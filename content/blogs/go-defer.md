+++
title= "Go Defer"
date= 2019-09-04T09:41:06+08:00
tags= ["go"]
+++
### defer 作用
go语言中有defer关键字；

其作用类似于js中的finally，用于清除函数所占系统资源；

### defer 用法
defer用在一个函数的函数体内; defer关键词后跟一个函数调用，后面所跟的函数调用和普通的函数调用没有区别；

如果defer函数有返回值，则函数执行完毕之后返回值会被丢弃。

defer 关键字执行函数的时机：

* 外部函数执行return语句时
* 函数执行结束
* goroutine报错（panick）

如果外部函数有return表达式，defer函数会在return表达式设置值之后，return之前执行；
比如：
``` golang
// f returns 42
func f() (result int) {
	defer func() {
		// result is accessed after it was set to 6 by the return statement
		result *= 7
	}()
	return 6
}

```
defer函数可以访问外部函数的变量；


如果有多个defer表达式，执行顺序为从最后一个到第一个；
``` golang
lock(l)
defer unlock(l)  // unlocking happens before surrounding function returns

// prints 3 2 1 0 before surrounding function returns
for i := 0; i <= 3; i++ {
	defer fmt.Print(i)
}
```


### 参考资料：
[https://golang.org/ref/spec#Defer_statements](https://golang.org/ref/spec#Defer_statements)


