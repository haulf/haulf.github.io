---
layout: post
title:  "Go教程10-接口与反射"
date:   2018-04-22 08:08:04
categories: Go语言编程
tags: Go 
excerpt: Go语言里面没有类和继承的概念，但是它里面有非常灵活的接口概念，通过接口可以实现很多面向对象的特性。
---

* content
{:toc}


## 1 接口是什么

Go语言里面没有类和继承的概念，但是它里面有非常灵活的接口概念，通过接口可以实现很多面向对象的特性。接口定义了一组方法，但是这些方法不包含实现代码。接口里也不能包含变量。

通过如下格式定义接口：

```go
type Namer interface {
    Method1(param_list) return_type
    Method2(param_list) return_type
    ...
}
```

上面的Namer是一个接口类型。按照约定，只包含一个方法的接口的名字是由方法名加[e]r后缀组成，例如Printer、Reader、Writer、Logger、Converter等等。还有一些不常用的方式（当后缀er不合适时），比如Recoverable，此时接口名以able结尾，或者以I开头（像.NET或Java中那样）。

Go语言中的接口都很简短，通常会包含0个、最多3个方法。

不像大多数面向对象编程语言，在Go语言中接口可以有值，一个接口类型的变量或一个接口值：var ai Namer，ai是一个多字（multiword）数据结构，它的值是nil。它本质上是一个指针，虽然不完全是一回事。指向接口值的指针是非法的，它们不仅一点用也没有，还会导致代码错误。

类型（比如结构体）实现接口方法集中的方法，每一个方法的实现说明了此方法是如何作用于该类型的：即**实现接口**。同时方法集也构成了该类型的接口。实现了Namer接口类型的变量可以赋值给ai（接收者值），此时方法表中的指针会指向被实现的接口方法。

类型不需要显式声明它实现了某个接口：接口被隐式地实现。多个类型可以实现同一个接口。实现某个接口的类型（除了实现接口方法外）可以有其它的方法。一个类型可以实现多个接口。接口类型可以包含一个实例的引用，该实例的类型实现了此接口（接口是动态类型）。即使接口在类型之后才定义， 二者处于不同的包中，被单独编译：只要类型实现了接口中的方法，它就实现了此接口。所有这些特性使得接口具有很大的灵活性。

#### 12.1.1 示例interfaces.go

```go
// @file:        interface.go
// @version:     1.0
// @author:      haulf
// @date:        2017.09.08
// @go version:  1.9
// @brief:       Interface test program.

package main

import "fmt"

type Shaper interface {
    Area() float32
}

type Square struct {
    side float32
}

func (sq *Square) Area() float32 {
    return sq.side * sq.side
}

func main() {
    // case one
    // sq1 := new(Square)
    // sq1.side = 5
    // areaIntf := sq1
    // fmt.Printf("The square has area: %f\n", areaIntf.Area())

    // case two
    sq1 := new(Square)
    sq1.side = 5
    var areaIntf Shaper
    areaIntf = sq1
    fmt.Printf("The square has area: %f\n", areaIntf.Area())

    // case three
    // sq1 := new(Square)
    // sq1.side = 5
    // areaIntf := Shaper(sq1)
    // fmt.Printf("The square has area: %f\n", areaIntf.Area())
}
```

上面的程序定义了一个接口Shaper(造形师)和一个结构体Square(正方形)。在接口Shaper中有一个方法Area()用来计算具体形状的面积。在程序的main()方法中创建了一个Square的实例。在主程序外边定义并实现了方法Area()，其接收者类型是Square，用来计算正方形的面积。从这里可知，结构体Square实现了接口Shaper，因此可以将一个Square类型的变量赋值给一个接口类型的变量：areaIntf = sq1。总体上看，比较喜欢第二种写法，这种写法最清晰。

现在接口变量areaIntf包含一个指向Square变量的引用，通过它可以调用Square上的方法Area()。当然也可以直接在Square的实例上调用此方法，但是在接口实例上调用此方法更令人兴奋，它使此方法更具有一般性。接口变量里包含了接收者实例的值和指向对应方法表的指针。这是多态的Go版本。多态是面向对象编程中一个广为人知的概念：根据当前的类型选择正确的方法，或者说：同一种类型在不同的实例上似乎表现出不同的行为。

**注意：**如果Square没有实现Area()方法，编译器将会给出清晰的错误信息：

```shell
cannot use sq1 (type *Square) as type Shaper in assignment:
   *Square does not implement Shaper (missing Area method)
```

如果Shaper有另外一个方法Perimeter()，但是Square没有实现它，即使没有人在Square实例上调用这个方法，编译器也会给出上面同样的错误。

#### 12.1.2 示例扩展interfaces_poly.go

扩展一下上面的例子，类型Rectangle也实现了Shaper接口。接着创建一个Shaper类型的数组，迭代它的每一个元素并在上面调用Area()方法，以此来展示多态行为：

```go
// @file:        interface_poly.go
// @version:     1.0
// @author:      haulf
// @date:        2017.09.08
// @go version:  1.9
// @brief:       Interface test program.

package main

import "fmt"

type Shaper interface {
    Area() float32
}

type Square struct {
    side float32
}

func (sq *Square) Area() float32 {
    return sq.side * sq.side
}

type Rectangle struct {
    length, width float32
}

func (r Rectangle) Area() float32 {
    return r.length * r.width
}

func main() {
    r := Rectangle{5, 3}
    q := &Square{5}
    shapers := []Shaper{Shaper(r), Shaper(q)}
    // shapers := []Shaper{r, q}

    fmt.Println("Looping through shapes for area ...")

    for n, _ := range shapers {
        fmt.Println("Shape details:", shapers[n])
        fmt.Println("Area of this shape is:", shapers[n].Area())
    }
}
```

在调用shapes[n].Area())这个时，只知道shapes[n] 是一个Shaper对象，最后它摇身一变成为了一个Square或Rectangle对象，并且表现出了相对应的行为。

#### 12.1.3 示例扩展valuable.go

