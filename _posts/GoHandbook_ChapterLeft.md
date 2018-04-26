

## 12 接口与反射

### 12.1 接口是什么

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

### 12.2 接口嵌套接口

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

### 12.3 类型断言：如何检测和转换接口变量的类型

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

### 12.4 类型判断：type-switch

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

### 12.5 测试一个值是否实现了某个接口

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

### 12.6 使用方法集与接口

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



### 12.7 第一个例子：使用Sorter接口排序

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

### 12.8 第二个例子：读和写

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

### 12.9 空接口

#### 12.9.1 概念

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

### 12.10 反射包

#### 12.10.1 方法和类型的反射

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

#### 12.10.2 通过反射修改值

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

#### 12.10.3 反射结构

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

### 12.11 Printf和反射

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



### 12.12 接口与动态类型

#### 12.12.1 Go的动态类型

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

#### 12.12.2 动态方法调用

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

#### 12.12.3 接口的提取

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

#### 12.12.4 显式地指明类型实现了某个接口

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

#### 12.12.5 空接口和函数重载

函数重载是不被允许的。在Go语言中函数重载可以用可变参数 ...T作为函数最后一个参数来实现。如果把T换为空接口，那么可以知道任何类型的变量都是满足 T (空接口）类型的，这样就允许传递任何数量任何类型的参数给函数，即重载的实际含义。

函数 fmt.Printf 就是这样做的：

```go
fmt.Printf(format string, a ...interface{}) (nint, errno error)
```

这个函数通过枚举 slice 类型的实参动态确定所有参数的类型。并查看每个类型是否实现了String() 方法，如果是就用于产生输出信息。

#### 12.12.6 接口的继承

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

### 12.13 总结：Go中的面向对象

总结一下前面看到的：Go没有类，而是松耦合的类型、方法对接口的实现。

OO语言最重要的三个方面分别是：封装，继承和多态，在Go中它们是怎样表现的呢？

- 封装（数据隐藏）：和别的OO语言有4个或更多的访问层次相比，Go把它简化为了2层:

  1）包范围内的：通过标识符首字母小写，对象只在它所在的包内可见

  2）可导出的：通过标识符首字母大写，对象对所在包以外也可见

类型只拥有自己所在包中定义的方法。

- 继承：用组合实现：内嵌一个（或多个）包含想要的行为（字段和方法）的类型；多重继承可以通过内嵌多个类型实现

- 多态：用接口实现：某个类型的实例可以赋给它所实现的任意接口类型的变量。类型和接口是松耦合的，并且多重继承可以通过实现多个接口实现。Go接口不是Java和C#接口的变体，而且：接口间是不相关的，并且是大规模编程和可适应的演进型设计的关键。

## 13 读写数据

除了fmt和os包，还需要用到bufio包来处理缓冲的输入和输出。

### 13.1 读取用户的输入

这里读取用户的输入，是指从键盘和标准输入os.Stdin读取输入。

#### 13.1.1 使用fmt包提供的Scan()和Sscan()开头函数

* 程序示例

```go
// @file:        read_input_1.go
// @version:     1.0
// @date:        2017.12.11
// @go version:  1.9
// @brief:       Standard input and output test.

package main

import (
    "fmt"
)

var (
    firstName, lastName, s string
    i                      int
    f                      float32
    inputString            = "56.12 / 5212 / Go"
    format                 = "%f / %d / %s"
)

func main() {
    fmt.Println("Please enter your full name: ")
    fmt.Scanln(&firstName, &lastName)
    // fmt.Scanf("%s %s", &firstName, &lastName)
    fmt.Printf("Your name is: %s %s!\n", firstName, lastName)

    fmt.Sscanf(inputString, format, &f, &i, &s)
    fmt.Println("From the inputString, we read: ", f, i, s)
}
```

* 程序说明

(1) Scanln()函数扫描来自标准输入的文本，将空格分隔的输入值依次存放到对应的参数里面，直到碰到换行为止(函数名称以ln结尾)。Scanf()函数与其类似，理解为格式化的输入(函数最后一个字符为f，即format，格式化)，它的第一个参数是格式字符串，用来决定如何读取数据进行格式化操作，后面的参数是对应的被格式化的值。

(2) Sscan()和以Sscan开头的函数则是从字符串读取数据来输入(函数以大写的字符S开头)。除此之外，与Scanf()相同。

#### 13.1.2 使用bufio包提供的缓冲读取（buffered reader）来读取数据

* 程序示例【read_input_2.go】

```go
// @file:        read_input_2.go
// @version:     1.0
// @date:        2017.12.11
// @go version:  1.9
// @brief:       Buffered read test.

package main

import (
    "bufio"
    "fmt"
    "os"
)

var inputReader *bufio.Reader
var input string
var err error

func main() {
    inputReader = bufio.NewReader(os.Stdin)
    fmt.Println("Please enter: ")
    input, err = inputReader.ReadString('\n')
    if err == nil {
        fmt.Printf("The input is: %s\n", input)
    }
}
```

* 程序说明

(1) inputReader是一个指向bufio.Reader的指针变量。`inputReader = bufio.NewReader(os.Stdin)`这行代码，将会创建一个读取器，并将其与标准输入进行绑定。

(2) bufio.NewReader()构造函数的签名为：`func NewReader(rd io.Reader) *Reader`。该函数的实参可以是满足io.Reader接口的任意对象，函数返回一个新的带缓冲的io.Reader对象，它将从指定读取器（例如标准输入os.Stdin）中读取内容。返回的读取器对象提供一个方法ReadString(delim byte)，该方法从输入中读取内容，直到碰到delim指定的字符，然后将读取到的内容连同delim字符一起放到缓冲区。ReadString返回读取到的字符串，如果碰到错误则返回nil。如果它一直读到文件结束，则返回读取到的字符串和io.EOF。如果读取过程中没有碰到delim字符，将返回错误err != nil。

(3) 在上面的例子中，程序会读取键盘输入，直到回车键（\n）被按下。屏幕是标准输出os.Stdout；os.Stderr用于显示错误信息，大多数情况下等同于os.Stdout。一般情况下，会省略变量声明，而使用 :=，例如：

```go
   inputReader := bufio.NewReader(os.Stdin)
   input, err := inputReader.ReadString('\n')
```

### 13.2 文件读写

#### 13.2.1 读文件

在Go语言中，文件使用指向os.File类型的指针来表示的，也叫做文件句柄。在前面章节使用到过标准输入 os.Stdin和标准输出os.Stdout，它们的类型都是*os.File。

* 程序示例

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

* 程序说明

变量`cmdlineFile`是`*os.File`类型的。该类型是一个结构，表示一个打开文件的描述符（文件句柄）。然后使用os包里的Open函数来打开一个文件。该函数的参数是文件名，类型为string。在上面的程序中，以只读模式打开cmdline文件。

如果文件不存在或者程序没有足够的权限打开这个文件，Open函数会返回一个错误：`inputFile, inputError =os.Open("cmdline")`。如果文件打开正常，就使用defer.Close()语句确保在程序退出前关闭该文件，然后使用bufio.NewReader来获得一个读取器变量。

通过使用bufio包提供的读取器（写入器也类似），如上面程序所示，可以很方便的操作相对高层的string对象，而避免了去操作比较底层的字节。

接着，在一个无限循环中使用`ReadString('\n')`或`ReadBytes('\n')`将文件的内容逐行`（行结束符 '\n'）`读取出来。一旦读取到文件末尾，变量readerError的值将变成非空（事实上，常量io.EOF的值是true），就会执行return语句从而退出循环。

注意：Unix和Linux的行结束符是\n，而Windows的行结束符是\r\n。在使用ReadString和ReadBytes方法的时候，不需要关心操作系统的类型，直接使用\n就可以了。另外，也可以使用ReadLine()方法来实现相同的功能。

**其它类似函数：**

1) 将整个文件的内容读到一个字符串里：

如果想这么做，可以使用io/ioutil包里的ioutil.ReadFile()方法，该方法第一个返回值的类型是[]byte，里面存放读取到的内容，第二个返回值是错误，如果没有错误发生，第二个返回值为nil。类似一地，函数WriteFile()可以将[]byte的值写入文件。

* 程序示例

```go
// @file:        read_write_file_1.go
// @version:     1.0
// @author:      haulf
// @date:        2017.12.25
// @go version:  1.9
// @brief:       Read and write test.

package main

import (
    "fmt"
    "io/ioutil"
    "os"
)

func main() {
    inputFile := "products.txt"
    outputFile := "products_copy.txt"

    buf, err := ioutil.ReadFile(inputFile)
    if err != nil {
        fmt.Fprintf(os.Stderr, "File Error: %s\n", err)
        // panic(err.Error())
    }

    fmt.Printf("%s\n", string(buf))
    err = ioutil.WriteFile(outputFile, buf, 0644) // oct, not hex
    if err != nil {
        panic(err.Error())
    }
}
```


2) 带缓冲的读取

在很多情况下，文件的内容是不按行划分的，或者干脆就是一个二进制文件。在这种情况下，ReadString()就无法使用了，可以使用 bufio.Reader 的 Read()，它只接收一个参数：

```go
   buf := make([]byte, 1024)
   ...

   n, err := inputReader.Read(buf)
   if (n == 0) { break}
```

变量 n 的值表示读取到的字节数.

3) 按列读取文件中的数据

如果数据是按列排列并用空格分隔的，可以使用 fmt 包提供的以 FScan 开头的一系列函数来读取它们。请看以下程序，将 3 列的数据分别读入变量 v1、v2 和 v3 内，然后分别把它们添加到切片的尾部。

示例read_file2.go：

```go
   package main

   import (
       "fmt"
       "os"
   )

   func main() {
       file, err := os.Open("products2.txt")
       if err != nil {
           panic(err)

       }

       defer file.Close()    

       var col1, col2, col3 []string

       for {
           var v1, v2, v3 string
           _, err := fmt.Fscanln(file, &v1, &v2, &v3)
           // scans until newline

           if err != nil {
                break
           }

           col1 = append(col1, v1)
           col2 = append(col2, v2)
           col3 = append(col3, v3)
       }    

       fmt.Println(col1)
       fmt.Println(col2)
       fmt.Println(col3)
    }
```

注意： path 包里包含一个子包叫 filepath，这个子包提供了跨平台的函数，用于处理文件名和路径。例如 Base() 函数用于获得路径中的最后一个元素（不包含后面的分隔符）：

```go
   import "path/filepath"
   filename := filepath.Base(path)
```

关于解析 CSV 文件，encoding/csv 包提供了相应的功能。具体请参考 http://golang.org/pkg/encoding/csv/

#### 13.2.2 compress包：读取压缩文件

compress包提供了读取压缩文件的功能，支持的压缩文件格式为：bzip2、flate、gzip、lzw 和 zlib。

下面的程序展示了如何读取一个 gzip 文件。

示例gzipped.go：

```go
   package main

   import (
       "fmt"
       "bufio"
       "os"
       "compress/gzip"
    )
    

   func main() {
       fName := "MyFile.gz"

       var r *bufio.Reader
       fi, err := os.Open(fName)
       if err != nil {
           fmt.Fprintf(os.Stderr, "%v, Can't open %s: error: %s\n", 
                                               os.Args[0],fName,
                err)
           os.Exit(1)
       }

       fz, err := gzip.NewReader(fi)
       if err != nil {
           r = bufio.NewReader(fi)
       } else {
           r = bufio.NewReader(fz)
       }

       for {
           line, err := r.ReadString('\n')
           if err != nil {
                fmt.Println("Done readingfile")
                os.Exit(0)
           }
           fmt.Println(line)
       }
    }
```
 

#### 13.2.3 写文件 

请看以下程序：

示例fileoutput.go：

```go
package main    

   import (
       "os"
       "bufio"
       "fmt"
    )

   func main () {
       // var outputWriter *bufio.Writer
       // var outputFile *os.File
       // var outputError os.Error
       // var outputString string

       outputFile, outputError := 
           os.OpenFile("output.dat", os.O_WRONLY|os.O_CREATE, 0666)

       if outputError != nil {
           fmt.Printf("An error occurred with file opening orcreation\n")
           return  

       }

       defer outputFile.Close()    
       outputWriter := bufio.NewWriter(outputFile)
       outputString := "hello world!\n"

       for i:=0; i<10; i++ {
           outputWriter.WriteString(outputString)
       }

       outputWriter.Flush()
    }
```

除了文件句柄，还需要 bufio 的 Writer。以只写模式打开文件 output.dat，如果文件不存在则自动创建：

   `outputFile, outputError := os.OpenFile(“output.dat”, os.O_WRONLY|os.O_ CREATE, 0666)`

可以看到，OpenFile 函数有三个参数：文件名、一个或多个标志（使用逻辑运算符“|”连接），使用的文件权限。

通常会用到以下标志：
- os.O_RDONLY：只读
- os.WRONLY：只写
- os.O_CREATE：创建：如果指定文件不存在，就创建该文件。
- os.O_TRUNC：截断：如果指定文件已存在，就将该文件的长度截为0。

在读文件的时候，文件的权限是被忽略的，所以在使用 OpenFile 时传入的第三个参数可以用0。而在写文件时，不管是 Unix 还是 Windows，都需要使用 0666。

然后，创建一个写入器（缓冲区）对象：

```go
   outputWriter := bufio.NewWriter(outputFile)
```

接着，使用一个 for 循环，将字符串写入缓冲区，写 10 次：`outputWriter.WriteString(outputString)`

缓冲区的内容紧接着被完全写入文件：`outputWriter.Flush()`

如果写入的东西很简单，可以使用 `fmt.Fprintf(outputFile, “Some test data.\n”)`直接将内容写入文件。fmt 包里的F 开头的 Print 函数可以直接写入任何io.Writer，包括文件。

程序 filewrite.go 展示了不使用 fmt.FPrintf 函数，使用其它函数如何写文件：

示例filewrite.go：

```go
   package main    

    import "os"    

   func main() {
       os.Stdout.WriteString("hello, world\n")
       f, _ := os.OpenFile("test", os.O_CREATE|os.O_WRONLY, 0)
       defer f.Close()
       f.WriteString("hello, world in a file\n")
    }
```

使用 `os.Stdout.WriteString(“hello, world\n”)`，可以输出到屏幕。
以只写模式创建或打开文件“test”，并且忽略了可能发生的错误：`f, _ := os.OpenFile(“test”, os.O_CREATE|os.O_WRONLY, 0)`
不使用缓冲区，直接将内容写入文件：`f.WriteString( )`

