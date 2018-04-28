---
layout: post
title:  "Go教程05-函数"
date:   2018-04-17 08:08:04
categories: Go语言编程
tags: Go 
excerpt: 函数是Go里面的基本代码块。
---

* content
{:toc}


## 函数

函数是Go里面的基本代码块。

## 1 介绍

函数是基本的代码块每，一个程序都包含很多的函数。Go是编译型语言，所以函数编写的顺序是无关紧要的。鉴于可读性的需求，最好把main()函数写在文件的前面，其它函数按照一定逻辑顺序进行编写。编写多个函数的主要目的是将一个需要很多行代码的复杂问题分解为一系列简单的任务来解决。而且，同一个任务可以被调用多次，有助于代码重用。事实上，好的程序是非常注意DRY原则的，即不要重复自己（Don't Repeat Yourself），意思是执行特定任务的代码只能在程序里面出现一次。

Go里面有三种类型的函数：

* 普通的带有名字的函数
* 匿名函数或者lambda函数
* 方法（Methods）

所有类型的函数都可以有参数与返回值。函数参数、返回值以及它们的类型被统称为**函数签名**。

**函数可以将其它函数调用作为它的参数**，只要这个被调用函数的返回值个数、返回值类型和返回值的顺序与调用函数所需求的实参是一致的，例如：

假设f1需要3个参数f1(a,b, c int)，同时f2返回3个参数f2(a, b int) (int, int, int)，就可以这样调用f1：
        `f1(f2(a, b))`
函数重载（function overloading）指的是可以编写多个同名函数，只要它们拥有不同的形参与/或者不同的返回值，**在Go里面函数重载是不被允许的**。Go语言不支持这项特性的主要原因是函数重载需要进行多余的类型匹配影响性能；没有重载意味着只是一个简单的函数调度。所以需要给不同的函数使用不同的名字，通常会根据函数的特征对函数进行命名。

如果需要申明一个在外部定义的函数，只需要给出函数名与函数签名，不需要给出函数体：

```go
   // implemented externally
   func flushICache(begin, end uintptr) 
```

函数也可以以申明的方式被使用，作为一个函数类型，就像：

```go
   type binOp func(int, int) int
```

在这里，不需要函数体{}。

函数是一等值（first-class value）：它们可以赋值给变量，就像`add := binOp`一样。这个变量知道自己指向的函数的签名，所以给它赋一个具有不同签名的函数值是不可能的。

函数值（functions value）之间可以相互比较：如果它们引用的是相同的函数或者都是nil的话，则认为它们是相同的函数。**函数不能在其它函数里面声明（不能嵌套），不过可以通过使用匿名函数来破除这个限制。**

目前Go没有泛型（generic）的概念，也就是说它不支持那种支持多种类型的函数。不过在大部分情况下可以通过接口（interface），特别是空接口与类型选择（type switch）与/或者通过使用反射（reflection）来实现相似的功能。使用这些技术将导致代码更为复杂、性能更为低下，所以在非常注意性能的的场合，最好是为每一个类型单独创建一个函数，而且代码可读性更强。



## 2 函数参数与返回值

函数能够接收参数供自己使用，也可以返回零个或多个值（通常把返回多个值称为返回一组值）。通过return关键字返回一组值。事实上，任何一个有返回值（单个或多个）的函数都必须以return或panic结尾。

在函数块里面，return之后的语句都不会执行。如果一个函数需要返回值，那么这个函数里面的每一个代码分支（code-path）都要有return语句。

函数定义时，它的形参一般是有名字的，不过也可以定义没有形参名的函数，只有相应的形参类型，就像这样：
`func f(int, int, float64)`
没有参数的函数通常被称为**niladic函数**（niladic function），就像 main.main()。

### 2.1 按值传递和按引用传递

