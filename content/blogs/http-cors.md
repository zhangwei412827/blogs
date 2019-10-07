+++
title= "Http Cors"
date= 2019-09-23T17:30:57+08:00
tags= ["http"]
+++
CORS定义

CORS是W3C的一个标准，全称是：cross orgin resource sharing，中文翻译是：跨域资源共享；

浏览器出于安全的原因，对跨域的请求会做特殊处理；

跨域情况下，在浏览器接收到请求的数据时（此处需特别注意，跨域请求也是可以到达服务器的），浏览器会检查服务器的响应头里是否明确可以进行跨域资源共享，如果响应头里包含 Access-Control-Allow-Origin 则允许跨域请求数据，否者就是不允许跨域，浏览器检测无此字段，则认为不可跨域请求数据，就会报错，触发XmlHttpRequest的onerror事件。

http预检请求

http的预检请求也就是浏览器自行发送的option请求；

当使用除get、post、head方法以外的其他方法发起http请求时浏览器会发送option请求；
请求头包含除一下字段的其他字段
Accept
Accept-Language
Content-Language
Last-Event-ID
Content-Type：只限于三个值application/x-www-form-urlencoded、multipart/form-data、text/plain
浏览器通过发送option请求确认服务器是否允许访问（查看Access-Control-Allow-Origin字段的值），如果允许跨域则向服务器正式发送请求，否者就拒绝发送。