### 13.3 文件拷贝

如何拷贝一个文件到另一个文件？最简单的方式就是使用 io 包：

示例filecopy.go：

```go
   // filecopy.go

   package main    

   import (
       "fmt"
       "io"
       "os"
    )

   func main() {
       CopyFile("target.txt", "source.txt")
       fmt.Println("Copy done!")
    }

   func CopyFile(dstName, srcName string) (written int64, err error) {
       src, err := os.Open(srcName)
       if err != nil {
           return
       }

       defer src.Close()    

       dst, err := os.OpenFile(dstName, os.O_WRONLY|os.O_CREATE, 
                              0644)
       if err != nil {
           return
       }

       defer dst.Close()    

       return io.Copy(dst, src)
    }
```

注意 defer 的使用：当打开目标文件时发生了错误，那么 defer 仍然能够确保 src.Close() 执行。如果不这么做，文件会一直保持打开状态并占用资源。

### 13.4 从命令行读取参数

#### 13.4.1 os 包

os包中有一个 string 类型的切片变量 os.Args，用来处理一些基本的命令行参数，它在程序启动后读取命令行输入的参数。来看下面的打招呼程序：

示例os_args.go：

```go
   // os_args.go

   package main 

   import (
       "fmt"
       "os"
       "strings"
    )

   func main() {
       who := "Alice "
       if len(os.Args) > 1 {
           who += strings.Join(os.Args[1:], " ")
       }
       fmt.Println("Good Morning", who)
    }
```

在 IDE 或编辑器中直接运行这个程序输出：Good Morning Alice
在命令行运行 os_args or ./os_args 会得到同样的结果。
但是在命令行加入参数，像这样：os_args John Bill Marc Luke，将得到这样的输出：GoodMorning Alice John Bill Marc Luke
这个命令行参数会放置在切片 os.Args[] 中（以空格分隔），从索引1开始（os.Args[0] 放的是程序本身的名字，在本例中是 os_args）。函数 strings.Join 以空格为间隔连接这些参数。

#### 13.4.2 flag 包

flag包有一个扩展功能用来解析命令行选项。但是通常被用来替换基本常量，例如，在某些情况下希望在命令行给常量一些不一样的值。

在 flag 包中一个 Flag 被定义成一个含有如下字段的结构体：

```go
   type Flag struct {
       Name     string // name as itappears on command line
       Usage    string // help message
       Value    Value  // value as set
       DefValue string // default value (as text); for usage message
    }
```

下面的程序 echo.go 模拟了Unix 的 echo 功能：

```go
   package main 

   import (
       "flag" // command line option parser
       "os"
    )  

   var NewLine = flag.Bool("n", false, "print newline") // echo -n flag, of type *bool  

   const (
       Space   = " "
       Newline = "\n"
    )   

   func main() {
       flag.PrintDefaults()
       flag.Parse() // Scans the arg list and sets up flags
       var s string = ""
       for i := 0; i < flag.NArg(); i++ {
           if i > 0 {
                s += " "
                if *NewLine { // -n is parsed,flag becomes true
                    s += Newline
                }
           }
           s += flag.Arg(i)
       }
       os.Stdout.WriteString(s)
    }
```

flag.Parse()扫描参数列表（或者常量列表）并设置 flag, flag.Arg(i) 表示第i个参数。Parse() 之后flag.Arg(i) 全部可用，flag.Arg(0) 就是第一个真实的 flag，而不是像 os.Args(0) 放置程序的名字。

flag.Narg()返回参数的数量。解析后 flag 或常量就可用了。flag.Bool() 定义了一个默认值是 false的flag：当在命令行出现了第一个参数（这里是 "n"），flag 被设置成 true（NewLine是 *bool 类型）。flag 被解引用到 *NewLine，所以当值是 true 时将添加一个 newline（"\n"）。

flag.PrintDefaults()打印 flag 的使用帮助信息，本例中打印的是：

```shell
   -n=false: print newline
```

另外，`flag.VisitAll(fnfunc(*Flag))` 是另一个有用的功能：按照字典顺序遍历 flag，并且对每个标签调用 fn 。

当在命令行（Windows）中执行：echo.exeA B C，将输出：A B C；执行 echo.exe -nA B C，将输出：

```shell
A
B
C
```

每个字符的输出都新起一行，每次都在输出的数据前面打印使用帮助信息：`-n=false: print newline`。

对于 flag.Bool 可以设置布尔型 flag 来测试的代码，例如定义一个`flag processedFlag`:

```go
   var processedFlag = flag.Bool(“proc”, false, “nothing processed yet”)
```

在后面用如下代码来测试：

```go
   if *processedFlag { // found flag -proc
       r = process()
   }
```

要给 flag 定义其它类型，可以使用 flag.Int()，flag.Float64，flag.String()。

### 13.5 用buffer读取文件

在下面的例子中，结合使用了缓冲读取文件和命令行 flag 解析这两项技术。如果不加参数，那么输入什么屏幕就打印什么。
参数被认为是文件名，如果文件存在的话就打印文件内容到屏幕。命令行执行 cat test 测试输出。

示例cat.go：

```go
   package main

   import (
       "bufio"
       "flag"
       "fmt"
       "io"
       "os"
    )  

   func cat(r *bufio.Reader) {
       for {
           buf, err := r.ReadBytes('\n')
           if err == io.EOF {
                break
           }
           fmt.Fprintf(os.Stdout, "%s", buf)
       }
       return
    }    

   func main() {
        flag.Parse()
       if flag.NArg() == 0 {
           cat(bufio.NewReader(os.Stdin))
       }

       for i := 0; i < flag.NArg(); i++ {
           f, err := os.Open(flag.Arg(i))
           if err != nil {
                fmt.Fprintf(os.Stderr, "%s:errorreading from %s: %s\n", 
                                       os.Args[0],flag.Arg(i), err.Error())
                continue
           }
           cat(bufio.NewReader(f))
       }
    }
```

### 13.6 用切片读写文件

切片提供了 Go 中处理 I/O 缓冲的标准方式，下面 cat 函数的第二版中，在一个切片缓冲内使用无限 for 循环（直到文件尾部 EOF）读取文件，并写入到标准输出（os.Stdout）。

```go
func cat(f *os.File) {
       const NBUF = 512
       var buf [NBUF]byte
       for {
           switch nr, err := f.Read(buf[:]); true {
           case nr < 0:
                fmt.Fprintf(os.Stderr,"cat: error reading: %s\n", 
                                                    err.Error())
                os.Exit(1)
           case nr == 0: // EOF
                return
           case nr > 0:
                if nw, ew := os.Stdout.Write(buf[0:nr]); nw!= nr {
                    fmt.Fprintf(os.Stderr,"cat: error writing: %s\n", 
                                                       ew.Error())
                }
           }
       }
    }
```

下面的代码来自于 cat2.go，使用了 os 包中的os.file 和 Read 方法；cat2.go 与 cat.go 具有同样的功能。

示例cat2.go：

```go
   package main    

   import (
       "flag"
       "fmt"
       "os"
    )

   func cat(f *os.File) {
       const NBUF = 512
       var buf [NBUF]byte
       for {
           switch nr, err := f.Read(buf[:]); true {
           case nr < 0:
                fmt.Fprintf(os.Stderr,"cat: error reading: %s\n", 
                                    err.Error())
                os.Exit(1)
           case nr == 0: // EOF
                return
           case nr > 0:
                if nw, ew :=os.Stdout.Write(buf[0:nr]); nw != nr {
                    fmt.Fprintf(os.Stderr,"cat: error writing: %s\n", 
                                         ew.Error())
                }
           }
       }
    }    

   func main() {
       flag.Parse() // Scans the arg list and sets up flags
       if flag.NArg() == 0 {
           cat(os.Stdin)
       }

       for i := 0; i < flag.NArg(); i++ {
           f, err := os.Open(flag.Arg(i))
           if f == nil {
                fmt.Fprintf(os.Stderr,"cat: can't open %s: error %s\n", 
                                                     flag.Arg(i), err)
                os.Exit(1)
           }

           cat(f)
           f.Close()
       }
    }
```

### 13.7 用defer 关闭文件

defer关键字对于在函数结束时关闭打开的文件非常有用，例如下面的代码片段：

```go
func data(name string) string {
       f := os.Open(name, os.O_RDONLY, 0)
       defer f.Close() // idiomatic Go code!
       contents := io.ReadAll(f)
       return contents
}
```

在函数 return 后执行了f.Close()

### 13.8 使用接口的实际例子：fmt.Fprintf

例子程序 io_interfaces.go 很好的阐述了 io 包中的接口概念。

示例io_interfaces.go：

```go
   // interfaces being used in the GO-package fmt

   package main    

   import (
       "bufio"
       "fmt"
       "os"
    )    

   func main() {
       // unbuffered
       fmt.Fprintf(os.Stdout, "%s\n", "hello world! -unbuffered")
       // buffered: os.Stdout implements io.Writer
       buf := bufio.NewWriter(os.Stdout)
       // and now so does buf.
       fmt.Fprintf(buf, "%s\n", "hello world! - buffered")
       buf.Flush()
    }
```

下面是 fmt.Fprintf() 函数的实际签名

```go
   func Fprintf(w io.Writer, format string, a ...interface{}) (n int, errerror)
```

其不是写入一个文件，而是写入一个 io.Writer 接口类型的变量，下面是 Writer 接口在 io 包中的定义：

```go
   type Writer interface {
       Write(p []byte) (n int, err error)
   }
```

fmt.Fprintf()依据指定的格式向第一个参数内写入字符串，第一参数必须实现了 io.Writer 接口。Fprintf() 能够写入任何类型，只要其实现了 Write 方法，包括 os.Stdout,文件（例如 os.File），管道，网络连接，通道等等，同样的也可以使用 bufio 包中缓冲写入。bufio 包中定义了`type Writer struct{...}`。

bufio.Writer实现了 Write 方法：

```go
   func (b *Writer) Write(p []byte) (nn int, err error)
```

它还有一个工厂函数：传给它一个 io.Writer 类型的参数，它会返回一个缓冲的 bufio.Writer 类型的 io.Writer:

```go
   func NewWriter(wr io.Writer) (b *Writer)
```

其适合任何形式的缓冲写入。

在缓冲写入的最后千万不要忘了使用 Flush()，否则最后的输出不会被写入。

### 13.9 JSON 数据格式

数据结构要在网络中传输或保存到文件，就必须对其编码和解码；目前存在很多编码格式：JSON，XML，gob，Google 缓冲协议等等。Go 语言支持所有这些编码格式；在后面的章节，将讨论前三种格式。

结构可能包含二进制数据，如果将其作为文本打印，那么可读性是很差的。另外结构内部可能包含匿名字段，而不清楚数据的用意。

通过把数据转换成纯文本，使用命名的字段来标注，让其具有可读性。这样的数据格式可以通过网络传输，而且是与平台无关的，任何类型的应用都能够读取和输出，不与操作系统和编程语言的类型相关。

下面是一些术语说明：

- 数据结构 --> 指定格式 = 序列化 或 编码（传输之前）

- 指定格式 --> 数据格式 = 反序列化 或 解码（传输之后）

序列化是在内存中把数据转换成指定格式（data -> string），反之亦然（string -> datastructure）

编码也是一样的，只是输出一个数据流（实现了 io.Writer 接口）；解码是从一个数据流（实现了 io.Reader）输出到一个数据结构。

都比较熟悉 XML 格式；但有些时候 JSON（JavaScriptObject Notation，参阅 http://json.org）被作为首选，主要是由于其格式上非常简洁。通常 JSON 被用于 web 后端和浏览器之间的通讯，但是在其它场景也同样的有用。

这是一个简短的 JSON 片段：

```json
{
       "Person": {
           "FirstName": "Laura",
           "LastName": "Lynn"
       }
}
```

尽管 XML 被广泛的应用，但是 JSON 更加简洁、轻量（占用更少的内存、磁盘及网络带宽）和更好的可读性，这也说明它越来越受欢迎。
Go语言的 json 包可以让在程序中方便的读取和写入 JSON 数据。
将在下面的例子里使用 json 包，并使用练习 10.1 vcard.go 中一个简化版本的 Address 和 VCard 结构。

示例json.go：

```go
   // json.go.go

   package main    

   import (
       "encoding/json"
       "fmt"
       "log"
       "os"
    )

    
  type Address struct {
       Type    string
       City    string
       Country string
    }

   type VCard struct {
       FirstName string
       LastName  string
       Addresses []*Address
       Remark    string
    }

   func main() {
       pa := &Address{"private", "Aartselaar","Belgium"}
       wa := &Address{"work", "Boom", "Belgium"}
       vc := VCard{"Jan", "Kersschot", []*Address{pa, wa},"none"}
       // fmt.Printf("%v: \n", vc) // {Jan Kersschot [0x126d2b800x126d2be0] none}:
       // JSON format:
       js, _ := json.Marshal(vc)
       fmt.Printf("JSON format: %s", js)
       // using an encoder:
       file, _ := os.OpenFile("vcard.json", os.O_CREATE|os.O_WRONLY,0)
       defer file.Close()
       enc := json.NewEncoder(file)
       err := enc.Encode(vc)
       if err != nil {
           log.Println("Error in encoding json")
       }
    }
```

json.Marshal()的函数签名是 func Marshal(v interface{}) ([]byte, error)，下面是数据编码后的 JSON 文本（实际上是一个 []bytes）：

```go
    {
       "FirstName": "Jan",
       "LastName": "Kersschot",
       "Addresses": [{
           "Type": "private",
           "City": "Aartselaar",
           "Country": "Belgium"
       }, {
           "Type": "work",
           "City": "Boom",
           "Country": "Belgium"
       }],
       "Remark": "none"
    }
```

出于安全考虑，在 web 应用中最好使用`json.MarshalforHTML()`函数，其对数据执行HTML转码，所以文本可以被安全地嵌在 `HTML <script> `标签中。

JSON与 Go 类型对应如下：
- bool 对应 JSON 的 booleans
- float64 对应 JSON 的 numbers
- string 对应 JSON 的 strings
- nil 对应 JSON 的 null

不是所有的数据都可以编码为 JSON 类型：只有验证通过的数据结构才能被编码：

