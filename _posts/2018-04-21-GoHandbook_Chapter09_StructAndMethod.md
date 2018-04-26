---
layout: post
title:  "结构体与方法"
date:   2018-04-21 08:08:04
categories: Go语言编程
tags: Go 
excerpt: Go让人感觉像是Python或Ruby这样的动态语言，但却又拥有像C或者Java这类语言的高性能和安全性。Go语言出现的目的是希望在编程领域创造最实用的方式来进行软件开发。
---

* content
{:toc}



## 结构体与方法

Go通过类型别名（alias types）和结构体的形式支持用户自定义类型，或者叫定制类型。一个带属性的结构体试图表示一个现实世界中的实体。结构体是复合类型（composite types），当需要定义一个类型，它由一系列属性组成，每个属性都有自己的类型和值的时候，就应该使用结构体。它把数据聚集在一起，然后可以访问这些数据，就好像它是一个独立实体的一部分。结构体也是值类型，因此可以通过new函数来创建。

组成结构体类型的那些数据称为字段（fields）。每个字段都有一个类型和一个名字。在一个结构体中，字段名字必须是唯一的。

结构体的概念在软件工程上旧的术语叫ADT（抽象数据类型：Abstract Data Type）。在一些老的编程语言中叫记录（Record），比如Cobol。在C家族的编程语言中它也存在，并且名字也是struct。在面向对象的编程语言中，跟一个无方法的轻量级类一样。不过因为Go语言中没有类的概念，因此在Go中结构体有着更为重要的地位。

## 1 结构体定义

结构体定义的一般方式如下：

```go
   type identifier struct {
       field1 type1
       field2 type2
       ...
    }
```

typeT struct {a, b int} 也是合法的语法，它更适用于简单的结构体。

结构体里的字段都有名字，像field1、field2等，如果字段在代码中从来也不会被用到，那么可以命名它为_。结构体的字段可以是任何类型，甚至是结构体本身，也可以是函数或者接口。可以声明结构体类型的一个变量，然后像下面这样给它的字段赋值：

```go
      var s T
      s.a = 5
      s.b = 8
```

数组可以看作是一种结构体类型，不过它使用下标而不是具名的字段。

### 1.1 使用new创建指向结构体指针

使用new函数给一个新的结构体变量分配内存，它返回指向已分配内存的指针：var t *T = new(T)，如果需要可以把这条语句放在不同的行（比如定义是包范围的，但是分配却没有必要在开始就做）。

```go
    var t *T
    t = new(T)
```

写这条语句的惯用方法是：t := new(T)，变量t是一个指向T的指针，此时结构体字段的值是它们所属类型的零值。声明var t T也会给t分配内存，并零值化内存，但是这个时候t是类型T。在这两种方式中，t通常被称做类型T的一个实例（instance）或对象（Object）。

* 程序示例

```go
// @file:        structs_fields.go
// @version:     1.0
// @author:      haulf
// @date:        2017.12.10
// @go version:  1.9
// @brief:       Struct test.

package main

import (
    "fmt"
)

type TestStruct struct {
    iField   int
    fField   float32
    strField string
}

func main() {
    ts := new(TestStruct)
    ts.iField = 10
    ts.fField = 15.5
    ts.strField = "aihaofeng"

    fmt.Printf("The int is: %d\n", ts.iField)
    fmt.Printf("The float is: %f\n", ts.fField)
    fmt.Printf("The string is: %s\n", ts.strField)
    fmt.Println(ts)
}
```

* 运行结果

```shell
The int is: 10
The float is: 15.500000
The string is: aihaofeng
&{10 15.5 aihaofeng}
```

使用fmt.Println打印一个结构体的默认输出可以很好的显示它的内容，类似使用%v选项。就像在面向对象语言所作的那样，可以使用点号符给字段赋值：structname.fieldname = value。同样的，使用点号符可以获取结构体字段的值：structname.fieldname。在Go语言中这叫**选择器（selector）**。无论变量是一个结构体类型还是一个结构体类型指针，都使用同样的**选择器符（selector-notation）**来引用结构体的字段：

```go
   type myStruct struct { 
       i int 
   }

   var v myStruct    // v是结构体类型变量
   var p *myStruct   // p是指向一个结构体类型变量的指针

   v.i
   p.i
```

### 1.2 初始化结构体实例

初始化一个结构体实例(一个结构体字面量：struct-literal)的更简短和惯用的方式如下：

```go
ms := &struct1{10, 15.5, "Chris"}  // 此时ms的类型是 *struct1
```

或者：

```go
var ms struct1
ms = struct1{10, 15.5, "Chris"}
```

**混合字面量语法**（compositeliteral syntax）&struct1{a, b, c}是一种简写，底层仍然会调用new()，这里值的顺序必须按照字段顺序来写。在下面的例子中能看到可以通过在值的前面放上字段名来初始化字段的方式。表达式new(Type)和&Type{}是等价的。

