+++
title= "Go Function Closure"
date= 2019-08-27T13:42:13+08:00
tag= ["golang"]
+++
### go语言中的closure（闭包）

go语言和JavaScript都支持closure（闭包）；

匿名函数被常用来实现闭包，go语言中也支持匿名函数；

closure的定义是：一个函数可以访问其函数体外的变量，我们称这个函数为closure（闭包）；

如下：
```go
package main

import "fmt"

// fibonacci is a function that returns
// a function that returns an int.
func fibonacci() func(int) int {
	a := 0
	b := 0
	return func(x int) int { 
		if x == 0 {
			return 0
		} 
		if x == 1 {
			b = 1
			return 1
		} 
		s := a + b
		a = b
		b = s
		return s
	}
}

func main() {
	f := fibonacci()
	for i := 0; i < 10; i++ {
		fmt.Println(f(i))
	}
}

```
利用closure实现一个fibonacci（斐波那契数列）