```go
// @file:        valuable.go
// @version:     1.0
// @author:      haulf
// @date:        2017.09.08
// @go version:  1.9
// @brief:       Interface test program.

package main

import "fmt"

type stockPosition struct {
    ticker     string
    sharePrice float32
    count      float32
}

// method to determine the value of a stock position
func (s stockPosition) getValue() float32 {
    return s.sharePrice * s.count

}

type car struct {
    make  string
    model string
    price float32
}

// method to determine the value of a car
func (c car) getValue() float32 {
    return c.price
}

// contract that defines different things that have value
type valuable interface {
    getValue() float32
}

func showValue(asset valuable) {
    fmt.Printf("Value of the asset is %f\n", asset.getValue())
}

func main() {
    var o valuable = stockPosition{"GOOG", 577.20, 4}
    showValue(o)
    o = car{"BMW", "M3", 66500}
    showValue(o)
}
```

上面的代码中定义了两个类型stockPosition和car，它们都有一个getValue() 方法，因此可以定义一个具有此方法的接口valuable。通过定义一个使用valuable 类型作为参数的函数 showValue()，则所有实现了valuable 接口的类型都可以使用这个函数。

#### 12.1.4 标准库示例

一个标准库的例子： io包里有一个接口类型Reader:

```go
   type Reader interface {
       Read(p []byte) (n int, err error)
    }
```

定义变量r：var r io.Reader

那么就可以写如下的代码：

```go
var r io.Reader
r = os.Stdin
r = bufio.NewReader(r)
r = new(bytes.Buffer)
f,_ := os.Open("test.txt")
r = bufio.NewReader(f)
```

上面r右边的类型都实现了Read()方法，并且有相同的方法签名，r的静态类型是io.Reader。

## 2 接口嵌套接口

一个接口可以包含一个或多个其它的接口，这相当于直接将这些内嵌接口的方法列举在外层接口中一样。

比如接口 File 包含了 ReadWrite 和 Lock 的所有方法，它还额外有一个 Close() 方法。

```go
type ReadWrite interface {
    Read(b Buffer) bool
    Write(b Buffer) bool
 }

type Lock interface {
    Lock()
    Unlock()
 }

type File interface {
    ReadWrite
    Lock
    Close()
 }
```

## 3 类型断言：如何检测和转换接口变量的类型

一个接口类型的变量varI中可以包含任何类型的值，必须有一种方式来检测它的动态类型，即运行时在变量中存储的值的实际类型。在执行过程中动态类型可能会有所不同，但是它总是可以分配给接口变量本身的类型。通常可以使用类型断言来测试在某个时刻varI是否包含类型T的值：

```go
v:= varI.(T)       // unchecked typeassertion
```

varI必须是一个接口变量，否则编译器会报错：`invalid type assertion: varI.(T)(non-interface type (type of varI) on left) `。

类型断言可能是无效的，虽然编译器会尽力检查转换是否有效，但是它不可能预见所有的可能性。如果转换在程序运行时失败，则会导致错误发生。更安全的方式是使用以下形式来进行类型断言：

```go
if v, ok := varI.(T); ok {  //checked type assertion
    Process(v)
    return
}

// varI is not of type T
```

如果转换合法，v是varI转换到类型T的值，ok会是true；否则v是类型T的零值，ok是false，也没有运行时错误发生。​

在大多数情况下，可能只是想在if中测试一下ok的值，此时使用以下的方法会是最方便的：

```go
if _, ok := varI.(T); ok {
    // ...
}
```

## 4 类型判断：type-switch

接口变量的类型也可以使用一种特殊形式的swtich 来检测：`type-swtich`：

```go
   switch t := areaIntf.(type) {
   case *Square:
       fmt.Printf("Type Square %T with value %v\n", t, t)
   case *Circle:
       fmt.Printf("Type Circle %T with value %v\n", t, t)
   case nil:
       fmt.Printf("nil value: nothing to check?\n")
   default:
       fmt.Printf("Unexpected type %T\n", t)
    }
```

areaIntf是一个接口变量。变量t得到了areaIntf的值和类型，所有case语句中列举的类型（nil除外）都必须实现对应的接口，如果被检测类型没有在 case语句列举的类型中，就会执行default语句。

可以用type-switch进行运行时类型分析，但是在type-switch不允许有fallthrough 。

如果仅仅是测试变量的类型，不用它的值，那么就可以不需要赋值语句，比如：

```go
   switch areaIntf.(type) {
   case *Square:
       // TODO
   case *Circle:
       // TODO
   ...
   default:
       // TODO
    }
```

下面的代码片段展示了一个类型分类函数，它有一个可变长度参数，可以是任意类型的数组，它会根据数组元素的实际类型执行不同的动作：

```go
func classifier(items ...interface{}) {
    for i, x := range items {
        switch x.(type) {
        case bool:
             fmt.Printf("Param #%d is abool\n", i)
        case float64:
           fmt.Printf("Param #%dis a float64\n", i)
        case int, int64:
             fmt.Printf("Param #%d is aint\n", i)
        case nil:
             fmt.Printf("Param #%d is anil\n", i)
        case string:
            fmt.Printf("Param #%d is astring\n", i)
        default:
             fmt.Printf("Param #%d isunknown\n", i)
        }
    }
}
```

可以这样调用此方法：classifier(13, -14.3, "BELGIUM", complex(1, 2), nil,false) 。

在处理来自于外部的、类型未知的数据时，比如解析诸如JSON或XML编码的数据，类型测试和转换会非常有用。

## 5 测试一个值是否实现了某个接口

这是类型断言中的一个特例：假定v是一个值，然后想测试它是否实现了Stringer接口，可以这样做：

```go
type Stringer interface {
    String() string
}
 
if sv, ok := v.(Stringer); ok {
    fmt.Printf("v implements String(): %s\n", sv.String()) //note: sv, not v
}
```

Print函数就是这样检测类型是否可以打印自身的。

**原则：**

- 接口是一种契约，实现类型必须满足它。它描述了类型的行为，规定类型可以做什么。接口彻底将类型可以做什么，以及如何做分离开来，使得相同接口的变量在不同的时刻表现出不同的行为，这就是多态的本质。
- 编写参数是接口变量的函数，这使得它们更具有一般性。
- 使用接口使代码更具有普适性。

标准库里到处都使用了这个原则，如果对接口概念没有良好的把握，是不可能理解它是如何构建的。

## 6 使用方法集与接口

作用于变量上的方法实际上是不区分变量到底是指针还是值的。当碰到接口类型值时，这会变得有点复杂，原因是接口变量中存储的具体值是不可寻址的。幸运的是，如果使用不当编译器会给出错误。

