---
layout: post
title:  "Go教程07_字典"
date:   2018-04-19 08:08:04
categories: Go语言编程
tags: Go 
excerpt: Go受Python影响，也包含字典。Map是一种特殊的数据结构。
---

* content
{:toc}


## Map

map是一种特殊的数据结构：一种元素对（pair）的无序集合，pair的一个元素是key，对应的另一个元素是value，所以这个结构也称为**关联数组**或**字典**。这是一种快速寻找值的理想结构：给定key，对应的value 可以迅速定位。map这种数据结构在其它编程语言中也称为字典（Python）、hash和HashTable等。

## 1 声明、初始化和make

### 1.1 概念

map是引用类型，可以使用如下声明：

   `var map1 map[keytype]valuetype`

在[keytype]和valuetype之间允许有空格，但是格式化工具gofmt移除了空格。

在声明的时候不需要知道map的长度，map是可以动态增长的。未初始化的map的值是nil。

key可以是任意可以用==或者!=操作符比较的类型，比如string、int、float，数组、切片和结构体不能作为key，但是指针和接口类型可以。如果要用结构体作为 key，可以提供Key()和Hash()方法，这样可以通过结构体的域计算出唯一的数字或者字符串的key。

value可以是任意类型的。通过使用空接口类型，可以存储任意值，但是使用这种类型作为值时需要先做一次类型断言。

map传递给函数的代价很小：在32位机器上占4个字节，64位机器上占8个字节，无论实际上存储了多少数据。通过key在map中寻找值是很快的，比线性查找快得多，但是仍然比从数组和切片的索引中直接读取要慢100倍，所以如果很在乎性能的话还是建议用切片来解决问题。

map也可以用函数作为自己的值，这样就可以用来做分支结构：key用来选择要执行的函数。

如果key1是map1的key，那么map1[key1]就是对应key1的值，就如同数组索引符号一样。数组可以视为一种简单形式的map，key是从0开始的整数。

key1对应的值可以通过赋值符号来设置为val1：`map1[key1] = val1`。

令`v := map1[key1]`可以将key1对应的值赋值为v；如果map中没有key1存在，那么v将被赋值为map1的值类型的空值。

常用的len(map1)方法可以获得map中的元素对的数目，这个数目是可以伸缩的，因为map-pairs在运行时可以动态添加和删除。

* 程序示例

```go
// @file:        make_maps.go
// @version:     1.0
// @author:      haulf
// @date:        2017.09.08
// @go version:  1.9
// @brief:       Demo test.

package main

import (
    "fmt"
)

func main() {
    var mapLit map[string]int
    //var mapCreated map[string]float32
    var mapAssigned map[string]int

    mapLit = map[string]int{"one": 1, "two": 2}
    mapCreated := make(map[string]float32)
    mapAssigned = mapLit // mapAssigned是mapList的引用，对mapAssigned的修改会影响到mapList

    mapCreated["key1"] = 4.5
    mapCreated["key2"] = 3.14159
    mapAssigned["two"] = 3

    fmt.Printf("Map literal at one is: %d\n", mapLit["one"])
    fmt.Printf("Map created at key2 is: %f\n", mapCreated["key2"])
    fmt.Printf("Map assigned at two is: %d\n", mapLit["two"])
    fmt.Printf("Map literal at ten is: %d\n", mapLit["ten"])
}
```

* 程序运行结果

```go
Map literal at one is: 1
Map created at key2 is: 3.141590
Map assigned at two is: 3
Map literal at ten is: 0
```

* 程序说明

mapLit说明了map literals的使用方法： map可以用{key1: val1, key2: val2}的描述方法来初始化，就像数组和结构体一样。

map是引用类型的：内存用make方法来分配。

map的初始化：

`var map1[keytype]valuetype = make(map[keytype]valuetype)`

或者简写为：`map1 := make(map[keytype]valuetype)`

mapAssigned也是mapList的引用，对mapAssigned的修改也会影响到mapLit的值。

**不要使用new、永远用make来构造map。**

**注意** 如果错误的使用new()分配了一个引用对象，会获得一个空引用的指针，相当于声明了一个未初始化的变量并且取了它的地址。

为了说明值可以是任意类型的，这里给出了一个使用func()int作为值的map：

* 程序示例

```go
// @file:        map_func.go
// @version:     1.0
// @author:      haulf
// @date:        2017.09.08
// @go version:  1.9
// @brief:       Demo test.

package main

import (
    "fmt"
)

func main() {
    mf := map[int]func() int{
        1: func() int { return 10 },
        2: func() int { return 20 },
        5: func() int { return 50 },
    }

    fmt.Println(mf)
}
```

* 程序运行结果

输出结果为：`map[1:0x10903be0 5:0x10903ba0 2:0x10903bc0]`。整型都被映射到函数地址。

### 1.2 map容量

和数组不同，map可以根据新增的key-value对动态的伸缩，因此它不存在固定长度或者最大限制，但是也可以选择标明map的初始容量capacity，就像这样：`make(map[keytype]valuetype, cap)`。例如：

   `map2 := make(map[string]float, 100)`

