---
layout: post
title:  "Go教程08-包"
date:   2018-04-20 08:08:04
categories: Go语言编程
tags: Go 
excerpt: 在Go语言中，像fmt、os等这样具有常用功能的内置包有150个以上，它们被称为标准库，大部分内置于Go本身。
---

* content
{:toc}


## 1 标准库概述

像fmt、os等这样具有常用功能的内置包在Go语言中有150个以上，它们被称为标准库，大部分(一些底层的除外)内置于Go本身。具体可以到Go官方网站查阅各个包的情况：https://golang.google.cn/pkg/

## 2 regexp包

如果是简单模式匹配，使用Match方法便可：

```go
ok, _ := regexp.Match(pat, []byte(searchIn))
```

在上面代码中，变量ok将返回true或者false。

可以使用MatchString进行匹配：

```go
ok, _ := regexp.MatchString(pat, searchIn)
```

- 程序示例
```go
// @file:        PackageRegexp.go
// @author:      haulf
// @date:        2017.11.20
// @brief:       Regexp test.

package main

import (
    "fmt"
    "regexp"
    "strconv"
)

func main() {
    originalString := "John: 2578.34 William: 4567.23 Steve: 5632.18"

    fmt.Println("Original string is:", originalString)

    fmt.Println("Start to match")
    patternString := "[0-9]+.[0-9]+"
    if ok, _ := regexp.Match(patternString, []byte(originalString)); ok {
        fmt.Println("Match Found!")
    }

    regexpObject, _ := regexp.Compile(patternString)

    str := regexpObject.ReplaceAllString(originalString, "##.#")
    fmt.Println(str)

    paramFunc := func(s string) string {
        v, _ := strconv.ParseFloat(s, 32)
        return strconv.FormatFloat(v*2, 'f', 2, 32)
    }
    strFunc := regexpObject.ReplaceAllStringFunc(originalString, paramFunc)
    fmt.Println(strFunc)
}
```

- 程序运行结果
```go
Match Found!
John: ##.# William: ##.# Steve: ##.#
John: 5156.68 William: 9134.46 Steve: 11264.36
```

- 程序说明
`regexp.Compile()`函数也可能返回一个错误，在使用时忽略对错误的判断是因为确信自己正则表达式是有效的。当用户输入或从数据中获取正则表达式的时候，有必要去检验它的正确性。另外也可以使用MustCompile方法，它可以像Compile方法一样检验正则的有效性，但是当正则不合法时程序将panic。

## 3 锁和sync包

在一些复杂的程序中，通常会通过不同线程执行不同应用来实现程序的并发。当不同线程要使用同一个变量时，经常会出现一个问题，即无法预知变量被不同线程修改的顺序！这通常被称为资源竞争,指不同线程对同一变量使用的竞争。显然这种情况无法让人容忍，那该如何解决这个问题呢？

经典的做法是一次只能让一个线程对共享变量进行操作。当变量被一个线程改变时(即线程访问临界区时)，为它上锁，直到这个线程执行完成并解锁后，其它线程才能访问它。出于对性能的考虑，map类型是不存在锁的机制来实现这种效果,所以map类型是非线程安全的。当并行访问一个共享的map类型的数据，map数据将会出错。

在Go语言中这种锁的机制是通过sync包中Mutex来实现的。sync来源于"synchronized"一词，这意味着线程将有序的对同一变量进行访问。sync.Mutex是一个互斥锁，它的作用是守护在临界区入口来确保同一时间只能有一个线程进入临界区。

假设info是一个需要上锁的放在共享内存中的变量。通过包含Mutex来实现的一个典型例子如下：

```go
   import  “sync”
   
   type Info struct {
       mu sync.Mutex
       // ... other fields, e.g.: Str string
    }
```

如果一个函数想要改变这个变量可以这样写:

```go
    func Update(info *Info) {
       info.mu.Lock()
       // critical section:
       info.Str = // new value
       // end critical section
       info.mu.Unlock()
    }
```

还有一个很有用的例子是通过Mutex来实现一个可以上锁的共享缓冲器:

```go
   type SyncedBuffer struct {
       lock     sync.Mutex
       buffer   bytes.Buffer
    }
```

在sync包中还有一个RWMutex锁：它能通过RLock()来允许同一时间多个线程对变量进行读操作，但是只能一个线程进行写操作。如果使用Lock()将和普通的Mutex作用相同。包中还有一个方便的Once类型变量的方法once.Do(call)，这个方法确保被调用函数只能被调用一次。

相对简单的情况下，通过使用sync包可以解决同一时间只能一个线程访问变量或map类型数据的问题。如果这种方式导致程序明显变慢或者引起其它问题，要重新思考来通过goroutines和channels来解决问题，这是在Go语言中所提倡用来实现并发的技术。