Go默认使用按值传递（call by value）来传递参数，也就是传递参数的副本。函数接收参数副本之后，在使用变量的过程中可能对副本的值进行更改，但不会影响到原来的变量。如果希望函数可以直接修改参数的值，而不是对参数的副本进行操作，需要将参数的地址（在变量名前面添加&符号，比如&variable）传递给函数，这就是**按引用传递**（call by reference），此时传递给函数的是一个指针。如果传递给函数的是一个指针，指针的值（一个地址）会被复制，但指针的值所指向的地址上的值不会被复制；可以通过这个指针的值来修改这个值所指向的地址上的值。几乎在任何情况下，传递指针（一个32位或者64位的值）的消耗都比传递副本来得少。在函数调用时，像切片（slice）、字典（map）、接口（interface）、通道（channel）这样的引用类型都是默认使用引用传递（即使没有显示的指出指针）。有些函数只是完成一个任务，并没有返回值。仅仅是利用了这种函数的副作用，就像输出文本到终端，发送一个邮件或者是记录一个错误等。但是绝大部分的函数还是带有返回值的。如果一个函数需要返回四到五个值，可以传递一个切片给函数（如果返回值具有相同类型）或者是传递一个结构体（如果返回值具有不同的类型）。因为传递一个指针允许直接修改变量的值，消耗也更少。

### 2.2 命名返回值

* 程序示例

```go
/*
@file:    multiple_return.go
@version: v1.0
@author:  haulf
@date:    2017.11.12
@brief:   Function test program.
*/

package main

import "fmt"

var num int = 10
var numx2, numx3 int

func main() {
    numx2, numx3 = getX2AndX3(num)
    PrintValues()
    numx2, numx3 = getX2AndX3New(num)
    PrintValues()
}

func PrintValues() {
    fmt.Printf("num = %d, 2x num = %d, 3x num = %d\n", num,
        numx2, numx3)
}

func getX2AndX3(input int) (int, int) {
    return 2 * input, 3 * input
}

func getX2AndX3New(input int) (x2 int, x3 int) {
    x2 = 2 * input
    x3 = 3 * input
    // return x2, x3
    return
}
```

* 运行结果

```go
   num = 10, 2x num = 20, 3x num = 30   
   num = 10, 2x num = 20, 3x num = 30 
```

* 程序说明

程序中getX2AndX3()函数和getX2AndX3New()函数演示了如何使用非命名返回值与命名返回值（named returnvariables）的特性。它们都带有一个int参数，返回两个int值。其中getX2AndX3New()函数的返回值在函数调用时就已经被赋予了一个初始零值。当需要返回多个非命名返回值时，需要使用一对圆括号把它们括起来，比如(int, int)。

命名返回值作为结果形参（result parameters）被初始化为相应类型的零值，当需要返回的时候，只需要一条简单的不带参数的return语句。需要注意的是，即使只有一个命名返回值，也需要使用一对圆括号括起来。

平时在写代码时尽量使用命名返回值，这样会使代码更清晰、更简短，同时更加容易读懂。

### 2.3 空白符

空白符（blank identifier）用来匹配一些不需要的值，然后丢弃掉。

* 程序示例

```go
/*
@file:    blank_identifier.go
@version: v1.0
@author:  haulf
@date:    2017.11.12
@brief:   Function test program.
*/

package main

import "fmt"

func main() {
    var i1 int
    var f1 float32
    i1, _, f1 = getThreeValues()
    fmt.Printf("The int: %d, the float: %f \n", i1, f1)
}

func getThreeValues() (int, int, float32) {
    return 5, 6, 7.5
}
```

* 运行结果

```go
The int: 5, the float: 7.500000
```

* 程序说明

  getThreeValues()函数是拥有三个返回值的不需要任何参数的函数。在程序中，将第一个与第三个返回值赋给了i1与f1，第二个返回值赋给了空白符_，然后自动丢弃掉。

### 2.4 改变外部变量的值

传递指针给函数不但可以节省内存，而且赋予了函数直接修改外部变量（outside variable）的能力，所以被修改的变量不再需要使用return返回。

* 程序示例

```go
/*
@file:    side_effect.go
@version: v1.0
@author:  haulf
@date:    2017.11.12
@brief:   Function test program.
*/

package main

import (
    "fmt"
)

// this function changes reply
func Multiply(a, b int, reply *int) {
    *reply = a * b
}

func main() {
    n := 0
    reply := &n
    Multiply(10, 5, reply)
    fmt.Println("Multiply:", *reply) // Multiply: 50
}
```

* 程序运行结果

```go
Multiply: 50
```