- JSON 对象只支持字符串类型的key；要编码一个 Go map 类型，map 必须是 map[string]T（T是json 包中支持的任何类型）
- Channel，复杂类型和函数类型不能被编码
- 不支持循环数据结构；它将引起序列化进入一个无限循环
- 指针可以被编码，实际上是对指针指向的值进行编码（或者指针是 nil） 

反序列化：
UnMarshal()的函数签名是 `func Unmarshal(data []byte, v interface{}) error把 JSON` 解码为数据结构。

首先创建一个结构 Message 用来保存解码的数据：var m Message 并调用 Unmarshal()，解析 []byte 中的 JSON 数据并将结果存入指针 m 指向的值。

虽然反射能够让 JSON 字段去尝试匹配目标结构字段；但是只有真正匹配上的字段才会填充数据。字段没有匹配不会报错，而是直接忽略掉。

 
解码任意的数据：

json包使用`map[string]interface{}`和`[]interface{}`储存任意的 JSON 对象和数组；其可以被反序列化为任何的 JSON blob 存储到接口值中。

来看这个 JSON 数据，被存储在变量 b 中：

```go
b== []byte({"Name": "Wednesday", "Age": 6,"Parents": ["Gomez", "Morticia"]})
```

不用理解这个数据的结构，可以直接使用 Unmarshal 把这个数据编码并保存在接口值中：

```go
   var f interface{}

   err := json.Unmarshal(b, &f)
```

f指向的值是一个 map，key 是一个字符串，value 是自身存储作为空接口类型的值：

```go
   map[string]interface{} {
       "Name": "Wednesday",
       "Age":  6,
       "Parents": []interface{} {
           "Gomez",
           "Morticia",
       },
  }
```

要访问这个数据，可以使用类型断言

```go
m:= f.(map[string]interface{})
```

可以通过 for range 语法和 type switch 来访问其实际类型：

```go
   for k, v := range m {
       switch vv := v.(type) {
       case string:
           fmt.Println(k, "is string", vv)
       case int:
           fmt.Println(k, "is int", vv)
       case []interface{}:
           fmt.Println(k, "is an array:")
           for i, u := range vv {
                fmt.Println(i, u)
           }
       default:
           fmt.Println(k, "is of a type I don’t knowhow to handle")
       }
   }
```

通过这种方式，可以处理未知的 JSON 数据，同时可以确保类型安全。

**解码数据到结构**

如果事先知道 JSON 数据，可以定义一个适当的结构并对 JSON 数据反序列化。下面的例子中，将定义：

```go
   type FamilyMember struct {
       Name    string
       Age     int
       Parents []string
   }
```

并对其反序列化：

```go
   var m FamilyMember
   err := json.Unmarshal(b, &m)
```

程序实际上是分配了一个新的切片。这是一个典型的反序列化引用类型（指针、切片和 map）的例子。

**编码和解码流**

json包提供 Decoder 和 Encoder 类型来支持常用 JSON 数据流读写。NewDecoder 和 NewEncoder 函数分别封装了 io.Reader 和 io.Writer 接口。

```go
   func NewDecoder(r io.Reader) *Decoder

   func NewEncoder(w io.Writer) *Encoder
```

要想把 JSON 直接写入文件，可以使用 json.NewEncoder 初始化文件（或者任何实现 io.Writer 的类型），并调用 Encode()；反过来与其对应的是使用 json.Decoder 和 Decode() 函数：

```go
   func NewDecoder(r io.Reader) *Decoder

   func (dec *Decoder) Decode(v interface{}) error
```

来看下接口是如何对实现进行抽象的：数据结构可以是任何类型，只要其实现了某种接口，目标或源数据要能够被编码就必须实现 io.Writer 或 io.Reader 接口。由于 Go 语言中到处都实现了 Reader 和 Writer，因此 Encoder 和Decoder 可被应用的场景非常广泛，例如读取或写入 HTTP 连接、websockets 或文件。

### 13.10 XML 数据格式

下面是XML 版本：

```xml
   <Person>
       <FirstName>Laura</FirstName>
       <LastName>Lynn</LastName>
   </Person>
```

如同 json 包一样，也有 Marshal() 和 UnMarshal() 从 XML 中编码和解码数据；但这个更通用，可以从文件中读取和写入（或者任何实现了 io.Reader 和 io.Writer 接口的类型）

和 JSON 的方式一样，XML 数据可以序列化为结构，或者从结构反序列化为 XML 数据；这些可以在例子 15.8（twitter_status.go）中看到。

encoding/xml 包实现了一个简单的 XML 解析器（SAX），用来解析XML 数据内容。下面的例子说明如何使用解析器：

示例xml.go：

```go
   // xml.go

   package main    

   import (
       "encoding/xml"
       "fmt"
       "strings"
    )    

   var t, token xml.Token
   var err error    

   func main() {
       input :="<Person><FirstName>Laura</FirstName><LastName>Lynn</LastName></Person>"
       inputReader := strings.NewReader(input)
       p := xml.NewDecoder(inputReader)    
       for t, err = p.Token(); err == nil; t, err = p.Token() {
           switch token := t.(type) {
           case xml.StartElement:
                name := token.Name.Local
                fmt.Printf("Token name:%s\n", name)
                for _, attr := range token.Attr {
                    attrName := attr.Name.Local
                    attrValue := attr.Value
                    fmt.Printf("Anattribute is: %s %s\n", attrName, 
                                                       attrValue)
                    // ...
                }

           case xml.EndElement:
                fmt.Println("End oftoken")
           case xml.CharData:
                content :=string([]byte(token))
                fmt.Printf("This is thecontent: %v\n", content)
                // ...
           default:
                // ...
           }
       }
   }
```

包中定义了若干 XML 标签类型：StartElement，Chardata（这是从开始标签到结束标签之间的实际文本），EndElement，Comment，Directive 或ProcInst。

包中同样定义了一个结构解析器：NewParser 方法持有一个 io.Reader（这里具体类型是 strings.NewReader）并生成一个解析器类型的对象。还有一个Token() 方法返回输入流里的下一个 XML token。在输入流的结尾处，会返回（nil，io.EOF）

XML文本被循环处理直到 Token() 返回一个错误，因为已经到达文件尾部，再没有内容可供处理了。通过一个 type-switch 可以根据一些 XML 标签进一步处理。Chardata 中的内容只是一个 []byte，通过字符串转换让其变得可读性强一些。

### 13.11 用Gob 传输数据

Gob是 Go 自己的以二进制形式序列化和反序列化程序数据的格式；可以在 encoding 包中找到。这种格式的数据简称为 Gob （即 Go binary 的缩写）。类似于 Python 的 "pickle" 和 Java 的 "Serialization"。

Gob通常用于远程方法调用参数和结果的传输，以及应用程序和机器之间的数据传输。 它和 JSON 或 XML 有什么不同呢？Gob 特定地用于纯 Go 的环境中，例如，两个用 Go 写的服务之间的通信。这样的话服务可以被实现得更加高效和优化。 Gob 不是可外部定义，语言无关的编码方式。因此它的首选格式是二进制，而不是像JSON 和 XML 那样的文本格式。 Gob 并不是一种不同于 Go 的语言，而是在编码和解码过程中用到了 Go 的反射。

Gob文件或流是完全自描述的：里面包含的所有类型都有一个对应的描述，并且总是可以用 Go 解码，而不需要了解文件的内容。

只有可导出的字段会被编码，零值会被忽略。在解码结构体的时候，只有同时匹配名称和可兼容类型的字段才会被解码。当源数据类型增加新字段后，Gob 解码客户端仍然可以以这种方式正常工作：解码客户端会继续识别以前存在的字段。并且还提供了很大的灵活性，比如在发送者看来，整数被编码成没有固定长度的可变长度，而忽略具体的 Go 类型。

假如在发送者这边有一个有结构 T：

```go
   type T struct { X, Y, Z int }
   var t = T{X: 7, Y: 0, Z: 8}
```

而在接收者这边可以用一个结构体 U 类型的变量 u 来接收这个值：

```go
   type U struct { X, Y *int8 }
   var u U
```

在接收者中，X 的值是7，Y 的值是0（Y的值并没有从 t 中传递过来，因为它是零值）

和 JSON 的使用方式一样，Gob 使用通用的 io.Writer 接口，通过 NewEncoder() 函数创建 Encoder 对象并调用 Encode()；相反的过程使用通用的 io.Reader 接口，通过 NewDecoder() 函数创建 Decoder对象并调用 Decode。

把示例的信息写进名为 vcard.gob 的文件作为例子。这会产生一个文本可读数据和二进制数据的混合，当试着在文本编辑中打开的时候会看到。

示例gob1.go：

示例 gob2.go 编码到文件：

### 13.12 Go 中的密码学

通过网络传输的数据必须加密，以防止被 hacker（黑客）读取或篡改，并且保证发出的数据和收到的数据检验和一致。 鉴于 Go母公司的业务，毫不惊讶地看到 Go 的标准库为该领域提供了超过30 个的包：

- hash 包：实现了adler32、crc32、crc64 和 fnv 校验；
- crypto 包：实现了其它的hash 算法，比如 md4、md5、sha1 等。以及完整地实现了 aes、blowfish、rc4、rsa、xtea 等加密算法。

下面的示例用 sha1 和 md5 计算并输出了一些校验值。

示例hash_sha1.go：

```go
   // hash_sha1.go

   package main    

   import (
       "fmt"
       "crypto/sha1"
       "io"
       "log"
   )    

   func main() {
       hasher := sha1.New()
       io.WriteString(hasher, "test")
       b := []byte{}
       fmt.Printf("Result: %x\n", hasher.Sum(b))
       fmt.Printf("Result: %d\n", hasher.Sum(b))
       //
       hasher.Reset()
       data := []byte("We shall overcome!")
       n, err := hasher.Write(data)
       if n!=len(data) || err!=nil {
            log.Printf("Hash write error: %v/ %v", n, err)
       }
       checksum := hasher.Sum(b)
       fmt.Printf("Result: %x\n", checksum)
    }
```

通过调用 sha1.New() 创建了一个新的 hash.Hash 对象，用来计算 SHA1 校验值。Hash 类型实际上是一个接口，它实现了 io.Writer 接口：

```go
   type Hash interface {
       // Write (via the embedded io.Writer interface) adds more data to therunning hash.
       // It never returns an error.
       io.Writer
       // Sum appends the current hash to b and returns the resulting slice.
       // It does not change the underlying hash state.
       Sum(b []byte) []byte
       // Reset resets the Hash to its initial state.
       Reset()
       // Size returns the number of bytes Sum will return.
       Size() int
       // BlockSize returns the hash's underlying block size.
       // The Write method must be able to accept any amount
       // of data, but it may operate more efficiently if all writes
       // are a multiple of the block size.
       BlockSize() int
    }
```

通过 io.WriteString 或 hasher.Write 计算给定字符串的校验值。

## 14 错误处理与测试

Go没有像 Java 和 .NET 那样的 try/catch 异常机制：不能执行抛异常操作。但是有一套defer-panic-and-recover 机制。

Go的设计者觉得 try/catch 机制的使用太泛滥了，而且从底层向更高的层级抛异常太耗费资源。它们给 Go 设计的机制也可以 “捕捉” 异常，但是更轻量，并且只应该作为（处理错误的）最后的手段。

Go是怎么处理普通错误的呢？通过在函数和方法中返回错误对象作为它们的唯一或最后一个返回值——如果返回 nil，则没有错误发生——并且主调（calling）函数总是应该检查收到的错误。

永远不要忽略错误，否则可能会导致程序崩溃！！

处理错误并且在函数发生错误的地方给用户返回错误信息：照这样处理就算真的出了问题，的程序也能继续运行并且通知给用户。panic and recover 是用来处理真正的异常（无法预测的错误）而不是普通的错误。

库函数通常必须返回某种错误提示给主调（calling）函数。

在前面的章节中了解了 Go 检查和报告错误条件的惯有方式：

- 产生错误的函数会返回两个变量，一个值和一个错误码；如果后者是 nil 就是成功，非 nil 就是发生了错误。

- 为了防止发生错误时正在执行的函数（如果有必要的话甚至会是整个程序）被中止，在调用函数后必须检查错误。

   下面这段来自 pack1 包的代码 Func1 测试了它的返回值：
   
```go
   if value, err := pack1.Func1(param1); err != nil {
       fmt.Printf(“Error %s in pack1.Func1 withparameter %v”, 
                  err.Error(), param1)
       return    // or: return err
   }

   // Process(value)
```

为了更清晰的代码，应该总是使用包含错误值变量的 if 复合语句

### 14.1 错误处理

Go有一个预先定义的 error 接口类型

```go
   type error interface {
       Error() string
   }
```

错误值用来表示异常状态。errors 包中有一个 errorString 结构体实现了 error 接口。当程序处于错误状态时可以用 os.Exit(1) 来中止运行。

#### 14.1.1 定义错误

任何时候当需要一个新的错误类型，都可以用 errors（必须先 import）包的 errors.New 函数接收合适的错误信息来创建，像下面这样：

```go
   err := errors.New(“math - square root ofnegative number”)
```

示例errors.go：

```go
   // errors.go

   package main    

   import (
       "errors"
       "fmt"
    )

   var errNotFound error = errors.New("Not found error")

   func main() {
       fmt.Printf("error: %v", errNotFound)
   }
   // error: Not found error
```

可以把它用于计算平方根函数的参数测试：

```go
   func Sqrt(f float64) (float64, error) {
       if f < 0 {
           return 0, errors.New (“math - square root ofnegative number”)
       }
       // implementation of Sqrt
  }
```

可以像下面这样调用 Sqrt 函数：

```go
   if f, err := Sqrt(-1); err != nil {
       fmt.Printf(“Error: %s\n”,err)
    }
```

由于 fmt.Printf 会自动调用 String() 方法 （参见 10.7 节），所以错误信息 “Error: math - square rootof negative number” 会打印出来。通常（错误信息）都会有像 “Error:” 这样的前缀，所以的错误信息不要以大写字母开头。

在大部分情况下自定义错误结构类型很有意义的，可以包含除了（低层级的）错误信息以外的其它有用信息，例如，正在进行的操作（打开文件等），全路径或名字。看下面例子中 os.Open 操作触发的 PathError 错误：

