---
layout: post
title:  "Go教程13-测试和性能优化"
date:   2018-04-25 08:08:04
categories: Go语言编程
tags: Go 
excerpt: 所有的包都应该有一定的必要文档，然后同样重要的是对包的测试。
---

* content
{:toc}


## 1 Go 中的单元测试和基准测试

首先所有的包都应该有一定的必要文档，然后同样重要的是对包的测试。

Go的测试工具 gotest。

名为 testing 的包被专门用来进行自动化测试，日志和错误报告。并且还包含一些基准测试函数的功能。

备注：gotest 是 Unix bash 脚本，所以在 Windows 下需要配置 MINGW 环境；在 Windows 环境下把所有的`pkg/linux_amd64`替换成 pkg/windows。

对一个包做（单元）测试，需要写一些可以频繁（每次更新后）执行的小块测试单元来检查代码的正确性。于是必须写一些 Go 源文件来测试代码。测试程序必须属于被测试的包，并且文件名满足这种形式`*_test.go`，所以测试代码和包中的业务代码是分开的。

`_test`程序不会被普通的 Go 编译器编译，所以当放应用部署到生产环境时它们不会被部署；只有 gotest 会编译所有的程序：普通程序和测试程序。

测试文件中必须导入 "testing" 包，并写一些名字以 TestZzz 打头的全局函数，这里的 Zzz 是被测试函数的字母描述，如 TestFmtInterface，TestPayEmployees 等。

测试函数必须有这种形式的头部：

```go
   func TestAbcde(t *testing.T)
```

T是传给测试函数的结构类型，用来管理测试状态，支持格式化测试日志，如 t.Log，t.Error，t.ErrorF 等。在函数的结尾把输出跟想要的结果对比，如果不等就打印一个错误。成功的测试则直接返回。

用下面这些函数来通知测试失败：

1）`func (t *T)Fail()`

标记测试函数为失败，然后继续执行（剩下的测试）。

2）`func (t *T)FailNow()`

标记测试函数为失败并中止执行；文件中别的测试也被略过，继续执行下一个文件。

3）`func (t *T)Log(args ...interface{})`

args 被用默认的格式格式化并打印到错误日志中。

4）`func (t *T)Fatal(args ...interface{})`

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

命令`go test –test.bench=.*`会运行所有的基准测试函数；代码中的函数会被调用 N 次（N是非常大的数，如 N =1000000），并展示 N 的值和函数执行的平均时间，单位为ns（纳秒，ns/op）。如果是用testing.Benchmark 调用这些函数，直接运行程序即可。

## 2 测试的具体例子

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

## 3 用（测试数据）表驱动测试

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

## 4 性能调试：分析并优化Go程序
### 4.1 时间和内存消耗

可以用这个便捷脚本 xtime 来测量：

```shell
   #!/bin/sh

   /usr/bin/time -f ‘%Uu %Ss %er %MkB %C’ “$@”
```

在Unix 命令行中像这样使用 xtime goprogexec，这里的 progexec 是一个 Go 可执行程序，这句命令行输出类似：56.63u 0.26s 56.92r 1642640kB progexec，分别对应用户时间，系统时间，实际时间和最大内存占用。

### 4.2 用go test调试

如果代码使用了 Go 中 testing 包的基准测试功能，可以用 gotest 标准的 -cpuprofile 和 -memprofile 标志向指定文件写入 CPU 或 内存使用情况报告。

使用方式：go test -x -v -cpuprofile=prof.out -file x_test.go

编译执行 x_test.go 中的测试，并向 prof.out 文件中写入 cpu 性能分析信息。

### 4.3 用pprof调试

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

## 5 参考资料
- https://github.com/Unknwon/the-way-to-go_ZH_CN/