时间间隔（开始和结束时间以秒为单位）是使用结构体的一个典型例子：

```go
type Interval struct {
    start int
    end   int
}
```

初始化方式：

```go
intr := Interval{0, 3}              (A)
intr := Interval{end:5, start:1}    (B)
intr := Interval{end:5}             (C)
```

在（A）中，值必须以字段在结构体定义时的顺序给出，&不是必须的。（B）显示了另一种方式，字段名加一个冒号放在值的前面，这种情况下值的顺序不必一致，并且某些字段还可以被忽略掉，就像（C）中那样。

## 2 使用工厂方法创建结构体实例

### 2.1 结构体工厂

Go语言不支持面向对象编程语言中那样的构造子方法，但是可以很容易的在Go中实现"构造子工厂"方法。为了方便通常会为类型定义一个工厂，按惯例，工厂的名字以new或New开头。假设定义了如下的File结构体类型：

```go
   type File struct {
       fd      int     // 文件描述符
       name    string  // 文件名
    }
```

下面是这个结构体类型对应的工厂方法，它返回一个指向结构体实例的指针：

```go
   func NewFile(fd int, name string) *File {
       if fd < 0 {
           return nil
       }
    
       return &File{fd, name}
    }
```

然后可以用f:= NewFile(10, "./test.txt")方式进行调用。如果File是一个结构体类型，那么表达式new(File)和&File{}是等价的。如果想知道结构体类型T的一个实例占用了多少内存，可以使用：

```go
size := unsafe.Sizeof(T{})。
```

### 2.2 如何强制使用工厂方法

通过应用可见性规则就可以禁止使用new函数，强制用户使用工厂方法，从而使类型变成私有的，就像在面向对象语言中那样。

```go
    type matrix struct {
       ...
    }   

   func NewMatrix(params) *matrix {
       m := new(matrix) // 初始化m

       return m
    }
```

在其它包里使用工厂方法：

```go
   package main

   import "matrix"

   ...

   wrong := new(matrix.matrix)     // 编译失败（matrix是私有的）
   right := matrix.NewMatrix(...)  // 实例化matrix的唯一方式
```

### 2.3 map和struct vs new()和make()

现在为止已经见到了可以使用make()的三种类型中的其中两个：

`slices  /  maps / channels`

下面的例子来说明了在映射上使用new和make的区别，以及可能的发生的错误：

示例new_make.go（不能编译）

```go
   package main
    
   type Foo map[string]string

   type Bar struct {
       thingOne string
       thingTwo int
    }  

   func main() {
       // OK
       y := new(Bar)
       (*y).thingOne = "hello"
       (*y).thingTwo = 1
   

       // NOT OK
       z := make(Bar) // 编译错误：cannot make type Bar
       (*y).thingOne = "hello"
       (*y).thingTwo = 1
 

       // OK
       x := make(Foo)
       x["x"] = "goodbye"
       x["y"] = "world"

   
       // NOT OK
       u := new(Foo)
       (*u)["x"] = "goodbye" // 运行时错误!! panic: assignment to entry in nil map
       (*u)["y"] = "world"
    }
```

试图make() 一个结构体变量，会引发一个编译错误，这还不是太糟糕，但是new()一个映射并试图使用数据填充它，将会引发运行时错误！因为new(Foo) 返回的是一个指向nil的指针，它尚未被分配内存。所以在使用map时要特别谨慎。

## 3 使用自定义包中的结构体

* 程序示例

【struct_pack/structPack.go】

```go
// @file:        structPack.go
// @version:     1.0
// @author:      haulf
// @date:        2017.09.08
// @go version:  1.9
// @brief:       Struct test.

package struct_pack

type ExpStruct struct {
    Mi1 int
    Mf1 float32
}
```

主程序【self_define.go】

```go
// @file:        self_define.go
// @version:     1.0
// @author:      haulf
// @date:        2017.12.10
// @go version:  1.9
// @brief:       Struct test.

package main

import (
    "./struct_pack"
    "fmt"
)

func main() {
    struct1 := new(struct_pack.ExpStruct)
    struct1.Mi1 = 10
    struct1.Mf1 = 16.

    fmt.Printf("Mi1 = %d\n", struct1.Mi1)
    fmt.Printf("Mf1 = %f\n", struct1.Mf1)
}
```

* 程序运行结果

```shell
Mi1 = 10
Mf1 = 16.000000
```

## 4 带标签的结构体

结构体中的字段除了有名字和类型外，还可以有一个可选的标签。它是一个附属于字段的字符串，可以是文档或其它的重要标记。标签的内容不可以在一般的编程中使用，只有包reflect能获取它。reflect包可以在运行时自省类型、属性和方法，比如在一个变量上调用reflect.TypeOf()可以获取变量的正确类型，如果变量是一个结构体类型，就可以通过Field来索引结构体的字段，然后就可以使用Tag属性。

* 程序示例

【struct_tag.go】