* 程序说明
在上面的程序中，reply是一个指向int变量的指针，通过这个指针，在函数内修改了这个int变量的数值。这仅仅是个指导性的例子，当需要在函数内改变一个占用内存比较大的变量时，性能优势就更加明显了。



## 3 变参函数
如果函数的最后一个参数是采用**...type**的形式，那么这个函数就可以处理一个变长的参数，这个长度可以为0，这样的函数称为**变参函数**。

```go
  func myFunc(a, b, arg ...int) {}
```
这个函数接受一个类似某个类型的切片的参数，该参数可以通过for循环结构来迭代。

### 3.1 接收变长参数

* 程序示例

```go
/*
@file:    var_num_params.go
@version: v1.0
@author:  haulf
@date:    2017.11.12
@brief:   Function test program.
*/

package main

import "fmt"

func main() {
    x := min(1, 3, 2, 0)
    fmt.Printf("The minimum is: %d\n", x)

    arr := []int{7, 9, 3, 5, 1}
    x = min(arr...)
    fmt.Printf("The minimum in the array arr is: %d", x)
}

func min(a ...int) (minValue int) {
    if len(a) == 0 {
        return 0
    }

    minValue = a[0]
    for _, v := range a {
        if v < minValue {
            minValue = v
        }
    }

    return
}
```

* 程序运行结果

```go
The minimum is: 0
The minimum in the array arr is: 1
```

### 3.2 将接收到的变长参数传递给其它函数

一个接收变长参数的函数可以将这个参数作为其它函数的参数进行传递：

```go
   function F1(s …string) {
       F2(s …)
       F3(s)
    }

   func F2(s … string) { }
   func F3(s []string) { }
```

变长参数可以作为对应类型的切片进行二次传递。但是如果变长参数的类型并不是都相同的呢？使用5个参数来进行传递并不是很明智的选择，有两种方案可以解决这个问题：

1. 使用结构体：

定义一个结构体类型，假设它叫Options，用以存储所有可能的参数：

```go
type Options struct {
           par1 type1,
           par2 type2,
           ...
       }
```

函数F1可以使用正常的参数a和b，以及一个没有任何初始化的Options结构： F1(a, b, Options {})。如果需要对选项进行初始化，则可以使用 F1(a, b, Options {par1:val1, par2:val2})。

2. 使用空接口：

如果一个变长参数的类型没有被指定，则可以使用默认的空接口interface{}，这样就可以接受任何类型的参数。该方案不仅可以用于长度未知的参数，还可以用于任何不确定类型的参数。一般而言会使用一个for-range循环以及switch结构对每个参数的类型进行判断。

```go
func typecheck(.., .., values ...interface{}) {
    for _, value := range values {
        switch v := value.(type) {
            case int: ...
            case float: ...
            case string: ...
            case bool: ...
            default: ...
        }
    }
}
```

## 4 defer和追踪

关键字defer允许推迟到函数返回之前（或任意位置执行return语句之后）一刻才执行某个语句或函数。为什么要在返回之后才执行这些语句？因为return语句同样可以包含一些操作，而不是单纯地返回某个值。defer的用法类似于面向对象编程语言Java和C#的finally语句块，它一般用于释放某些已分配的资源。

### 4.1 多个defer任务执行顺序

当有多个defer行为被注册时，它们会以逆序执行。这种情况类似栈，即后进先出。

- 程序示例

```go
// file:    DeferParamSequence.go
// author:  haulf 
// date:    2017.11.12
// brief:   Test the sequence of multi defer params.

package main

import "fmt"

func main() {
    f()
}

func f() {
    for i := 0; i < 5; i++ {
        defer fmt.Printf("i=%d ", i)
    }
}
```

- 运行结果

```go
i=4 i=3 i=2 i=1 i=0
```

### 4.2 defer使用场景

关键字defer主要进行一些函数执行完成后的收尾工作。

- 关闭文件流

```go
func parseCmdlineParameters()  {
    cmdlineFile, inputError := os.Open("cmdline")
    if inputError != nil {
        return
    }
    defer cmdlineFile.Close()

    inputReader := bufio.NewReader(cmdlineFile)
    for {
        inputString, readerError := inputReader.ReadString('\n')
        if readerError == io.EOF {
            return
        }

        resultSlice := strings.Fields(inputString)
        for _, item := range resultSlice {
            fmt.Println(item)
        }
    }
}
```