示例method_set.go：

```go
// @file:        method_set.go
// @version:     1.0
// @author:      haulf
// @date:        2017.09.08
// @go version:  1.9
// @brief:       Method set test.

package main

import (
    "fmt"
)

type List []int

func (l List) Len() int {
    return len(l)
}

func (l *List) Append(val int) {
    *l = append(*l, val)
}

type Appender interface {
    Append(int)
}

func CountInto(a Appender, start, end int) {
    for i := start; i <= end; i++ {
        a.Append(i)
    }
}

type Lener interface {
    Len() int
}

func LongEnough(l Lener) bool {
    return l.Len()*10 > 42
}

func main() {
    // A bare value
    var lst List

  CountInto(&lst, 5, 15) // 第一个参数，自定义Append()的接收者为一个指针对象
    if LongEnough(lst) {
        fmt.Printf("- lst is long enough\n")
    }

    fmt.Println("The lst value is:")
    for _, i := range lst {
        fmt.Print(" ", i)
    }
    fmt.Println("")

    // A pointer value
    plst := new(List)
    CountInto(plst, 2, 10) //VALID:Identical receiver type

    if LongEnough(plst) {
        // VALID: a *List can be dereferenced for the receiver
        fmt.Printf("- plst is long enough\n")
    }

    fmt.Println("The plst value is:")
    for _, i := range *plst {
        fmt.Print(" ", i)
    }
    fmt.Println("")
}
```

**总结：**

1、在接口上调用方法时，必须有和方法定义时相同的接收者类型，或者是可以从具体类型P直接可以辨识的：

- 指针方法可以通过指针调用。
- 值方法可以通过值调用。


- 接收者是值的方法可以通过指针调用，因为指针会首先被解引用。


- 接收者是指针的方法不可以通过值调用，因为存储在接口中的值没有地址。


- 将一个值赋值给一个接口赋值时，编译器会确保所有可能的接口方法都可以在此值上被调用，因此不正确的赋值在编译期就会失败。

2、Go语言规范定义了接口方法集的调用规则：

- 类型`*T `的可调用方法集包含接受者为`*T`或T的所有方法集。
- 类型T的可调用方法集包含接受者为T的所有方法，而不包含接受者为`*T`的方法。



## 7 第一个例子：使用Sorter接口排序

一个很好的例子是来自标准库的sort包，要对一组数字或字符串排序，只需要实现三个方法：反映元素个数的Len()方法、比较第i和j个元素的Less(i,j) 方法以及交换第i和j个元素的Swap(i, j)方法。排序函数的算法只会使用到这三个方法，可以使用任何排序算法来实现。

【sort/sort.go】

```go
// @file:        sort.go
// @version:     1.0
// @author:      haulf
// @date:        2017.09.08
// @go version:  1.9
// @brief:       Sort test.

package sort

type Sorter interface {
    Len() int           // length
    Less(i, j int) bool // compare
    Swap(i, j int)      // swap
}

// Bubble Sort
func Sort(data Sorter) {
    for pass := 1; pass < data.Len(); pass++ {
        for i := 0; i < data.Len()-pass; i++ {
            if data.Less(i+1, i) {
                data.Swap(i, i+1)
            }
        }
    }
}

func IsSorted(data Sorter) bool {
    n := data.Len()

    for i := n - 1; i > 0; i-- {
        if data.Less(i, i-1) {
            return false
        }
    }

    return true
}

// Convenience types for common cases
type IntArray []int

func (p IntArray) Len() int           { return len(p) }
func (p IntArray) Less(i, j int) bool { return p[i] < p[j] }
func (p IntArray) Swap(i, j int)      { p[i], p[j] = p[j], p[i] }

type StringArray []string

func (p StringArray) Len() int           { return len(p) }
func (p StringArray) Less(i, j int) bool { return p[i] < p[j] }
func (p StringArray) Swap(i, j int)      { p[i], p[j] = p[j], p[i] }

// Convenience wrappers for common cases
func SortInts(a []int)                 { Sort(IntArray(a)) }
func SortStrings(a []string)           { Sort(StringArray(a)) }
func IntsAreSorted(a []int) bool       { return IsSorted(IntArray(a)) }
func StringsAreSorted(a []string) bool { return IsSorted(StringArray(a)) }
```



【sort_main.go】

```go
// @file:        sort_main.go
// @version:     1.0
// @author:      haulf
// @date:        2017.09.08
// @go version:  1.9
// @brief:       Sort test.

package main

import (
    "./sort"
    "fmt"
)

func ints() {
    data := []int{74, 59, 238, -784, 9845, 959, 905, 0, 0, 42, 7586, -5467984, 7586}
    a := sort.IntArray(data) //conversion to type IntArray
    sort.Sort(a)

    if !sort.IsSorted(a) {
        panic("fails")
    }

    fmt.Printf("The sorted int array is: %v\n", a)
}

func strings() {
    data := []string{"monday", "friday", "tuesday", "wednesday", "sunday", "thursday", "", "saturday"}
    a := sort.StringArray(data)
    sort.Sort(a)

    if !sort.IsSorted(a) {
        panic("fail")
    }

    fmt.Printf("The sorted string array is: %v\n", a)
}

type day struct {
    num       int
    shortName string
    longName  string
}

type dayArray struct {
    data []*day
}

func (p *dayArray) Len() int           { return len(p.data) }
func (p *dayArray) Less(i, j int) bool { return p.data[i].num < p.data[j].num }
func (p *dayArray) Swap(i, j int)      { p.data[i], p.data[j] = p.data[j], p.data[i] }

func days() {
    Sunday := day{0, "SUN", "Sunday"}
    Monday := day{1, "MON", "Monday"}
    Tuesday := day{2, "TUE", "Tuesday"}
    Wednesday := day{3, "WED", "Wednesday"}
    Thursday := day{4, "THU", "Thursday"}
    Friday := day{5, "FRI", "Friday"}
    Saturday := day{6, "SAT", "Saturday"}

    data := []*day{&Tuesday, &Thursday, &Wednesday, &Sunday, &Monday, &Friday, &Saturday}
    a := dayArray{data}
    sort.Sort(&a)

    if !sort.IsSorted(&a) {
        panic("fail")
    }

    fmt.Println("The week time sorted: ")
    for _, d := range data {
        fmt.Printf("    %s ", d.longName)
    }
    fmt.Printf("\n")
}

func main() {
    ints()
    strings()
    days()
}
```

