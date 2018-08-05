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

#### cgo对可移植性的影响

默认情况下，Go的runtime环境变量CGO\_ENABLED=1，即默认开始cgo，允许你在Go代码中调用C代码，Go的pre-compiled标准库的.a文件也是在这种情况下编译出来的。在$GOROOT/pkg/darwin\_amd64中，我们遍历所有预编译好的标准库.a文件，并用nm输出每个.a的未定义符号，我们看到下面一些包是对外部有依赖的（动态链接）

```
=> crypto/x509.a
                 U _CFArrayGetCount
                 U _CFArrayGetValueAtIndex
                 U _CFDataAppendBytes
                 ... ...
                 U _SecCertificateCopyNormalizedIssuerContent
                 U _SecCertificateCopyNormalizedSubjectContent
                 ... ...
                 U ___stack_chk_fail
                 U ___stack_chk_guard
                 U __cgo_topofstack
                 U _kCFAllocatorDefault
                 U _memcmp
                 U _sysctlbyname

=> net.a
                 U ___error
                 U __cgo_topofstack
                 U _free
                 U _freeaddrinfo
                 U _gai_strerror
                 U _getaddrinfo
                 U _getnameinfo
                 U _malloc

=> os/user.a
                 U __cgo_topofstack
                 U _free
                 U _getgrgid_r
                 U _getgrnam_r
                 U _getgrouplist
                 U _getpwnam_r
                 U _getpwuid_r
                 U _malloc
                 U _realloc
                 U _sysconf

=> plugin.a
                 U __cgo_topofstack
                 U _dlerror
                 U _dlopen
                 U _dlsym
                 U _free
                 U _malloc
                 U _realpath$DARWIN_EXTSN

=> runtime/cgo.a
                 ... ...
                 U _abort
                 U _fprintf
                 U _fputc
                 U _free
                 U _fwrite
                 U _malloc
                 U _nanosleep
                 U _pthread_attr_destroy
                 U _pthread_attr_getstacksize
                 ... ...
                 U _setenv
                 U _strerror
                 U _unsetenv

=> runtime/race.a
                 U _OSSpinLockLock
                 U _OSSpinLockUnlock
                 U __NSGetArgv
                 U __NSGetEnviron
                 U __NSGetExecutablePath
                 U ___error
                 U ___fork
                 U ___mmap
                 U ___munmap
                 U ___stack_chk_fail
                 U ___stack_chk_guard
                 U __dyld_get_image_header
                .... ...
```

我们以os/user为例，在CGO\_ENABLED=1，即cgo开启的情况下，os/user包中的lookupUserxxx系列函数采用了c版本的实现，我们看到在$GOROOT/src/os/user/lookup\_unix.go中的build tag中包含了**+build cgo**。这样一来，在CGO\_ENABLED=1，该文件将被编译，该文件中的c版本实现的lookupUser将被使用

```
// +build darwin dragonfly freebsd !android,linux netbsd openbsd solaris
// +build cgo

package user
... ...
func lookupUser(username string) (*User, error) {
    var pwd C.struct_passwd
    var result *C.struct_passwd
    nameC := C.CString(username)
    defer C.free(unsafe.Pointer(nameC))
    ... ...
}
```

这样来看，凡是依赖上述包的Go代码最终编译的可执行文件都是要有外部依赖的。不过我们依然可以通过disable CGO\_ENABLED来编译出纯静态的Go程序：

```
$CGO_ENABLED=0 go build -o server_cgo_disabled server.go

$otool -L server_cgo_disabled
server_cgo_disabled:
$nm server_cgo_disabled |grep " U "
```

如果你使用build的 “-x -v”选项，你将看到go compiler会重新编译依赖的包的静态版本，包括net、mime/multipart、crypto/tls等，并将编译后的.a\(以包为单位\)放入临时编译器工作目录\($WORK\)下，然后再静态连接这些版本。

#### internal linking和external linking

在CGO\_ENABLED=1这个默认值的情况下，是否可以实现纯静态连接呢？答案是可以。在$GOROOT/cmd/cgo/doc.go中，文档介绍了cmd/link的两种工作模式：internal linking和external linking。

* ### internal linking