- 解锁一个加锁的资源

```go
  mu.Lock() 

  defer mu.Unlock()
```

- 打印最终报告

```go
  printHeader() 

  defer printFooter()
```

- 关闭数据库链接

```go
   //open a database connection 
   defer disconnectFromDB()
```

### 4.3 使用defer任务实现代码追踪

一个基础但十分实用的实现代码执行追踪的方案就是在进入和离开某个函数打印相关的消息，即可以提炼为下面两个函数：

```go
   func trace(s string) { fmt.Println("entering:", s) }
   func untrace(s string) { fmt.Println("leaving:", s) }
```

以下代码展示了何时调用两个函数：

```go
// file:    DeferTracing.go
// author:  haulf
// date:    2017.11.12
// brief:   Tracing function with defer.

package main

import "fmt"

func trace(s string) {
    fmt.Println("entering:", s)
}

func untrace(s string) {
    fmt.Println("leaving:", s)
}

func a() {
    trace("a")
    defer untrace("a")
    fmt.Println("in a")
}

func b() {
    trace("b")
    defer untrace("b")
    fmt.Println("in b")
    a()
}

func main() {
    b()
}
```

上面的代码还可以修改为更加简便的版本：

```go
// file:    DeferTracing.go
// author:  haulf
// date:    2017.11.12
// brief:   Tracing function with defer.

package main

import "fmt"

func trace(s string) string {
    fmt.Println("entering:", s)
    return s
}

func untrace(s string) {
    fmt.Println("leaving:", s)
}

func a() {
    defer untrace(trace("a"))
    fmt.Println("in a")
}

func b() {
    defer untrace(trace("b"))
    fmt.Println("in b")
    a()
}

func main() {
    b()
}
```

### 4.4 使用defer语句来记录函数的参数和返回值

下面的代码展示了另一种在调试时使用defer语句的方法。

- 程序示例

```go
// file:    RecordParametersAndReturns.go
// author:  aihaofeng
// date:    2017.11.12
// brief:   Record function's parameters and returns by defer.

package main

import (
    "io"
    "log"
)

func main() {
    myfunction("Go")
}

func myfunction(s string) (n int, err error) {
    defer func() {
        log.Printf("myfunction(%q) = %d, %v", s, n, err)
    }()
    return 7, io.EOF
}
```

## 5 内置函数

Go语言拥有一些不需要进行导入操作就可以使用的内置函数。它们有时可以针对不同的类型进行操作，例如：len、cap和append，或必须用于系统级的操作，例如：panic。因此，它们需要直接获得编译器的支持。

| 名称              | 说明                                      |
| ----------------- | ---------------------------------------- |
| close             | 用于管道通信                                   |
| len、cap           | len 用于返回某个类型的长度或数量（字符串、数组、切片、map 和管道）；  cap 是容量的意思，用于返回某个类型的最大容量（只能用于切片和 map） |
| new、make          | new 和 make 均是用于分配内存。new 用于值类型和用户定义的类型，如自定义结构，make 用于内置引用类型（切片、map 和管道）。  它们的用法就像是函数，但是将类型作为参数：new(type)、make(type)。new(T) 分配类型 T 的零值并返回其地址，也就是指向类型 T 的指针。它也可以被用于基本类型：v := new(int)。make(T) 返回类型 T 的初始化之后的值，因此它比 new 进行更多的工作。new() 是一个函数，不要忘记它的括号。 |
| copy、append       | 用于复制和连接切片                                |
| panic、recover     | 两者均用于错误处理机制                              |
| print、println     | 底层打印函数（详见第 4.2 节），在部署环境中建议使用 fmt 包       |
| complex、real imag | 用于创建和操作复数                                |



## 6 递归函数

当一个函数在其函数体内调用自身，则称之为递归。

* 程序示例

```go
/*
@file:    fibonacci.go
@version: v1.0
@author:  haulf
@date:    2017.11.12
@brief:   Function test program.
*/

package main

import "fmt"

func main() {
    result := 0
    for i := 0; i <= 10; i++ {
        result = fibonacci(i)
        fmt.Printf("fibonacci(%d) is: %d\n", i, result)
    }
}

func fibonacci(n int) (res int) {
    if n <= 1 {
        res = 1
    } else {
        res = fibonacci(n-1) + fibonacci(n-2)
    }
    return
}
```