## 4 精密计算和big包

有些时候通过编程的方式去进行计算是不精确的。如果使用Go语言中的float64类型进行浮点运算，返回结果将精确到15位，足以满足大多数任务的需要。当对超出int64或者uint64类型这样的大数进行计算时，如果对精度没有要求，float32或者float64可以胜任，但如果对精度有严格要求的时候，不能使用浮点数，在内存中它们只能被近似的表示。

对于整数的高精度计算Go语言中提供了big包。其中包含了math包：有用来表示大整数的big.Int和表示大有理数的big.Rat类型（可以表示为2/5或3.1416这样的分数，而不是无理数或π）。这些类型可以实现任意位类型的数字，只要内存足够大。缺点是更大的内存和处理开销使它们使用起来要比内置的数字类型慢很多。

大的整型数字是通过big.NewInt(n) 来构造的，其中n为int64类型整数。大有理数是用过big.NewRat(N,D)方法构造。N（分子）和D（分母）都是int64型整数。因为Go语言不支持运算符重载，所以所有大数字类型都有像是Add()和Mul()这样的方法。它们作用于作为receiver的整数和有理数，大多数情况下它们修改receiver并以receive作为返回结果。因为没有必要创建big.Int类型的临时变量来存放中间结果，所以这样的运算可通过内存链式存储。

* 程序示例

```go
// @file:        Big.go
// @author:      haulf
// @date:        2017.12.09
// @brief:       Big test.

package main

import (
    "fmt"
    "math"
    "math/big"
)

func main() {
    // Here are some calculations with bigInts:
    im := big.NewInt(math.MaxInt64)
    in := im
    io := big.NewInt(1956)
    ip := big.NewInt(1)
    ip.Mul(im, in).Add(ip, im).Div(ip, io)
    fmt.Printf("Big Int: %v\n", ip)

    // Here are some calculations with bigInts:
    rm := big.NewRat(math.MaxInt64, 1956)
    rn := big.NewRat(-1956, math.MaxInt64)
    ro := big.NewRat(19, 56)
    rp := big.NewRat(1111, 2222)
    rq := big.NewRat(1, 1)
    rq.Mul(rm, rn).Add(rq, ro).Mul(rq, rp)
    fmt.Printf("Big Rat: %v\n", rq)
}
```

- 运行结果

```shell
Big Int: 43492122561469640008497075573153004
Big Rat: -37/112
```

## 5 自定义包和可见性

包是Go语言中代码组织和代码编译的主要方式。当写自己包的时候，要使用短小的不含有_(下划线)的小写单词来命名。

### 5.1 导入外部安装包

如果要在的应用中使用一个或多个外部包，首先必须使用go install在的本地机器上安装它们。假设想使用http://codesite.ext/author/goExample/goex 这种托管在Google Code、GitHub和Launchpad等代码网站上的包，可以通过如下命令安装：

`go install codesite.ext/author/goExample/goex`

将一个名为codesite.ext/author/goExample/goex的map安装在$GOROOT/src/目录下。     

### 5.2 包的初始化

程序的执行开始于导入包，初始化main包然后调用main函数。

一个没有导入的包将通过分配初始值给所有的包级变量和调用源码中定义的包级init函数来初始化。一个包可能有多个init函数甚至在一个源码文件中。它们的执行是无序的。这是最好的例子来测定包的值是否只依赖于相同包下的其它值或者函数。

init函数是不能被调用的。

导入的包在包自身初始化前被初始化，而一个包在程序执行中只能初始化一次。

## 6 为自定义包使用godoc

godoc工具在显示自定义包中的注释也有很好的效果：注释必须以//开始并无空行放在声明（包，类型，函数）前。godoc会为每个文件生成一系列的网页。

- 命令行下进入目录下并输入命令

 `godoc -http =:6060 -goroot="."`

`.`是指当前目录，`-goroot`参数可以是/path/to/my/package1这样的形式指出package1在源码中的位置或接受用冒号形式分隔的路径，无根目录的路径为相对于当前目录的相对路径。

- 在浏览器打开地址：http://localhost:6060

然后会看到本地的godoc页面从左到右一次显示出目录中的包。

如果在一个团队中工作，并在源代码树被存储在网络硬盘上，就可以使用godoc给所有团队成员连续文档的支持。通过设置`sync_minutes=n`，甚至可以让它每n分钟自动更新文档！

## 7 使用go install安装自定义包

`go install`是Go中自动包安装工具：如需要将包安装到本地它会从远端仓库下载包：检出、编译和安装一气呵成。

在包安装前的先决条件是要自动处理包自身依赖关系的安装。被依赖的包也会安装到子目录下，但是没有文档和示例。

`go install`使用了GOPATH变量。