```go
// @file:        struct_tag.go
// @version:     1.0
// @author:      haulf
// @date:        2017.12.10
// @go version:  1.9
// @brief:       Struct test.

package main

import (
    "fmt"
    "reflect"
)

type TagType struct { // tags
    field1 bool   "An important answer"
    field2 string "The name of the thing"
    field3 int    "How much thereare"
}

func main() {
    tt := TagType{true, "Barak Obama", 1}
    for i := 0; i < 3; i++ {
        refTag(tt, i)
    }
}

func refTag(tt TagType, ix int) {
    ttType := reflect.TypeOf(tt)
    ixField := ttType.Field(ix)

    fmt.Printf("%v\n", ixField.Tag)
}
```

* 程序运行结果

```shell
An important answer
The name of the thing
How much thereare
```

## 5 匿名字段和内嵌结构体

### 5.1 定义

结构体可以包含一个或多个匿名（或内嵌）字段，即这些字段没有显式的名字，只有字段的类型是必须的，此时类型就是字段的名字。匿名字段本身可以是一个结构体类型，即结构体可以包含内嵌结构体。可以粗略地将这个和面向对象语言中的继承概念相比较，随后将会看到它被用来模拟类似继承的行为。Go语言中的继承是通过内嵌或组合来实现的，所以可以说，在Go语言中，相比较于继承，组合更受青睐。

* 程序示例

【structs_anonymous_fields.go】

```go
// @file:        structs_anonymous_fields.go
// @version:     1.0
// @author:      haulf
// @date:        2017.12.10
// @go version:  1.9
// @brief:       Struct test.

package main

import (
    "fmt"
)

type innerS struct {
    in1 int
    in2 int
}

type outerS struct {
    b      int
    c      float32
    int    // anonymous field
    innerS //anonymous field
}

func main() {
    outer := new(outerS)
    outer.b = 6
    outer.c = 7.5
    outer.int = 60

    outer.in1 = 5
    outer.in2 = 10

    fmt.Printf("outer.b is: %d\n", outer.b)
    fmt.Printf("outer.c is: %f\n", outer.c)

    fmt.Printf("outer.int is: %d\n", outer.int)

    fmt.Printf("outer.in1 is: %d\n", outer.in1)
    fmt.Printf("outer.in2 is:%d\n", outer.in2)

    // 使用结构体字面量
    outer2 := outerS{6, 7.5, 60, innerS{5, 10}}
    fmt.Println("outer2 is:", outer2)
}
```

* 程序运行结果

```shell
outer.b is: 6
outer.c is: 7.500000
outer.int is: 60
outer.in1 is: 5
outer.in2 is:10
outer2 is: {6 7.5 60 {5 10}}
```

从上面的例子中是通过类型outer.int的名字来获取存储在匿名字段中的数据。因此，在一个结构体中对于每一种数据类型只能有一个匿名字段。

### 5.2 内嵌结构体

同样地，结构体也是一种数据类型，它也可以作为一个匿名字段来使用。在上面的例子中，外层结构体通过outer.in1直接进入内层结构体的字段，内嵌结构体甚至可以来自其它包。内层结构体被简单的插入或者内嵌进外层结构体。这个简单的“继承”机制提供了一种方式，使得可以从另外一个或一些类型继承部分或全部实现。

* 程序示例

【embedd_struct.go】

```go
// @file:        embedd_struct.go
// @version:     1.0
// @author:      haulf
// @date:        2017.12.10
// @go version:  1.9
// @brief:       Struct test.

package main

import (
    "fmt"
)

type A struct {
    ax, ay int
}

type B struct {
    A
    bx, by float32
}

func main() {
    b := B{A{1, 2}, 3.0, 4.0}

    fmt.Println(b.ax, b.ay, b.bx, b.by)
    fmt.Println(b.A)
}
```

* 程序运行结果

```shell
1 2 3 4
{1 2}
```

### 5.3 命名冲突

当结构体中的两个字段拥有相同的名字（可能是继承来的名字）时该怎么办呢？

1. 外层名字会覆盖内层名字（但是两者的内存空间都保留），这提供了一种重载字段或方法的方式。
2. 如果相同的名字在同一级别出现了两次，如果这个名字被程序使用了，将会引发一个错误（不使用没关系）。没有办法来解决这种问题引起的二义性，必须由程序员自己修正。

例子：

```go
   type A struct {a int}
   type B struct {a, b int}

   type C struct {A; B}
   var c C;
```

规则：

使用c.a是错误的，到底是c.A.a还是c.B.a呢？这种情况会导致编译器错误。

* 程序示例

```go
// @file:        MyStruct.go
// @version:     1.0
// @author:      haulf
// @date:        2017.09.08
// @go version:  1.9
// @brief:       Demo test.

package main

import (
    "fmt"
)

type A struct {
    a int
    b int
}

type B struct {
    a int
    b int
}

type C struct {
    A
    B
}

func main() {
    var c C
    c.A.a = 5
    c.A.b = 6
    c.B.a = 7
    c.B.b = 8

    // error: ./MyStruct.go:37:15: ambiguous selector c.a
    // fmt.Println(c.a)

    // OK
    fmt.Println(c.A.a)
    fmt.Println(c.A.b)
    fmt.Println(c.B.a)
    fmt.Println(c.B.b)
}
```