* 程序运行结果

```go
fibonacci(0) is: 1
fibonacci(1) is: 1
fibonacci(2) is: 2
fibonacci(3) is: 3
fibonacci(4) is: 5
fibonacci(5) is: 8
fibonacci(6) is: 13
fibonacci(7) is: 21
fibonacci(8) is: 34
fibonacci(9) is: 55
fibonacci(10) is: 89
```

* 程序说明

每个数均为前两个数之和，这就是斐波那契数列，它是最经典的递归例子。

数列如下所示：

`1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144,233, 377, 610, 987, 1597, 2584, 4181, 6765, 10946, …`

许多问题都可以使用优雅的递归来解决，比如说著名的快速排序算法。

在使用递归函数时经常会遇到的一个重要问题就是栈溢出：一般出现在大量的递归调用导致的程序栈内存分配耗尽。这个问题可以通过一个名为懒惰求值的技术解决，在 Go语言中，可以使用channel和goroutine来实现。


## 7 将函数作为参数

函数可以作为其它函数的参数进行传递，然后在其它函数内调用执行，一般称之为**回调**。

* 程序示例

```go
/*
@file:    function_params.go
@version: v1.0
@author:  haulf
@date:    2017.11.12
@brief:   Function test program.
*/

package main

import (
    "fmt"
)

func main() {
    callback(1, Add)
}

func Add(a, b int) {
    fmt.Printf("The sum of %d and %d is: %d\n", a, b, a+b)
}

func callback(y int, f func(int, int)) {
    f(y, 2) // this becomes Add(1, 2)
}
```

* 程序运行结果

```go
The sum of 1 and 2 is: 3
```

将函数作为参数的最好的例子是函数strings.IndexFunc()。该函数的签名是funcIndexFunc(s string, f func(c int) bool) int，它的返回值是在函数f(c)返回true、-1或从未返回时的索引值。

例如 strings.IndexFunc(line, unicode.IsSpace)就会返回line中第一个空白字符的索引值。当然，也可以书写自己的函数：

```go
func IsAscii(c int) bool {
     if c > 255 {
         return false
     }
     return true
 }
```



## 8 匿名函数

当不希望给函数起名字的时候，可以使用**匿名函数**，例如：

```go
func(x, y int) int { 
    return x + y 
}
```

**匿名函数同样被称之为闭包**：它们被允许调用定义在其它环境下的变量。闭包可使得某个函数捕捉到一些外部状态，例如：函数被创建时的状态。另一种表示方式为：一个闭包继承了函数所声明时的作用域。这种状态（作用域内的变量）都被共享到闭包的环境中，因此这些变量可以在闭包中被操作，直到被销毁。闭包经常被用作**包装函数**：它们会预先定义好1个或多个参数以用于包装。另一个不错的应用场景就是使用闭包来完成更加简洁的错误检查。

匿名函数不能够独立存在，但可以被赋值于某个变量，即保存函数的地址到变量中，例如：

```go
fplus := func(x, y int) int { 
    return x + y 
}
```

然后通过变量名对函数进行调用：

```go
fplus(3,4)
```

当然，也可以直接对匿名函数进行调用：

```go
func(x,y int) int { 
    return x + y 
} (3, 4)
```

因为匿名函数没有函数名称，所以表示参数列表的第一对圆括号必须紧挨着关键字func。花括号{}包含着函数体，最后的一对圆括号表示对该匿名函数的调用，里面可以向匿名函数传递参数。匿名函数像所有函数一样可以接受或不接受参数，其可以被赋值给变量并作为值使用。

### 8.1 defer语句和匿名函数

关键字defer经常配合匿名函数使用，它可以用于改变函数的命名返回值。

* **程序示例**

```go
// @file:        return_defer.go
// @version:     1.0
// @author:      haulf
// @date:        2017.11.05
// @go version:  1.9
// @brief:       Return defer test.

package main

import "fmt"

func f() (ret int) {
    defer func() {
        ret++
    }()
    return 1
}

func main() {
    fmt.Println(f())
}
```

* **程序说明**

defer后面的函数会在最后执行。在这个匿名函数中，对返回值进行了加1操作。

