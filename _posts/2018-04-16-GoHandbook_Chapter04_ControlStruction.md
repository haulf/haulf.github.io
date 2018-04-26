---
layout: post
title:  "控制结构"
date:   2018-04-16 08:08:04
categories: Go语言编程
tags: Go 
excerpt: Go让人感觉像是Python或Ruby这样的动态语言，但却又拥有像C或者Java这类语言的高性能和安全性。Go语言出现的目的是希望在编程领域创造最实用的方式来进行软件开发。
---

* content
{:toc}


## 控制结构

经常会需要只有在满足一些特定情况时执行某些代码，也就是说在代码里进行条件判断。针对这种需求，Go提供了下面这些条件结构和分支结构：

* if-else结构
* switch结构
* select结构，用于channel的选择。


* for (range) 结构
* break和continue，用于中途改变循环的状态。
* return，用于结束某个函数的执行。
* goto，用来调整程序的执行位置。

Go完全省略了if、switch和for结构中条件语句两侧的括号，相比Java、C++和C#中减少了很多视觉混乱的因素，同时也使的代码更加简洁。

## 1 if-else结构

if是用于测试某个条件（布尔型或逻辑型）的语句。如果该条件成立，则会执行if后由大括号括起来的代码块，否则就忽略该代码块继续执行后续的代码。即使当代码块之间只有一条语句时，大括号也不可被省略。关键字if和else之后的左大括号{必须和关键字在同一行。如果使用了`else-if`结构，则前段代码块的右大括号}必须和`else-if`关键字在同一行。在有些情况下，条件语句两侧的括号是可以被省略的；当条件比较复杂时，则可以使用括号让代码更易读。条件允许是符合条件，需使用`&&`、`||`或`!`，可以使用括号来提升某个表达式的运算优先级，并提高代码的可读性。当if结构内有break、continue、goto或者return语句时，Go代码的常见写法是省略else部分。

无论满足哪个条件都会返回x或者y时，一般使用以下写法：

```go
   if condition {
       return x
    }

   return y
```

- 判断一个字符串是否为空。

```go
   if str == "" { 
    ... 
   }

   if len(str) == 0 {
    ...
   }
```

- 判断运行Go程序的操作系统类型，这可以通过常量`runtime.GOOS`来判断。

```go
if runtime.GOOS == "windows" {
    ...
} else { // Unix-like
    ...
}
```

这段代码一般被放在init()函数中执行。 

- 函数Abs()用于返回一个整型数字的绝对值

```go
func Abs(x int) int {
    if x < 0 {
        return -x
    }

    return x
}
```

- isGreater()用于比较两个整型数字的大小

```go
func isGreater(x, y int) bool {
    if x > y {
        return true
    }

    return false
}
```

在第四种情况中，if可以包含一个初始化语句。这种写法具有固定的格式（在初始化语句后方必须加上分号）：

```go
if initialization; condition {
    // do something
}
```

例如:

```go
val := 10
if val > max {
    // do something
}
```

也可以这样写:

```go
if val := 10; val > max {
    // do something
}
```

但要注意的是，使用简短方式:=声明的变量的作用域只存在于if结构中（在if结构的大括号之间，如果使用if-else结构则在else代码块中变量也会存在）。如果变量在 if结构之前就已经存在，那么在if结构中，该变量原来的值会被隐藏。最简单的解决方案就是不要在初始化语句中声明变量。

下面的代码片段展示了如何通过在初始化语句中获取函数process()的返回值，并在条件语句中作为判定条件来决定是否执行if结构中的代码：

```go
if value := process(data); value > max {
    ...
}
```

## 2 测试多返回值函数的错误

Go语言的函数经常使用两个返回值来表示执行是否成功：返回某个值以及true表示成功；返回零值（或nil）或false表示失败。当不使用true或false的时候，也可以使用一个error类型的变量来代替作为第二个返回值：成功执行的话，error的值为nil，否则就会包含相应的错误信息（Go语言中的错误类型为 error: var err error。这样一来，就很明显需要用一个if语句来测试执行结果。由于其符号的原因，这样的形式又称之为comma,ok模式。

例如：`anInt, _ = strconv.Atoi(origStr)`

如果origStr不能被转换为整数，anInt的值会变成0，而_无视了错误，程序会继续运行。这样做是非常不好的：程序应该在最接近的位置检查所有相关的错误，至少需要暗示用户有错误发生并对函数进行返回，甚至中断程序。