当map增长到容量上限的时候，如果再增加新的key-value对，map的大小会自动加 1。出于性能的考虑，对于大的map或者会快速扩张的map，即使只是大概知道容量，也最好先标明。

这里有一个map的具体例子，即将音阶和对应的音频映射起来：

```go
noteFrequency := map[string]float32 {
       "C0": 16.35, "D0": 18.35, "E0": 20.60,"F0": 21.83,
       "G0": 24.50, "A0": 27.50, "B0": 30.87,"A4": 440}
```

### 1.3 用切片作为map的值

既然一个key只能对应一个value，而value又是一个原始类型，那么如果一个key要对应多个值怎么办？例如，当要处理unix机器上的所有进程，以父进程（pid为整形）作为key，所有的子进程（以所有子进程的pid组成的切片）作为value。通过将value定义为[]int类型或者其它类型的切片，就可以优雅的解决这个问题。

这里有一些定义这种map的例子：

```go
   mp1 := make(map[int][]int)
   mp2 := make(map[int]*[]int)
```

## 2 测试键值对是否存在及删除元素

测试map1中是否存在key1, 可以这么用：

`val1, isPresent = map1[key1]`

isPresent返回一个bool值：如果key1存在于map1，val1就是key1对应的value值，并且isPresent为true；如果key1不存在，val1就是一个空值，并且 isPresent会返回false。

如果只是想判断某个key是否存在而不关心它对应的值到底是多少，可以这么做：

```go
   _, ok := map1[key1] // 如果key1存在则ok == true，否在ok为false
```

或者和if混合使用：

```go
   if _, ok := map1[key1]; ok {
       // ...
   }
```

从map1中删除key1：直接delete(map1, key1)就可以。

如果key1不存在，该操作不会产生错误。

* 程序示例

```go
// @file:        map_test_element.go
// @version:     1.0
// @author:      haulf
// @date:        2017.11.17
// @go version:  1.9
// @brief:       Map test.

package main

import (
    "fmt"
)

func main() {
    var value int
    var isPresent bool
    map1 := make(map[string]int)
    map1["New Delhi"] = 55
    map1["Beijing"] = 20
    map1["Washington"] = 25

    value, isPresent = map1["Beijing"]
    if isPresent {
        fmt.Printf("The value of \"Beijing\" in map1 is:%d\n", value)
    } else {
        fmt.Printf("map1 does not containBeijing")
    }

    value, isPresent = map1["Paris"]
    fmt.Printf("Is \"Paris\" in map1 ?: %t\n", isPresent)
    fmt.Printf("Value is: %d\n", value)

    // delete an item:
    delete(map1, "Washington")

    value, isPresent = map1["Washington"]
    if isPresent {
        fmt.Printf("The value of \"Washington\" in map1 is:%d\n", value)
    } else {
        fmt.Println("map1 does not contain Washington")
    }
}
```

* 程序运行结果

```go
The value of "Beijing" in map1 is:20
Is "Paris" in map1 ?: false
Value is: 0
map1 does not contain Washington
```



## 3 for-range的配套用法

可以使用for循环构造map：

```go
   for key, value := range map1 {
       ...
   }
```

第一个返回值key是map中的key值，第二个返回值则是该key对应的value值；这两个都是仅for循环内部可见的局部变量。其中第一个返回值key值是一个可选元素。如果只关心值，可以这么使用：

```go
   for _, value := range map1 {
       ...
   }
```

如果只想获取key，可以这么使用：

```go
   for key := range map1 {
       fmt.Printf("key is: %d\n", key)
   }
```

* 程序示例

```go
// @file:        map_for_range.go
// @version:     1.0
// @author:      haulf
// @date:        2017.11.17
// @go version:  1.9
// @brief:       Map test.

package main

import (
    "fmt"
)

func main() {
    map1 := make(map[int]float32)
    map1[1] = 1.0
    map1[2] = 2.0
    map1[3] = 3.0
    map1[4] = 4.0

    for key := range map1 {
        fmt.Printf("key is: %d\n", key)
    }

    for _, value := range map1 {
        fmt.Printf("value is: %f\n", value)
    }

    for key, value := range map1 {
        fmt.Printf("key is: %d - value is: %f\n", key, value)
    }
}
```

* 程序运行结果

```go
key is: 1
key is: 2
key is: 3
key is: 4
value is: 1.000000
value is: 2.000000
value is: 3.000000
value is: 4.000000
key is: 2 - value is: 2.000000
key is: 3 - value is: 3.000000
key is: 4 - value is: 4.000000
key is: 1 - value is: 1.000000
```

* 程序说明

map不是按照key排列的，也不是按照value排列的。



## 4 map类型的切片

如果想获取一个map类型的切片，则必须使用两次make()函数，第一次用来分配切片，第二次用来分配切片中每个map元素。

* 程序示例