* **程序运行结果**

```shell
2
```

### 8.2 go关键字和匿名函数

匿名函数还可以配合go关键字来作为goroutine 使用。

```go
    // Scan files in multiple goroutines.
    for i := 0; i < *para; i++ {
        go func() {
            defer wg.Done()

            output := make([]Node, 0, 1024) // Local output list.
            for node := range ch {
                if node.scan() {
                    output = append(output, node)
                }
            }
            // Add to the global output list.
            mutex.Lock()
            allOutput = append(allOutput, output...)
            mutex.Unlock()
        }()
    }
```

### 8.3 将匿名函数作为返回值

* **程序示例**

```go
// @file:        function_return.go
// @version:     1.0
// @author:      haulf
// @date:        2017.11.05
// @go version:  1.9
// @brief:       Function return test.

package main

import "fmt"

func main() {
    // make an Add2 function, give it a name p2, and call it:
    p2 := Add2()
    fmt.Printf("Call Add2 for 3 gives: %v\n", p2(3))
    
    // make a special Adder function, a gets value 3:
    TwoAdder := Adder(2)
    fmt.Printf("The result is: %v\n", TwoAdder(3))
}

func Add2() func(b int) int {
    return func(b int) int {
        return b + 2
    }
}

func Adder(a int) func(b int) int {
    return func(b int) int {
        return a + b
    }
}

```

- 程序说明

函数Add2和Adder均会返回签名为func(b int) int的函数：

```go
func Add2() (func(b int) int)

func Adder(a int) (func(b int) int)
```

函数Add2不接受任何参数，但函数Adder接受一个int类型的整数作为参数。也可以将Adder返回的函数存到变量中。

下例为一个略微不同的实现：

- 程序示例

```go
// @file:        function_closure.go
// @version:     1.0
// @author:      haulf
// @date:        2017.11.05
// @go version:  1.9
// @brief:       Function return test.

package main

import "fmt"

func main() {
    var f = Adder()
    fmt.Print("")
    fmt.Print(f(1), " - ")
    fmt.Print(f(20), " - ")
    fmt.Print(f(300))
    fmt.Print("")
}

func Adder() func(int) int {
    var x int
    return func(delta int) int {
        x += delta
        return x
    }
}
```

- 程序运行结果

```shell
1 - 21 - 321
```

- 程序说明

函数Adder()现在被赋值到变量f中（类型为func(int) int）。三次调用函数f的过程中函数Adder()中变量 delta 的值分别为：1、20和300。可以看到，在多次调用中，变量x的值是被保留的，即0 + 1 = 1，然后1 + 20 = 21，最后21 + 300 = 321 。

闭包函数保存并积累其中的变量的值，不管外部函数退出与否，它都能够继续操作外部函数中的局部变量。这些局部变量同样可以是参数，例如之前例子中的Adder(as int)。

在闭包中使用到的变量可以是在闭包函数体内声明的，也可以是在外部函数声明的：

```go
    var g int
    go func(i int) {
        s := 0
        for j := 0; j < i; j++ { s += j }
        g = s
    }(1000) // Passes argument 1000 to the function literal. 
```

这样闭包函数就能够被应用到整个集合的元素上，并修改它们的值。然后这些变量就可以用于表示或计算全局或平均值。

### 8.4 使用匿名函数来调试程序

在分析和调试复杂的程序时，无数个函数在不同的代码文件中相互调用，如果这时候能够准确地知道哪个文件中的具体哪个函数正在执行，对于调试是十分有帮助的。可以使用runtime或log包中的特殊函数来实现这样的功能。包runtime中的函数Caller()提供了相应的信息，因此可以在需要的时候实现一个where()匿名函数来打印函数执行的位置：

```go
    where := func() {
        _, file, line, _ := runtime.Caller(1)
        log.Printf("%s:%d", file, line)
    }
    where()
    // some code
    where()
    // some more code
    where()
```

也可以设置 log 包中的 flag 参数来实现：

```go
    log.SetFlags(log.Llongfile)
    log.Print("")
```

或使用一个更加简短版本的 where 函数：

```go
    var where = log.Print
    func func1() {
    where()
    ... some code
    where()
    ... some code
    where()
    }
```

## 9 计算函数执行时间

