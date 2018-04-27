---
layout: post
title:  "Go教程11_I/O操作"
date:   2018-04-23 08:08:04
categories: Go语言编程
tags: Go 
excerpt: 除了fmt和os包，还需要用到bufio包来处理缓冲的输入和输出。
---

* content
{:toc}


除了fmt和os包，还需要用到bufio包来处理缓冲的输入和输出。

## 1 读取用户的输入

这里读取用户的输入，是指从键盘和标准输入os.Stdin读取输入。

### 1.1 使用fmt包提供的Scan()和Sscan()开头函数

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

### 1.2 使用bufio包提供的缓冲读取（buffered reader）来读取数据

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

## 2 文件读写

### 2.1 读文件

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

### 2.2 compress包：读取压缩文件

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
 

### 2.3 写文件 

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

## 3 文件拷贝

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

## 4 从命令行读取参数

### 4.1 os 包

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

### 4.2 flag 包

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

## 5 用buffer读取文件

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

## 6 用切片读写文件

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

## 7 用defer关闭文件

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

## 8 使用接口的实际例子：fmt.Fprintf

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

## 9 JSON 数据格式

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

## 10 XML数据格式

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

## 11 用Gob传输数据

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

## 12 Go中的密码学

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
## 13 参考资料
- https://github.com/Unknwon/the-way-to-go_ZH_CN/
