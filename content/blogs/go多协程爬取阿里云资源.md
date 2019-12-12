+++
title= "Go多协程爬取阿里云资源"
date= 2019-11-28T17:18:11+08:00
tags=["go"]
+++
### 用go写了个爬取好未来存放在阿里云oss的音频附件，代码如下：
``` go
package main

import (
	"encoding/json"
	"fmt"
	"io/ioutil"
	"log"
	"net/http"
	"os"
	"strconv"
	"strings"
	"sync"

	"github.com/aliyun/aliyun-oss-go-sdk/oss"
)

var _ClassIds = []string{}

const (
	Endpoint     string = "http://oss-cn-beijing.aliyuncs.com"
	AccessID     string = "这里存放阿里云accessId"
	AccessKey    string = "这里存放阿里云accessKey"
	RootPath     string = "classrecord/"
	getLessonApi string = "获取大班ID API" 
	getPaperApi  string = "获取大班ID试题API"
)

// todo 如果解析csv文件，这里创建一个struct 保存课程信息
// type CLassInfo struct {
// 	Cid  string
// 	Cnum string
// }

type ClassInfo struct {
	Id           int    `json:"id"`
	LiveNumberId string `json:"liveNumberId"`
}
type Result struct {
	Result ClassInfo `json:"result"`
}

type Task struct {
	TaskId int64  `json:"taskId"`
	Stem   string `json:"stem"`
}
type Paper struct {
	PaperId int64  `json:"paperId"`
	Tasks   []Task `json:"tasks"`
}

//taskId>stem Map type
type TaskMap map[string]string

func iterationClassIdWorker(id int, path chan string, bk *oss.Bucket, wg *sync.WaitGroup) {
	for {
		p := <-path
		count := 0
		init := false
		// todo 这里先通过小班id 转换大班id，封装方法，返回大班id、error哈，如果error就不用跑后面了，等处理下个班，别忘了wg.Done()
		// todo 这里通过大班id、课次号、获取题目列表， 封装2个方法哈，在一级方法里合并语音题、分角色朗读两次请求的结果，在二级方法里请求具体题目列表，返回 map[string]string, error

		//小班id
		cId := strings.Split(p, "/")[1]
		//课次号
		cNum := strings.Split(p, "/")[2]
		taskMap, err := getTaskMap(cId, cNum)
		if err != nil {
			log.Println("Get taskMap error: ", err)
			break
		}

		log.Println("workerid:", id, "开始下载班级音频，班级id:", p)
		marker := oss.Marker("")
		for {
			lret, err := bk.ListObjects(oss.Prefix(p), marker)
			if err != nil {
				log.Println("client.Bucket, path:", p, "error :", err)
				break
			}
			for _, obj := range lret.Objects {
				count++
				suffix := obj.Key[len(obj.Key)-4:]
				if suffix == ".wav" || suffix == ".mp3" {
					tms := strings.Split(obj.Key, "/")
					fn := tms[4]
					if !init {
						err := os.Mkdir("./wavFile/"+tms[1], 0777)
						if err != nil {
							log.Println("create dir error:", err)
						}
						os.Chmod("./wavFile/"+tms[1], 0777)
						init = true
					}
					// todo 这里再解析下目标文件名，匹配下题目题干，并且修改文件名，封装方法哈，输入文件名和题目列表，返回文件名

					//fn包含taskid，替换taskid为stem
					fnSlc := strings.Split(fn, "-")
					if len(fnSlc) == 6 {
						//查找taskMap
						if _, ok := taskMap[fnSlc[3]]; ok {
							fn = strings.Replace(fn, fnSlc[3], taskMap[fnSlc[3]], 1)
						}
					}
					err := bk.GetObjectToFile(obj.Key, "./wavFile/"+tms[1]+"/"+fn)
					if err != nil {
						log.Println("download error, file:", fn, "error :", err)
					}
				}
			}
			marker = oss.Marker(lret.NextMarker)
			if !lret.IsTruncated {
				log.Println("workerid:", id, "下载班级音频完成，班级id:", p, ",获取完成")
				break
			}
		}
		// todo 注意下上面各个资源的声明周期，该清空要清空
		log.Println("workerid:", id, "下载班级音频完成，班级id:", p, ",下载文件数量:", count)
		wg.Done()
	}
}

//转换paper tasks到map
func getTaskMap(cid string, cNum string) (TaskMap, error) {
	papers, err := getPaerListByLnum(cid, cNum)
	if err != nil {
		return nil, err
	}
	taskMap := make(TaskMap)
	for _, paper := range papers {
		for _, task := range paper.Tasks {
			taskMap[strconv.FormatInt(int64(task.TaskId), 10)] = task.Stem
		}
	}

	return taskMap, nil
}

//根据小班ID获取大班ID
func getLnumByCid(cid string) (string, error) {
	// fmt.Printf("classId: %v", cid)
	resp, err := http.Get(getLessonApi + cid)
	if err != nil {
		return "", err
	}
	defer resp.Body.Close()
	s, err := ioutil.ReadAll(resp.Body)

	mResult := &Result{}
	if err != nil {
		return "", err
	}
	err = json.Unmarshal(s, &mResult)
	if err != nil {
		return "", err
	}

	return mResult.Result.LiveNumberId, nil
}

//根据大班ID获取paperList
func getPaerListByLnum(cid string, cNum string) ([]Paper, error) {
	// fmt.Printf("classId: %v", cid)
	//?liveNum=0101-M3hx6cEnePdHkjvgNzkvDz&classNum=9&type=10
	//根据小班ID获取大班ID
	lnum, err := getLnumByCid(cid)
	if err != nil {
		return nil, err
	}

	//包含”语音“和”朗读“
	papers := &[]Paper{}

	wg := sync.WaitGroup{}
	papersChn := make(chan []Paper, 2)

	//语音题worker
	go getPaperListByLnumAndCnum(lnum, cNum, 2, papersChn, &wg)
	wg.Add(1)
	//分角色朗读worker
	go getPaperListByLnumAndCnum(lnum, cNum, 10, papersChn, &wg)
	wg.Add(1)

	//等待worker执行完毕
	wg.Wait()
	//关闭channel
	close(papersChn)

	//range papers channel 获取数据
	for ps := range papersChn {
		//合并papers
		*papers = append(*papers, ps...)
	}

	return *papers, nil
}

//根据大班ID和课次号获取paperList
func getPaperListByLnumAndCnum(lnum string, cNum string, typ int, papersChn chan []Paper, wg *sync.WaitGroup) ([]Paper, error) {
	//获取 语音题  2 分角色朗读  10
	queryData := fmt.Sprintf("?liveNum=%s&classNum=%s&type=%d", lnum, cNum, typ)
	resp, err := http.Get(getPaperApi + queryData)
	if err != nil {
		return nil, err
	}
	defer resp.Body.Close()
	s, err := ioutil.ReadAll(resp.Body)

	voicePapers := &[]Paper{}
	if err != nil {
		return nil, err
	}
	err = json.Unmarshal(s, &voicePapers)
	if err != nil {
		return nil, err
	}

	papersChn <- *voicePapers

	//获取完毕执行Done
	wg.Done()

	return *voicePapers, nil
}

func main() {
	client, err := oss.New(Endpoint, AccessID, AccessKey)
	if err != nil {
		log.Fatalln("aliyun oss init error, error :", err)
	}
	bk, err := client.Bucket("ss-dtc")
	if err != nil {
		log.Println("client.Bucket, path:", RootPath+_ClassIds[0], "error :", err)
		return
	}
	wg := sync.WaitGroup{}
	os.Mkdir("./wavFile", 0x777)
	os.Chmod("./wavFile", 0777)
	pch := make(chan string)
	for i := 0; i < 10; i++ {
		go iterationClassIdWorker(i, pch, bk, &wg)
	}
	count := 1
	for _, cid := range _ClassIds {
		log.Println("启动课程下载数量：", count)
		count++
		wg.Add(1)
		// todo 这里需要修改为使用 classInfo对象传递
		pch <- RootPath + cid + "/"
	}
	wg.Wait()
}

```
### 需求说明
* 下载所有给定小班和课次下的音频文件
* 通过小班ID获取大班ID
* 通过大班ID和课次号获取所有试题
* 替换下载的音频文件名为题干名（原文件名包含的有题干ID）