**程序运行结果：**

```shell
The sorted int array is: [-5467984 -784 0 0 42 59 74 238 905 959 7586 7586 9845]
The sorted string array is: [ friday monday saturday sunday thursday tuesday wednesday]
The week time sorted:
    Sunday     Monday     Tuesday     Wednesday     Thursday     Friday     Saturday

```

**说明：**

1、panic("fail")用于停止处于在非正常情况下的程序，当然也可以先打印一条信息，然后调用os.Exit(1)来停止程序。

2、上面的例子演示了接口的意义和使用方式。对于基本类型的排序，标准库已经提供了相关的排序函数，不需要再重复造轮子了。对于一般性的排序，sort 包定义了一个接口：

```go
type Interface interface {
    Len() int
    Less(i, j int) bool
    Swap(i, j int)
}
```

这个接口总结了需要用于排序的抽象方法，函数Sort(data Interface)用来对此类对象进行排序，可以用它们来实现对其它数据（非基本类型）进行排序。在上面的例子中，也是这么做的，不仅可以对int和string序列进行排序，也可以对用户自定义类型dayArray进行排序。

## 8 第二个例子：读和写

读和写是软件中很普遍的行为，提起它们会立即想到读写文件、缓存（比如字节或字符串切片）、标准输入输出、标准错误以及网络连接、管道等等，或者读写的自定义类型。为了让代码尽可能通用，Go采取了一致的方式来读写数据。

io包提供了用于读和写的接口io.Reader和io.Writer：

```go
   type Reader interface {
       Read(p []byte) (n int, err error)
    }    

   type Writer interface {
       Write(p []byte) (n int, err error)
   }
```

只要类型实现了读写接口，提供Read()和Write方法，就可以从它读取数据，或向它写入数据。一个对象要是可读的，它必须实现`io.Reader`接口，这个接口只有一个签名是`Read(p []byte) (nint, err error)`的方法，它从调用它的对象上读取数据，并把读到的数据放入参数中的字节切片中，然后返回读取的字节数和一个error对象。如果没有错误发生返回'nil'。如果已经到达输入的尾端，会返回`io.EOF("EOF")`。如果读取的过程中发生了错误，就会返回具体的错误信息。类似地，一个对象要是可写的，它必须实现io.Writer接口，这个接口也只有一个签名是`Write(p []byte)(n int, err error)`的方法，它将指定字节切片中的数据写入调用它的对象里，然后返回实际写入的字节数和一个error对象（如果没有错误发生就是nil）。

io包里的Readers和Writers都是不带缓冲的，bufio包里提供了对应的带缓冲的操作，在读写UTF-8编码的文本文件时它们尤其有用。

在实际编程中尽可能的使用这些接口，会使程序变得更通用，可以在任何实现了这些接口的类型上使用读写方法。例如一个JPEG图形解码器，通过一个Reader参数，它可以解码来自磁盘、网络连接或以gzip压缩的HTTP流中的JPEG图形数据，或者其它任何实现了Reader接口的对象。

## 9 空接口

### 9.1 概念

空接口或者最小接口不包含任何方法，它对实现不做任何要求：

```go
type Any interface {}
```

任何其它类型都实现了空接口（它不仅仅像Java/C#中Object 引用类型），any或Any是空接口一个很好的别名或缩写。空接口类似Java/C# 中所有类的基类：Object 类，二者的目标也很相近。可以给一个空接口类型的变量`var val interface {}`赋任何类型的值。

示例empty_interface.go：

```go
// @file:        empty_interface.go
// @version:     1.0
// @author:      haulf
// @date:        2017.09.08
// @go version:  1.9
// @brief:       Empty interface test.

package main

import "fmt"

var i = 5
var str = "ABC"

type Person struct {
    name string
    age  int
}

type Any interface{}

func main() {
    var val Any
    val = 5
    fmt.Printf("val has the value: %v\n", val)

    val = str
    fmt.Printf("val has the value: %v\n", val)

    pers1 := new(Person)
    pers1.name = "Rob Pike"
    pers1.age = 55

    val = pers1
    fmt.Printf("val has the value: %v\n", val)

    switch t := val.(type) {
    case int:
        fmt.Printf("Type int %T\n", t)
    case string:
        fmt.Printf("Type string %T\n", t)
    case bool:
        fmt.Printf("Type boolean %T\n", t)
    case *Person:
        fmt.Printf("Type pointer to Person %T\n", t)
    default:
        fmt.Printf("Unexpected type %T", t)
    }
}
```

**输出结果**

```go
val has the value: 5
val has the value: ABC
val has the value: &{Rob Pike 55}
Type pointer to Person *main.Person
```

在上面的例子中，接口变量 val 被依次赋予一个int、string和 Person实例的值，然后使用type-swtich来测试它的实际类型。每个interface {} 变量在内存中占据两个字长：一个用来存储它包含的类型，另一个用来存储它包含的数据或者指向数据的指针。

示例empty_interface_switch.go说明了空接口在type-swtich中联合lambda函数的用法：

```go
// @file:        empty_interface_switch.go
// @version:     1.0
// @author:      haulf
// @date:        2017.10.17
// @go version:  1.9
// @brief:       Empty interface switch test.

package main

import "fmt"

type specialString string

var whatIsThis specialString = "hello"

func TypeSwitch() {
    testFunc := func(any interface{}) {
        switch v := any.(type) {
        case bool:
            fmt.Printf("any %v is abool type", v)
        case int:
            fmt.Printf("any %v is anint type", v)
        case float32:
            fmt.Printf("any %v is afloat32 type", v)
        case string:
            fmt.Printf("any %v is astring type", v)
        case specialString:
            fmt.Printf("any %v is aspecial String!", v)
        default:
            fmt.Println("unknowntype!")
        }
    }
    testFunc(whatIsThis)
}

func main() {
    TypeSwitch()
}
```

**输出结果：**

```
any hello is aspecial String!
```

#### 12.9.2 构建通用类型或包含不同类型变量的数组