* 程序运行结果

```shell
5
6
7
8
```

另外的情况：

```go
   type D struct {B; b float32}
   var d D;
```

规则：

使用d.b是没问题的：它是float32，而不是B的b。如果想要内层的b可以通过d.B.b得到。

## 6 方法

### 6.1 方法是什么

在Go语言中，结构体就像是类的一种简化形式，那么面向对象程序员可能会问：类的方法在哪里呢？在Go中有一个概念，它和方法有着同样的名字，并且大体上意思相同：Go方法是作用在接收者（receiver）上的一个函数，接收者是某种类型的变量。因此，方法是一种特殊类型的函数。接收者类型可以是（几乎）任何类型，不仅仅是结构体类型。任何类型都可以有方法，甚至可以是函数类型，可以是int、bool、string或数组的别名类型。但是接收者不能是一个接口类型，因为接口是一个抽象定义，但是方法却是具体实现。最后接收者不能是一个指针类型，但是它可以是任何其它允许类型的指针。

一个类型加上它的方法等价于面向对象中的一个类。一个重要的区别是，在Go中，类型的代码和绑定在它上面的方法的代码可以不放置在一起，它们可以存在在不同的源文件，唯一的要求是它们必须是同一个包的。

类型`T（或 *T）`上的所有方法的集合叫做类型`T（或 *T）`的方法集。

因为方法是函数，所以同样的，不允许方法重载，即对于一个类型只能有一个给定名称的方法。但是如果基于接收者类型，是有重载的：具有同样名字的方法可以在2个或多个不同的接收者类型上存在，比如在同一个包里这么做是允许的：

```go
   func (a *denseMatrix) Add(b Matrix) Matrix
   func (a *sparseMatrix) Add(b Matrix) Matrix
```

别名类型不能有它原始类型上已经定义过的方法。

定义方法的一般格式如下：

```go
   func (recv receiver_type) methodName(parameter_list) (return_value_list){ ... }
```

在方法名之前，func关键字之后的括号中指定receiver。

如果recv是receiver的实例，Method1是它的方法名，那么方法调用遵循传统的object.name选择器符号：recv.Method1()。

如果recv一个指针，Go会自动解引用。如果方法不需要使用recv的值，可以用_替换它，比如：

```go
   func (_ receiver_type) methodName(parameter_list) (return_value_list) {... }
```

recv就像是面向对象语言中的this或self，但是Go中并没有这两个关键字。随个人喜好，可以使用this或self作为receiver的名字。下面是一个结构体上的简单方法的例子：

* 程序示例

【method .go】

```go
// @file:        method.go
// @version:     1.0
// @author:      haulf
// @date:        2017.12.10
// @go version:  1.9
// @brief:       Method test.

package main

import (
    "fmt"
)

type TwoInts struct {
    a int
    b int
}

func main() {
    two1 := new(TwoInts)
    two1.a = 12
    two1.b = 10

    fmt.Printf("The sum is: %d\n", two1.AddThem())
    fmt.Printf("Add them to the param: %d\n", two1.AddToParam(20))
    two2 := TwoInts{3, 4}

    fmt.Printf("The sum is: %d\n", two2.AddThem())
}

func (tn *TwoInts) AddThem() int {
    return tn.a + tn.b
}

func (tn *TwoInts) AddToParam(param int) int {
    return tn.a + tn.b + param
}
```

* 程序运行结果

```shell
The sum is: 22
Add them to the param: 42
The sum is: 7
```
  

下面是非结构体类型上方法的例子：

* 程序示例

【method2.go】

```go
// @file:        method2.go
// @version:     1.0
// @author:      haulf
// @date:        2017.12.10
// @go version:  1.9
// @brief:       Method test.

package main

import (
    "fmt"
)

type IntVector []int

func (v IntVector) Sum() (s int) {
    for _, x := range v {
        s += x
    }
    return
}

func main() {
    fmt.Println(IntVector{1, 2, 3}.Sum()) // 输出是6
}
```


练习iteration_list.go

下面这段代码有什么错？

```go
   package main
   

   import "container/list"
   

    func (p *list.List) Iter() {
       // ...
    }

    
   func main() {
       lst := new(list.List)
       for _= range list.Iter() {    

       }
    }
```

类型和作用在它上面定义的方法必须在同一个包里定义，这就是为什么不能在int、float或类似这些的类型上定义方法。试图在int类型上定义方法会得到一个编译错误：`cannot define new methods on non-local type int`

比如想在time.Time上定义如下方法：

```go
   func (t time.Time) first3Chars() string {
       return time.LocalTime().String()[0:3]
   }
```