```go
// @file:        map_slice.go
// @version:     1.0
// @author:      haulf
// @date:        2017.11.17
// @go version:  1.9
// @brief:       Map test.

package main

import (
    "fmt"
)

func main() {
    // Version A:
    items := make([]map[int]int, 5) // items是一个切片，容量为5。它里面的元素类型是map[int]int
    for i := range items {          
        items[i] = make(map[int]int, 1) // 分配切片中的每个map元素。items[i]是一个Map。
        items[i][1] = 2
    }

    fmt.Printf("Version A: Value of items: %v\n", items)

    // Version B: NOT GOOD!
    items2 := make([]map[int]int, 5)
    for _, item := range items2 {
        // item is only a copy of the slice element.
        item = make(map[int]int, 1)
        item[1] = 2 // This 'item' will be lost on the next iteration.
    }

    fmt.Printf("Version B: Value of items: %v\n", items2)
}
```

* 程序运行结果

```go
Version A: Value of items: [map[1:2] map[1:2] map[1:2] map[1:2] map[1:2]]
Version B: Value of items: [map[] map[] map[] map[] map[]]
```

* 程序说明

应该像A版本那样通过索引使用切片的map元素。在B版本中获得的项只是map值的一个拷贝而已，所以真正的map元素没有得到初始化。

## 5 map的排序

map默认是无序的。不管是按照key，还是按照value，默认都不排序。如果想为map排序，需要将key（或者value）拷贝到一个切片，再对切片排序（使用sort包），然后可以使用切片的for-range方法打印出所有的key和value。

* 程序示例

```go
// @file:        map_sort.go
// @version:     1.0
// @author:      haulf
// @date:        2017.11.19
// @go version:  1.9
// @brief:       Map sort test.

package main

import (
    "fmt"
    "sort"
)

var (
    barVal = map[string]int{"alpha": 34, "bravo": 56, "charlie": 23,
        "delta": 87, "echo": 56, "foxtrot": 12,
        "golf": 34, "hotel": 16, "indio": 87,
        "juliet": 65, "kili": 43, "lima": 98}
)

func main() {
    fmt.Println("\nunsorted:")

    for k, v := range barVal {
        fmt.Printf("Key: %v, Value: %v \n ", k, v)
    }

  keys := make([]string, len(barVal)) // 构造一个长度为len(barVal)，元素类型为string的切片
    i := 0
    for k, _ := range barVal {
        keys[i] = k
        i++
    }

    sort.Strings(keys)
    fmt.Println("\n\nsorted:")
    for _, k := range keys {
        fmt.Printf("Key: %v, Value: %v \n ", k, barVal[k])
    }
}
```

* 程序运行结果

```go
unsorted:
 Key: indio, Value: 87
 Key: kili, Value: 43
 Key: alpha, Value: 34
 Key: bravo, Value: 56
 Key: charlie, Value: 23
 Key: delta, Value: 87
 Key: foxtrot, Value: 12
 Key: echo, Value: 56
 Key: golf, Value: 34
 Key: hotel, Value: 16
 Key: juliet, Value: 65
 Key: lima, Value: 98


sorted:
 Key: alpha, Value: 34
 Key: bravo, Value: 56
 Key: charlie, Value: 23
 Key: delta, Value: 87
 Key: echo, Value: 56
 Key: foxtrot, Value: 12
 Key: golf, Value: 34
 Key: hotel, Value: 16
 Key: indio, Value: 87
 Key: juliet, Value: 65
 Key: kili, Value: 43
 Key: lima, Value: 98
```

* 程序说明

但是如果想要一个排序的列表最好使用结构体切片，这样会更有效：

```go
   type name struct {
       key string
       value int
   }
```



## 6 将map的键值对调

这里倒置是指调换key和value。如果map的值类型可以作为key且所有的value是唯一的，那么通过下面的方法可以简单的做到键值对调。

* 程序示例

```go
// @file:        map_invert.go
// @version:     1.0
// @author:      aihaofeng
// @date:        2017.11.19
// @go version:  1.9
// @brief:       Map invert test.

package main

import (
    "fmt"
)

var (
    barVal = map[string]int{"alpha": 34, "bravo": 56, "charlie": 23,
        "delta": 87, "echo": 56, "foxtrot": 12,
        "golf": 34, "hotel": 16, "indio": 87,
        "juliet": 65, "kili": 43, "lima": 98}
)

func main() {
    invMap := make(map[int]string, len(barVal))
    for k, v := range barVal {
        invMap[v] = k
    }

    fmt.Println("\ninverted:\n")
    for k, v := range invMap {
        fmt.Printf("Key: %v, Value: %v \n ", k, v)
    }
}
```

* 程序运行结果  


```go
inverted:

 Key: 56, Value: bravo
 Key: 16, Value: hotel
 Key: 87, Value: delta
 Key: 65, Value: juliet
 Key: 34, Value: golf
 Key: 23, Value: charlie
 Key: 12, Value: foxtrot
 Key: 43, Value: kili
 Key: 98, Value: lima
```

* 程序说明

如果原始value值不唯一那么这么做肯定会出错；为了保证不出错，当遇到不唯一的key时应当立刻停止，这样可能会导致没有包含原map的所有键值对！一种解决方法就是仔细检查唯一性并且使用多值map，比如使用 `map[int][]string` 类型。