在前面的例子中看到了能被搜索和排序的int数组、float数组以及string数组，那么对于其它类型的数组呢，是不是必须得自己编程实现它们？现在知道该怎么做了，就是通过使用空接口。让给空接口定一个别名类型` Element：type Element interface{}`，然后定义一个容器类型的结构体Vector，它包含一个Element类型元素的切片：

```go
   type Vector struct {
       a []Element
   }
```

Vector里能放任何类型的变量，因为任何类型都实现了空接口，实际上Vector里放的每个元素可以是不同类型的变量。为它定义一个At()方法用于返回第i个元素：

```go
   func (p *Vector) At(i int) Element {
       return p.a[i]
   }
```

再定一个Set()方法用于设置第i个元素的值：

```go
   func (p *Vector) Set(i int, e Element) {
       p.a[i] = e
   }
```

Vector中存储的所有元素都是Element类型，要得到它们的原始类型（unboxing：拆箱）需要用到类型断言。`TODO：The compiler rejects assertions guaranteed to fail`，类型断言总是在运行时才执行，因此它会产生运行时错误。

#### 12.9.3 复制数据切片至空接口切片

假设有一个 myType 类型的数据切片，想将切片中的数据复制到一个空接口切片中，类似：

```go
   var dataSlice []myType = FuncReturnSlice()
   var interfaceSlice []interface{} = dataSlice
```

可惜不能这么做，编译时会出错：`cannot use dataSlice (type []myType) as type []interface { } inassignment`。原因是它们在内存中的布局是不一样的。必须使用 for-range 语句来一个一个显式地复制：

```go
   var dataSlice []myType = FuncReturnSlice()
   var interfaceSlice []interface{} = make([]interface{}, len(dataSlice))

   for ix, d := range dataSlice {
       interfaceSlice[i] = d
   }
```

#### 12.9.4 通用类型的节点数据结构

可以使用空接口作为数据字段的类型，这样就能写出通用的代码。下面是实现一个二叉树的部分代码：通用定义、用于创建空节点的 NewNode 方法，及设置数据的 SetData 方法。

示例node_structures.go：

```go
// @file:        node_structures.go
// @version:     1.0
// @author:      haulf
// @date:        2017.10.17
// @go version:  1.9
// @brief:       Interface test.

package main

import "fmt"

type Node struct {
    le   *Node
    data interface{}
    ri   *Node
}

func NewNode(left, right *Node) *Node {
    return &Node{left, nil, right}
}

func (n *Node) SetData(data interface{}) {
    n.data = data
}

func main() {
    root := NewNode(nil, nil)
    root.SetData("root node")

    // make child (leaf) nodes:
    a := NewNode(nil, nil)
    a.SetData("left node")
    b := NewNode(nil, nil)
    b.SetData("right node")

    root.le = a
    root.ri = b

    // Output: &{0x125275f0 root node 0x125275e0}
    fmt.Printf("%v\n", root)
}
```

#### 12.9.5 接口到接口转换

只要底层类型实现了必要的方法，那么一个接口的值可以赋值给另一个接口变量，这个转换是在运行时进行检查的，转换失败时会产生一个运行时错误。

假定：

```go
   var ai AbsInterface // 在此接口中声明了方法Abs()

   type SqrInterface interface {
       Sqr() float
   }

   var si SqrInterface

   pp := new(Point) // 假定 *Point 实现了方法 Abs(), Sqr()

   var empty interface{}
```

那么下面的语句和类型断言是合法的：

```go
   empty = pp                //everything satisfies empty
   ai = empty.(AbsInterface) // underlying value pp implements Abs()
   // (runtime failure otherwise)
   // // *Point has Sqr() even though AbsInterface doesn’t

   si = ai.(SqrInterface) 
   empty = si             // *Point implements empty set

   //Note: statically checkable so type assertion not necessary.
```
 

下面是函数调用的一个例子：

```go
   type myPrintInterface interface {
       print()
    }    

   func f3(x myInterface) {
       x.(myPrintInterface).print() // type assertion to myPrintInterface
   }
```
x转换为myPrintInterface类型是完全动态的：只要x的底层类型（动态类型）定义了print方法这个调用就可以正常运行。

## 10 反射包

### 10.1 方法和类型的反射

反射是用程序检查其所拥有的结构（尤其是类型）的一种能力，这是元编程的一种形式。反射可以在运行时检查类型和变量，例如它的大小、方法和动态的调用这些方法。变量的最基本信息就是类型和值。反射包的Type用来表示一个Go类型，反射包的Value用来获取Go值。

可以利用函数reflect.TypeOf()和reflect.ValueOf()返回被检查对象的类型和值。例如，x被定义为：var x float64 = 3.4，那么reflect.TypeOf(x)返回变量x的类型是float64，reflect.ValueOf(x)返回变量x的值3.4。

实际上，反射是通过检查一个接口的值来进行的。变量先被转换成一个空接口。这从Go源代码中两个函数签名能够很明显地看出来：

【Go/scr/reflect/type.go】

```go
// TypeOf returns the reflection Type that represents the dynamic type of i.
// If i is a nil interface value, TypeOf returns nil.
func TypeOf(i interface{}) Type {
    eface := *(*emptyInterface)(unsafe.Pointer(&i))
    return toType(eface.typ)
}
```

【Go/scr/reflect/value.go】

```go
// ValueOf returns a new Value initialized to the concrete value
// stored in the interface i. ValueOf(nil) returns the zero Value.
func ValueOf(i interface{}) Value {
    if i == nil {
        return Value{}
    }

    // TODO: Maybe allow contents of a Value to live on the stack.
    // For now we make the contents always escape to the heap. It
    // makes life easier in a few places (see chanrecv/mapassign
    // comment below).
    escapes(i)

    return unpackEface(i)
}
```

接口的值包含type和value。

反射可以从接口值反射到对象，也可以从对象反射回接口值。

reflect.Type结构体和reflect.Value结构体都有许多方法用于检查和操作它们。一个重要的例子是Value有一个Type()方法返回reflect.Value的Type。另一个是Type和Value都有Kind()方法返回一个常量来表示具体的类型，如Uint、Float64、Slice等等。同样地，Value有叫做Int()和Float()的方法可以获取存储在内部的值。

【Go/scr/reflect/type.go】

