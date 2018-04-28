---
layout: post
title:  "Go教程02-开发环境搭建与配置"
date:   2018-04-14 08:08:04
categories: Go语言编程
tags: Go 
excerpt: Go是跨平台的，可以在Windows、Linux、MacOS等操作系统下进行开发。本部分介绍Go的安装与运行环境、开发工具等。
---

* content
{:toc}


## 1 安装与运行环境
### 1.1 平台与架构

Go语言开发团队开发了适用于以下操作系统的编译器：
- Linux
- FreeBSD
- Mac OS X（也称为Darwin）

### 1.2 Go环境变量

Go开发环境依赖于一些操作系统环境变量，最好在安装Go之间就已经设置好它们。这里列举几个最为重要的环境变量：
- `$GOROOT`  表示Go在的电脑上的安装位置，它的值一般都是`$HOME/go`。
- `$GOARCH`  表示目标机器的处理器架构，它的值可以是386、amd64或arm。
- `$GOOS`    表示目标机器的操作系统，它的值可以是darwin、freebsd、linux或windows。
- `$GOBIN`   表示编译器和链接器的安装位置，默认是`$GOROOT/bin`。
- `$GOARM`专门针对基于arm架构的处理器，它的值可以是5或6，默认为6。
- `$GOMAXPROCS`用于设置应用程序可使用的处理器个数与核数。
- `$GOPATH`默认采用和`$GOROOT`一样的值，但从Go1.1版本开始，必须修改为其它路径。它可以包含多个包含Go语言源码文件、包文件和可执行文件的路径，而这些路径下又必须分别包含三个规定的目录：src、pkg和bin，这三个目录分别用于存放源码文件、包文件和可执行文件。

Go编译器支持交叉编译，也就是说可以在一台机器上构建运行在具有不同操作系统和处理器架构上运行的应用程序，也就是说编写源代码的机器可以和目标机器有完全不同的特性。

为了区分本地机器和目标机器，可以使用`$GOHOSTOS`和`$GOHOSTARCH`设置目标机器的参数，这两个变量只有在进行交叉编译的时候才会用到，如果不进行显示设置，它们的值会和本地机器（`$GOOS`和`$GOARCH`）一样。


### 1.3 在Linux上安装Go

#### 1.3.1 配置Go环境变量

在Linux系统下一般通过文件`$HOME/.bashrc` 配置自定义环境变量，根据不同的发行版也可能是文件`$HOME/.profile`。

`export GOROOT=$HOME/go`

为了确保相关文件在文件系统的任何地方都能被调用，还需要添加以下内容：

`export PATH=$PATH:$GOROOT/bin`

在开发Go项目时，还需要一个环境变量来保存的工作目录。

`export GOPATH=$HOME/Applications/Go`

`$GOPATH`可以包含多个工作目录，取决于个人情况。如果设置了多个工作目录，那么当在之后使用 go get（远程包安装命令）时远程包将会被安装在第一个目录下。

在完成这些设置后，需要在终端输入指令source .bashrc 以使这些环境变量生效。然后重启终端，输入 go env 和 env 来检查环境变量是否设置正确。


### 1.4 在Mac OS X上安装Go

如果想要在的Mac系统上安装Go，则必须使用Intel 64位处理器，Go不支持PowerPC处理器。在Mac系统下使用到的C工具链是Xcode的一部分，因此需要通过安装Xcode来完成这些工具的安装。并不需要安装完整的Xcode，而只需要安装它的命令行工具部分。在安装Go程序时会弹出提示安装Xcode的命令行工具，按照提示一步步安装。

安装完成后，在终端里输入go version，若能够查看到版本信息，则说明安装成功了。

接着需要配置环境变量。在自己的主目录下新建`.bash_profile`文件，在此文件中输入：

```shell
# haulf begin, add
export GOPATH=/Users/aihaofeng/Documents/go
export GOBIN=$GOPATH/bin
export PATH=$PATH:$GOBIN
# haulf end
```

接着运行source命令让设置生效：

```shell
source .bash_profile
```

这里设置的GOPATH为`/Users/aihaofeng/Documents/go`，也就是把工作目录设置在Documents下的go目录，所以需要在自己的Documents目录下创建名称为go目录。

如果Mac电脑上安装的是ohmyzsh，则并不使用bash，因此上面的配置不会生效。需要修改的是`.zshrc`文件。

```shell
➜  /Users/aihaofeng subl .zshrc

# haulf begin, 20171117
# Go environment config.
export GOPATH="/Users/aihaofeng/go"
export GOBIN="/Users/aihaofeng/go/bin"
export PATH="$PATH:$GOBIN"
# haulf end
```

