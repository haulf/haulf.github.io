---
layout: post
title:  "数组与切片"
date:   2018-04-18 08:08:04
categories: Go语言编程
tags: Go 
excerpt: Go让人感觉像是Python或Ruby这样的动态语言，但却又拥有像C或者Java这类语言的高性能和安全性。Go语言出现的目的是希望在编程领域创造最实用的方式来进行软件开发。
---

* content
{:toc}


## 数组与切片

容器是可以包含大量条目（item）的数据结构, 例如数组、切片和map。从这里可以看到Go明显受到Python的影响。

以`[]`符号标识的数组类型几乎在所有的编程语言中都是一个基本主力。Go语言中的数组也是类似的，只是有一些特点。Go没有C那么灵活，但是拥有切片（slice）类型。这是一种建立在Go语言数组类型之上的抽象，要想理解切片必须先理解数组。数组有特定的用处，但是却有一些呆板，所以在Go语言的代码里并不是特别常见。相对的，切片确实随处可见的。它们构建在数组之上并且提供更强大的能力和便捷。

## 1 数组声明和初始化

### 1.1 数组基本概念

数组是具有相同唯一类型的一组已经编号并且长度固定的数据项序列。这种类型可以是任意的原始类型例如整形、字符串或者自定义类型。数组长度必须是一个常量表达式，并且必须是一个非负整数。数组长度也是数组类型的一部分，所以[5]int和[10]int是属于不同类型的。数组的编译时值初始化是按照数组顺序完成的。

数组元素可以通过索引来读取，索引从0开始，第一个元素索引为0，第二个索引为1，以此类推。元素的数目，也称为**长度或者数组大小**，必须是固定的并且在声明该数组时就给出（编译时需要知道数组长度以便分配内存）。数组长度最大为2Gb。

声明的格式是：`var identifier [len]type` 例如：`var arr1 [5]int`

每个元素是一个整型值，当声明数组时所有的元素都会被自动初始化为默认值0。arr1的长度是5，索引范围从0到len(arr1)-1。第一个元素是arr1[0]，第三个元素是 arr1[2]；总体来说索引i代表的元素是arr1[i]，最后一个元素是arr1[len(arr1)-1]。

由于索引的存在，遍历数组的方法自然就是使用for结构:

- 通过for初始化数组项
- 通过for打印数组元素
- 通过for依次处理元素

* 程序示例

```go
/*
@file:    for_arrays.go
@version: v1.0
@author:  haulf
@date:    2017.11.12
@brief:   Array test program.
*/

package main

import "fmt"

func main() {
    var arr1 [5]int

    for i := 0; i < len(arr1); i++ {
        arr1[i] = i * 2
    }

    for i := 0; i < len(arr1); i++ {
        fmt.Printf("Array at index %d is %d\n", i, arr1[i])
    }
}

```

* 程序运行结果

```go
Array at index 0 is 0
Array at index 1 is 2
Array at index 2 is 4
Array at index 3 is 6
Array at index 4 is 8
```

Go语言中的数组是一种**值类型**（不像C/C++中是指向首元素的指针），所以可以通过new()来创建： 

`var arr1 = new([5]int)`

那么这种方式和`var arr2 [5]int`的区别是什么呢？arr1是通过new()来创建的，其类型是*[5]int，而arr2是直接申明的，其类型是[5]int。这样的结果就是当把一个数组赋值给另一个时，需要在做一次数组内存的拷贝操作。例如：

```go
arr2 := arr1
arr2[2] = 100
```

这样两个数组就有了不同的值，在赋值后修改arr2不会对arr1生效。所以在函数中数组作为参数传入时，如func1(arr2)，会产生一次数组拷贝，func1方法不会修改原始的数组arr2。