类型在其它的，或是非本地的包里定义，在它上面定义方法都会得到和上面同样的错误。

但是有一个间接的方式：可以先定义该类型（比如：int或float）的别名类型，然后再为别名类型定义方法。或者像下面这样将它作为匿名类型嵌入在一个新的结构体中。当然方法只在这个别名类型上有效。

* 程序示例【method_on_time.go】

```go
// @file:        method_on_time.go
// @version:     1.0
// @author:      haulf
// @date:        2017.12.10
// @go version:  1.9
// @brief:       Method test.

package main

import (
    "fmt"
    "time"
)

type myTime struct {
    time.Time // anonymous field
}

func (t myTime) first3Chars() string {
    return t.Time.String()[0:3]
}

func main() {
    m := myTime{time.Now()}
    // 调用匿名Time上的String方法
    fmt.Println("Full time now:", m.String())
    // 调用myTime.first3Chars
    fmt.Println("First 3 chars:", m.first3Chars())
}
```

* 程序运行结果

```shell
Full time now: 2017-12-10 14:16:55.072421 +0800 CST m=+0.000322643
First 3 chars: 201
```

### 6.2 函数和方法的区别

函数将变量作为参数：`Function1(recv)`。方法在变量上被调用：`recv.Method1()`

在接收者是指针时，方法可以改变接收者的值（或状态），这点函数也可以做到（当参数作为指针传递，即通过引用调用时，函数也可以改变参数的状态）。

不要忘记Method1后边的括号()，否则会引发编译器错误：method recv.Method1 is not an expression, must be called

接收者必须有一个显式的名字，这个名字必须在方法中被使用。

receiver_type叫做（接收者）基本类型，这个类型必须在和方法同样的包中被声明。

在Go中，（接收者）类型关联的方法不写在类型结构里面，就像类那样；耦合更加宽松；类型和方法之间的关联由接收者来建立。

方法没有和数据定义（结构体）混在一起：它们是正交的类型；表示（数据）和行为（方法）是独立的。

### 6.3 指针或值作为接收者

鉴于性能的原因，recv最常见的是一个指向receiver_type的指针（因为不想要一个实例的拷贝，如果按值调用的话就会是这样），特别是在receiver类型是结构体时，就更是如此了。

如果想要方法改变接收者的数据，就在接收者的指针类型上定义该方法。否则，就在普通的值类型上定义方法。

下面的例子pointer_value.go作了说明：change()接受一个指向B的指针，并改变它内部的成员；write()接受通过拷贝接受B的值并只输出B的内容。注意Go为做了探测工作，自己并没有指出是否在指针上调用方法，Go替做了这些事情。b1是值而b2是指针，方法都支持运行了。

程序示例

【pointer_value.go】

```go
// @file:        pointer_value.go
// @version:     1.0
// @author:      haulf
// @date:        2017.12.10
// @go version:  1.9
// @brief:       Method test.

package main

import (
    "fmt"
)

type B struct {
    thing int
}

func (b *B) change()      { b.thing = 1 }
func (b B) write() string { return fmt.Sprint(b) }

func main() {
    var b1 B // b1是值
    b1.change()
    fmt.Println(b1.write())

    b2 := new(B) // b2是指针
    b2.change()
    fmt.Println(b2.write())
}
```

程序运行结果

```shell
{1}
{1}
```

试着在write()中改变接收者b的值：将会看到它可以正常编译，但是开始的b没有被改变。知道方法将指针作为接收者不是必须的，如下面的例子，只是需要 Point3的值来做计算：

```go
   type Point3 struct { x, y, z float }

   // A method on Point3
   func (p Point3) Abs float {
       return math.Sqrt(p.xp.x + p.yp.y + p.z*p.z)
   }
```

这样做稍微有点昂贵，因为Point3是作为值传递给方法的，因此传递的是它的拷贝，这在Go中合法的。也可以在指向这个类型的指针上调用此方法（会自动解引用）。

假设p3定义为一个指针：`p3 := &Point{ 3, 4, 5}`，可以使用p3.Abs()来替代`(*p3).Abs()`。

像例子method1.go中接收者类型是`*TwoInts`的方法AddThem()，它能在类型TwoInts的值上被调用，这是自动间接发生的。因此two2.AddThem可以替代(&two2).AddThem()。

 

在值和指针上调用方法：

可以有连接到类型的方法，也可以有连接到类型指针的方法。但是这没关系：对于类型T，如果在`*T`上存在方法`Meth()`，并且t是这个类型的变量，那么`t.Meth()`会被自动转换为`(&t).Meth()`。

指针方法和值方法都可以在指针或非指针上被调用，如下面程序所示，类型List在值上有一个方法Len()，在指针上有一个方法Append()，但是可以看到两个方法都可以在两种类型的变量上被调用。

### 6.4 方法和未导出字段

考虑 person2.go 中的 person 包：类型 Person 被明确的导出了，但是它的字段没有被导出。例如在use_person2.go 中 p.firstName 就是错误的。该如何在另一个程序中修改或者只是读取一个 Person 的名字呢？