有时候，能够知道一个计算执行消耗的时间是非常有意义的，尤其是在对比和基准测试中。最简单的一个办法就是在计算开始之前设置一个起始时候，再由计算结束时的结束时间，最后取出它们的差值，就是这个计算所消耗的时间。想要实现这样的做法，可以使用time包中的Now()和Sub()函数：

```go
    start := time.Now()
    longCalculation()
    end := time.Now()
    delta := end.Sub(start)
    fmt.Printf("longCalculation took this amount of time: %s\n", delta)
```

## 10 通过内存缓存技术来提升性能

当在进行大量的计算时，提升性能最直接有效的一种方式就是避免重复计算。通过在内存中缓存和重复利用相同计算的结果，称之为**内存缓存**。最明显的例子就是生成斐波那契数列的程序。

要计算数列中第n个数字，需要先得到之前两个数的值，但很明显绝大多数情况下前两个数的值都是已经计算过的。即每个更后面的数都是基于之前计算结果的重复计算，正如示例fibonnaci.go所展示的那样。而要做就是将第n个数的值存在数组中索引为n的位置，然后在数组中查找是否已经计算过，如果没有找到，则再进行计算。

* **程序示例**

```go
// @file:        fibonacci_memory_optimization.go
// @version:     1.0
// @author:      haulf
// @date:        2017.11.05
// @go version:  1.9
// @brief:       Memory optimization test.

package main

import (
    "fmt"
    "time"
)

const LIM = 41

var fibs [LIM]uint64

func main() {
    var result uint64 = 0
    start := time.Now()
    for i := 0; i < LIM; i++ {
        result = fibonacci(i)
        fmt.Printf("fibonacci(%d) is: %d\n", i, result)
    }
    end := time.Now()
    delta := end.Sub(start)
    fmt.Printf("longCalculation took this amount of time: %s\n", delta)
}

func fibonacci(n int) (res uint64) {
    // memoization: check if fibonacci(n) is already known in array:
    if fibs[n] != 0 {
        res = fibs[n]
        return
    }
    if n <= 1 {
        res = 1
    } else {
        res = fibonacci(n-1) + fibonacci(n-2)
    }
    fibs[n] = res
    return
}
```

* **程序运行结果**

```shell
fibonacci(0) is: 1
fibonacci(1) is: 1
fibonacci(2) is: 2
fibonacci(3) is: 3
fibonacci(4) is: 5
fibonacci(5) is: 8
fibonacci(6) is: 13
fibonacci(7) is: 21
fibonacci(8) is: 34
fibonacci(9) is: 55
fibonacci(10) is: 89
fibonacci(11) is: 144
fibonacci(12) is: 233
fibonacci(13) is: 377
fibonacci(14) is: 610
fibonacci(15) is: 987
fibonacci(16) is: 1597
fibonacci(17) is: 2584
fibonacci(18) is: 4181
fibonacci(19) is: 6765
fibonacci(20) is: 10946
fibonacci(21) is: 17711
fibonacci(22) is: 28657
fibonacci(23) is: 46368
fibonacci(24) is: 75025
fibonacci(25) is: 121393
fibonacci(26) is: 196418
fibonacci(27) is: 317811
fibonacci(28) is: 514229
fibonacci(29) is: 832040
fibonacci(30) is: 1346269
fibonacci(31) is: 2178309
fibonacci(32) is: 3524578
fibonacci(33) is: 5702887
fibonacci(34) is: 9227465
fibonacci(35) is: 14930352
fibonacci(36) is: 24157817
fibonacci(37) is: 39088169
fibonacci(38) is: 63245986
fibonacci(39) is: 102334155
fibonacci(40) is: 165580141
longCalculation took this amount of time: 66.441µs
```

* **程序说明**

上面的测试程序是依照这个原则实现的，下面是计算到第40位数字的性能对比：

- 普通写法：4.730270 秒
- 内存缓存：0.001000 秒

内存缓存的优势显而易见，而且还可以将它应用到其它类型的计算中，例如使用map而不是数组或切片。

内存缓存的技术在使用计算成本相对昂贵的函数时非常有用（不仅限于例子中的递归），譬如大量进行相同参数的运算。这种技术还可以应用于纯函数中，即相同输入必定获得相同输出的函数。
