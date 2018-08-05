## Go的可移植性

[https://tonybai.com/2017/06/27/an-intro-about-go-portability/\#comment-7247](https://tonybai.com/2017/06/27/an-intro-about-go-portability/#comment-7247) 原创

说到一门编程语言可移植性，我们一般从下面两个方面考量：

* 语言自身被移植到不同平台的容易程度
* 通过这种语言编译出来的应用程序对平台的适应性

```
go tool dist list //go 1.10
android/386
android/amd64
android/arm
android/arm64
darwin/386
darwin/amd64
darwin/arm
darwin/arm64
dragonfly/amd64
freebsd/386
freebsd/amd64
freebsd/arm
linux/386
linux/amd64
linux/arm
linux/arm64
linux/mips
linux/mips64
linux/mips64le
linux/mipsle
linux/ppc64
linux/ppc64le
linux/s390x
nacl/386
nacl/amd64p32
nacl/arm
netbsd/386
netbsd/amd64
netbsd/arm
openbsd/386
openbsd/amd64
openbsd/arm
plan9/386
plan9/amd64
plan9/arm
solaris/amd64
windows/386
windows/amd64
```

Go官方似乎没有给出明确的porting guide，关于将Go语言porting到其他平台上的内容更多是在[golang-dev](https://groups.google.com/forum/#!forum/golang-dev)这样的小圈子中讨论的事情。但就Go语言这么短的时间就能很好的支持这么多平台来看，Go的porting还是相对easy的。从个人对Go的了解来看，这一定程度上得益于Go独立实现了runtime：

![](/assets/import.png)

runtime是支撑程序运行的基础。我们最熟悉的莫过于libc（C运行时），它是目前主流操作系统上应用最普遍的运行时，通常以

[动态链接库](http://tonybai.com/2010/12/13/also-talk-about-shared-library/)的形式\(比如：/lib/x86\_64-linux-gnu/libc.so.6\)随着系统一并发布，它的功能大致有如下几个：

```
提供基础库函数调用，比如：strncpy；
封装syscall（注:syscall是操作系统提供的API口，当用户层进行系统调用时，代码会trap(陷入)到内核层面执行），并提供同语言的库函数调用，
比如：malloc、fread等；
提供程序启动入口函数，比如：linux下的__libc_start_main。
```

[libc](http://tonybai.com/2006/07/08/plauger-c-standard-lib-assert-header/)等c runtime lib是很早以前就已经实现的了，甚至有些老旧的libc还是单线程的。一些从事c/c++开发多年的程序员早年估计都有过这样的经历：那就是链接runtime库时甚至需要选择链接支持多线程的库还是只支持单线程的库。除此之外，c runtime的版本也参差不齐。这样的c runtime状况完全不能满足go语言自身的需求；另外Go的目标之一是原生支持并发，并使用[goroutine模型](http://tonybai.com/2017/06/23/an-intro-about-goroutine-scheduler/)，c runtime对此是无能为力的，因为c runtime本身是基于线程模型的。综合以上因素，Go自己实现了runtime，并封装了syscall，为不同平台上的go user level代码提供封装完成的、统一的go标准库；同时Go runtime实现了对goroutine模型的支持。

独立实现的go runtime层将Go user-level code与OS syscall解耦，把Go porting到一个新平台时，将runtime与新平台的syscall对接即可\(当然porting工作不仅仅只有这些\)；同时，runtime层的实现基本摆脱了Go程序对libc的依赖，这样静态编译的Go程序具有很好的平台适应性。比如：一个compiled for linux amd64的Go程序可以很好的运行于不同linux发行版（centos、ubuntu）下。

```
otool工具(linux上可以用ldd)可以用来查看两个文件的对外部动态库的依赖情况
nm工具可以查看到具体是哪个函数符号需要由外部动态库提供
```