- GOROOT: 指的是Go的安装目录。
- GOPATH：指工作目录。
- GOBIN：go install时位置。

目前在Mac上通过`go env`查看到的信息如下：

```shell
GOARCH="amd64"
GOBIN="/Users/aihaofeng/go/bin"
GOEXE=""
GOHOSTARCH="amd64"
GOHOSTOS="darwin"
GOOS="darwin"
GOPATH="/Users/aihaofeng/go"
GORACE=""
GOROOT="/usr/local/go"
GOTOOLDIR="/usr/local/go/pkg/tool/darwin_amd64"
GCCGO="gccgo"
CC="clang"
GOGCCFLAGS="-fPIC -m64 -pthread -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fdebug-prefix-map=/var/folders/4t/mvmdp4313gdfmjstyx2n2bth0000gn/T/go-build717412833=/tmp/go-build -gno-record-gcc-switches -fno-common"
CXX="clang++"
CGO_ENABLED="1"
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
PKG_CONFIG="pkg-config"
```

### 1.5 在Windows上安装Go

在下载页面页面下载Windows系统下的一键安装包。在完成安装包的安装之后，只需要配置`$GOPATH`这一个环境变量就可以开始使用Go语言进行开发了，其它的环境变量安装包会进行自动设置。在默认情况下，Go被安装在目录c:\go下，但如果在安装过程中修改安装目录，则可能需要手动修改所有的环境变量的值。

### 1.6 安装目录清单

Go安装目录（$GOROOT）的文件夹结构应该如下所示：

```shell
➜  /Users/aihaofeng ls /usr/local/go
AUTHORS         CONTRIBUTORS    PATENTS         VERSION         bin             doc             lib             pkg             src
CONTRIBUTING.md LICENSE         README.md       api             blog            favicon.ico     misc            robots.txt      test
```

各个目录或文件的含义如下：
- `/bin`：包含可执行文件，如：编译器，Go工具。
- `/doc`：包含示例程序，代码工具，本地文档等。
- `/lib`：包含文档模版。
- `/misc`：包含与支持Go编辑器有关的配置文件以及cgo的示例。
- `/os_arch`：包含标准库的包的对象文件`（.a）`。
- `/src`：包含源代码构建脚本和标准库的包的完整源代码。
- `/src/cmd`：包含Go和C的编译器和命令行脚本。

### 1.7 Go运行时

尽管Go编译器产生的是本地可执行代码，这些代码仍旧运行在Go的runtime（这部分的代码可以在runtime包中找到）当中。这个runtime类似Java和.NET语言所用到的虚拟机，它负责管理包括内存分配、垃圾回收、栈处理、goroutine、channel、切片（slice）、map和反射（reflection）等等。

runtime主要由C语言编写（Go1.5开始自举），并且是每个Go包的最顶级包。可以在目录$GOROOT/src/runtime中找到相关内容。

垃圾回收器Go拥有简单却高效的标记-清除回收器。它的主要思想来源于IBM的可复用垃圾回收器，旨在打造一个高效、低延迟的并发回收器。目前gccgo还没有回收器，同时适用gc和gccgo的新回收器正在研发中。使用一门具有垃圾回收功能的编程语言不代表可以避免内存分配所带来的问题，分配和回收内容都是消耗CPU资源的一种行为。

Go的可执行文件都比相对应的源代码文件要大很多，这恰恰说明了Go的 runtime 嵌入到了每一个可执行文件当中。当然，在部署到数量巨大的集群时，较大的文件体积也是比较头疼的问题。但总体来说，Go的部署工作还是要比Java和Python轻松得多。因为Go不需要依赖任何其它文件，它只需要一个单独的静态文件，这样也不会像使用其它语言一样在各种不同版本的依赖文件之间混淆。

## 2 开发环境与其它工具

因为Go语言还是一门相对年轻的编程语言，所以不管是在集成开发环境还是相关的插件方面，发展都不是很成熟。不过目前还是有一些IDE能够较好地支持Go的开发，有些开发工具甚至是跨平台的，可以在Linux、Mac OS X或者Windows下工作。

### 2.1 编辑器和集成开发环境

- SublimeText是一个革命性的跨平台（Linux、Mac OS X、Windows）文本编辑器，它支持编写非常多的编程语言代码。对于Go而言，有一个插件叫做GoSublime来支持代码补全和代码模版。
- GoLand是一款非常出名的Go开发工具。

### 2.2 调试器

应用程序的开发过程中调试是必不可少的一个环节，因此有一个好的调试器是非常重要的，可惜的是，Go在这方面的发展还不是很完善。目前可用的调试器是gdb。如果不想使用调试器，可以按照下面的一些有用的方法来达到基本调试的目的：