### 2.1 检查函数返回值错误

通用方式为：

```go
value, err := pack1.Function1(param1)
if err!= nil {
    fmt.Printf("An error occured in pack1.Function1 
        with parameter %v", param1)
    return err
}

// 未发生错误，继续执行
```

示例：string_conversion2.go

```go
package main

import (
    "fmt"
    "strconv"
)

func main() {
    var orig string = "ABC"
    // var an int
    var newS string
    // var err error

    fmt.Printf("The size of ints is: %d\n", strconv.IntSize)  
  
    // anInt, err = strconv.Atoi(origStr)
    an, err := strconv.Atoi(orig)
    if err != nil {
        fmt.Printf("orig %s is not an integer - 
             exiting with error\n",  orig)
        return
    } 
                   
    fmt.Printf("The integer is %d\n", an)
    an = an + 5
    newS = strconv.Itoa(an)
    fmt.Printf("The new string is: %s\n", newS)
}
```

这是测试err变量是否包含一个真正的错误的习惯用法。如果确实存在错误，则会打印相应的错误信息然后通过return提前结束函数的执行。还可以使用携带返回值的 return形式，例如return err。这样一来，函数的调用者就可以检查函数执行过程中是否存在错误了。

由于本例的函数调用者属于main函数，程序会直接停止运行。

### 2.2 函数返回错误时终止程序运行

如果想要在错误发生的同时终止程序的运行，可以使用os包的Exit函数：

```go
if err != nil {
    fmt.Printf("Program stopping with error %v", err)
    os.Exit(1) //此处的退出代码 1 可以使用外部脚本获取到
}
```

有时候，会发现这种习惯用法被连续重复地使用在某段代码中。

### 2.3 函数没有返回错误的处理方法

当没有错误发生时，代码继续运行就是唯一要做的事情，所以if语句块后面不需要使用else分支。

示例：尝试通过os.Open方法打开一个名为name的只读文件：

```go
f, err := os.Open(name)
if err !=nil {
    return err   // 返回err
}
doSomething(f)  // 没有错误发生，文件对象被传入到某个函数中
doSomething
```

### 2.4 将错误的获取放置在if语句的初始化部分

```go
if err := file.Chmod(0664); err !=nil {
    fmt.Println(err)
     return err
}
```

### 2.5 将ok-pattern的获取放置在if语句的初始化部分并进行判断

```go
if value, ok := readData(); ok {
    …
}
```

【注意事项】

如果您像下面一样，没有为多返回值的函数准备足够的变量来存放结果：

```go
func mySqrt(f float64) (v float64, ok bool) {
    if f < 0 { return } // error case
    return math.Sqrt(f),true
}

func main() {
    t := mySqrt(25.0)
    fmt.Println(t)
}
```

这会得到一个编译错误：multiple-value mySqrt() in single-value context。正确的做法是：

```go
if t, ok := mySqrt(25.0); ok {
    fmt.Println(t)
}
```

【注意事项2】

当将字符串转换为整数时，且确定转换一定能够成功时，可以将Atoi函数进行一层忽略错误的封装：

```go
func atoi (s string) (n int) {
    n, _ = strconv.Atoi(s)
    return
}
```

实际上，fmt包最简单的打印函数也有2个返回值：

```go
count, err := fmt.Println(x) // number of bytes printed, nil or 0, error
```

当打印到控制台时，可以将该函数返回的错误忽略；但当输出到文件流、网络流等具有不确定因素的输出对象时，应该始终检查是否有错误发生。

## 3 switch结构

相比较C和Java等其它语言而言，Go语言中的switch结构使用上更加灵活。

### 3.1 通用形式的switch语句    

它接受任意形式的表达式：

```go
    switch var1 {
        case val1:
            ...
        case val2:
            ...
        default:
            ...
    }
```

