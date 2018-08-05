## Go的可移植性

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

Go官方似乎没有给出明确的porting guide，关于将Go语言porting到其他平台上的内容更多是在[golang-dev](https://groups.google.com/forum/#!forum/golang-dev)这样的小圈子中讨论的事情。但就Go语言这么短的时间就能很好的支持这么多平台来看，Go的porting还是相对easy的。从个人对Go的了解来看，这一定程度上得益于Go独立实现了runtime

