### go语言中的一些小case

* 对于一些关乎性能的 JSON 解析的处理, 我们可以通过 json.RawMessage 进行性能的提升.

* Go语言的源码文件有三大类，即：命令源码文件、库源码文件和测试源码文件。他们的功用各不相同，而写法也各有各的特点。命令源码文件总是作为可执行的程序的入口。库源码文件一般用于集中放置各种待被使用的程序实体（全局常量、全局变量、接口、结构体、函数等等）。而测试源码文件主要用于对前两种源码文件中的程序实体的功能和性能进行测试。另外，后者也可以用于展现前两者中程序的使用方法。

* 代码包:

> 一个包可以在源文件中多次导入
>
> Go 源文件可以多次导入相同的包, 但导入名称必须不同. 这些相同的包导入引用相同的包实例.
>
> 例如:
>
> ```
> package main
>
> import "fmt"
> import "io"
> import inout "io"
>
> func main() {
>     fmt.Println(&inout.EOF == &io.EOF) // true
> }
> ```
>
> package mypkg 后的注释 // import “x.y.z.mypkg” 对于 Go 编译器是有意义的
>
> 例如, 当导入此包的源文件由标准 Go 编译器编译时, 以下包的导入路径必须为 “x.y.z.mypkg”.
>
> ```
> package mypkg // import "x.y.z.mypkg"
> ...
> ```

* 流程控制

> swith 和 select 中的 default 分支可以放在所有 case 分支之前, 之后, 或者之间.
>
> 与许多其他语言相比, 一个明显的区别在于 switch-case 控制流程块中默认分支的顺序可以是任意的. 例如, 以下三个 switch-case 控制流程块彼此等效.
>
> ```
> switch n := rand.Intn(3); n {
>     case 0: fmt.Println("n == 0")
>     case 1: fmt.Println("n == 1")
>     default: fmt.Println("n == 2")
> }
>
> switch n := rand.Intn(3); n {
>     default: fmt.Println("n == 2")
>     case 0: fmt.Println("n == 0")
>     case 1: fmt.Println("n == 1")
> }
>
> switch n := rand.Intn(3); n {
>     case 0: fmt.Println("n == 0")
>     default: fmt.Println("n == 2")
>     case 1: fmt.Println("n == 1")
> }
> ```
>
> defer 匿名函数可以修改嵌套函数的命名返回结果
>
> 例如:
>
> ```
> package main
>
> import "fmt"
>
> func Triple(n int) (r int) {
> 	defer func() {
> 		r += n // modify the return value
> 	}()
>
> 	return n + n // <=> r = n + n; return
> }
>
> func main() {
> 	fmt.Println(Triple(5)) // 15
> } 
> ```
>
> 对于切片 s, 循环 for i = range s {…} 不等于 for i = 0; i &lt; len\(s\); i++ {…},对于两个循环, 迭代变量 i 的相应最终值可能不同.
>
> ```
> package main
>
> import "fmt"
>
> var i int
>
> func fa(s []int, n int) int {
> 	i = n
> 	for i = 0; i < len(s); i++ {}
> 	return i
> }
>
> func fb(s []int, n int) int {
> 	i = n
> 	for i = range s {}
> 	return i
> }
>
> func main() {
> 	s := []int{2, 3, 5, 7, 11, 13}
> 	fmt.Println(fa(s, -1), fb(s, -1)) // 6 5
> 	s = nil
> 	fmt.Println(fa(s, -1), fb(s, -1)) // 0 -1
> }
> ```
>
> 使用 os.Exit 函数退出程序并使用 runtime.Goexit 函数退出 goroutine
>
> 我们可以通过调用 os.Exit 函数从任何函数中退出程序. os.Exit 函数调用取一个 int 码作为参数并且将该码返回给操作系统.
>
> 例如:
>
> ```
>  // exit-example.go
> package main
>
> import "os"
> import "time"
>
> func main() {
> 	go func() {
> 		time.Sleep(time.Second)
> 		os.Exit(1)
> 	}()
> 	select{}
> }
>
> $ go run a.go
> exit status 1
> $ echo $?
> 1
> ```
>
> 我们可以通过调用 runtime.Goexit 函数来退出一个 goroutine. runtime.Goexit 函数没有参数.
>
> 一个例子:
>
> ```
> package main
>
> import "fmt"
> import "runtime"
>
> func main() {
> 	c := make(chan int)
> 	go func() {
> 		defer func() {c <- 1}()
> 		defer fmt.Println("Go")
> 		func() {
> 			defer fmt.Println("C")
> 			runtime.Goexit()
> 		}()
> 		fmt.Println("Java")
> 	}()
> 	<-c
> } 
> ```