```go
   // PathError records an error and the operation and file path thatcaused it.

   type PathError struct {
       Op string    // “open”, “unlink”, etc.
       Path string  // The associatedfile.
       Err error  // Returned by thesystem call.
    }    

   func (e *PathError) String() string {
       return e.Op + “ ” + e.Path + “: “+ e.Err.Error()
   }
```

如果有不同错误条件可能发生，那么对实际的错误使用类型断言或类型判断（type-switch）是很有用的，并且可以根据错误场景做一些补救和恢复操作。

```go
   //  err != nil

   if e, ok := err.(*os.PathError); ok {
       // remedy situation
    }
```

或：

```go
   switch err := err.(type) {
       case ParseError:
           PrintParseError(err)
       case PathError:
           PrintPathError(err)
       ...
       default:
           fmt.Printf(“Not a special error, just %s\n”, err)
    }
```

作为第二个例子考虑用 json 包的情况。当 json.Decode 在解析 JSON 文档发生语法错误时，指定返回一个 SyntaxError 类型的错误：

```go
   type SyntaxError struct {
       msg    string // description oferror
   // error occurred after reading Offset bytes, from which line andcolumnnr can be obtained
       Offset int64
    }    

   func (e *SyntaxError) String() string { return e.msg }
```

在调用代码中可以像这样用类型断言测试错误是不是上面的类型：

```go
   if serr, ok := err.(*json.SyntaxError); ok {
       line, col := findLine(f, serr.Offset)
       return fmt.Errorf(“%s:%d:%d: %v”, f.Name(), line, col, err)
    }
```

包也可以用额外的方法（methods）定义特定的错误，比如 net.Errot：

```go
   package net

   type Error interface {
       Timeout() bool   // Is the error atimeout?
       Temporary() bool // Is the error temporary?
    }
```

正如所看到的一样，所有的例子都遵循同一种命名规范：错误类型以“Error” 结尾，错误变量以 “err” 或 “Err” 开头。

syscall是低阶外部包，用来提供系统基本调用的原始接口。它们返回整数的错误码；类型 syscall.Errno 实现了 Error 接口。

大部分 syscall 函数都返回一个结果和可能的错误，比如：

```go
   r, err := syscall.Open(name, mode, perm)
   if err != 0 {
       fmt.Println(err.Error())
    }
```

os包也提供了一套像 os.EINAL 这样的标准错误，它们基于syscall 错误：

```go
   var (
       EPERM        Error =Errno(syscall.EPERM)
       ENOENT        Error =Errno(syscall.ENOENT)
       ESRCH        Error =Errno(syscall.ESRCH)
       EINTR        Error =Errno(syscall.EINTR)
      EIO            Error =Errno(syscall.EIO)
       ...
    )
```

#### 14.1.2 用 fmt 创建错误对象

通常想要返回包含错误参数的更有信息量的字符串，例如：可以用 fmt.Errorf() 来实现：它和 fmt.Printf() 完全一样，接收有一个或多个格式占位符的格式化字符串和相应数量的占位变量。和打印信息不同的是它用信息生成错误对象。

比如在前面的平方根例子中使用：

```go
   if f < 0 {
       return 0, fmt.Errorf(“math: square root ofnegative number %g”, f)
    }
```


第二个例子：从命令行读取输入时，如果加了 help 标志，可以用有用的信息产生一个错误：

```go
   if len(os.Args) > 1 && (os.Args[1] == “-h” || os.Args[1] == “--help”) {
       err = fmt.Errorf(“usage: %s infile.txtoutfile.txt”, filepath.Base(os.Args[0]))
       return
    }
```

### 14.2 运行时异常和 panic

当发生像数组下标越界或类型断言失败这样的运行错误时，Go 运行时会触发运行时 panic，伴随着程序的崩溃抛出一个 runtime.Error 接口类型的值。这个错误值有个 RuntimeError()方法用于区别普通错误。

panic可以直接从代码初始化：当错误条件（所测试的代码）很严苛且不可恢复，程序不能继续运行时，可以使用 panic 函数产生一个中止程序的运行时错误。panic 接收一个做任意类型的参数，通常是字符串，在程序死亡时被打印出来。Go 运行时负责中止程序并给出调试信息。在示例 13.2 panic.go 中阐明了它的工作方式：

```go
package main    

import "fmt"    

func main() {
    fmt.Println("Starting the program")
    panic("A severe error occurred: stopping the program!")
    fmt.Println("Ending the program")
}
```

一个检查程序是否被已知用户启动的具体例子：

```go
   var user = os.Getenv(“USER”)    

   func check() {
       if user == “” {
           panic(“Unknown user: no value for $USER”)
       }

   }
```

可以在导入包的 init() 函数中检查这些。

当发生错误必须中止程序时，panic 可以用于错误处理模式：

```go
   if err != nil {
       panic(“ERROR occurred:”+ err.Error())
   }
```
 
在多层嵌套的函数调用中调用 panic，可以马上中止当前函数的执行，所有的 defer 语句都会保证执行并把控制权交还给接收到 panic 的函数调用者。这样向上冒泡直到最顶层，并执行（每层的） defer，在栈顶处程序崩溃，并在命令行中用传给 panic 的值报告错误情况：这个终止过程就是 panicking。

标准库中有许多包含 Must 前缀的函数，像 regexp.MustComplie 和 template.Must；当正则表达式或模板中转入的转换字符串导致错误时，这些函数会 panic。

不能随意地用 panic 中止程序，必须尽力补救错误让程序能继续执行。

### 14.3 从panic 中恢复（Recover）

正如名字一样，这个（recover）内建函数被用于从 panic 或 错误场景中恢复：让程序可以从 panicking 重新获得控制权，停止终止过程进而恢复正常执行。

recover只能在 defer 修饰的函数（参见 6.4 节）中使用：用于取得 panic 调用中传递过来的错误值，如果是正常执行，调用 recover 会返回 nil，且没有其它效果。

总结：panic 会导致栈被展开直到 defer 修饰的 recover() 被调用或者程序中止。

下面例子中的 protect 函数调用函数参数 g 来保护调用者防止从 g 中抛出的运行时 panic，并展示panic 中的信息：

```go
   func protect(g func()) {

       defer func() {
           log.Println(“done”)
           // Println executes normally even if there is a panic
           if err := recover(); err != nil {
               log.Printf(“run time panic: %v”, err)
           }
       }()

       log.Println(“start”)
       g() //   possible runtime-error
    }
```

这跟 Java 和 .NET 这样的语言中的catch 块类似。

log包实现了简单的日志功能：默认的 log 对象向标准错误输出中写入并打印每条日志信息的日期和时间。除了 Println 和 Printf 函数，其它的致命性函数都会在写完日志信息后调用 os.Exit(1)，那些退出函数也是如此。而 Panic 效果的函数会在写完日志信息后调用 panic；可以在程序必须中止或发生了临界错误时使用它们，就像当 web 服务器不能启动时那样。

log包用那些方法（methods）定义了一个 Logger 接口类型，如果想自定义日志系统的话可以参考（参见 http://golang.org/pkg/log/#Logger）。

这是一个展示 panic，defer 和 recover怎么结合使用的完整例子：

示例panic_recover.go：

```go
   // panic_recover.go

   package main    

   import (
       "fmt"
    )    

   func badCall() {
       panic("bad end")
    }

   func test() {
       defer func() {
           if e := recover(); e != nil {
                fmt.Printf("Panicing%s\r\n", e)
           }
       }()

       badCall()

       fmt.Printf("After bad call\r\n") // <-- wordt niet bereikt
    }    

   func main() {
       fmt.Printf("Calling test\r\n")
       test()
       fmt.Printf("Test completed\r\n")
    }    
```

 defer-panic-recover在某种意义上也是一种像 if，for 这样的控制流机制。

Go标准库中许多地方都用了这个机制，例如，json 包中的解码和regexp 包中的 Complie 函数。Go 库的原则是即使在包的内部使用了 panic，在它的对外接口（API）中也必须用 recover 处理成返回显式的错误。

### 14.4 自定义包中的错误处理和 panicking

这是所有自定义包实现者应该遵守的最佳实践：

1）在包内部，总是应该从 panic 中 recover：不允许显式的超出包范围的 panic()

2）向包的调用者返回错误值（而不是panic）。

在包内部，特别是在非导出函数中有很深层次的嵌套调用时，对主调函数来说用 panic 来表示应该被翻译成错误的错误场景是很有用的（并且提高了代码可读性）。

这在下面的代码中被很好地阐述了。有一个简单的 parse 包用来把输入的字符串解析为整数切片；这个包有自己特殊的ParseError。

当没有东西需要转换或者转换成整数失败时，这个包会 panic（在函数 fields2numbers 中）。但是可导出的 Parse 函数会从 panic 中recover 并用所有这些信息返回一个错误给调用者。为了演示这个过程，在 panic_recover.go 中调用了 parse 包不可解析的字符串会导致错误并被打印出来。

示例parse.go：

```go
   // parse.go

   package parse

   import (
       "fmt"
       "strings"
       "strconv"
    )
    

   // A ParseError indicates an error in converting a word into an integer.
   type ParseError struct {
       Index int      // The index intothe space-separated list of words.
       Word  string   // The word that generated the parse error.
       Err error // The raw error that precipitated this error, if any.
    }
    

   // String returns a human-readable error message.
   func (e *ParseError) String() string {
      return fmt.Sprintf("pkg parse: error parsing %q as int",e.Word)
   }
    

   // Parse parses the space-separated words in in put as integers.
   func Parse(input string) (numbers []int, err error) {
       defer func() {
           if r := recover(); r != nil {
                var ok bool
                err, ok = r.(error)
                if !ok {
                    err = fmt.Errorf("pkg:%v", r)
                }
           }
       }()    

       fields := strings.Fields(input)
       numbers = fields2numbers(fields)
       return
    }    

   func fields2numbers(fields []string) (numbers []int) {
       if len(fields) == 0 {
           panic("no words to parse")
       }

       for idx, field := range fields {
           num, err := strconv.Atoi(field)
           if err != nil {
                panic(&ParseError{idx,field, err})
           }

           numbers = append(numbers, num)
       }
       return

    }
```

示例 13.5 panic_package.go：

```go
   // panic_package.go

   package main


   import (
       "fmt"
       "./parse/parse"
    )    

   func main() {
       var examples = []string{
                "1 2 3 4 5",
                "100 50 25 12.56.25",
               "2 + 2 = 4",
                "1st class",
                "",
       }

       for _, ex := range examples {
           fmt.Printf("Parsing %q:\n ", ex)
           nums, err := parse.Parse(ex)
           if err != nil {
                // here String() method fromParseError is used
                fmt.Println(err) 
                continue
           }

           fmt.Println(nums)
       }
    }
```

### 14.5 一种用闭包处理错误的模式

每当函数返回时，应该检查是否有错误发生：但是这会导致重复乏味的代码。结合 defer/panic/recover 机制和闭包可以得到一个马上要讨论的更加优雅的模式。不过这个模式只有当所有的函数都是同一种签名时可用，这样就有相当大的限制。一个很好的使用它的例子是 web 应用，所有的处理函数都是下面这样：

```go
   func handler1(w http.ResponseWriter, r *http.Request) { ... }
```

假设所有的函数都有这样的签名：

```go
   func f(a type1, b type2)
```

参数的数量和类型是不相关的。

给这个类型一个名字：

```go
   fType1 = func f(a type1, b type2)
```

在的模式中使用了两个帮助函数：

1）check：这是用来检查是否有错误和 panic 发生的函数：

```go
   func check(err error) { if err != nil { panic(err) } }
```

2）errorhandler：这是一个包装函数。接收一个 fType1 类型的函数 fn 并返回一个调用 fn 的函数。里面就包含有 defer/recover 机制，这在 13.3 节中有相应描述。

```go
   func errorHandler(fn fType1) fType1 {
       return func(a type1, b type2) {
           defer func() {
                if e, ok := recover().(error);ok {
                    log.Printf(“run time panic: %v”, err)
                }
           }()
           fn(a, b)
       }
    }
```

当错误发生时会 recover 并打印在日志中；除了简单的打印，应用也可以用 template 包为用户生成自定义的输出。check() 函数会在所有的被调函数中调用，像这样：

```go
   func f1(a type1, b type2) {
       ...
       f, _, err := // call function/method
       check(err)
       t, err := // call function/method
       check(err)
       _, err2 := // call function/method
       check(err2)
       ...
    }
```

通过这种机制，所有的错误都会被 recover，并且调用函数后的错误检查代码也被简化为调用 check(err) 即可。在这种模式下，不同的错误处理必须对应不同的函数类型；它们（错误处理）可能被隐藏在错误处理包内部。可选的更加通用的方式是用一个空接口类型的切片作为参数和返回值。


阅读下面的完整程序。不要执行它，写出程序的输出结果。然后编译执行并验证的预想。

### 14.6 启动外部命令和程序

os包有一个 StartProcess 函数可以调用或启动外部系统命令和二进制可执行文件；它的第一个参数是要运行的进程，第二个参数用来传递选项或参数，第三个参数是含有系统环境基本信息的结构体。

这个函数返回被启动进程的 id（pid），或者启动失败返回错误。

exec包中也有同样功能的更简单的结构体和函数；主要是 exec.Command(name string, arg...string) 和 Run()。首先需要用系统命令或可执行文件的名字创建一个 Command 对象，然后用这个对象作为接收者调用 Run()。下面的程序（因为是执行 Linux 命令，只能在 Linux 下面运行）演示了它们的使用：

示例exec.go：

```go
   // exec.go

   package main

   import (
       "fmt"
       "os/exec"
       "os"
    )
 

   func main() {

   // 1) os.StartProcess //

   /*********************/

   /* Linux: */

   env := os.Environ()
   procAttr := &os.ProcAttr{
                Env: env,
                Files: []*os.File{
                    os.Stdin,
                    os.Stdout,
                    os.Stderr,
                },
           }

   // 1st example: list files

   pid, err := os.StartProcess("/bin/ls",[]string{"ls", "-l"}, procAttr)  

   if err != nil {
           fmt.Printf("Error %v starting process!", err)  //
           os.Exit(1)
   }

   fmt.Printf("The process id is %v", pid)

   // 2nd example: show all processes

   pid, err = os.StartProcess("/bin/ps", []string{"-e","-opid,ppid,comm"}, procAttr)  


   if err != nil {
           fmt.Printf("Error %v starting process!", err)  //
           os.Exit(1)
    }    

   fmt.Printf("The process id is %v", pid)

   // 2) exec.Run //

   /***************/

   // Linux:  OK, but not for ls ?
   //no error, but doesn't show anything ?
   // cmd := exec.Command("ls", "-l")  

   //no error, but doesn't show anything ?

   // cmd := exec.Command("ls")          

       cmd := exec.Command("gedit") // this opens a gedit-window
       err = cmd.Run()
       if err != nil {
           fmt.Printf("Error %v executing command!", err)
           os.Exit(1)
       }

       fmt.Printf("The command is %v", cmd)
    // The command is &{/bin/ls [ls -l][]  <nil> <nil> <nil>0xf840000210 <nil> true [0xf84000ea50 0xf84000e9f0 0xf84000e9c0][0xf84000ea50 0xf84000e9f0 0xf84000e9c0] [][] 0xf8400128c0}

    }

   // in Windows: uitvoering: Error fork/exec /bin/ls: The system cannotfind the path specified. starting process!
```

