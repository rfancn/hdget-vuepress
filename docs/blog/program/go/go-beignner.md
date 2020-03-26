---
date: '2017-11-17 13:06:17'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- program
- golang
title: Golang学习笔记
---

##### go变量声明与C语言的不同:

- go变量声明引入var关键字可以一次同时声明多个变量

        var myvar1, myvar2, myvar3 int
        OR
        var (
            myvar1 int
            myvar2 string
        )

- 变量类型声明与C类似，类型关键字在变量后面且没分号结束，C语言类型关键字在变量前面且以分号结束

    go: var myvar int
     c:  int myvar;

##### go赋值与c语言的不同

- go变量声明引入声明赋值符，当使用声明赋值符时，该变量不应该是已被声明的，否则编译出错

    var myvar := 1
    OR
    myvar := 1

- go引入多重赋值功能，可以一次给多个变量赋值

    myvar1, myvar2 = 1, 2
    OR
    i, j = j, i

- 与C不同的是Go的在不同类型之间的项目赋值时需要显式转换

- Go的if语句也不要求用()将条件括起来，同时，{}还是必须有的。

##### go常量与c语言的不同

- go语言中的常量可以是无类型的，一个数值可以是int, uint, int32, int64...., c语言中是必须有类型的

    go: const MYVAR = 12
    c:  const long MYVAR; MARVAR = 12l;

- go语言引入特殊iota常量，在每一个const关键字出现被重置为0， 然后在下一个const出现之前，没出现一个iota,其所代表的数字会自动增加1

##### go的for循环与c语言的不同

- for 语句的三个组成部分 并不需要用括号括起来，但循环体必须用 { } 括起来。
- 循环初始化语句和后置语句都是可选的

    func main() {
        sum := 1
        for ; sum < 1000; {
            sum += sum
        }
    }



##### 常见技巧

- Go中没有while， 可以用for来代替

    func main() {
        sum := 1
        for sum < 1000 {
            sum += sum
        }
    }

- 如果省略了循环条件，循环就不会结束，因此可以用更简洁地形式表达死循环。

    func main() {
        for {
        }
    }

- 没有条件的 switch 同 switch true 一样。这一构造使得可以用更清晰的形式来编写长的 if-then-else 链。

    func main() {
        t := time.Now()
        switch {
        case t.Hour() < 12:
            fmt.Println("Good morning!")
        case t.Hour() < 17:
            fmt.Println("Good afternoon.")
        default:
            fmt.Println("Good evening.")
        }
    }





##### 常见错误

- 首字母大写的名称是被导出的。在导入包之后，你只能访问包所导出的名字，任何未导出的名字是不能被包外的代码访问的。
- 当两个或多个连续的函数命名参数是同一类型，则除了最后一个类型之外，其他都可以省略
    
    var x int, y int
    可以写成
    var x, y int

- Go 的返回值可以被命名，并且就像在函数体开头声明的变量那样使用。返回值的名称应当具有一定的意义，可以作为文档使用。没有参数的 return 语句返回各个返回变量的当前值。这种用法被称作“裸”返回。直接返回语句仅应当用在像下面这样的短函数中。在长的函数中它们会影响代码的可读性。

    func split(sum int) (x, y int) {
        x = sum * 4 / 9
        y = sum - x
        return
    }

- 函数外的每个语句都必须以关键字开始（ var 、 func 、等等）， := 结构不能使用在函数外。

- 变量在定义时没有明确的初始化时会赋值为零值 。零值是：数值类型为 0 ，布尔类型为 false ，字符串为 "" （空字符串）。

- 使用声明赋值符时需保证该变量是未被声明的

    var myvar int
    /* 以下声明和赋值会报错 */
    myvar := 1

- 不同类型的整型之间不能比较，但各种类型的整型变量都可以与字面常量比较

    
    var i int32
    var j int64

    i, j = 1, 2
    /* 编译错误 */
    if i == j {
        fmt.Println("i == j")
    }

    /* 编译通过 */
    if i == 1 || j == 2 {
        fmt.Println("i, j")
    }

- 赋值时如果未声明变量类型，编译自动推测，浮点类型会被自动声明为float64, 如果将该变量赋值给其他float32会出错

    var myvar1 float32
    /* myvar2 会被自动推测声明为float64 */
    myvar2 := 12.0
    /* 编译错误 */
    myvar1 = myvar2

- 不能直接比较浮点数，因为浮点数并不是一种精确的表达方式
- 常量不能使用 := 语法定义