这可以通过面向对象语言一个众所周知的技术来完成：提供 getter 和 setter 方法。对于 setter 方法使用 Set 前缀，对于 getter 方法只适用成员名。

程序示例【person/person.go】

```go
// @file:        person.go
// @version:     1.0
// @author:      haulf
// @date:        2017.12.10
// @go version:  1.9
// @brief:       Method test.

package person

type Person struct {
    firstName string
    lastName  string
}

func (p *Person) GetFirstName() string {
    return p.firstName
}

func (p *Person) SetFirstName(newName string) {
    p.firstName = newName
} 
```

程序示例【use_person.go】

```go
// @file:        use_person.go
// @version:     1.0
// @author:      haulf
// @date:        2017.12.10
// @go version:  1.9
// @brief:       Method test.

package main

import (
    "./person"
    "fmt"
)

func main() {
    p := new(person.Person)
    // p.firstName undefined
    // (cannot refer to unexported field or method firstName)
    // p.firstName = "Eric"
    p.SetFirstName("Eric")
    fmt.Println(p.GetFirstName()) // Output: Eric
}
```

**并发访问对象**

对象的字段（属性）不应该由2个或2个以上的不同线程在同一时间去改变。如果在程序发生这种情况，为了安全并发访问，可以使用包sync中的方法来处理。

### 6.5 内嵌类型的方法和继承

当一个匿名类型被内嵌在结构体中时，匿名类型的可见方法也同样被内嵌，这在效果上等同于外层类型继承了这些方法：将父类型放在子类型中来实现。这个机制提供了一种简单的方式来模拟经典面向对象语言中的子类和继承相关的效果。

下面是一个示例：假定有一个Engine接口类型，一个Car结构体类型，它包含一个Engine类型的匿名字段：

```go
   type Engine interface {
       Start()
       Stop()
    }   

   type Car struct {
       Engine
    }
```

可以构建如下的代码：

```go
   func (c *Car) GoToWorkIn() {
       // get in car
       c.Start()
       // drive to work
       c.Stop()
       // get out of car
    }
```

下面是 method3.go的完整例子，它展示了内嵌结构体上的方法可以直接在外层类型的实例上调用：

```go
// @file:        method3.go
// @version:     1.0
// @author:      haulf
// @date:        2017.12.10
// @go version:  1.9
// @brief:       Method test.

package main

import (
    "fmt"
    "math"
)

type Point struct {
    x, y float64
}

func (p *Point) Abs() float64 {
    return math.Sqrt(p.x*p.x + p.y*p.y)
}

type NamedPoint struct {
    Point  // 内嵌的匿名结构体
    name string
}

func main() {
    n := &NamedPoint{Point{3, 4}, "Pythagoras"}
    fmt.Println(n.Abs()) // 打印5
}
```

内嵌将一个已存在类型的字段和方法注入到了另一个类型里。匿名字段上的方法“晋升”成为了外层类型的方法。当然类型可以有只作用于本身实例而不作用于内嵌“父”类型上的方法。

可以覆写方法（像字段一样）：和内嵌类型方法具有同样名字的外层类型的方法，会覆写内嵌类型对应的方法。

在示例method4.go中添加：

```go
// @file:        method4.go
// @version:     1.0
// @author:      haulf
// @date:        2017.12.10
// @go version:  1.9
// @brief:       Method test.

package main

import (
    "fmt"
    "math"
)

type Point struct {
    x, y float64
}

func (p *Point) Abs() float64 {
    return math.Sqrt(p.x*p.x + p.y*p.y)
}

type NamedPoint struct {
    Point
    name string
}

func (n *NamedPoint) Abs() float64 {
    return n.Point.Abs() * 100.
}

func main() {
    n := &NamedPoint{Point{3, 4}, "Pythagoras"}
    fmt.Println(n.Abs()) // 打印500
}
```

现在fmt.Println(n.Abs())会打印500。NamedPoint结构体的Abs()方法会覆盖Point这个内嵌结构体的，具有相同名称的Abs()方法。

因为一个结构体可以嵌入多个匿名类型，所以实际上可以有一个简单版本的多重继承，就像：type Child struct { Father; Mother}。结构体内嵌于和自己在同一个包中的结构体时，可以彼此访问对方所有的字段和方法。

### 6.6 如何在类型中嵌入功能

主要有两种方法来实现在类型中嵌入功能：

A：聚合（或组合）：包含一个所需功能类型的具名字段。 

B：内嵌：内嵌（匿名地）所需功能类型。

为了使这些概念具体化，假设有一个Customer类型，想让它通过Log类型来包含日志功能，Log类型只是简单地包含一个累积的消息（当然它可以是复杂的）。如果想让特定类型都具备日志功能，可以实现一个这样的Log类型，然后将它作为特定类型的一个字段，并提供Log()，它返回这个日志的引用。

方式A可以通过如下方法实现：