```go
// A Kind represents the specific kind of type that a Type represents.
// The zero Kind is not a valid kind.
type Kind uint

const (
    Invalid Kind = iota
    Bool
    Int
    Int8
    Int16
    Int32
    Int64
    Uint
    Uint8
    Uint16
    Uint32
    Uint64
    Uintptr
    Float32
    Float64
    Complex64
    Complex128
    Array
    Chan
    Func
    Interface
    Map
    Ptr
    Slice
    String
    Struct
    UnsafePointer
)
```

对于变量x，如果v:=reflect.ValueOf(x)，那么v.Kind()返回的是变量x的具体类型float64，所以下面的表达式是true：

`v.Kind() == reflect.Float64`

使用Kind()方法总是得到底层的类型名称：

```go
        type MyInt int
        var m MyInt = 5
        v:= reflect.ValueOf(m)
```

方法v.Kind() 返回reflect.Int。

变量v的Interface()方法可以得到还原（接口）值，所以可以这样打印v的值：

```go
fmt.Println(v.Interface())
```

尝试运行下面的代码：

示例reflect1.go：

```go
// @file:        reflect_1.go
// @version:     1.0
// @author:      haulf
// @date:        2017.10.19
// @go version:  1.9
// @brief:       Reflect test.

package main

import (
    "fmt"
    "reflect"
)

func main() {
    var x float64 = 3.4
    fmt.Println("type:", reflect.TypeOf(x))

    v := reflect.ValueOf(x)
    fmt.Println("value:", v)
    fmt.Println("type:", v.Type())
    fmt.Println("kind:", v.Kind())
    fmt.Println("value:", v.Float())
    fmt.Println(v.Interface())
    fmt.Printf("value is %5.2e\n", v.Interface())
    y := v.Interface().(float64)
    fmt.Println(y)
}
```

运行结果：

```shell
type: float64
value: 3.4
type: float64
kind: float64
value: 3.4
3.4
value is 3.40e+00
3.4

```

x是一个float64类型的值，`reflect.ValueOf(x).Float()`返回这个float64类型的实际值。这种使用方法同样的适用于Int()、Bool()、Complex()、String()等。

### 10.2 通过反射修改值

假设想把float64类型的变量x的值改为3.1415。Value 有一些方法可以完成这个任务，但是必须小心使用：`v.SetFloat(3.1415)`。这将产生一个错误：`reflect.Value.SetFloat using unaddressable value`。

为什么会这样呢？原因是v不是可设置的。是否可设置是Value的一个属性，并且不是所有的反射值都有这个属性：可以使用CanSet()方法测试是否可设置。当v := reflect.ValueOf(x)函数通过传递一个x拷贝创建了v，那么v的改变并不能更改原始的x。要想v的更改能作用到x，就必须传递x的地址：

```go
v = reflect.ValueOf(&x)
```

通过 `Type()` 看到 v 现在的类型是`*float64`并且仍然是不可设置的。要想让其可设置需要使用`Elem()`函数，这间接的使用指针：

```go
v = v.Elem()
```

现在`v.CanSet()`返回 true 并且`v.SetFloat(3.1415)`设置成功了！

示例reflect_2.go：

```go
// @file:        reflect2.go
// @version:     1.0
// @author:      haulf
// @date:        2017.10.21
// @go version:  1.9
// @brief:       Reflect test.

package main

import (
    "fmt"

    "reflect"
)

func main() {
    var x float64 = 3.4
    v := reflect.ValueOf(x)

    // setting a value:
    // Error: reflect.Value.SetFloat using unaddressable value
    // v.SetFloat(3.1415)

    fmt.Println("settability of v:", v.CanSet())

    v = reflect.ValueOf(&x) //Note: take the address of x.
    fmt.Println("type of v:", v.Type())
    fmt.Println("settability of v:", v.CanSet())

    v = v.Elem()
    fmt.Println("The Elem of v is: ", v)
    fmt.Println("settability of v:", v.CanSet())
    v.SetFloat(3.1415) // this works!
    fmt.Println(v.Interface())
    fmt.Println(v)
}
```

反射中有些内容是需要用地址去改变它的状态的。

### 10.3 反射结构

有些时候需要反射一个结构类型。NumField()方法返回结构内的字段数量；通过一个for循环用索引取得每个字段的值`Field(i)`。同样能够调用签名在结构上的方法，例如，使用索引n来调用：`Method(n).Call(nil)`。

示例reflect_struct.go：

```go
// @file:        reflect_struct.go
// @version:     1.0
// @author:      haulf
// @date:        2017.10.21
// @go version:  1.9
// @brief:       Reflect test.

package main

import (
    "fmt"
    "reflect"
)

type NotknownType struct {
    s1, s2, s3 string
}

func (n NotknownType) String() string {
    return n.s1 + " - " + n.s2 + " - " + n.s3
}

// variable to investigate:
var secret interface{} = NotknownType{"Ada", "Go", "Oberon"}

func main() {
    value := reflect.ValueOf(secret) // <main.NotknownType Value>
    typ := reflect.TypeOf(secret)    // main.NotknownType

    typeValue := value.Type() //main.NotknownType
    fmt.Println(typeValue)

    fmt.Println(typ)
    knd := value.Kind()
    fmt.Println(knd)

    // iterate through the fields of the struct:
    for i := 0; i < value.NumField(); i++ {
        fmt.Printf("Field %d: %v\n", i, value.Field(i))
    }

    // call the first method, which is String():
    results := value.Method(0).Call(nil)
    fmt.Println(results)
}
```

输出结果：

```shell
main.NotknownType
main.NotknownType
struct
Field 0: Ada
Field 1: Go
Field 2: Oberon
[Ada - Go - Oberon]
```

但是如果尝试更改一个值，会得到一个错误：

```go
panic: reflect.Value.SetString using valueobtained using unexported field
```

这是因为结构中只有被导出字段（首字母大写）才是可设置的；来看下面的例子：

示例reflect_struct_2.go：