> internal linking的大致意思是若用户代码中仅仅使用了net、os/user等几个标准库中的依赖cgo的包时，cmd/link默认使用internal linking，而无需启动外部external linker\(如:gcc、clang等\)，不过由于cmd/link功能有限，仅仅是将.o和pre-compiled的标准库的.a写到最终二进制文件中。因此如果标准库中是在CGO\_ENABLED=1情况下编译的，那么编译出来的最终二进制文件依旧是动态链接的，即便在go build时传入`-ldflags ‘extldflags “-static”‘`亦无用，因为根本没有使用external linker：
>
> ```
> $go build -o server-fake-static-link  -ldflags '-extldflags "-static"' server.go
> $otool -L server-fake-static-link
> server-fake-static-link:
>     /usr/lib/libSystem.B.dylib (compatibility version 0.0.0, current version 0.0.0)
>     /System/Library/Frameworks/CoreFoundation.framework/Versions/A/CoreFoundation (compatibility version 0.0.0, current version 0.0.0)
>     /System/Library/Frameworks/Security.framework/Versions/A/Security (compatibility version 0.0.0, current version 0.0.0)
>     /usr/lib/libSystem.B.dylib (compatibility version 0.0.0, current version 0.0.0)
>     /usr/lib/libSystem.B.dylib (compatibility version 0.0.0, current version 0.0.0)
> ```

* ### external linking

> 而external linking机制则是cmd/link将所有生成的.o都打到一个.o文件中，再将其交给外部的链接器，比如[gcc](http://tonybai.com/tag/gcc)或clang去做最终链接处理。如果此时，我们在cmd/link的参数中传入`-ldflags ‘extldflags “-static”‘`，那么gcc/clang将会去做静态链接，将.o中undefined的符号都替换为真正的代码。我们可以通过-linkmode=external来强制cmd/link采用external linker，还是以server.go的编译为例：
>
> ```
> $go build -o server-static-link  -ldflags '-linkmode "external" -extldflags "-static"' server.go
> # command-line-arguments
> /Users/tony/.bin/go18/pkg/tool/darwin_amd64/link: running clang failed: exit status 1
> ld: library not found for -lcrt0.o
> clang: error: linker command failed with exit code 1 (use -v to see invocation)
> ```

> 可以看到，cmd/link调用的clang尝试去静态连接libc的.a文件，但由于我的mac上仅仅有libc的dylib，而没有.a，因此静态连接失败。我找到一个ubuntu 16.04环境：重新执行上述构建命令：
>
> ```
> # go build -o server-static-link  -ldflags '-linkmode "external" -extldflags "-static"' server.go
> # ldd server-static-link
>     not a dynamic executable
> # nm server-static-link|grep " U "
> ```

> 就这样，我们在CGO\_ENABLED=1的情况下，也编译构建出了一个纯静态链接的Go程序。
>
> 如果你的代码中使用了C代码，并依赖cgo在go中调用这些c代码，那么cmd/link将会自动选择external linking的机制
>
> ```
> //testcgo.go
> package main
>
> //#include <stdio.h>
> // void foo(char *s) {
> //    printf("%s\n", s);
> // }
> // void bar(void *p) {
> //    int *q = (int*)p;
> //    printf("%d\n", *q);
> // }
> import "C"
> import (
>     "fmt"
>     "unsafe"
> )
>
> func main() {
>     var s = "hello"
>     C.foo(C.CString(s))
>
>     var i int = 5
>     C.bar(unsafe.Pointer(&i))
>
>     var i32 int32 = 7
>     var p *uint32 = (*uint32)(unsafe.Pointer(&i32))
>     fmt.Println(*p)
> }
> ```

> 编译testcgo.go：
>
> ```
> # go build -o testcgo-static-link  -ldflags '-extldflags "-static"' testcgo.go
> # ldd testcgo-static-link
>     not a dynamic executable
>
> vs.
> # go build -o testcgo testcgo.go
> # ldd ./testcgo
>     linux-vdso.so.1 =>  (0x00007ffe7fb8d000)
>     libpthread.so.0 => /lib/x86_64-linux-gnu/libpthread.so.0 (0x00007fc361000000)
>     libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007fc360c36000)
>     /lib64/ld-linux-x86-64.so.2 (0x000055bd26d4d000)
> ```

> 小结：
>
> ```
> 你的程序用了哪些标准库包？如果仅仅是非net、os/user等的普通包，那么你的程序默认将是纯静态的，不依赖任何c lib等外部动态链接库；
> 如果使用了net这样的包含cgo代码的标准库包，那么CGO_ENABLED的值将影响你的程序编译后的属性：是静态的还是动态链接的；
> CGO_ENABLED=0的情况下，Go采用纯静态编译；
> 如果CGO_ENABLED=1，但依然要强制静态编译，需传递-linkmode=external给cmd/link。
> ```