### 14.7 Go 中的单元测试和基准测试

首先所有的包都应该有一定的必要文档，然后同样重要的是对包的测试。

Go的测试工具 gotest。

名为 testing 的包被专门用来进行自动化测试，日志和错误报告。并且还包含一些基准测试函数的功能。

备注：gotest 是 Unix bash 脚本，所以在 Windows 下需要配置 MINGW 环境；在 Windows 环境下把所有的 pkg/linux_amd64 替换成 pkg/windows。

对一个包做（单元）测试，需要写一些可以频繁（每次更新后）执行的小块测试单元来检查代码的正确性。于是必须写一些 Go 源文件来测试代码。测试程序必须属于被测试的包，并且文件名满足这种形式*_test.go，所以测试代码和包中的业务代码是分开的。

`_test`程序不会被普通的 Go 编译器编译，所以当放应用部署到生产环境时它们不会被部署；只有 gotest 会编译所有的程序：普通程序和测试程序。

测试文件中必须导入 "testing" 包，并写一些名字以 TestZzz 打头的全局函数，这里的 Zzz 是被测试函数的字母描述，如 TestFmtInterface，TestPayEmployees 等。

测试函数必须有这种形式的头部：

```go
   func TestAbcde(t *testing.T)
```

T是传给测试函数的结构类型，用来管理测试状态，支持格式化测试日志，如 t.Log，t.Error，t.ErrorF 等。在函数的结尾把输出跟想要的结果对比，如果不等就打印一个错误。成功的测试则直接返回。

用下面这些函数来通知测试失败：

1）func (t *T)Fail()

标记测试函数为失败，然后继续执行（剩下的测试）。

2）func (t *T)FailNow()

标记测试函数为失败并中止执行；文件中别的测试也被略过，继续执行下一个文件。

3）func (t *T)Log(args ...interface{})

args 被用默认的格式格式化并打印到错误日志中。

4）func (t *T)Fatal(args ...interface{})

结合 先执行 3），然后执行 2）的效果。

运行 go test 来编译测试程序，并执行程序中所有的 TestZZZ 函数。如果所有的测试都通过会打印出 PASS。

gotest可以接收一个或多个函数程序作为参数，并指定一些选项。

结合 --chatty 或 -v 选项，每个执行的测试函数以及测试状态会被打印。

例如：

```go
   go test fmt_test.go --chatty
   === RUN fmt.TestFlagParser
   --- PASS: fmt.TestFlagParser
   === RUN fmt.TestArrayPrinter
   --- PASS: fmt.TestArrayPrinter
   ...
```

testing包中有一些类型和函数可以用来做简单的基准测试；测试代码中必须包含以 `BenchmarkZzz` 打头的函数并接收一个 `*testing.B` 类型的参数，比如：

```go
   func BenchmarkReverse(b *testing.B) {
       ...

   }
```

命令 go test –test.bench=.* 会运行所有的基准测试函数；代码中的函数会被调用 N 次（N是非常大的数，如 N =1000000），并展示 N 的值和函数执行的平均时间，单位为ns（纳秒，ns/op）。如果是用testing.Benchmark 调用这些函数，直接运行程序即可。

### 14.8 测试的具体例子

写了一个叫 main_oddeven.go 的程序用来测试前 100 个整数是否是偶数。这个函数属于 even 包。

下面是一种可能的方案：
示例even_main.go：

```go
   package main    

   import (
       "fmt"
       "even/even"
    )    

   func main() {
       for i:=0; i<=100; i++ {
           fmt.Printf("Is the integer %d even? %v\n", i, even.Even(i))
       }
    }
```

上面使用了 even.go 中的even 包：

示例even/even.go：

```go
   package even    

   func Even(i int) bool {        //Exported function
       return i%2 == 0
   }    

   func Odd(i int) bool {        //Exported function
       return i%2 != 0
   }
```

在 even 包的路径下，创建一个名为 oddeven_test.go 的测试程序：

示例even/oddeven_test.go：

```go
   package even    

   import "testing"    

   func TestEven(t *testing.T) {
       if !Even(10) {
           t.Log(" 10 must be even!")
           t.Fail()
       }

       if Even(7) {
           t.Log(" 7 is not even!")
           t.Fail()
       }    

    }    

   func TestOdd(t *testing.T) {
       if !Odd(11) {
           t.Log(" 11 must be odd!")
           t.Fail()
       }
       if Odd(10) {
           t.Log(" 10 is not odd!")
           t.Fail()
       }

    }
```

由于测试需要具体的输入用例且不可能测试到所有的用例（非常像一个无穷的数），所以必须对要使用的测试用例思考再三。

至少应该包括：

- 正常的用例

- 反面的用例（错误的输入，如用负数或字母代替数字，没有输入等）

- 边界检查用例（如果参数的取值范围是0 到 1000，检查 0 和 1000 的情况）

可以直接执行 go install 安装 even 或者创建一个 以下内容的 Makefile：

```makefile
   include $(GOROOT)/src/Make.inc

   TARG=even

   GOFILES=\
          even.go\

   include $(GOROOT)/src/Make.pkg
```

然后执行 make（或 gomake）命令来构建归档文件 even.a

测试代码不能在 GOFILES 参数中引用，因为不希望生成的程序中有测试代码。如果包含了测试代码，gotest 会给出错误提示！go test 会生成一个单独的包含测试代码的 _test 程序。

现在可以用命令：go test（或 make test）来测试 even 包。

因为示例中的测试函数不会调用 t.Log 和 t.Fail，所以会得到一个 PASS 的结果。在这个简单例子中一切都正常执行。

为了看到失败时的输出，把函数 TestEven 改为：

```go
   func TestEven(t *testing.T) {
       if Even(10) {
           t.Log(“Everything OK: 10 is even, just a test tosee failed 
                                                        output!”)
           t.Fail()
        }
    }
```

现在会调用 t.Log 和 t.Fail，得到的结果如下：

```go
   --- FAIL: even.TestEven (0.00 seconds)

   Everything OK: 10 is even, just a test to see failed output!

   FAIL
```

### 14.9 用（测试数据）表驱动测试

编写测试代码时，一个较好的办法是把测试的输入数据和期望的结果写在一起组成一个数据表：表中的每条记录都是一个含有输入和期望值的完整测试用例，有时还可以结合像测试名字这样的额外信息来让测试输出更多的信息。

实际测试时简单迭代表中的每条记录，并执行必要的测试。

可以抽象为下面的代码段：

```go
   var tests = []struct{     // Testtable
       in string
       out string    

   }{
       {“in1”, “exp1”},
       {“in2”, “exp2”},
       {“in3”, “exp3”},
   ...
  } 

   func TestFunction(t *testing.T) {
       for i, tt := range tests {
           s := FuncToBeTested(tt.in)
           if s != tt.out {
                t.Errorf(“%d. %q => %q, wanted: %q”, i, tt.in, s,tt.out)
           }
       }
   }
```

如果大部分函数都可以写成这种形式，那么写一个帮助函数 verify 对实际测试会很有帮助：

```go
   func verify(t *testing.T, testnum int, testcase, input, output, expectedstring) {
       if input != output {
           t.Errorf(“%d. %s with input = %s: output %s !=%s”, testnum, testcase, input, output, expected)
       }
    }
```

   TestFunction则变为：

```go
   func TestFunction(t *testing.T) {
       for i, tt := range tests {
           s := FuncToBeTested(tt.in)
           verify(t, i, “FuncToBeTested: “, tt.in, s, tt.out)
       }
   }
```

### 14.10 性能调试：分析并优化 Go 程序
#### 14.10.1 时间和内存消耗

可以用这个便捷脚本 xtime 来测量：

```shell
   #!/bin/sh

   /usr/bin/time -f ‘%Uu %Ss %er %MkB %C’ “$@”
```

在Unix 命令行中像这样使用 xtime goprogexec，这里的 progexec 是一个 Go 可执行程序，这句命令行输出类似：56.63u 0.26s 56.92r 1642640kB progexec，分别对应用户时间，系统时间，实际时间和最大内存占用。

#### 14.10.2 用 go test 调试

如果代码使用了 Go 中 testing 包的基准测试功能，可以用 gotest 标准的 -cpuprofile 和 -memprofile 标志向指定文件写入 CPU 或 内存使用情况报告。

使用方式：go test -x -v -cpuprofile=prof.out -file x_test.go

编译执行 x_test.go 中的测试，并向 prof.out 文件中写入 cpu 性能分析信息。

#### 14.10.3 用 pprof 调试

可以在单机程序 progexec 中引入 runtime/pprof 包；这个包以 pprof 可视化工具需要的格式写入运行时报告数据。对于 CPU 性能分析来说需要添加一些代码：

```go
   var cpuprofile = flag.String(“cpuprofile”, “”, “write cpuprofile to file”)

   func main() {
       flag.Parse()
       if *cpuprofile != “” {
           f, err := os.Create(*cpuprofile)
           if err != nil {
                log.Fatal(err)
           }

           pprof.StartCPUProfile(f)
           defer pprof.StopCPUProfile()
       }
   ...
```

代码定义了一个名为 cpuprofile 的 flag，调用 Go flag 库来解析命令行 flag，如果命令行设置了 cpuprofile flag，则开始 CPU 性能分析并把结果重定向到那个文件。（os.Create 用拿到的名字创建了用来写入分析数据的文件）。这个分析程序最后需要在程序退出之前调用 StopCPUProfile 来刷新挂起的写操作到文件中；用 defer 来保证这一切会在 main 返回时触发。


现在用这个 flag 运行程序：progexec-cpuprofile=progexec.prof

然后可以像这样用 gopprof 工具：gopprof progexec progexec.prof

gopprof程序是 Google pprofC++ 分析器的一个轻微变种；关于此工具更多的信息，参见http://code.google.com/p/google-perftools/。

如果开启了 CPU 性能分析，Go 程序会以大约每秒 100 次的频率阻塞，并记录当前执行的 goroutine 栈上的程序计数器样本。

此工具一些有趣的命令：

1）topN
用来展示分析结果中最开头的 N 份样本，例如：top5 它会展示在程序运行期间调用最频繁的 5 个函数，输出如下：

```shell
   Total: 3099 samples
   626 20.2% 20.2% 626 20.2% scanblock
   309 10.0% 30.2% 2839 91.6% main.FindLoops
   ...
```

第 5 列表示函数的调用频度。

2）web 或 web 函数名
该命令生成一份 SVG 格式的分析数据图表，并在网络浏览器中打开它（还有一个 gv 命令可以生成 PostScript 格式的数据，并在 GhostView 中打开，这个命令需要安装 graphviz）。函数被表示成不同的矩形（被调用越多，矩形越大），箭头指示函数调用链。

3）list 函数名 或 weblist 函数名

展示对应函数名的代码行列表，第 2 列表示当前行执行消耗的时间，这样就很好地指出了运行过程中消耗最大的代码。

如果发现函数 runtime.mallocgc（分配内存并执行周期性的垃圾回收）调用频繁，那么是应该进行内存分析的时候了。找出垃圾回收频繁执行的原因，和内存大量分配的根源。

为了做到这一点必须在合适的地方添加下面的代码：

```go
   var memprofile = flag.String(“memprofile”, “”, “write memoryprofile to this file”)
   ...

   CallToFunctionWhichAllocatesLotsOfMemory()
   if *memprofile != “” {
       f, err := os.Create(*memprofile)
       if err != nil {
           log.Fatal(err)
       }
       pprof.WriteHeapProfile(f)
       f.Close()
      return
    }
```

用 -memprofile flag 运行这个程序：progexec-memprofile=progexec.mprof

然后可以像这样再次使用 gopprof 工具：gopprof progexec progexec.mprof

top5，list 函数名 等命令同样适用，只不过现在是以 Mb 为单位测量内存分配情况，这是 top 命令输出的例子：

```shell
   Total: 118.3 MB
       66.1 55.8% 55.8% 103.7 87.7% main.FindLoops
       30.5 25.8% 81.6% 30.5 25.8% main.*LSG·NewLoop
       ...
```

从第 1 列可以看出，最上面的函数占用了最多的内存。

同样有一个报告内存分配计数的有趣工具：

```shell
   gopprof --inuse_objects progexec progexec.mprof
```

对于 web 应用来说，有标准的 HTTP 接口可以分析数据。在 HTTP 服务中添加

```go
   import _ “http/pprof”
```

会为 /debug/pprof/ 下的一些 URL 安装处理器。然后可以用一个唯一的参数——服务中的分析数据的 URL 来执行 gopprof 命令——它会下载并执行在线分析。

```shell
   gopprof http://localhost:6060/debug/pprof/profile # 30-second CPUprofile

   gopprof http://localhost:6060/debug/pprof/heap # heap profile
```


## 15 协程与通道

作为一门21世纪的语言，Go原生支持应用之间的通信和程序的并发。程序可以在不同的处理器和计算机上同时执行不同的代码段。Go语言为构建并发程序的基本代码块是 协程(goroutine)与通道(channel)。它们需要语言、编译器和runtime的支持。Go语言提供的垃圾回收器对并发编程至关重要。

不要通过共享内存来通信，而通过通信来共享内存。

通信强制协作。

### 15.1 并发、并行和协程

#### 15.1.1 什么是协程