- 在合适的位置使用打印语句输出相关变量的值（print/println 和fmt.Print/fmt.Println/fmt.Printf）。

- 在fmt.Printf中使用下面的说明符来打印有关变量的相关信息：
 - %+v 打印包括字段在内的实例的完整信息
 - %#v 打印包括字段和限定类型名称在内的实例的完整信息
 - %T 打印某个类型的完整说明

- 使用panic语句来获取栈跟踪信息（直到 panic 时所有被调用函数的列表）。

- 使用关键字defer来跟踪代码执行过程。

### 2.3 构建并运行Go程序

使用Go自带的更加方便的工具来构建应用程序：

- `go build` 编译并安装自身包和依赖包
- `go install` 安装自身包和依赖包

### 2.4 格式化代码

Go开发团队不想要Go语言像许多其它语言那样总是在为代码风格而引发无休止的争论，浪费大量宝贵的开发时间，因此制作了一个工具：go fmt（gofmt）。这个工具可以将的源代码格式化成符合官方统一标准的风格，属于语法风格层面上的小型重构。遵循统一的代码风格是Go开发中无可撼动的铁律，因此必须在编译或提交版本管理系统之前使用gofmt来格式化的代码。

尽管这种做法也存在一些争论，但使用gofmt后不再需要自成一套代码风格而是和所有人使用相同的规则。这不仅增强了代码的可读性，而且在接手外部Go项目时，可以更快地了解其代码的含义。此外，大多数开发工具也都内置了这一功能。

Go对于代码的缩进层级方面使用tab还是空格并没有强制规定，一个tab可以代表4个或8个空格。在实际开发中，1个tab应该代表4个空格。

在命令行输入`gofmt –w program.go`会格式化该源文件的代码然后将格式化后的代码覆盖原始内容（如果不加参数`-w`则只会打印格式化后的结果而不重写文件）；`gofmt -w *.go`会格式化并重写所有Go 源文件；`gofmt map1`会格式化并重写map1目录及其子目录下的所有Go源文件。

gofmt也可以通过在参数`-r`后面加入用双引号括起来的替换规则实现代码的简单重构，规则的格式：`<原始内容> -> <替换内容>`。

实例：

```go
   gofmt -r “(a) -> a” –w*.go
```

上面的代码会将源文件中没有意义的括号去掉。

```go
   gofmt -r “a[n:len(a)] -> a[n:]” –w *.go
```

上面的代码会将源文件中多余的`len(a)`去掉。

```go
   gofmt –r ‘A.Func1(a,b)-> A.Func2(b,a)’ –w *.go
```

上面的代码会将源文件中符合条件的函数的参数调换位置。

### 2.5 生成代码文档

godoc工具会从Go程序和包文件中提取顶级声明的首行注释以及每个对象的相关注释，并生成相关文档。它也可以作为一个提供在线文档浏览的web服务器，`http://golang.org`就是通过这种形式实现的。

一般用法：
- `go doc package` 获取包的文档注释，例如：`go doc fmt`会显示使用godoc生成的fmt包的文档注释。
- `go doc package/subpackage`获取子包的文档注释，例如：`go doc container/list`。
- `go doc package function`获取某个函数在某个包中的文档注释，例如：`go doc fmt Printf`会显示有关`fmt.Printf()`的使用说明。

这个工具只能获取在Go安装目录下`.../go/src`中的注释内容。此外，它还可以作为一个本地文档浏览web服务器。在命令行输入`godoc -http=:6060`，然后使用浏览器打开http://localhost:6060 后，就可以看到本地文档浏览服务器提供的页面。

godoc也可以用于生成非标准库的Go源码文件的文档注释。

### 2.6 其它工具

Go自带的工具集主要使用脚本和Go语言自身编写的，目前版本的Go实现了以下三个工具：

- `go install` 是安装Go包的工具，主要用于安装非标准库的包文件，将源代码编译成对象文件。
- `go fix` 用于将的Go代码从旧的发行版迁移到最新的发行版，它主要负责简单的、重复的、枯燥无味的修改工作，如果像API等复杂的函数修改，工具则会给出文件名和代码行数的提示以便让开发人员快速定位并升级代码。Go开发团队一般也使用这个工具升级Go内置工具以及谷歌内部项目的代码。go fix之所以能够正常工作是因为Go在标准库就提供生成抽象语法树和通过抽象语法树对代码进行还原的功能。该工具会尝试更新当前目录下的所有Go源文件，并在完成代码更新后在控制台输出相关的文件名称。
- `go test` 是一个轻量级的单元测试框架。

可以在终端里输入go，直接查看其帮助信息，以看到go支持的命令工具。