变量var1可以是任何类型，而val1和val2则可以是同类型的任意值。**类型不被局限于常量或整数，但必须是相同的类型；或者最终结果为相同类型的表达式。**前花括号{必须和switch关键字在同一行。

可以同时测试多个可能符合条件的值，使用逗号分割它们，例如：case val1, val2, val3。每一个case分支都是唯一的，从上至下逐一测试，直到匹配为止。一旦成功地匹配到每个分支，在执行完相应代码后就会退出整个switch代码块，也就是说不需要特别使用break语句来表示结束。因此，程序也不会自动地去执行下一个分支的代码。如果在执行完每个分支的代码后，还希望继续执行后续分支的代码，可以使用fallthrough关键字来达到目的。

```go
    switch i {
        case 0: // 空分支，只有当 i == 0 时才会进入分支
        case 1:
            f() // 当 i == 0 时函数不会被调用
    }
```

并且：

```go
    switch i {
        case 0: fallthrough
        case 1:
            f() // 当 i == 0 时函数也会被调用
    }
```

在`case ...:`语句之后，不需要使用花括号将多行语句括起来，但可以在分支中进行任意形式的编码。当代码块只有一行时，可以直接放置在case语句之后。同样可以使用return语句来提前结束代码块的执行。当在switch语句块中使用return语句，并且函数是有返回值的，还需要在switch之后添加相应的return语句以确保函数始终会返回。

可选的default分支可以出现在任何顺序，但最好将它放在最后。它的作用类似与if-else语句中的else，表示不符合任何已给出条件时，执行相关语句。

示例switch1.go：

```go
package main

import "fmt"

func main() {
    var num1 int = 100

    switch num1 {
    case 98, 99:
        fmt.Println("It's equal to 98")
    case 100: 
        fmt.Println("It's equal to 100")
    default:
        fmt.Println("It's not equal to 98 or 100")
    }
}
```

### 3.2 没有提供判断值的switch语句

switch语句的第二种形式是不提供任何被判断的值（实际上默认为判断是否为true），然后在每个case分支中进行测试不同的条件。当任一分支的测试结果为true时，该分支的代码会被执行。这看起来非常像链式的if-else语句，但是在测试条件非常多的情况下，提供了可读性更好的书写方式。

```go
    switch {
        case condition1:
            ...
        case condition2:
            ...
        default:
            ...
    }
```

例如：

```go
    switch {
        case i < 0:
            f1()
        case i == 0:
            f2()
        case i > 0:
            f3()
    }
```

任何支持进行相等判断的类型都可以作为测试表达式的条件，包括int、string、指针等。

示例switch2.go：

```go
package main

import "fmt"

func main() {
    var num1 int = 7

    switch {
        case num1 < 0:
            fmt.Println("Number is negative")
        case num1 > 0 && num1 < 10:
            fmt.Println("Number is between 0 and 10")
        default:
            fmt.Println("Number is 10 or greater")
    }
}
```

输出：Number is between 0 and 10

### 3.3 包含初始化语句的switch

switch语句的第三种形式是包含一个初始化语句：

```go
    switch initialization {
        case val1:
            ...
        case val2:
            ...
        default:
            ...
    }
```

这种形式可以非常优雅地进行条件判断：

```go
    switch result := calculate(); {
        case result < 0:
            ...
        case result > 0:
            ...
        default:
            // 0
    }
```

在下面这个代码片段中，变量a和b被平行初始化，然后作为判断条件：

```go
    switch a, b := x[i], y[j]; {
        case a < b: t = -1
        case a == b: t = 0
        case a > b: t = 1
    }
```

switch语句还可以被用于type-switch来判断某个interface变量中实际存储的变量类型。

## 4 for结构

如果想要重复执行某些语句，Go语言中只有for结构可以使用。不要小看它，这个for结构比其它语言中的更为灵活。

### 4.1 基于计数器的迭代

文件for1.go演示了最简单的基于计数器的迭代，基本形式为：

```go
   for 初始化语句; 条件语句; 修饰语句 {}
```

示例for1.go：

```go
package main

import "fmt"

func main() {
    for i := 0; i < 5; i++ {
        fmt.Printf("This is the %d iteration\n", i)
    }
}
```

还可以在循环中同时使用多个计数器：

```go
   for i, j := 0, N; i < j; i, j = i+1, j-1 {}
```

这得益于Go语言具有的平行赋值的特性。

可以将两个for循环嵌套起来：

```go
for i:=0; i<5; i++ {
    for j:=0; j<10; j++ {
        println(j)
    }
}
```

如果使用for循环迭代一个Unicode 编码的字符串，会发生什么？

示例for_string.go：

```go
package main

import "fmt"

func main() {
    str := "Go is a beautiful language!"
    fmt.Printf("The length of str is: %d\n", len(str))
    for ix :=0; ix < len(str); ix++ {
        fmt.Printf("Character on position %d is: %c \n", 
                                             ix, str[ix])
    }
    str2 := "日本語"
    fmt.Printf("The length of str2 is: %d\n", len(str2))
    for ix :=0; ix < len(str2); ix++ {
        fmt.Printf("Character on position %d is: %c \n", ix, str2[ix])
    }
}
```

如果打印str和str2的长度，会分别得到27和9。由此可以发现，ASCII编码的字符占用1个字节，既每个索引都指向不同的字符，而非ASCII 编码的字符（占有2到4个字节）不能单纯地使用索引来判断是否为同一个字符。

### 4.2 基于条件判断的迭代

for结构的第二种形式是**没有头部的条件判断迭代**（类似其它语言中的while循环），基本形式为：

```go
for 条件语句 {}
```

也可以认为这是没有初始化语句和修饰语句的for结构，因此;;便是多余的了。

示例for2.go：

```go
package main

import "fmt"

func main() {
    var i int = 5

    for i >= 0 {
        i = i - 1
        fmt.Printf("The variable i is now: %d\n", i)
    }
}
```

### 4.3 无限循环

条件语句是可以被省略的，如 i:=0; ; i++ 或 for { } 或 for ;; { }（;;会在使用gofmt时被移除）。这些循环的本质就是无限循环。最后一个形式也可以被改写为for true { }，但一般情况下都会直接写for { }。

如果for循环的头部没有条件语句，那么就会认为条件永远为true，因此循环体内必须有相关的条件判断以确保会在某个时刻退出循环。想要直接退出循环体，可以使用 break语句或return语句直接返回。但这两者之间有所区别，break只是退出当前的循环体，而return语句提前对函数进行返回，不会执行后续的代码。

无限循环的经典应用是服务器，用于不断等待和接受新的请求。

```go
    for t, err = p.Token(); err == nil; t, err = p.Token() {
        ...
    }
```

### 4.4 for-range结构

这是Go特有的一种的迭代结构，它在许多情况下都非常有用。它可以迭代任何一个集合（包括数组和map）。语法上很类似其它语言中for-each语句，但依旧可以获得每次迭代所对应的索引。一般形式为：`for ix, val := range coll { }`。val始终为集合中对应索引的值拷贝，因此它一般只具有只读性质，对它所做的任何修改都不会影响到集合中原有的值。 

示例：

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

在上面的例子中，利用for-range结构打印切片resultSlice里的每一个值。

## 5 Break与continue

示例for3.go：

```go
  for {
        i = i - 1
        fmt.Printf("The variable i is now: %d\n", i)
        if i < 0 {
            break
        }
    }
```

因此每次迭代都会对条件进行检查（i < 0），以此判断是否需要停止循环。如果退出条件满足，则使用break语句退出循环。

一个 break的作用范围为该语句出现后的最内部的结构，它可以被用于任何形式的for循环（计数器、条件判断等）。但在switch或select语句中，break语句的作用结果是跳过整个代码块，执行后续的代码。

下面的示例中包含了嵌套的循环体，break只会退出最内层的循环： 

示例for4.go：

```go
package main

func main() {
    for i:=0; i<3; i++ {
        for j:=0; j<10; j++ {
            if j>5 {
                break   
            }
            print(j)
        }
        print("  ")
    }
}
```

输出： 012345 012345 012345

关键字continue忽略剩余的循环体而直接进入下一次循环的过程，但不是无条件执行下一次循环，执行之前依旧需要满足循环的判断条件。

示例for5.go：

```go
package main

func main() {
    for i := 0; i < 10; i++ {
        if i == 5 {
            continue
        }
        print(i)
        print(" ")
    }
}
```

输出：0 1 2 3 4 6 7 8 9

显然，5 被跳过了。

另外，关键字 continue 只能被用于 for 循环中。

## 6 标签与goto

for、switch 或 select 语句都可以配合标签（label）形式的标识符使用，即某一行第一个以冒号（:）结尾的单词（gofmt 会将后续代码自动移至下一行）。标签的名称是大小写敏感的，为了提升可读性，一般建议使用全部大写字母。

如果必须使用goto，应当只使用正序的标签（标签位于goto语句之后），但注意标签和goto语句之间不能出现定义新变量的语句，否则会导致编译失败。

示例：

```go
// compile error goto2.go:8: goto TARGET jumps over declaration of b at goto2.go:8
package main

import "fmt"

func main() {
        a := 1
        goto TARGET // compile error
        b := 9
    TARGET:  
        b += a
        fmt.Printf("a is %v *** b is %v", a, b)
}

```