一个应用程序是运行在机器上的一个进程。进程是一个运行在自己内存地址空间里的独立执行体。一个进程由一个或多个操作系统线程组成，这些线程其实是共享同一个内存地址空间的一起工作的执行体。几乎所有"正式"的程序都是多线程的，以便让用户或计算机不必等待，或者能够同时服务多个请求（如Web服务器），或增加性能和吞吐量（例如，通过对不同的数据集并行执行代码）。一个并发程序可以在一个处理器或者内核上使用多个线程来执行任务，但是只有在同一个程序在某一个时间点在多个些处理内核或处理器上同时执行的任务才是真正的并行。

并行是一种通过使用多处理器以提高速度的能力。所以并发程序可以是并行的，也可以不是。

公认的，使用多线程的应用难以做到准确，最主要的问题是内存中的数据共享，它们会被多线程以无法预知的方式进行操作，导致一些无法重现或者随机的结果（称作竞态）。

不要使用全局变量或者共享内存，它们会给代码在并发运算的时候带来危险。解决之道在于同步不同的线程，对数据加锁，这样同时就只有一个线程可以变更数据。在Go 的标准库sync中有一些工具用来在低级别的代码中实现加锁。不过过去的软件开发经验告诉这会带来更高的复杂度，更容易使代码出错以及更低的性能，所以这个经典的方法明显不再适合现代多核/多处理器编程：thread-per-connection模型不够有效。

Go更倾向于其它的方式，在诸多比较合适的范式中，有个被称作Communicating SequentialProcesses（顺序通信处理），还有一个叫做 message passing-model（消息传递）（已经运用在了其它语言中，比如Eralng）。

在Go中，应用程序并发处理的部分被称作goroutines（协程），它可以进行更有效的并发运算。在协程和操作系统线程之间并无一对一的关系：协程是根据一个或多个线程的可用性，映射（多路复用，执行于）在它们之上的；协程调度器在Go运行时很好的完成了这个工作。

协程工作在相同的地址空间中，所以共享内存的方式一定是同步的；这个可以使用sync包来实现，不过很不鼓励这样做：Go使用channels来同步协程。

当系统调用（比如等待I/O）阻塞协程时，其它协程会继续在其它线程上工作。协程的设计隐藏了许多线程创建和管理方面的复杂工作。

协程是轻量的，比线程更轻。它们痕迹非常不明显（使用少量的内存和资源）：使用4K的栈内存就可以在堆中创建它们。因为创建非常廉价，必要的时候可以轻松创建并运行大量的协程（在同一个一个地址空间中100,000个连续的协程）。并且它们对栈进行了分割，从而动态的增加（或缩减）内存的使用；栈的管理是自动的，但不是由垃圾回收器管理的，而是在协程退出后自动释放。

协程可以运行在多个操作系统线程之间，也可以运行在线程之内，让可以很小的内存占用就可以处理大量的任务。由于操作系统线程上的协程时间片，可以使用少量的操作系统线程就能拥有任意多个提供服务的协程，而且Go运行时可以聪明的意识到哪些协程被阻塞了，暂时搁置它们并处理其它协程。

存在两种并发方式：确定性的（明确定义排序）和非确定性的（加锁/互斥从而未定义排序）。Go的协程和通道理所当然的支持确定性的并发方式（例如通道具有一个 sender和一个receiver）。

协程是通过使用关键字go调用（执行）一个函数或者方法来实现的（也可以是匿名或者lambda函数）。这样会在当前的计算过程中开始一个同时进行的函数，在相同的地址空间中并且分配了独立的栈，比如：go sum(bigArray)，在后台计算总和。

协程的栈会根据需要进行伸缩，不出现栈溢出；开发者不需要关心栈的大小。当协程结束的时候，它会静默退出：用来启动这个协程的函数不会得到任何的返回值。

任何Go程序都必须有的main()函数也可以看做是一个协程，尽管它并没有通过go来启动。协程可以在程序初始化的过程中运行（在init()函数中）。

在一个协程中，比如它需要进行非常密集的运算，可以在运算循环中周期的使用runtime.Gosched()：这会让出处理器，允许运行其它协程；它并不会使当前协程挂起，所以它会自动恢复执行。使用Gosched()可以使计算均匀分布，使通信不至于迟迟得不到响应。

#### 15.1.2 并发和并行的差异

Go的并发原语提供了良好的并发设计基础：表达程序结构以便表示独立地执行的动作；所以Go的的重点不在于并行的首要位置：并发程序可能是并行的，也可能不是。并行是一种通过使用多处理器以提高速度的能力。但往往是，一个设计良好的并发程序在并行方面的表现也非常出色。

在2012年1月的运行时实现中，Go默认没有并行指令，只有一个独立的核心或处理器被专门用于Go程序，不论它启动了多少个协程；所以这些协程是并发运行的，但它们不是并行运行的：同一时间只有一个协程会处在运行状态。

这个情况在以后可能会发生改变，不过届时，为了使的程序可以使用多个核心运行，这时协程就真正的是并行运行了，必须使用GOMAXPROCS变量。这会告诉运行时有多少个协程同时执行。

并且只有gc编译器真正实现了协程，适当的把协程映射到操作系统线程。使用gccgo编译器，会为每一个协程创建操作系统线程。

#### 15.1.3 使用GOMAXPROCS

在gc编译器下（6g或者8g）必须设置GOMAXPROCS为一个大于默认值1的数值来允许运行时支持使用多于1个的操作系统线程，所有的协程都会共享同一个线程除非将 GOMAXPROCS设置为一个大于1的数。当GOMAXPROCS大于1时，会有一个线程池管理许多的线程。通过gccgo编译器GOMAXPROCS有效的与运行中的协程数量相等。假设n是机器上处理器或者核心的数量。如果设置环境变量GOMAXPROCS>=n，或者执行runtime.GOMAXPROCS(n)，接下来协程会被分割（分散）到n个处理器上。更多的处理器并不意味着性能的线性提升。有这样一个经验法则，对于n个核心的情况设置GOMAXPROCS为n-1以获得最佳性能，也同样需要遵守这条规则：协程的数量 > 1 + GOMAXPROCS > 1。

所以如果在某一时间只有一个协程在执行，不要设置GOMAXPROCS！

还有一些通过实验观察到的现象：在一台1颗CPU的笔记本电脑上，增加GOMAXPROCS到9会带来性能提升。在一台32核的机器上，设置GOMAXPROCS=8会达到最好的性能，在测试环境中，更高的数值无法提升性能。如果设置一个很大的GOMAXPROCS只会带来轻微的性能下降；设置GOMAXPROCS=100，使用top命令和H选项查看到只有7个活动的线程。

增加GOMAXPROCS的数值对程序进行并发计算是有好处的；

总结：GOMAXPROCS等同于（并发的）线程数量，在一台核心数多于1个的机器上，会尽可能有等同于核心数的线程在并行运行。

#### 15.1.4 如何用命令行指定使用的核心数量

使用 flags 包，如下：

```go
var numCores = flag.Int("n", 2, "number of CPU cores touse")
```

在main()方法中，

```go
flag.Parse()
runtime.GOMAXPROCS(*numCores)
```

协程可以通过调用runtime.Goexit()来停止，尽管这样做几乎没有必要。

- **程序示例**

```go
// @file:        goroutine1.go
// @version:     1.0
// @author:      haulf
// @date:        2017.11.07
// @go version:  1.9
// @brief:       Goroutine test.

package main

import (
    "fmt"
    "time"
)

func main() {
    fmt.Println("In main()")
    go longWait()
    go shortWait()
    fmt.Println("About to sleep in main()")
    // sleep works with a Duration in nanoseconds (ns) !
    time.Sleep(10 * 1e9)
    fmt.Println("At the end of main()")
}

func longWait() {
    fmt.Println("Beginning longWait()")
    time.Sleep(5 * 1e9) // sleep for 5 seconds
    fmt.Println("End of longWait()")
}

func shortWait() {
    fmt.Println("Beginning shortWait()")
    time.Sleep(2 * 1e9) // sleep for 2 seconds
    fmt.Println("End of shortWait()")
}
```

- **程序运行结果**

```shell
In main()
About to sleep in main()
Beginning shortWait()
Beginning longWait()
End of shortWait()
End of longWait()
At the end of main()
```

重新运行一次：

```shell
In main()
About to sleep in main()
Beginning longWait()
Beginning shortWait()
End of shortWait()
End of longWait()
At the end of main()
```

- **程序说明**

main()、longWait() 和shortWait()三个函数作为独立的处理单元启动后开始并行运行。每一个函数都在运行的开始和结束阶段输出了消息。为了模拟它们运算的时间消耗，使用了time包中的Sleep函数。Sleep()可以按照指定的时间来暂停函数或协程地执行，这里使用了纳秒（ns，符号1e9表示1乘10的9次方，e=指数）。

让main()函数暂停10秒从而确定它会在另外两个协程之后结束。如果不这样（如果让main()函数停止4 秒），main()会提前结束，longWait()则无法完成。如果不在main()中等待，协程会随着程序的结束而消亡。

当main()函数返回的时候，程序退出，它不会等待任何其它非main协程的结束。这就是为什么在服务器程序中，每一个请求都会启动一个协程来处理，server()函数必须保持运行状态。通常使用一个无限循环来达到这样的目的。

另外，协程是独立的处理单元，一旦陆续启动一些协程，无法确定它们是什么时候真正开始执行的。从上面的运行结果来看，不能确longWait()和shortWait()哪一个先开始执行。

为了对比使用一个线程，连续调用的情况，移除go关键字，重新运行程序。

协程更有用的一个例子应该是在一个非常长的数组中查找一个元素。将数组分割为若干个不重复的切片，然后给每一个切片启动一个协程进行查找计算。这样许多并行的线程可以用来进行查找任务，整体的查找时间会缩短（除以协程的数量）。

#### 15.1.5 Go协程（goroutines）和其它语言协程（coroutines）

在其它语言中，比如在C#，Lua或者Python里都有协程的概念。这个名字表明它和Go协程有些相似，不过有两点不同：

- Go协程意味着并行（或者可以以并行的方式部署），其它语言中的协程一般来说不是这样的。
- Go协程通过通道来通信；其它语言中的协程通过让出和恢复操作来通信

Go协程比其它语言中的协程更强大，也很容易从其它语言中协程的逻辑复用到Go协程。

### 15.2 协程间的信道

#### 15.2.1 概念

在上面的例子中，协程是独立执行的，它们之间没有通信。它们必须通信才会变得更有用：彼此之间发送和接收信息并且协调/同步它们的工作。协程可以使用共享变量来通信，但是很不提倡这样做，因为这种方式给所有的共享内存的多线程都带来了困难。

Go有一个特殊的类型，通道（channel），可以通过它们发送类型化的数据在协程之间通信，可以避开所有内存共享导致的坑。通道的通信方式保证了同步性。数据通过通道，同一时间只有一个协程可以访问数据，所以不会出现数据竞争。数据的归属（可以读写数据的能力）被传递。

工厂的传送带是个很有用的例子。一个机器（生产者协程）在传送带上放置物品，另外一个机器（消费者协程）拿到物品并打包。

通道服务于通信的两个目的：值的交换，同步的，保证了两个计算（协程）任何时候都是可知状态。

通常使用这样的格式来声明通道：

```go
var identifier chan datatype
```

未初始化的通道的值是nil。

通道只能传输一种类型的数据，比如chan int 或者chan string，所有的类型都可以用于通道，空接口interface{}也可以，甚至可以创建通道的通道。

通道实际上是类型化消息的队列：使数据得以传输。它是先进先出（FIFO）结构，所以可以保证发送给它们的元素的顺序。通道也是引用类型，可以使用make()函数来给它分配内存。这里先声明了一个字符串通道ch1，然后创建了它（实例化）：

```go
   var ch1 chan string
   ch1 = make(chan string)
```

当然可以更短： 

```go
ch1 := make(chan string)
```

这里构建一个int通道的通道： 

```go
chanOfChans := make(chan chan int)
```

或者函数通道：

```go
funcChan := chan func()。
```

所以通道是对象的第一类型：可以存储在变量中，作为函数的参数传递，从函数返回以及通过通道发送它们自身。另外它们是类型化的，允许类型检查，比如尝试使用整数通道发送一个指针。

#### 15.2.2 通信操作符<-

这个操作符直观的标示了数据的传输：信息按照箭头的方向流动。

- 对象进入通道（发送）

`ch <- int1` 表示：用通道ch发送变量int1(把变量int1放入到通道ch中进行发送)

- 从通道中取出数据（接收），三种方式：

`int2 = <- ch` 表示：从通道ch中取出新的值赋给变量int2，这里是假设int2已经声明过了。如果没有的话可以写成：int2 := <- ch。

`<- ch` 可以单独调用获取通道的下一个值，当前值会被丢弃，但是可以用来验证，所以以下代码是合法的：

```go
   if <- ch != 1000{
       ...
    }
```

操作符` <-`也被用来发送和接收，Go 尽管不必要，为了可读性，通道的命名通常以ch开头或者包含chan。通道的发送和接收操作都是自动的：它们通常一气呵成。下面的示例展示了通信操作。



- **程序示例**

```go
// @file:        goroutine2.go
// @version:     1.0
// @author:      haulf
// @date:        2017.11.08
// @go version:  1.9
// @brief:       Goroutine test.

package main

import (
    "fmt"
    "time"
)

func main() {
    ch := make(chan string)
    go sendData(ch)
    go getData(ch)
    time.Sleep(1e9)
}

func sendData(ch chan string) {
    ch <- "Washington"
    ch <- "Tripoli"
    ch <- "London"
    ch <- "Beijing"
    ch <- "Tokio"
}

func getData(ch chan string) {
    var input string
    // time.Sleep(1e9)
    for {
        input = <-ch
        fmt.Printf("%s ", input)
    }
}
```

- **程序运行结果**

```shell
Washington Tripoli London Beijing Tokio %

```

- **程序说明**

main()函数中启动了两个协程：sendData()通过通道ch发送了5个字符串，getData()按顺序接收它们并打印出来。

如果两个协程之间需要通信，必须给它们同一个通道作为参数。

如果注释掉main()函数中的time.Sleep(1e9)，则没有数据打印出来。

发现协程之间的同步非常重要：

- main()等待了1秒让两个协程完成，如果不这样，sendData()发送的数据就没有机会输出。

- getData()使用了无限循环，它随着sendData()的发送完成和ch变空也结束了。

- 如果移除一个或所有go关键字，程序无法运行，Go运行时会抛出panic：