程序示例【embed_func1.go】

```go
// @file:        embed_func1.go
// @version:     1.0
// @author:      haulf
// @date:        2017.12.10
// @go version:  1.9
// @brief:       Function test.

package main

import (
    "fmt"
)

type Log struct {
    msg string
}

type Customer struct {
    Name string
    log  *Log
}

func main() {
    c := new(Customer)
    c.Name = "Barak Obama"
    c.log = new(Log)
    c.log.msg = "1 - Yes we can!"

    // 上面的代码可以用下面更加简洁的方式代替
    //c = &Customer{"Barak Obama", &Log{"1 - Yes wecan!"}}

    c.Log().Add("2 - After me the world will be a better place!")

    fmt.Println(c.Log().String())
    //fmt.Println(c.Log())
}

func (l *Log) Add(s string) {
    l.msg += "\n" + s
}

func (l *Log) String() string {
    return l.msg
}

func (c *Customer) Log() *Log {
    return c.log
}
```

程序运行结果

```shell
1 - Yes wecan!
2 - After me the world will be a better place!
```


相对的方式B可能会像这样：

程序示例【embed_func1_B.go】

```go
// @file:        embed_func1_B.go
// @version:     1.0
// @author:      haulf
// @date:        2017.12.10
// @go version:  1.9
// @brief:       Function test.

package main

import (
    "fmt"
)

type Log struct {
    msg string
}

type Customer struct {
    Name string
    Log
}

func main() {
    c := &Customer{"Barak Obama", Log{"1 - Yes wecan!"}}
    c.Add("2 - After me the world will be a better place!")
    fmt.Println(c)
}

func (l *Log) Add(s string) {
    l.msg += "\n" + s
}

func (l *Log) String() string {
    return l.msg
}

func (c *Customer) String() string {
    return c.Name + "\nLog:" + fmt.Sprintln(c.Log)
}
```

程序运行结果

```shell
Barak Obama
Log:{1 - Yes wecan!
2 - After me the world will be a better place!}
```

内嵌的类型不需要指针，Customer结构体也不需要Add方法，它使用Log的Add方法，Customer有自己的String方法，并且在它里面调用了Log的String 方法。如果内嵌类型嵌入了其它类型，也是可以的，那些类型的方法可以直接在外层类型中使用。因此，一个好的策略是创建一些小的、可复用的类型作为一个工具箱，用于组成域类型。

### 6.7 多重继承

多重继承指的是类型获得多个父类型行为的能力，它在传统的面向对象语言中通常是不被实现的（C++和Python例外）。因为在类继承层次中，多重继承会给编译器引入额外的复杂度。但是在Go语言中，通过在类型中嵌入所有必要的父类型，可以很简单的实现多重继承。

作为一个例子，假设有一个类型CameraPhone，通过它可以Call()，也可以TakeAPicture()，但是第一个方法属于类型Phone，第二个方法属于类型Camera。只要嵌入这两个类型就可以解个问题。

* 程序示例

```go
// @file:        camera_phone.go
// @version:     1.0
// @author:      haulf
// @date:        2017.12.10
// @go version:  1.9
// @brief:       Struct test.

package main

import (
    "fmt"
)

type Camera struct{}

func (c *Camera) TakeAPicture() string {
    return "Click"
}

type Phone struct{}

func (p *Phone) Call() string {
    return "Ring Ring"
}

type CameraPhone struct {
    Camera
    Phone
}

func main() {
    cp := new(CameraPhone)
    fmt.Println("Our new CameraPhone exhibits multiple behaviors...")
    fmt.Println("It exhibits behavior of a Camera: ", cp.TakeAPicture())
    fmt.Println("It works like a Phone too: ", cp.Call())
}
```

* 程序运行结果

```shell
Our new CameraPhone exhibits multiple behaviors...
It exhibits behavior of a Camera:  Click
It works like a Phone too:  Ring Ring
```

### 6.8 通用方法和方法命名

在编程中一些基本操作会一遍又一遍的出现，比如打开（Open）、关闭（Close）、读（Read）、写（Write）、排序（Sort）等等，并且它们都有一个大致的意思：打开（Open）可以作用于一个文件、一个网络连接、一个数据库连接等等。具体的实现可能千差万别，但是基本的概念是一致的。在Go语言中，通过使用接口，标准库广泛的应用了这些规则。在标准库中这些通用方法都有一致的名字，比如Open()、Read()、Write()等。想写规范的Go程序，就应该遵守这些约定，给方法合适的名字和签名，就像那些通用方法那样。这样做会使Go开发的软件更加具有一致性和可读性。比如：如果需要一个convert-to-string方法，应该命名为String()，而不是ToString()。上面的程序中，都是命名为String()的方法。

### 6.9 和其它面向对象语言比较Go的类型和方法

在如C++、Java、C#和Ruby这样的面向对象语言中，方法在类的上下文中被定义和继承：在一个对象上调用方法时，运行时会检测类以及它的超类中是否有此方法的定义，如果没有会导致异常发生。