```go
// @file:        reflect_struct_2.go
// @version:     1.0
// @author:      haulf
// @date:        2017.10.21
// @go version:  1.9
// @brief:       Reflect test.

package main

import (
    "fmt"
    "reflect"
)

type T struct {
    A int
    B string
}

func main() {
    t := T{23, "skidoo"}
    s := reflect.ValueOf(&t).Elem()

    typeOfT := s.Type()

    for i := 0; i < s.NumField(); i++ {
        f := s.Field(i)
        fmt.Printf("%d: %s %s = %v\n", i,
            typeOfT.Field(i).Name, f.Type(), f.Interface())
    }

    s.Field(0).SetInt(77)
    s.Field(1).SetString("Sunset Strip")
    fmt.Println("t is now", t)
}
```

输出结果：

```shell
0: A int = 23
1: B string = skidoo
t is now {77 Sunset Strip}
```

## 11 Printf和反射

在Go语言的标准库中，反射的功能被大量地使用。举个例子，fmt包中的Printf（以及其它格式化输出函数）都会使用反射来分析它的...参数。

Printf的函数声明为：

```go
func Printf(format string, args ... interface{}) (n int, err error)
```

Printf中的...参数为空接口类型。Printf使用反射包来解析这个参数列表。所以，Printf能够知道它每个参数的类型。因此格式化字符串中只有%d而没有 %u 和 %ld，因为它知道这个参数是unsigned 还是 long。这也是为什么 Print 和 Println 在没有格式字符串的情况下还能如此漂亮地输出。

为了让大家更加具体地了解 Printf 中的反射，实现了一个简单的通用输出函数。其中使用了type-switch 来推导参数类型，并根据类型来输出每个参数的值。

示例print.go：

```go
    packagemain

   import (
       "os"
       "strconv"
    )

   type Stringer interface {
       String() string
    }

   type Celsius float64

   func (c Celsius) String() string {
       return strconv.FormatFloat(float64(c),'f', 1, 64) + " °C"

    }   

   type Day int  

   var dayName = []string{"Monday", "Tuesday","Wednesday", "Thursday", "Friday","Saturday", "Sunday"}   

   func (day Day) String() string {

       return dayName[day]

    }

   func print(args ...interface{}) {
       for i, arg := range args {
           if i > 0 {os.Stdout.WriteString(" ")}
           switch a := arg.(type) { // type switch
                case Stringer:    os.Stdout.WriteString(a.String())
                case int:        os.Stdout.WriteString(strconv.Itoa(a))
                case string:    os.Stdout.WriteString(a)
                // more types
                default:        os.Stdout.WriteString("???")
           }
       }
   }
    
   func main() {
       print(Day(1), "was", Celsius(18.36))  // Tuesday was 18.4 °C
   }
```



## 12 接口与动态类型

### 12.1 Go的动态类型

在经典的面向对象语言（像 C++，Java 和 C#）中数据和方法被封装为类的概念：类包含它们两者，并且不能剥离。Go没有类：数据（结构体或更一般的类型）和方法是一种松耦合的正交关系。Go中的接口跟Java或C#类似：都是必须提供一个指定方法集的实现，但是Go中的接口更加灵活通用：任何提供了接口方法实现代码的类型都隐式地实现了该接口，而不用显式地声明。

和其它语言相比，Go是唯一结合了接口值、静态类型检查（是否该类型实现了某个接口）、运行时动态转换的语言，并且不需要显式地声明类型是否满足某个接口。该特性允许在不改变已有的代码的情况下定义和使用新接口。

接收一个（或多个）接口类型作为参数的函数，可以被实现了该接口的类型实例调用。实现了某个接口的类型可以被传给任何以此接口为参数的函数。

类似于 Python 和 Ruby 这类动态语言中的动态类型（duck typing）；这意味着对象可以根据提供的方法被处理（例如，作为参数传递给函数），而忽略它们的实际类型：它们能做什么比它们是什么更重要。

这在程序duck_dance.go中得以阐明，函DuckDance接受一个IDuck接口类型变量。仅当DuckDance被实现了IDuck接口的类型调用时程序才能编译通过。

示例duck_dance.go：

```go
// @file:        duck_dance.go
// @version:     1.0
// @author:      haulf
// @date:        2017.10.20
// @go version:  1.9
// @brief:       Interface test.

package main

import "fmt"

type IDuck interface {
    Quack()
    Walk()
}

func DuckDance(duck IDuck) {
    for i := 1; i <= 3; i++ {
        duck.Quack()
        duck.Walk()
    }
}

type Bird struct {
    // ...
}

func (b *Bird) Quack() {
    fmt.Println("I am quacking!")
}

func (b *Bird) Walk() {
    fmt.Println("I am walking!")
}

func main() {
    b := new(Bird)
    DuckDance(b)
}
```

如果Bird 没有实现 Walk()（把它注释掉），会得到一个编译错误：

```shell
   cannot use b (type *Bird) as type IDuck in function argument:
   *Bird does not implement IDuck (missing Walk method)
```

如果对 cat 调用函数 DuckDance()，Go 会提示编译错误，但是 Python 和 Ruby 会以运行时错误结束。

### 12.2 动态方法调用

像Python，Ruby这类语言，动态类型是延迟绑定的（在运行时进行）：方法只是用参数和变量简单地调用，然后在运行时才解析（它们很可能有像 responds_to 这样的方法来检查对象是否可以响应某个方法，但是这也意味着更大的编码量和更多的测试工作）

Go的实现与此相反，通常需要编译器静态检查的支持：当变量被赋值给一个接口类型的变量时，编译器会检查其是否实现了该接口的所有函数。如果方法调用作用于像 interface{} 这样的“泛型”上，可以通过类型断言来检查变量是否实现了相应接口。

例如，用不同的类型表示 XML 输出流中的不同实体。然后为 XML 定义一个如下的“写”接口（甚至可以把它定义为私有接口）：

```go
   type xmlWriter interface {
       WriteXML(w io.Writer) error
   }
```

现在可以实现适用于该流类型的任何变量的 StreamXML 函数，并用类型断言检查传入的变量是否实现了该接口；如果没有，就调用内建的 encodeToXML 来完成相应工作：

```go
   // Exported XML streaming function.

   func StreamXML(v interface{}, w io.Writer) error {
       if xw, ok := v.(xmlWriter); ok {
           // It’s an xmlWriter, use method of asserted type.
           return xw.WriteXML(w)
       }

       // No implementation, so we have to use our own function (with perhapsreflection):
       return encodeToXML(v, w)
    }

   // Internal XML encoding function.
   func encodeToXML(v interface{}, w io.Writer) error {
       // ...
   }
```