```shell
Washington Tripoli London Beijing Tokio fatal error: all goroutines are asleep - deadlock!

goroutine 1 [chan receive]:
main.getData(0xc42006c060)
    /Users/aihaofeng/Documents/go_v20171107/study_manal/goroutine/goroutine2.go:34 +0x4e
main.main()
    /Users/aihaofeng/Documents/go_v20171107/study_manal/goroutine/goroutine2.go:18 +0x66
```

为什么会这样？运行时会检查所有的协程是否在等待（可以读取或者写入某个通道），意味着程序无法处理，这是死锁（deadlock）形式。不要使用打印状态来表明通道的发送和接收顺序，因为由于打印状态和通道实际发生读写的时间延迟会导致和真实发生的顺序不同。

#### 15.2.3 通道阻塞

默认情况下，通信是同步且无缓冲的，在有接受者接收数据之前，发送不会结束。可以想象一个无缓冲的通道在没有空间来保存数据的时候，必须要一个接收者准备好接收通道的数据然后发送者可以直接把数据发送给接收者，所以通道的发送/接收操作在对方准备好之前是阻塞的。

1）对于同一个通道，发送操作（协程或者函数中的），在接收者准备好之前是阻塞的。如果通道中的数据无人接收，就无法再给通道传入其它数据，新的输入无法在通道非空的情况下传入，所以发送操作会等待通道再次变为可用状态，就是通道值被接收时。

2）对于同一个通道，接收操作是阻塞的（协程或函数中的），直到发送者可用。如果通道中没有数据，接收者就阻塞了。

- **程序示例**

```go
// @file:        channel_block.go
// @version:     1.0
// @author:      haulf
// @date:        2017.11.08
// @go version:  1.9
// @brief:       Goroutine test.

package main

import "fmt"

func main() {
    ch1 := make(chan int)
    go pump(ch1)       // pump hangs
    fmt.Println(<-ch1) // prints only 0
}

func pump(ch chan int) {
    for i := 0; ; i++ {
        ch <- i
    }
}
```



- **程序运行结果**

0



- **程序说明**

在上面的程序中，一个协程在无限循环中给通道发送整数数据。不过因为没有接收者，只输出了一个数字0。pump()函数为通道提供数值，也被叫做生产者。

为通道解除阻塞，定义了suck()函数来在无限循环中读取通道。

- **程序示例**

```go
// @file:        channel_block.go
// @version:     1.0
// @author:      haulf
// @date:        2017.11.08
// @go version:  1.9
// @brief:       Goroutine test.

package main

import (
    "fmt"
    "time"
)

func main() {
    ch1 := make(chan int)
    go pump(ch1) // pump hangs
    go suck(ch1)
    time.Sleep(1e9)
    fmt.Println(<-ch1) // prints only 0
}

func pump(ch chan int) {
    for i := 0; ; i++ {
        ch <- i
    }
}

func suck(ch chan int) {
    for {
        fmt.Println(<-ch)
    }
}
```

- **程序运行结果**

```shell
....
220758
220759
220760
220761
220762
220764
220763
```

- **程序说明**

给程序1秒的时间来运行，这期间输出了上万个整数。



#### 15.2.4 通过一个或多个通道交换数据进行协程同步

通信是一种同步形式。通过通道，两个协程在通信中某刻同步交换数据。无缓冲通道成为了多个协程同步的完美工具。甚至可以在通道两端互相阻塞对方，形成了叫做死锁的状态。Go运行时会检查并panic，停止程序。死锁几乎完全是由糟糕的设计导致的。无缓冲通道会被阻塞。设计无阻塞的程序可以避免这种情况，或者使用带缓冲的通道。

blocking.go

解释为什么下边这个程序会导致panic：所有的协程都休眠了 - 死锁！

```go
   package main   

   import (
       "fmt"
    )
    

   func f1(in chan int) {
       fmt.Println(<-in)
    }
    

   func main() {
       out := make(chan int)
       out <- 2
       go f1(out)
    }
```

上面程序中的out是一个无缓冲的通道。只有一个发送协程，即只有生产者，没有消费者，因此会出现阻塞，这就是列死锁情况。



#### 15.2.5 同步通道-使用带缓冲的通道

一个无缓冲通道只能包含1个元素，有时显得很局限。给通道提供了一个缓存，可以在扩展的make命令中设置它的容量，如下：

```go
buf := 100
ch1 := make(chan string, buf)  // buf是通道可以同时容纳的元素（这里是string）个数
```

在缓冲被全部使用之前，给一个带缓冲的通道发送数据是不会阻塞的，而从通道读取数据也不会阻塞，直到缓冲空了。缓冲容量和类型无关，可以给一些通道设置不同的容量，只要它们拥有同样的元素类型。内置的cap函数可以返回缓冲区的容量。如果容量大于0，通道就是异步的了：缓冲满载（发送）或变空（接收）之前通信不会阻塞，元素会按照发送的顺序被接收。如果容量是0或者未设置容量，通信仅在收发双方准备好的情况下才可以成功。

```go
ch := make(chan type, value)
- value == 0 // synchronous, unbuffered(阻塞）
- value > 0  // asynchronous, buffered（非阻塞）取决于value元素
```

#### 15.2.6 在协程中用通道输出结果

为了知道计算何时完成，可以通过信道回报。

```go
   ch := make(chan int)
   go sum(bigArray, ch) // bigArray puts the calculated sum on ch
   // .. do something else for a while
   sum := <- ch // wait for, and retrieve the sum
```

也可以使用通道来达到同步的目的，这个很有效的用法在传统计算机中称为信号量（semaphore）。或者换个方式，通过通道发送信号告知处理已经完成。在其它协程运行时让main程序无限阻塞的通常做法是在main函数的最后放置一个{}，也可以使用通道让main程序等待协程完成，就是所谓的信号量模式。

#### 15.2.7 信号量模式

下边的代码片段中，协程通过在通道ch中放置一个值来处理结束的信号。main协程等待<-ch直到从中获取到值。

期望从这个通道中获取返回的结果，像这样：

```go
   func compute(ch chan int){
       // when it completes, signal on the channel.
       ch <- someComputation() 
    }

   func main(){
       ch := make(chan int)         // allocate a channel.
       go compute(ch)               // stat something in a goroutines
       doSomethingElseForAWhile()
       result := <- ch
    }
```

这个信号也可以是其它的，不返回结果，比如下面这个协程中的匿名函数（lambda）协程：

```go
   ch := make(chan int)

   go func(){
       // doSomething
       ch <- 1 // Send a signal; value does not matter
    }

   doSomethingElseForAWhile()
   <- ch    // Wait for goroutineto finish; discard sent value.
```

或者等待两个协程完成，每一个都会对切片s的一部分进行排序，片段如下：

```go
   done := make(chan bool)

   // doSort is a lambda function, so a closure which knows the channel done:
   doSort := func(s []int){
       sort(s)
       done <- true
    }

    i:= pivot(s)

   go doSort(s[:i])
   go doSort(s[i:])
   <-done
   <-done
```

下边的代码用完整的信号量模式对长度为N的float64切片进行了N个doSomething()计算并同时完成，通道sem分配了相同的长度（包含空接口类型的元素），等待所有的计算都完成后，发送信号（通过放入值）。在循环中从通道sem不停的取出数据来等待所有的协程完成。

```go
   type Empty interface {}
   var empty Empty
   ...
   data := make([]float64, N)
   res := make([]float64, N)

   sem := make(chan Empty, N)
   ...
   for i, xi := range data {
       go func (i int, xi float64) {
           res[i] = doSomething(i, xi)
           sem <- empty
       } (i, xi)
    }

   // wait for goroutines to finish
   for i := 0; i < N; i++ { <-sem }
```

i、xi都是作为参数传入闭合函数的，从外层循环中隐藏了变量i和xi。让每个协程有一份i和xi的拷贝；另外，for循环的下一次迭代会更新所有协程中i和xi的值。切片 res没有传入闭合函数，因为协程不需要单独拷贝一份。切片res也在闭合函数中但并不是参数。

#### 15.2.8 实现并行的for循环

在之前的代码片段中，for循环的每一个迭代是并行完成的：

```go
   for i, v := range data {
       go func (i int, v float64) {
           doSomething(i, v)
           ...
       } (i, v)
    }
```

在for循环中并行计算迭代可能带来很好的性能提升。不过所有的迭代都必须是独立完成的。

#### 15.2.9 用带缓冲通道实现一个信号量

信号量是实现互斥锁常见的同步机制，它限制对资源的访问，解决读写问题。比如没有实现信号量的sync的Go包，使用带缓冲的通道可以轻松实现：

- 带缓冲通道的容量和要同步的资源容量相同。
- 通道的长度（当前存放的元素个数）与当前资源被使用的数量相同。
- 容量减去通道的长度就是未处理的资源个数（标准信号量的整数值）。

不用管通道中存放的是什么，只关注长度。因此创建了一个长度可变但容量为0（字节）的通道：

```go
   type Empty interface {}
   type semaphore chan Empty
```

将可用资源的数量N来初始化信号量 

```go
  semaphore：sem = make(semaphore, N)
```

然后直接对信号量进行操作：

```go
   // acquire n resources
   func (s semaphore) P(n int) {
       e := new(Empty)

       for i := 0; i < n; i++ {
           s <- e
       }
    }    

   // release n resouces
   func (s semaphore) V(n int) {
       for i:= 0; i < n; i++{
           <- s
       }
    }
```

可以用来实现一个互斥的例子：

```go
   /* mutexes */
   func (s semaphore) Lock() {
       s.P(1)
    }

   func (s semaphore) Unlock(){
       s.V(1)
    }
    
   /* signal-wait */
   func (s semaphore) Wait(n int) {
       s.P(n)
    }   

   func (s semaphore) Signal() {
       s.V(1)
    }
```



**习惯用法：通道工厂模式**

编程中常见的另外一种模式如下：不将通道作为参数传递给协程，而用函数来生成一个通道并返回（工厂角色）；函数内有个匿名函数被协程调用。

channel_idiom.go：

```go
   package main

   import (
       "fmt"
       "time"
    )   

   func main() {
       stream := pump()
       go suck(stream)
       time.Sleep(1e9)
    }   

   func pump() chan int {
       ch := make(chan int)

       go func() {
           for i := 0; ; i++ {
                ch <- i
           }
       }()

       return ch
    }

   func suck(ch chan int) {
       for {
           fmt.Println(<-ch)
       }
    }
```

#### 15.2.10 给通道使用for循环

for循环的range语句可以用在通道ch上，便可以从通道中获取值，像这样：

```go
   for v := range ch {
       fmt.Printf("The value is %v\n", v)
   }
```

它从指定通道中读取数据直到通道关闭，才继续执行下边的代码。很明显，另外一个协程必须写入数据ch（不然代码就阻塞在for循环了），而且必须在写入完成后才关闭。suck函数可以这样写，且在协程中调用这个动作，程序变成了这样：

示例channel_idiom2.go：

```go
   package main

   import (
       "fmt"
       "time"
    )

   func main() {
       suck(pump())
       time.Sleep(1e9)
    } 

   func pump() chan int {
       ch := make(chan int)

       go func() {
           for i := 0; ; i++ {
                ch <- i
           }
       }()
       return ch
    }

   func suck(ch chan int) {
       go func() {
           for v := range ch {
                fmt.Println(v)
           }
       }()
    }
```

**习惯用法：通道迭代模式**

这个模式用到了前边示例中的模式，通常，需要从包含了地址索引字段items的容器给通道填入元素。为容器的类型定义一个方法Iter()，返回一个只读的通道items，如下：

```go
   func (c *container) Iter () <- chan items {
       ch := make(chan item)

       go func () {
           for i:= 0; i < c.Len(); i++ {   // or use a for-range loop
                ch <- c.items[i]
           }
       } ()

       return ch
    }
```

在协程里，一个for循环迭代容器c中的元素（对于树或图的算法，这种简单的for循环可以替换为深度优先搜索）。

调用这个方法的代码可以这样迭代容器：

```go
   for x := range container.Iter() { ... }
```

可以运行在自己的协程中，所以上边的迭代用到了一个通道和两个协程（可能运行在两个线程上）。就有了一个特殊的生产者-消费者模式。如果程序在协程给通道写完值之前结束，协程不会被回收；设计如此。这种行为看起来是错误的，但是通道是一种线程安全的通信。在这种情况下，协程尝试写入一个通道，而这个通道永远不会被读取，这可能是个bug而并非期望它被静默的回收。


**习惯用法：生产者消费者模式**

假设有Produce()函数来产生Consume函数需要的值。它们都可以运行在独立的协程中，生产者在通道中放入给消费者读取的值。整个处理过程可以替换为无限循环：

```go
   for {
       Consume(Produce())
    }
```

#### 15.2.11 通道的方向

通道类型可以用注解来表示它只发送或者只接收：

```go
   var send_only chan<- int        // channel can only receive data
   var recv_only <-chan int       // channel can onley send data
```

只接收的通道（<-chan T）无法关闭，因为关闭通道是发送者用来表示不再给通道发送值了，所以对只接收通道是没有意义的。通道创建的时候都是双向的，但也可以分配有方向的通道变量，就像以下代码：

```go
   var c = make(chan int) // bidirectional

   go source(c)
   go sink(c)   

   func source(ch chan<- int){
       for { ch <- 1 }
   }    

   func sink(ch <-chan int) {
       for { <-ch }
   }
```


**习惯用法：管道和选择器模式**

更具体的例子还有协程处理它从通道接收的数据并发送给输出通道：

```go
   sendChan := make(chan int)
   reciveChan := make(chan string)
   go processChannel(sendChan, receiveChan)

   func processChannel(in <-chan int, out chan<- string) {
       for inValue := range in {
           result := ... /// processing inValue
           out <- result
       }
   }
```

通过使用方向注解来限制协程对通道的操作。

这里有一个例子，打印了输出的素数，使用选择器作为它的算法。每个 prime 都有一个选择器。
 
版本1：示例14.7-sieve1.go