如果想修改原数组，那么arr2必须通过&操作符以引用方式传过来，例如func1(&arr2）。

* 程序示例

```go
/*
@file:    pointer_array.go
@version: v1.0
@author:  haulf
@date:    2017.11.12
@brief:   Array test program.
*/

package main

import "fmt"

func f(a [3]int) {
    fmt.Println(a)
}

func fp(a *[3]int) {
    fmt.Println(a)
}

func main() {
    var ar [3]int
    f(ar)   // passes a copy of ar
    fp(&ar) // passes a pointer to ar
}

```

* 程序运行结果

```go
   [0 0 0]
   &[0 0 0]
```

另一种方法就是生成数组切片并将其传递给函数。

### 1.2 数组常量

如果数组值已经提前知道了，那么可以通过数组常量的方法来初始化数组，而不用依次使用[]=方法。

* 程序示例

```go
/*
@file:    array_literals.go
@version: v1.0
@author:  haulf
@date:    2017.11.12
@brief:   Array test program.
*/

package main

import "fmt"

func main() {
    // var arrAge = [5]int{18, 20, 15, 22, 16}
    // var arrLazy = [...]int{5, 6, 7, 8, 22}
    // var arrLazy = []int{5, 6, 7, 8, 22}
    var arrKeyValue = [5]string{3: "Chris", 4: "Ron"}
    // var arrKeyValue = []string{3: "Chris", 4: "Ron"}

    for i := 0; i < len(arrKeyValue); i++ {
        fmt.Printf("Person at %d is %s\n", i, arrKeyValue[i])
    }
}
```

* 程序运行结果

```go
Person at 0 is
Person at 1 is
Person at 2 is
Person at 3 is Chris
Person at 4 is Ron
```

* 程序说明

第一种变化：

var arrAge = [5]int{18, 20, 15, 22, 16}

[5]int可以从左边起开始忽略：[10]int {1, 2, 3} :这是一个有10个元素的数组，除了前三个元素外其它元素都为0。

第二种变化：

```go
var arrLazy = [...]int{5, 6, 7, 8, 22}
```

`...`可同样可以忽略，从技术上说它们其实变化成了切片。

第三种变化：key: value syntax
```go
var arrKeyValue = [5]string{3: "Chris", 4: "Ron"}
```

只有索引3和4被赋予实际的值，其它元素都被设置为空的字符串。在这里数组长度同样可以写成...或者直接忽略。

可以取任意数组常量的地址来作为指向新实例的指针。

* 程序示例

```go
/*
@file:    for_arrays2.go
@version: v1.0
@author:  haulf
@date:    2017.11.12
@brief:   Array test program.
*/

package main

import "fmt"

func fp(a *[3]int) {
    fmt.Println(a)
}

func main() {
    for i := 0; i < 3; i++ {
        fp(&[3]int{i, i * i, i * i * i})
    }
}
```

* 程序运行结果

```go
&[0 0 0]
&[1 1 1]
&[2 4 8]
```

### 1.3 多维数组

数组通常是一维的，但是可以用来组装成多维数组，例如：`[3][5]int`，`[2][2][2]float64`。内部数组总是长度相同的。Go语言的多维数组是矩形式的（唯一的例外是切片的数组）。

* 程序示例

```go
/*
@file:    multidim_array.go
@version: v1.0
@author:  haulf
@date:    2017.11.12
@brief:   Array test program.
*/

package main

const (
    WIDTH  = 1920
    HEIGHT = 1080
)

type pixel int

var screen [WIDTH][HEIGHT]pixel

func main() {

    for y := 0; y < HEIGHT; y++ {
        for x := 0; x < WIDTH; x++ {
            screen[x][y] = 0
        }
    }
}
```

### 1.4 将数组传递给函数

把第一个大数组传递给函数会消耗很多内存。有两种方法可以避免这种现象：

- 传递数组的指针
- 使用数组的切片

接下来的例子阐明了第一种方法：

* 程序示例

```go
/*
@file:    array_sum.go
@version: v1.0
@author:  haulf
@date:    2017.11.12
@brief:   Array test program.
*/

package main

import "fmt"

func main() {
    array := [3]float64{7.0, 8.5, 9.1}
    x := Sum(&array) // Note the explicit address-of operator

    // to pass a pointer to the array
    fmt.Printf("The sum of the array is: %f", x)
}

func Sum(a *[3]float64) (sum float64) {
    // derefencing *a to get back to the array is not necessary!
    for _, v := range a {
        sum += v
    }

    return
}
```

* 程序运行结果

```go
The sum of the array is: 24.600000
```

* 程序说明

在Go中这种方式并不常用，通常使用切片。

## 2 切片

### 2.1 切片声明和初始化

**1. 基本概念**

切片（slice）是对数组一个连续片段的引用（该数组称之为**切片的相关数组**，通常是匿名的），所以**切片是一个引用类型**（因此更类似于C/C++中的数组类型，或者 Python中的list类型）。这个片段可以是整个数组，或者是由起始和终止索引标识的一些项的子集。需要注意的是，终止索引标识的项不包括在切片内。切片提供了一个相关数组的动态窗口。

切片是可索引的，并且可以由len()函数获取长度。给定项的切片索引可能比相关数组的相同元素的索引小。和数组不同的是，切片的长度可以在运行时修改，最小为0，最大为相关数组的长度。切片是一个长度可变的数组。

切片提供了计算容量的函数cap()可以测量切片最长可以达到多少。它等于切片的长度+数组除切片之外的长度。如果s是一个切片，cap(s)就是从s[0]到数组末尾的数组长度。切片的长度永远不会超过它的容量，所以对于切片s来说该不等式永远成立：`0 <= len(s) <= cap(s)`。

多个切片如果表示同一个数组的片段，它们可以共享数据。因此一个切片和相关数组的其它切片是共享存储的。相反，不同的数组总是代表不同的存储。数组实际上是切片的构建块。

因为切片是引用，所以它们不需要使用额外的内存并且比使用数组更有效率，所以在Go代码中，切片比数组更常用。

**2. 切片声明和初始化**

声明切片的格式是： `var identifier []type`。在切片声明时不需要说明长度。

一个切片在未初始化之前默认为nil，长度为0。

切片的初始化格式是：

     `var slice1 []type = arr1[start:end]`

这表示slice1是由数组arr1从start索引到end-1索引之间的元素构成的子集（切分数组，start:end被称为**slice表达式**）。所以slice1[0]就等于 arr1[start]。这可以在arr1被填充前就定义好。

如果某个人写：`var slice1 []type = arr1[:]`，那么slice1就等于完整的arr1数组。这种表示方式是`arr1[0:len(arr1)]`的一种缩写。另外一种表述方式是：`slice1 = &arr1`。

`arr1[2:]`和`arr1[2:len(arr1)]`相同，都包含了数组从第三个到最后的所有元素。`arr1[:3]`和`arr1[0:3]`相同，包含了从第一个到第三个元素（不包括第三个）。如果想去掉slice1的最后一个元素，只要`slice1 = slice1[:len(slice1)-1]`。

一个由数字1、2、3组成的切片可以这么生成：

     `s := [3]int{1,2,3}`

甚至更简单的方法：

     `s := []int{1,2,3}`

s2:= s[:]是用切片组成的切片，拥有相同的元素，但是仍然指向相同的相关数组。

一个切片s可以这样扩展到它的大小上限：`s = s[:cap(s)]`，如果再扩大的话就会导致运行时错误。

对于每一个切片（包括 string），以下状态总是成立的：

  s == s[:i] + s[i:] // i是一个整数且: 0 <= i <= len(s)

   len(s) < cap(s)

切片也可以用类似数组的方式初始化：

     `var x = []int{2, 3, 5, 7, 11}`

这样就创建了一个长度为5的数组并且创建了一个相关切片。

切片在内存中的组织方式实际上是一个有3个域的结构体：指向相关数组的指针，切片长度以及切片容量。

* 程序示例

```go
/*
@file:    array_slices.go
@version: v1.0
@author:  haulf
@date:    2017.11.12
@brief:   Array test program.
*/

package main

import "fmt"

func main() {
    var arr1 [6]int
    var slice1 []int = arr1[2:5] // item at index 5 not included!

    // load the array with integers: 0,1,2,3,4,5
    for i := 0; i < len(arr1); i++ {
        arr1[i] = i
    }

    // print the slice
    for i := 0; i < len(slice1); i++ {
        fmt.Printf("Slice at %d is %d\n", i, slice1[i])
    }

    fmt.Printf("The length of arr1 is %d\n", len(arr1))
    fmt.Printf("The length of slice1 is %d\n", len(slice1))
    fmt.Printf("The capacity of slice1 is %d\n", cap(slice1))

    // grow the slice
    slice1 = slice1[0:4]

    for i := 0; i < len(slice1); i++ {
        fmt.Printf("Slice at %d is %d\n", i, slice1[i])
    }

    fmt.Printf("The length of slice1 is %d\n", len(slice1))
    fmt.Printf("The capacity of slice1 is %d\n", cap(slice1))

    // grow the slice beyond capacity
    // // panic: runtime error: slice bound out of range
    //slice1 = slice1[0:7 ]
}
```

* 程序运行结果

```go
Slice at 0 is 2
Slice at 1 is 3
Slice at 2 is 4
The length of arr1 is 6
The length of slice1 is 3
The capacity of slice1 is 4
Slice at 0 is 2
Slice at 1 is 3
Slice at 2 is 4
Slice at 3 is 5
The length of slice1 is 4
The capacity of slice1 is 4
```

* 程序说明

如果s2是一个slice，可以将s2向后移动一位s2 = s2[1:]，但是末尾没有移动。切片只能向后移动，s2 = s2[-1:]会导致编译错误。切片不能被重新分片以获取数组的前一个元素。

**注意**

绝对不要用指针指向 slice。切片本身已经是一个引用类型，所以它本身就是一个指针!!

### 2.2 将切片作为参数传递给函数

如果有一个函数需要对数组做操作，可能总是需要把参数声明为切片。当调用该函数时，把数组分片，创建为一个切片引用并传递给该函数。

* 程序示例

```go
/*
@file:    slice_param.go
@version: v1.0
@author:  haulf
@date:    2017.11.12
@brief:   Array test program.
*/

package main

import (
    "fmt"
)

func sum(a []int) int {
    s := 0
    for i := 0; i < len(a); i++ {
        s += a[i]
    }
    return s
}

func main() {
    var arr = [5]int{0, 1, 2, 3, 4}
    fmt.Println("The sum is:", sum(arr[:])) 
}
```

* 程序运行结果

```shell
The sum is: 10
```

* 程序说明

在上面的程序中，是在一数组的基础上生成了一个切片传递给函数sum。

### 2.3 用make()创建一个切片

当切片的相关数组还没有定义时，可以使用make()函数来创建一个切片 同时创建好相关数组：`var slice1 []type = make([]type, len)`，也可以简写为`slice1 := make([]type, len)`，这里len是数组的长度并且也是slice的初始长度。

所以定义s2 := make([]int, 10)，那么cap(s2) == len(s2)== 10。

这里的make()函数接受2个参数：元素的类型以及切片的元素个数。

如果想创建一个slice1，它不占用整个数组，而只是占用以len为个数个项，可以这样创建：slice1 := make([]type, len, cap)。

make的使用方式是：func make([]T, len, cap)，其中cap是可选参数。

所以下面两种方法可以生成相同的切片:

```go
   make([]int, 50, 100)
   new([100]int)[0:50]
```

* 程序示例

```go
// @file:        make_slice.go
// @version:     1.0
// @author:      haulf
// @date:        2017.11.15
// @go version:  1.9
// @brief:       Slice test.

package main

import (
    "fmt"
)

func main() {
    var slice1 []int = make([]int, 10)

    // load the array/slice:
    for i := 0; i < len(slice1); i++ {
        slice1[i] = 5 * i
    }

    // print the slice:
    for i := 0; i < len(slice1); i++ {
        fmt.Printf("Slice at %d is %d\n", i, slice1[i])
    }

    fmt.Printf("\nThe length of slice1 is %d\n", len(slice1))
    fmt.Printf("The capacity of slice1 is %d\n", cap(slice1))
}
```

* 运行结果

```go
Slice at 0 is 0
Slice at 1 is 5
Slice at 2 is 10
Slice at 3 is 15
Slice at 4 is 20
Slice at 5 is 25
Slice at 6 is 30
Slice at 7 is 35
Slice at 8 is 40
Slice at 9 is 45

The length of slice1 is 10
The capacity of slice1 is 10
```



因为字符串是纯粹不可变的字节数组，它们也可以被切分成切片。

### 2.4 new()和make()的区别

看起来二者没有什么区别，都在堆上分配内存，但是它们的行为不同，适用于不同的类型。

* new(T) 为每个新的类型T分配一片内存，初始化为0，并且返回类型为*T的内存地址。这种方法返回一个指向类型为T，值为0的地址的指针，它适用于值类型如数组和结构体；它相当于&T{}。
* make(T) 返回一个类型为T的初始值，它只适用于3种内建的引用类型：切片、map和channel。

   `var v []int = make([]int, 10, 50)`或者`v := make([]int, 10, 50)`

这样分配一个有50个int值的数组，并且创建一个长度为10，容量为50的切片。 

### 2.5 多维切片

和数组一样，切片通常也是一维的，但是也可以由一维组合成高维。通过分片的分片（或者切片的数组），长度可以任意动态变化，所以Go语言的多维切片可以任意切分，而且内层的切片必须通过make()函数单独分配。

### 2.6 bytes包

类型[]byte的切片十分常见，Go语言有一个bytes包专门用来解决这种类型的操作方法。bytes包和字符串包十分类似，而且它还包含一个十分有用的类型Buffer:

```go
   import "bytes"    

   type Buffer struct {
       ...
    }
```

这是一个长度可变的bytes的buffer，提供Read和Write方法，因为读写长度未知的bytes最好使用buffer。

Buffer可以这样定义：`var buffer bytes.Buffer`，或者使用new获得一个指针：`var r *bytes.Buffer = new(bytes.Buffer)`。或者通过函数：`func NewBuffer(buf []byte)*Buffer`，创建一个Buffer对象并且用buf初始化好；NewBuffer最好用在从buf读取的时候使用。

 

**通过buffer串联字符串**

在下面的代码段中，创建一个buffer，通过buffer.WriteString(s)方法将字符串s追加到后面，最后再通过buffer.String()方法转换为string：

```go
   var buffer bytes.Buffer

   for {
       // method getNextString() not shown here
       if s, ok := getNextString(); ok { 
           buffer.WriteString(s)
       } else {
           break
       }
    }

   fmt.Print(buffer.String(), "\n")
```

这种实现方式比使用 += 要更节省内存和CPU，尤其是要串联的字符串数目特别多的时候。


## 3 for-range结构

这种构建方法可以应用与数组和切片:

```go
   for ix, value := range slice1 {
       ...
    }
```

## 4 切片重组（reslice）

已经知道切片创建的时候通常比相关数组小，例如：

   slice1 := make([]type, start_length, capacity)

其中 start_length 作为切片初始长度而 capacity 作为相关数组的长度。

这么做的好处是的切片在达到容量上限后可以扩容。改变切片长度的过程称之为**切片重组(reslicing)**，做法如下：slice1 = slice1[0:end]，其中 end 是新的末尾索引（即长度）。

将切片扩展 1 位可以这么做：

   sl = sl[0:len(sl)+1]


## 5 切片的复制与追加

如果想增加切片的容量，必须创建一个新的更大的切片并把原分片的内容都拷贝过来。下面的代码描述了从拷贝切片的**copy()函数**和向切片追加新元素的**append()函数**。

* 程序示例

```go
// @file:        copy_append_slice.go
// @version:     1.0
// @author:      haulf
// @date:        2017.11.15
// @go version:  1.9
// @brief:       Slice test.

package main

import (
    "fmt"
)

func main() {
    sliceFrom := []int{1, 2, 3}
    sliceTo := make([]int, 10)
    n := copy(sliceTo, sliceFrom)
    fmt.Println(sliceTo)
    fmt.Printf("Copied %d elements\n", n) // n == 3

    sliceNew := []int{1, 2, 3}
    sliceNew = append(sliceNew, 4, 5, 6)
    fmt.Println(sliceNew)
}
```

* 程序运行结果

```go
[1 2 3 0 0 0 0 0 0 0]
Copied 3 elements
[1 2 3 4 5 6]
```

`func append(s []T, x …T) []T` 其中append方法将0个或多个具有相同类型的元素追加到切片后面并且返回新的切片。追加的元素必须和原切片的元素同类型。如果 s的容量不足以存储新增元素，append会分配新的切片来保证已有切片元素和新增元素的存储。因此，返回的切片可能已经指向一个不同的相关数组了。append方法总是返回成功，除非系统内存耗尽了。

如果想将切片y追加到切片x后面，只要将第二个参数扩展成一个列表即可：`x = append(x, y…)`。

注意： append 在大多数情况下很好用，但是如果想完全掌控整个追加过程，可以实现一个这样的AppendByte方法：

```go
   func AppendByte(slice []byte, data ...byte) []byte {
       m := len(slice)
       n := m + len(data)

       if n > cap(slice) { // if necessary, reallocate
           // allocate double what's needed, for future growth.
           newSlice := make([]byte, (n+1)*2)
           copy(newSlice, slice)
           slice = newSlice
       }

       slice = slice[0:n]
       copy(slice[m:n], data)
     
       return slice
    }
```

`func copy(dst, src []T) int` copy方法将类型为T的切片从源地址src拷贝到目标地址dst，覆盖dst的相关元素，并且返回拷贝的元素个数。源地址和目标地址可能会有重叠。拷贝个数是src和dst的长度最小值。如果src是字符串那么元素类型就是byte。如果还想继续使用src，在拷贝结束后执行src = dst。


## 6 字符串、数组和切片的应用
### 6.1 从字符串生成字节切片

假设s是一个字符串（本质上是一个字节数组），那么就可以直接通过`c := []bytes(s)`来获取一个字节的切片c。另外，还可以通过copy函数来达到相同的目的：`copy(dst []byte, src string)`。

同样的，还可以使用for-range来获得每个元素：

```go
// @file:        for_range.go
// @version:     1.0
// @author:      haulf
// @date:        2017.11.15
// @go version:  1.9
// @brief:       Slice test.

package main

import (
    "fmt"
)

func main() {
    s := "\u00ff\u754c"
    for i, c := range s {
        fmt.Printf("%d:%c ", i, c)
    }
}
```

Unicode字符会占用2个字节，有些甚至需要3个或者4个字节来进行表示。如果发现错误的UTF8字符，则该字符会被设置为U+FFFD并且索引向前移动一个字节。和字符串转换一样，同样可以使用`c := []int(s)` 语法，这样切片中的每个int都会包含对应的Unicode代码，因为字符串中的每次字符都会对应一个整数。类似的，也可以将字符串转换为元素类型为rune的切片：`r := []rune(s)`。

可以通过代码`len([]int(s))`来获得字符串中字符的数量，但使用`utf8.RuneCountInString(s)`效率会更高一点。

还可以将一个字符串追加到某一个字符数组的尾部：

```go
   var b []byte
   var s string
   b= append(b, s...)
```

### 6.2 获取字符串的某一部分

使用`substr := str[start:end]`可以从字符串str获取到从索引start开始到end-1位置的子字符串。同样的，str[start:] 则表示获取从start开始到 len(str)-1位置的子字符串，而str[:end]表示获取从0开始到end-1的子字符串。

### 6.3 字符串和切片的内存结构

在内存中，一个字符串实际上是一个双字结构，即一个指向实际数据的指针和记录字符串长度的整数。因为指针对用户来说是完全不可见，因此可以依旧把字符串看做是一个值类型，也就是一个字符数组。

### 6.4 修改字符串中的某个字符

Go语言中的字符串是不可变的，也就是说str[index]这样的表达式是不可以被放在等号左侧的。如果尝试运行`str[i] = 'D'`会得到错误：cannot assign to str[i]。必须先将字符串转换成字节数组，然后再通过修改数组中的元素值来达到修改字符串的目的，最后将字节数组转换回字符串格式。

例如，将字符串"hello"转换为"cello"：

```go
   s:= "hello"
   c:= []byte(s)
   c[0] = ’c’
   s2 := string(c) // s2 == "cello"
```

所以，可以通过操作切片来完成对字符串的操作。

### 6.5 字节数组对比函数

下面的Compare函数会返回两个字节数组字典顺序的整数对比结果，即 

```go
   func Compare(a, b[]byte) int {
       for i:=0; i < len(a) && i < len(b); i++ {
           switch {
           case a[i] > b[i]:
                return 1
           case a[i] < b[i]:
                return -1
           }
       }

       // 数组的长度可能不同
       switch {
       case len(a) < len(b):
           return -1

       case len(a) > len(b):
           return 1
       }

       return 0 // 数组相等
    }
```

### 6.6 搜索及排序切片和数组

标准库提供了sort包来实现常见的搜索和排序操作。可以使用sort包中的函数`func Ints(a []int)`来实现对int类型的切片排序。例如`sort.Ints(arri)`，其中变量 arri就是需要被升序排序的数组或切片。为了检查某个数组是否已经被排序，可以通过函数`IntsAreSorted(a []int) bool`来检查，如果返回true则表示已经被排序。

类似的，可以使用函数`func Float64s(a []float64)`来排序float64的元素，或使用函数`func Strings(a []string)`排序字符串元素。

想要在数组或切片中搜索一个元素，该数组或切片必须先被排序（因为标准库的搜索算法使用的是二分法）。然后，可以使用函数`func SearchInts(a []int, n int) int`进行搜索，并返回对应结果的索引值。

当然，还可以搜索float64和字符串：

```go
   func SearchFloat64s(a []float64, x float64) int
   func SearchStrings(a []string, x string) int
```

### 6.7 append函数常见操作

append非常有用，它能够用于各种方面的操作：

* 将切片b的元素追加到切片a之后：`a = append(a, b…)`
* 复制切片a的元素到新的切片b上：

```go
       b = make([]T, len(a))
       copy(b, a)
```

* 删除位于索引i的元素：`a = append(a[:i], a[i+1:]…)`
* 切除切片a中从索引i至j位置的元素：`a = append(a[:i], a[j:]…)`
* 为切片a扩展j个元素长度：`a = append(a, make([]T, j)…)`
* 在索引i的位置插入元素x：`a = append(a[:i], append([]T{x},a[i:]...)…)`
* 在索引i的位置插入长度为j的新切片：`a = append(a[:i], append(make([]T,j), a[i:]...)…)`
* 在索引i的位置插入切片b的所有元素：`a = append(a[:i], append(b,a[i:]...)…)`
* 取出位于切片a最末尾的元素x：`x, a = a[len(a)-1], a[:len(a)-1]`
* 将元素x追加到切片a：`a = append(a, x)`

因此，可以使用切片和append操作来表示任意可变长度的序列。

从数学的角度来看，切片相当于向量，如果需要的话可以定义一个向量作为切片的别名来进行操作。

### 6.8 切片和垃圾回收

切片的底层指向一个数组，该数组的实际体积可能要大于切片所定义的体积。只有在没有任何切片指向的时候，底层的数组内层才会被释放，这种特性有时会导致程序占用多余的内存。

函数FindDigits将一个文件加载到内存，然后搜索其中所有的数字并返回一个切片。

```go
   var digitRegexp = regexp.MustCompile("[0-9]+")

   func FindDigits(filename string) []byte {
       b, _ := ioutil.ReadFile(filename)
       return digitRegexp.Find(b)
    }
```

这段代码可以顺利运行，但返回的[]byte指向的底层是整个文件的数据。只要该返回的切片不被释放，垃圾回收器就不能释放整个文件所占用的内存。换句话说，一点点有用的数据却占用了整个文件的内存。

想要避免这个问题，可以通过拷贝需要的部分到一个新的切片中：

```go
   func FindDigits(filename string) []byte {
      b, _ := ioutil.ReadFile(filename)
      b = digitRegexp.Find(b)
      c := make([]byte, len(b))
      copy(c, b)
     
      return c
    }
```