Go在这里用了和gob相同的机制：定义了两个接口GobEncoder和GobDecoder。这样就允许类型自己实现从流编解码的具体方式；如果没有实现就使用标准的反射方式。

因此Go提供了动态语言的优点，却没有其它动态语言在运行时可能发生错误的缺点。

对于动态语言非常重要的单元测试来说，这样即可以减少单元测试的部分需求，又可以发挥相当大的作用。

Go的接口提高了代码的分离度，改善了代码的复用性，使得代码开发过程中的设计模式更容易实现。用Go接口还能实现依赖注入模式。

### 12.3 接口的提取

提取接口是非常有用的设计模式，可以减少需要的类型和方法数量，而且不需要像传统的基于类的面向对象语言那样维护整个的类层次结构。

Go接口可以让开发者找出自己写的程序中的类型。假设有一些拥有共同行为的对象，并且开发者想要抽象出这些行为，这时就可以创建一个接口来使用。 来扩展 11.1 节的示例 11.2 interfaces_poly.go，假设需要一个新的接口TopologicalGenus，用来给shape排序（这里简单地实现为返回 int）。需要做的是给想要满足接口的类型实现Rank()方法：

示例multi_interfaces_poly.go：

```go
   //multi_interfaces_poly.go
   package main
   
   import "fmt"    

   type Shaper interface {
       Area() float32
   }    

   type TopologicalGenus interface {
       Rank() int
   } 

   type Square struct {
       side float32
   }

   func (sq *Square) Area() float32 {
       return sq.side * sq.side
   }

   func (sq *Square) Rank() int {
       return 1
   }

   type Rectangle struct {
       length, width float32
   }

   func (r Rectangle) Area() float32 {
       return r.length * r.width
   }

   func (r Rectangle) Rank() int {
       return 2
   }

   func main() {
       r := Rectangle{5, 3} // Area() of Rectangle needs a value
       q := &Square{5}      // Area()of Square needs a pointer
       shapes := []Shaper{r, q}
       fmt.Println("Looping through shapes for area ...")
       for n, _ := range shapes {
           fmt.Println("Shape details: ", shapes[n])
           fmt.Println("Area of thisshape is: ", shapes[n].Area())
       }

       topgen := []TopologicalGenus{r, q}
       fmt.Println("Looping through topgen for rank ...")
       for n, _ := range topgen {
           fmt.Println("Shape details: ", topgen[n])
           fmt.Println("Topological Genus of this shape is: ", 
                                            topgen[n].Rank())
       }
    }
```

所以不用提前设计出所有的接口；整个设计可以持续演进，而不用废弃之前的决定。类型要实现某个接口，它本身不用改变，只需要在这个类型上实现新的方法。

### 12.4 显式地指明类型实现了某个接口

如果希望满足某个接口的类型显式地声明它们实现了这个接口，可以向接口的方法集中添加一个具有描述性名字的方法。例如：

```go
   type Fooer interface {
       Foo()
       ImplementsFooer()
   }
```

类型Bar必须实现ImplementsFooer方法来满足Footer接口，以清楚地记录这个事实。

```go
   type Bar struct{}
   func (b Bar) ImplementsFooer() {} func (b Bar) Foo() {}
```

大部分代码并不使用这样的约束，因为它限制了接口的实用性。但是有些时候，这样的约束在大量相似的接口中被用来解决歧义。

### 12.5 空接口和函数重载

函数重载是不被允许的。在Go语言中函数重载可以用可变参数 ...T作为函数最后一个参数来实现。如果把T换为空接口，那么可以知道任何类型的变量都是满足 T (空接口）类型的，这样就允许传递任何数量任何类型的参数给函数，即重载的实际含义。

函数 fmt.Printf 就是这样做的：

```go
fmt.Printf(format string, a ...interface{}) (nint, errno error)
```

这个函数通过枚举 slice 类型的实参动态确定所有参数的类型。并查看每个类型是否实现了String() 方法，如果是就用于产生输出信息。

### 12.6 接口的继承

当一个类型包含（内嵌）另一个类型（实现了一个或多个接口）的指针时，这个类型就可以使用（另一个类型）所有的接口方法。

例如：

```go
   type Task struct {
       Command string
       *log.Logger
   }
```

这个类型的工厂方法像这样：

```go
   func NewTask(command string, logger *log.Logger) *Task {
       return &Task{command, logger}
   }
```

当 log.Logger 实现Log() 方法后，Task 的实例task就可以调用该方法：

```go
   task.Log()
```

类型可以通过继承多个接口来提供像多重继承 一样的特性：

```go
   type ReaderWriter struct {
       *io.Reader
       *io.Writer
   }
```

上面概述的原理被应用于整个Go包，多态用得越多，代码就相对越少。这被认为是Go编程中的重要的最佳实践。

有用的接口可以在开发的过程中被归纳出来。添加新接口非常容易，因为已有的类型不用变动（仅仅需要实现新接口的方法）。已有的函数可以扩展为使用接口类型的约束性参数：通常只有函数签名需要改变。对比基于类的OO类型的语言在这种情况下则需要适应整个类层次结构的变化。

## 13 总结：Go中的面向对象

总结一下前面看到的：Go没有类，而是松耦合的类型、方法对接口的实现。

OO语言最重要的三个方面分别是：封装，继承和多态，在Go中它们是怎样表现的呢？

- 封装（数据隐藏）：和别的OO语言有4个或更多的访问层次相比，Go把它简化为了2层:

  1）包范围内的：通过标识符首字母小写，对象只在它所在的包内可见

  2）可导出的：通过标识符首字母大写，对象对所在包以外也可见

类型只拥有自己所在包中定义的方法。

- 继承：用组合实现：内嵌一个（或多个）包含想要的行为（字段和方法）的类型；多重继承可以通过内嵌多个类型实现

- 多态：用接口实现：某个类型的实例可以赋给它所实现的任意接口类型的变量。类型和接口是松耦合的，并且多重继承可以通过实现多个接口实现。Go接口不是Java和C#接口的变体，而且：接口间是不相关的，并且是大规模编程和可适应的演进型设计的关键。

## 14 参考资料
- https://github.com/Unknwon/the-way-to-go_ZH_CN/