在Go语言中，这样的继承层次是完全没必要的：如果方法在此类型定义了，就可以调用它，和其它类型上是否存在这个方法没有关系。在这个意义上，Go具有更大的灵活性。

下面的模式就很好的说明了这个问题：

Go不需要一个显式的类定义，如同Java、C++、C#等那样，相反地，“类”是通过提供一组作用于一个共同类型的方法集来隐式定义的。类型可以是结构体或者任何用户自定义类型。比如：想定义自己的Integer类型，并添加一些类似转换成字符串的方法，在Go中可以如下定义：

```go
   type Integer int

   func (i *Integer) String() string {
       return strconv.Itoa(i)
   }
```

在Java或C#中，这个方法需要和类Integer的定义放在一起，在Ruby中可以直接在基本类型int上定义这个方法。 

**总结**

在Go中，类型就是类（数据和关联的方法）。Go不知道类似面向对象语言的类继承的概念。继承有两个好处：代码复用和多态。在Go中，代码复用通过组合和委托实现，多态通过接口的使用来实现，有时这也叫组件编程（Component Programming）。许多开发者说相比于类继承，Go的接口提供了更强大、却更简单的多态行为。 

如果真的需要更多面向对象的能力，看一下goop包（Go Object-Oriented Programming），它由Scott Pakin编写。它给Go提供了JavaScript风格的对象（基于原型的对象），并且支持多重继承和类型独立分派，通过它可以实现喜欢的其它编程语言里的一些结构。

## 7 类型的String()方法和格式化描述符

当定义了一个有很多方法的类型时，通常会使用String()方法来定制类型的字符串形式的输出，换句话说，一种可阅读性和打印性的输出。如果类型定义了String()方法，它会被用在fmt.Printf()中生成默认的输出，等同于使用格式化描述符%v产生的输出。还有fmt.Print()和fmt.Println()也会自动使用String()方法。

程序示例【method_string.go】

```go
// @file:        method_string.go
// @version:     1.0
// @author:      haulf
// @date:        2017.12.10
// @go version:  1.9
// @brief:       Struct test.

package main

import (
    "fmt"
    "strconv"
)

type TwoInts struct {
    a int
    b int
}

func main() {
    two1 := new(TwoInts)
    two1.a = 12
    two1.b = 10
    fmt.Printf("two1 is: %v\n", two1)
    fmt.Println("two1 is:", two1)
    fmt.Printf("two1 is: %T\n", two1)
    fmt.Printf("two1 is: %#v\n", two1)
}

func (tn *TwoInts) String() string {
    return "(" + strconv.Itoa(tn.a) + "/" + strconv.Itoa(tn.b) + ")"
}
```

程序运行结果

```shell
two1 is: (12/10)
two1 is: (12/10)
two1 is: *main.TwoInts
two1 is: &main.TwoInts{a:12, b:10}
```

当广泛使用一个自定义类型时，最好为它定义String()方法。从上面的例子也可以看到，格式化描述符%T会给出类型的完全规格，%#v会给出实例的完整输出，包括它的字段（在程序自动生成Go代码时也很有用）。

不要在String()方法里面调用涉及String()方法的方法，它会导致意料之外的错误，比如下面的例子，它导致了一个无限迭代（递归）调用（TT.String() 调用 fmt.Sprintf，而 fmt.Sprintf 又会反过来调用 TT.String()...），很快就会导致内存溢出：

```go
   type TT float64

   func (t TT) String() string {
       return fmt.Sprintf("%v", t)
   }

   t. String()
```

## 8 垃圾回收和SetFinalizer

Go开发者不需要写代码来释放程序中不再使用的变量和结构占用的内存，在Go运行时中有一个独立的进程，即垃圾收集器（GC），会处理这些事情。它搜索不再使用的变量然后释放它们的内存。可以通过runtime包访问GC进程。

通过调用`runtime.GC()`函数可以显式的触发GC，但这只在某些罕见的场景下才有用，比如当内存资源不足时调用`runtime.GC()`，它会此函数执行的点上立即释放一大片内存，此时程序可能会有短时的性能下降（因为GC进程在执行）。

如果想知道当前的内存状态，可以使用：

```go
   var m runtime.MemStats
   runtime.ReadMemStats(&m)
   fmt.Printf("%d Kb\n", m.Alloc / 1024)
```

上面的程序会给出已分配内存的总量，单位是Kb。

如果需要在一个对象obj被从内存移除前执行一些特殊操作，比如写到日志文件中，可以通过如下方式调用函数来实现：

```go
   runtime.SetFinalizer(obj, func(obj *typeObj))
```

`func(obj*typeObj)` 需要一个 typeObj 类型的指针参数obj，特殊操作会在它上面执行。func也可以是一个匿名函数。在对象被GC进程选中并从内存中移除以前，SetFinalizer都不会执行，即使程序正常结束或者发生错误。