```go
// Copyright 2009 The Go Authors. All rights reserved.
// Use of this source code is governed by a BSD-style
// license that can be found in the LICENSE file.package main

package main

import "fmt"

// Send the sequence 2, 3, 4, ... to channel 'ch'.
func generate(ch chan int) {
    for i := 2; ; i++ {
        ch <- i // Send 'i' to channel 'ch'.
    }
}

// Copy the values from channel 'in' to channel 'out',
// removing those divisible by 'prime'.
func filter(in, out chan int, prime int) {
    for {
        i := <-in // Receive value of new variable 'i' from 'in'.
        if i%prime != 0 {
            out <- i // Send 'i' tochannel 'out'.
        }
    }
}

// The prime sieve: Daisy-chain filter processes together.
func main() {
    ch := make(chan int) // Create a new channel.
    go generate(ch)      // Startgenerate() as a goroutine.
    for {
        prime := <-ch
        fmt.Print(prime, " ")
        ch1 := make(chan int)
        go filter(ch, ch1, prime)
        ch = ch1
    }
}
```

协程 filter(in, out chan int, prime int) 拷贝整数到输出通道，丢弃掉可以被prime整除的数字，然后每个prime又开启了一个新的协程，生成器和选择器并发请求。

第二个版本引入了上边的习惯用法：函数sieve、generate和filter都是工厂；它们创建通道并返回，而且使用了协程的lambda函数。main函数现在短小清晰：它调用 sieve() 返回了包含素数的通道，然后通过 fmt.Println(<-primes) 打印出来。


版本2：示例sieve2.go

```go
// Copyright 2009 The Go Authors. All rights reserved.
// Use of this source code is governed by a BSD-style
// license that can be found in the LICENSE file.

package main

import (
    "fmt"
)

// Send the sequence 2, 3, 4, ... to returned channel
func generate() chan int {
    ch := make(chan int)

    go func() {
        for i := 2; ; i++ {
            ch <- i
        }
    }()

    return ch
}

// Filter out input values divisible by 'prime', send rest to returnedchannel
func filter(in chan int, prime int) chan int {
    out := make(chan int)

    go func() {
        for {
            if i := <-in; i%prime != 0 {
                out <- i
            }
        }
    }()

    return out
}

func sieve() chan int {
    out := make(chan int)

    go func() {
        ch := generate()

        for {
            prime := <-ch
            ch = filter(ch, prime)
            out <- prime
        }
    }()

    return out
}

func main() {
    primes := sieve()

    for {
        fmt.Println(<-primes)
    }
}
```

### 15.3 协程同步：关闭通道-测试阻塞的通道

通道可以被显示的关闭。它们和文件不同，不必每次都关闭。只有在当需要告诉接收者不会再提供新值的时候，才需要关闭通道。发送者需要关闭通道，接收者不会需要。

继续看示例goroutine2.go：

(1) 如何在通道的sendData()完成的时候发送一个信号?

(2) getData()又如何检测到通道是否关闭或阻塞？

第一个可以通过函数close(ch)来完成：这个将通道标记为无法通过发送操作<-接受更多的值；给已经关闭的通道发送或者再次关闭都会导致运行时的panic。在创建一个通道后使用defer语句是个不错的办法（类似这种情况）：

```go
   ch := make(chan float64)
   defer close(ch)
```

第二个问题可以使用逗号，ok操作符来检测通道是否被关闭。

如何来检测可以收到没有被阻塞（或者通道没有被关闭）？

```go
   v, ok := <-ch   // ok is trueif v received value
```

通常和 if 语句一起使用：

```go
   if v, ok := <-ch; ok {
       process(v)
   }
```

或者在for循环中接收的时候，当关闭或者阻塞的时候使用break：

```go
   v, ok := <-ch

   if !ok {
       break
   }

   process(v)
```

可以通过 _ = ch <- v 来实现非阻塞发送，因为空标识符获取到了发送给ch的任何东西。在示例程序中使用这些可以改进为版本goroutine3.go，输出相同。

实现非阻塞通道的读取，需要使用select。

示例goroutine3.go：

```go
   package main

   import "fmt"

 
   func main() {
       ch := make(chan string)
       go sendData(ch)
       getData(ch)
   }

    
   func sendData(ch chan string) {
       ch <- "Washington"
       ch <- "Tripoli"
       ch <- "London"
       ch <- "Beijing"
       ch <- "Tokio"
       close(ch)
    }

   func getData(ch chan string) {
       for {
           input, open := <-ch
           if !open {
                break
           }

           fmt.Printf("%s ", input)
       }
    }
```

改变了以下代码：

- 现在只有sendData()是协程，getData()和main()在同一个线程中：
   go sendData(ch)
   getData(ch)

- 在sendData()函数的最后，通过close(ch)关闭了通道。

- 在for循环的getData()中，在每次接收通道的数据之前都使用if !open来检测。使用for-range语句来读取通道是更好的办法，因为这会自动检测通道是否关闭：

```go
   for input := range ch {
       process(input)
   }
```

**阻塞和生产者-消费者模式：**

在之前的通道迭代器中，两个协程经常是一个阻塞另外一个。如果程序工作在多核心的机器上，大部分时间只用到了一个处理器。可以通过使用带缓冲（缓冲空间大于0）的通道来改善。比如，缓冲大小为100，迭代器在阻塞之前，至少可以从容器获得100个元素。如果消费者协程在独立的内核运行，就有可能让协程不会出现阻塞。

由于容器中元素的数量通常是已知的，需要让通道有足够的容量放置所有的元素。这样，迭代器就不会阻塞（尽管消费者协程仍然可能阻塞）。然后，这样有效的加倍了迭代容器所需要的内存使用量，所以通道的容量需要限制一下最大值。记录运行时间和性能测试可以帮助找到最小的缓存容量带来最好的性能。

### 15.4 使用select切换协程

从不同的并发执行的协程中获取值可以通过关键字select来完成，它和switch控制语句非常相似也被称作通信开关；它的行为像是“准备好了吗”的轮询机制；select监听进入通道的数据，也可以是用通道发送值的时候。

```go
   select {
   case u:= <- ch1:
           ...
   case v:= <- ch2:
           ...
           ...
   default: // no value ready to be received
           ...
   }
```

default语句是可选的；fallthrough行为，和普通的switch相似，是不允许的。在任何一个case中执行break或者return，select就结束了。

select做的就是：选择处理列出的多个通信情况中的一个。

- 如果都阻塞了，会等待直到其中一个可以处理。

- 如果多个可以处理，随机选择一个。

- 如果没有通道操作可以处理并且写了default语句，它就会执行：default永远是可运行的（这就是准备好了，可以执行）。

在select中使用发送操作并且有default可以确保发送不被阻塞！如果没有case，select就会一直阻塞。

elect语句实现了一种监听模式，通常用在（无限）循环中；在某种情况下，通过break语句使循环退出。

在程序goroutine_select.go中有2个通道 ch1和ch2，三个协程pump1()、pump2()和suck()。这是一个典型的生产者消费者模式。在无限循环中，ch1和ch2通过 pump1()和pump2()填充整数；suck()也是在无限循环中轮询输入的，通过select语句获取ch1和ch2的整数并输出。选择哪一个case取决于哪一个通道收到了信息。程序在 main执行1秒后结束。

示例goroutine_select.go：

```go
   package main

   import (
       "fmt"
       "time"
    )

   func main() {
       ch1 := make(chan int)
       ch2 := make(chan int)

       go pump1(ch1)
       go pump2(ch2)
       go suck(ch1, ch2)
       time.Sleep(1e9)
    }  

   func pump1(ch chan int) {
       for i := 0; ; i++ {
           ch <- i * 2
       }
    } 

   func pump2(ch chan int) {
       for i := 0; ; i++ {
           ch <- i + 5
       }
    }

   func suck(ch1, ch2 chan int) {
       for {
           select {
           case v := <-ch1:
                fmt.Printf("Received onchannel 1: %d\n", v)
           case v := <-ch2:
                fmt.Printf("Received onchannel 2: %d\n", v)
           }
       }
   }
```
一秒内的输出非常惊人，如果给它计数（goroutine_select2.go），得到了90000个左右的数字。


**习惯用法：后台服务模式**

服务通常是是用后台协程中的无限循环实现的，在循环中使用select获取并处理通道中的数据：

```go
   // Backend goroutine.
   func backend() {
       for {
         select {
           case cmd := <-ch1:
               // Handle ...
           case cmd := <-ch2:
                ...
           case cmd := <-chStop:
                // stop server
           }
       }
    }
```

在程序的其它地方给通道 ch1，ch2 发送数据，比如：通道stop用来清理结束服务程序。

另一种方式（但是不太灵活）就是（客户端）在chRequest上提交请求，后台协程循环这个通道，使用switch根据请求的行为来分别处理。

### 15.5 通道、超时和计时器（Ticker）

time包中有一些有趣的功能可以和通道组合使用。其中就包含了 time.Ticker 结构体，这个对象以指定的时间间隔重复的向通道 C 发送时间值：

```go
   type Ticker struct {
       C <-chan Time // the channel on which the ticks are delivered.
       // contains filtered or unexported fields
       ...
    }
```

时间间隔的单位是ns（纳秒，int64），在工厂函数time.NewTicker中以Duration类型的参数传入：func Newticker(dur) *Ticker。

在协程周期性的执行一些事情（打印状态日志，输出，计算等等）的时候非常有用。

调用Stop()使计时器停止，在defer语句中使用。这些都很好的适应select语句:

```go
   ticker := time.NewTicker(updateInterval)
   defer ticker.Stop()
   ...
   select {
   case u:= <-ch1:
       ...
   case v:= <-ch2:
       ...
   case <-ticker.C:
       logState(status) // call some logging function logState
   default: // no value ready to be received
       ...
   }
```

`time.Tick()`函数声明为`Tick(d Duration) <-chan Time`，当想返回一个通道而不必关闭它的时候这个函数非常有用：它以d为周期给返回的通道发送时间，d是纳秒数。如果需要像下边的代码一样，限制处理频率（函数`client.Call()`是一个RPC调用，这里暂不赘述：

```go
   import "time"   

   rate_per_sec := 10
   var dur Duration = 1e9 / rate_per_sec
   chRate := time.Tick(dur) // a tick every 1/10th of a second

   for req := range requests {
       <- chRate // rate limit our Service.Method RPC calls
       go client.Call("Service.Method", req, ...)
   }
```

这样只会按照指定频率处理请求：chRate阻塞了更高的频率。每秒处理的频率可以根据机器负载（和/或）资源的情况而增加或减少。

定时器（Timer）结构体看上去和计时器（Ticker）结构体的确很像（构造为NewTimer(d Duration)），但是它只发送一次时间，在Drationd之后。

还有time.After(d) 函数，声明如下：

```go
  func After(d Duration) <-chan Time
```

在Duration d之后，当前时间被发到返回的通道；所以它和NewTimer(d).C是等价的；它类似Tick()，但是After()只发送一次时间。下边有个很具体的示例，很好的阐明了select中default的作用：

示例timer_goroutine.go：

```go
   package main

    

   import (

       "fmt"

       "time"

    )

    

   func main() {
       tick := time.Tick(1e8)
       boom := time.After(5e8)

       for {
           select {
           case <-tick:
                fmt.Println("tick.")
           case <-boom:
                fmt.Println("BOOM!")
                return
           default:
                fmt.Println("    .")
                time.Sleep(5e7)
           }
       }
    }
```


**习惯用法：简单超时模式**

要从通道ch中接收数据，但是最多等待1秒。先创建一个信号通道，然后启动一个lambda协程，协程在给通道发送数据之前是休眠的：

```go
   timeout := make(chan bool, 1)

   go func() {
           time.Sleep(1e9) // one second
           timeout <- true
   }()
```

然后使用select语句接收ch或者timeout的数据：如果ch在1秒内没有收到数据，就选择到了time分支并放弃了ch的读取。


第二种形式：取消耗时很长的同步调用

也可以使用time.After()函数替换 timeout-channel。可以在select中使用以发送信号超时或停止协程的执行。以下代码，在 timeoutNs纳秒后执行 select 的 timeout 分支时，client.Call 不会给通道 ch 返回值：

```go
   ch := make(chan error, 1)

   go func() { ch <- client.Call("Service.Method", args,&reply) } ()

   select {
   case resp := <-ch
      // use resp and reply

   case <-time.After(timeoutNs):
       // call timed out
       break
   }
```

注意缓冲大小设置为1是必要的，可以避免协程死锁以及确保超时的通道可以被垃圾回收。

第三种形式：假设程序从多个复制的数据库同时读取。只需要一个答案，需要接收首先到达的答案，Query函数获取数据库的连接切片并请求。并行请求每一个数据库并返回收到的第一个响应：

```go
    func Query(conns []conn, query string) Result {
        ch := make(chan Result, 1)
        for _, conn := range conns {
            go func(c Conn) {
                select {
                case ch <- c.DoQuery(query):
                default:
                }
            }(conn)
        }
        return <- ch
    }
```

再次声明，结果通道ch必须是带缓冲的：以保证第一个发送进来的数据有地方可以存放，确保放入的首个数据总会成功，所以第一个到达的值会被获取而与执行的顺序无关。正在执行的协程可以总是可以使用runtime.Goexit()来停止。
 

**在应用中缓存数据：**

应用程序中用到了来自数据库（或者常见的数据存储）的数据时，经常会把数据缓存到内存中，因为从数据库中获取数据的操作代价很高；如果数据库中的值不发生变化就没有问题。但是如果值有变化，需要一个机制来周期性的从数据库重新读取这些值：缓存的值就不可用（过期）了，而且也不希望用户看到陈旧的数据。

### 15.6 协程和恢复（recover）

一个用到recover的程序停掉了服务器内部一个失败的协程而不影响其它协程的工作。

```go
func server(workChan <-chan *Work) {
    for work := range workChan {
        go safelyDo(work)   // start the goroutine for that work
    }
}

func safelyDo(work *Work) {
    defer func {
        if err := recover(); err != nil {
            log.Printf("Work failed with %s in %v", err, work)
        }
    }()
    do(work)
}
```

上边的代码，如果do(work)发生panic，错误会被记录且协程会退出并释放，而其它协程不受影响。

因为recover总是返回nil，除非直接在defer修饰的函数中调用，defer 修饰的代码可以调用那些自身可以使用panic和recover避免失败的库例程（库函数）。举例，safelyDo()中deffer修饰的函数可能在调用recover之前就调用了一个logging函数，panicking状态不会影响logging代码的运行。因为加入了恢复模式，函数 do（以及它调用的任何东西）可以通过调用panic来摆脱不好的情况。但是恢复是在panicking的协程内部的：不能被另外一个协程恢复。


***
**参考资料**
- https://github.com/Unknwon/the-way-to-go_ZH_CN/
