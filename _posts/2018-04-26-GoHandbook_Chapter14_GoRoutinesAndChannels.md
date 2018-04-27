
---
layout: post
title:  "Go教程14_协程与通道"
date:   2018-04-26 08:08:04
categories: Go语言编程
tags: Go 
excerpt: 作为一门21世纪的语言，Go原生支持应用之间的通信和程序的并发。程序可以在不同的处理器和计算机上同时执行不同的代码段。Go语言为构建并发程序的基本代码块是 协程(goroutine)与通道(channel)。它们需要语言、编译器和runtime的支持。Go语言提供的垃圾回收器对并发编程至关重要。
---

* content
{:toc}



作为一门21世纪的语言，Go原生支持应用之间的通信和程序的并发。程序可以在不同的处理器和计算机上同时执行不同的代码段。Go语言为构建并发程序的基本代码块是 协程(goroutine)与通道(channel)。它们需要语言、编译器和runtime的支持。Go语言提供的垃圾回收器对并发编程至关重要。

不要通过共享内存来通信，而通过通信来共享内存。

通信强制协作。

## 1 并发、并行和协程

### 1.1 什么是协程

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

### 1.2 并发和并行的差异

Go的并发原语提供了良好的并发设计基础：表达程序结构以便表示独立地执行的动作；所以Go的的重点不在于并行的首要位置：并发程序可能是并行的，也可能不是。并行是一种通过使用多处理器以提高速度的能力。但往往是，一个设计良好的并发程序在并行方面的表现也非常出色。

在2012年1月的运行时实现中，Go默认没有并行指令，只有一个独立的核心或处理器被专门用于Go程序，不论它启动了多少个协程；所以这些协程是并发运行的，但它们不是并行运行的：同一时间只有一个协程会处在运行状态。

这个情况在以后可能会发生改变，不过届时，为了使的程序可以使用多个核心运行，这时协程就真正的是并行运行了，必须使用GOMAXPROCS变量。这会告诉运行时有多少个协程同时执行。

并且只有gc编译器真正实现了协程，适当的把协程映射到操作系统线程。使用gccgo编译器，会为每一个协程创建操作系统线程。

### 1.3 使用GOMAXPROCS

在gc编译器下（6g或者8g）必须设置GOMAXPROCS为一个大于默认值1的数值来允许运行时支持使用多于1个的操作系统线程，所有的协程都会共享同一个线程除非将 GOMAXPROCS设置为一个大于1的数。当GOMAXPROCS大于1时，会有一个线程池管理许多的线程。通过gccgo编译器GOMAXPROCS有效的与运行中的协程数量相等。假设n是机器上处理器或者核心的数量。如果设置环境变量GOMAXPROCS>=n，或者执行runtime.GOMAXPROCS(n)，接下来协程会被分割（分散）到n个处理器上。更多的处理器并不意味着性能的线性提升。有这样一个经验法则，对于n个核心的情况设置GOMAXPROCS为n-1以获得最佳性能，也同样需要遵守这条规则：协程的数量 > 1 + GOMAXPROCS > 1。

所以如果在某一时间只有一个协程在执行，不要设置GOMAXPROCS！

还有一些通过实验观察到的现象：在一台1颗CPU的笔记本电脑上，增加GOMAXPROCS到9会带来性能提升。在一台32核的机器上，设置GOMAXPROCS=8会达到最好的性能，在测试环境中，更高的数值无法提升性能。如果设置一个很大的GOMAXPROCS只会带来轻微的性能下降；设置GOMAXPROCS=100，使用top命令和H选项查看到只有7个活动的线程。

增加GOMAXPROCS的数值对程序进行并发计算是有好处的；

总结：GOMAXPROCS等同于（并发的）线程数量，在一台核心数多于1个的机器上，会尽可能有等同于核心数的线程在并行运行。

### 1.4 如何用命令行指定使用的核心数量

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

### 1.5 Go协程（goroutines）和其它语言协程（coroutines）

在其它语言中，比如在C#，Lua或者Python里都有协程的概念。这个名字表明它和Go协程有些相似，不过有两点不同：

- Go协程意味着并行（或者可以以并行的方式部署），其它语言中的协程一般来说不是这样的。
- Go协程通过通道来通信；其它语言中的协程通过让出和恢复操作来通信

Go协程比其它语言中的协程更强大，也很容易从其它语言中协程的逻辑复用到Go协程。

## 2 协程间的信道

### 2.1 概念

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

### 2.2 通信操作符<-

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

### 2.3 通道阻塞

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


### 2.4 通过一个或多个通道交换数据进行协程同步

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


### 2.5 同步通道-使用带缓冲的通道

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

### 2.6 在协程中用通道输出结果

为了知道计算何时完成，可以通过信道回报。

```go
   ch := make(chan int)
   go sum(bigArray, ch) // bigArray puts the calculated sum on ch
   // .. do something else for a while
   sum := <- ch // wait for, and retrieve the sum
```

也可以使用通道来达到同步的目的，这个很有效的用法在传统计算机中称为信号量（semaphore）。或者换个方式，通过通道发送信号告知处理已经完成。在其它协程运行时让main程序无限阻塞的通常做法是在main函数的最后放置一个{}，也可以使用通道让main程序等待协程完成，就是所谓的信号量模式。

### 2.7 信号量模式

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

### 2.8 实现并行的for循环

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

### 2.9 用带缓冲通道实现一个信号量

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

### 2.10 给通道使用for循环

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

### 2.11 通道的方向

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

## 3 协程同步：关闭通道-测试阻塞的通道

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

## 4 使用select切换协程

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

## 5 通道、超时和计时器（Ticker）

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

## 6 协程和恢复（recover）

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


## 7 参考资料

- https://github.com/Unknwon/the-way-to-go_ZH_CN/
