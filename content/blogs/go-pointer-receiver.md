+++
title= "Go Pointer Receiver"
date= 2019-08-26T14:13:02+08:00
tag= ["golang"]
+++
### golang struct类型
go语言中没有class概念，但有一个struct类型，可以实现class。
struct中可以包含基础类型做为其属性，比如boolean、string、int、float等，
实例化可通过**v :=T{}**，其中，打括号中是struct的属性值，类似面向对象语言中实例化一个class所用的new。
### golang pointer类型
go语言中，如果一个变量存储的是另一个变量所在的地址，则这个变量的类型为pointer类型，也就是指针； 

比如：

``` go
var house = "I am a string!"
padr := &house
```
变量padr就是一个pointer类型的变量，其存储的是字符串house的地址；
打印出来，就类似“0xc000084030”，
打印类型，则为“*string”
在golang中，指针类型就是“*T”
### golang methods with pointer receiver
go语言中的方法可以被指针类型所接收，该值可以为pointe类型，通过这种关系，可以把struct和该方法关联；

这种隐式的给struct指定方法的方式可以引用一句名言：***if it looks like a duck, work like a duck and quacks like a duck it must be a duck.***

比如：

``` go
package main

import (
	"fmt"
	"math"
)

type Vertex struct {
	X, Y float64
}

func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}

func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}

func main() {
	v := Vertex{3, 4}
	v.Scale(10)
	fmt.Println(v.Abs())
}

```

上例中，在main方法中实例化Vertex，Vertext是struct类型；

Scale方法是一个可以被指针类型所接收的方法，通过在方法名前加”(v *Vertext)“，在方法内可以改变v的值，也就改变了原始值（在main中定义的v）；

通过调用v.Abs()，看到，在Scale中改变了值，在v.Abs的值会被改变。

在方法Abs中，方法名前加了“(v Vertex)”，Vertext前没有加“*”，其实是对main中定一个v的值得拷贝；

而，Scale中方法名前“(v *Vertex)”,是对值的引用。

### 思考下面代码会输出什么？
``` 
package main

    import (
        "fmt"
        "time"
    )

    type field struct {
        name string
    }

    func (p *field) print() {
        fmt.Println(p.name)
    }

    func main() {
        data := []field{{"one"},{"two"},{"three"}}
        for _,v := range data {
            go v.print()
        }
        time.Sleep(3 * time.Second)
    }
```
### 再进一步思考下面代码会输出什么？
```
package main

    import (
        "fmt"
        "time"
    )

    type field struct {
        name string
    }

    func (p *field) print() {
        fmt.Println(p.name)
    }

    func main() {
        data := []*field{{"one"},{"two"},{"three"}}
        for _,v := range data {
            go v.print()
        }
        time.Sleep(3 * time.Second)
    }
```
上面两个思考题的代码唯一区别是main函数中定义的data 切片类型不同，一个存储的是field类型，一个是指针类型（存储field的地址）。
