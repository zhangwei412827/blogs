+++
title= "Css Inline"
date= 2019-08-16T09:36:58+08:00
tags = ["css"]
+++
有个同事问我，说为啥我用span标签写了一个序列，内容换行怎么围绕着这个span标签了？？
我说，这正是css行内元素的特性啊！
如果不想被“围绕”，可以使用block元素，通过float或者是flex等都可以实现。
css需要去感悟，理解，而不仅仅是死记硬背。
### 如下：
``` html
<html>
<body>
<div id='div1'>
	<span id='span1'>1</span>
    <span id='span2'>内容内容内容内容内容内容内容内容内容内容</span>
</div>
<style type='text/css'>
	#div1{
    	width:100px;
        height:100px;
        background:red;
    }
    #span1{
    	background:black;
        color:#fff;
        padding:1px 5px;
    }
    #span2{
    word-break:break-all;
    }
</style>
</body>
</html>

```
### 效果：
![效果图](/images/css-inline.png)
