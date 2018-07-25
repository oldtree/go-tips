# go语言的nil

nil 是 Go 中一个熟悉而重要的预定义标识符. 它是很多类型零值\(zero values\)的字面量表示. 很多有一些其他流行语言经验的 Go 程序员新手视 nil 为其他语言 null\(或者 NULL\)的副本. 这是部分正确的, 但是 Go 中的 nil 和其他语言的 null\(或者 NULL\)有很多不同之处.

### Go 中的 nil 是一个预定义标识符 {#Go-中的-nil-是一个预定义标识符}

### nil 可以表示很多类型的零值 {#nil-可以表示很多类型的零值}

在 Go 中, nil 可以表示以下类型的零值:

* > * pointer \(包括类型不安全的\)
  > * map
  > * slice
  > * function
  > * channel
  > * interface

换句话说, 在 Go 中, nil 可能是许多不同类型的值.

### nil 不是默认类型 {#nil-不是默认类型}

Go 中的每个其他预定义标识符都有一个默认类型. 比如,

* > * true 和 false 的默认类型都是 bool 类型.
  > * iota 的默认类型是 int.

但是 nil 没有默认类型, 尽管它有很多可能的类型. 编译器必须有足够的信息来从上下文中推导出 nil 的类型.

例如：

> ```
> package main
>
> func main() {
>     // 以下代码行不能编译.
>     /*
>         v := nil
>     */
>
>     // 编译器必须有足够的信息才可以推断 nil 的类型.
>     _ = (*struct{})(nil)
>     _ = []int(nil)
>     _ = map[int]bool(nil)
>     _ = chan string(nil)
>     _ = (func())(nil)
>     _ = interface{}(nil)
>
>     // 这些代码行和上面的代码行等价.
>     var _ *struct{} = nil
>     var _ []int = nil
>     var _ map[int]bool = nil
>     var _ chan string = nil
>     var _ func() = nil
>     var _ interface{} = nil
> }
> ```

### Go 中的 nil 不是一个关键字 {#Go-中的-nil-不是一个关键字}

预定义的 nil 可以被覆盖\(shadowed\).

> ```
> package main
>
> import "fmt"
>
> func main() {
>     nil := 123
>     fmt.Println(nil) // 123
>
>     // 如下代码行会产生编译错误, 因为当前作用域 nil 代表一个 nil 值.
>     /*
>     var _ map[string]int = nil
>     */
> }
> ```

### 不同类型的 nil 值的大小\(Size\)可能不一样 {#不同类型的-nil-值的大小-Size-可能不一样}

一种类型的所有值的内存布局总是相同的. 该类型的 nil 值也不例外. nil 的大小总是与相同类型的非 nil 的大小相同. 因此, 表示不同类型的不同零值的 nil 标识符可能具有不同的大小.

> ```
> package main
>
> import (
>     "fmt"
>     "unsafe"
> )
>
> func main() {
>     var p *struct{} = nil
>     fmt.Println(unsafe.Sizeof(p)) // 8
>
>     var s []int = nil
>     fmt.Println(unsafe.Sizeof(s)) // 24, 译注: slice 的实现包含一个 array(unsafe.Pointer 类型), 一个 len (int 类型) 以及一个 cap (int 类型), 所以 size 为 24, 参见: https://golang.org/src/runtime/slice.go#L11
>
>     var m map[int]bool = nil
>     fmt.Println(unsafe.Sizeof(m)) // 8
>
>     var c chan string = nil
>     fmt.Println(unsafe.Sizeof(c)) // 8
>
>     var f func() = nil
>     fmt.Println(unsafe.Sizeof(f)) // 8
>
>     var i interface{} = nil
>     fmt.Println(unsafe.Sizeof(i)) // 16, 译注: interface{} 的内部实现是一个 eface 类型, 包含一个 _type 指针类型 以及一个 data unsafe.Pointer 类型, 所以 size 为 16, 参见: https://golang.org/src/runtime/runtime2.go#L148
> }
> ```

大小是编译器和架构相关的. 上述打印结果适用于 64 位架构和标准 Go 编译器. 对于 32 位架构, 打印大小将减半. 对于标准的 Go 编译器, 不同种类的相同类型的两个 nil 值的大小总是相同的. 例如, 两个不同 slice 类型 \[\]int 和 \[\]string 的两个 nil 值的大小是相同的.

### 两种不同类型的两个 nil 值可能不可比较 {#两种不同类型的两个-nil-值可能不可比较}

> ```
> // 如下代码无法编译.
> var _ = (*int)(nil) == (*bool)(nil)         // error: mismatched types.
> var _ = (chan int)(nil) == (chan bool)(nil) // error: mismatched types.
> ```

在 Go 中, 两种不同可比较类型的两个值仅当其中一个隐式的转化为另一个类型的时候才可以比较. 具体而言, 有三种情况不同可比较的两个值才可以比较:

1. > 1. 两个值中的一个的类型是另一个的底层类型.
   > 2. 两个值之一的类型实现另一个值的类型\(必须是接口类型\).
   > 3. 两个值中的一个的类型是定向 channel 类型, 另一个是双向 channel 类型, 两种类型具有相同的元素类型，并且两种类型中的一种不是定义的类型.

nil 值也不例外.

下面例子中的代码行都可以编译.

> ```
> type IntPtr *int
> // IntPtr 的底层类型是 *int.
> var _ = IntPtr(nil) == (*int)(nil)
>
> // Go 中每一个类型都实现了空 interface{} 类型.
> var _ = (interface{})(nil) == (*int)(nil)
>
> // 定向 channel 类型的值可以转为一个具有相同元素类型的双向 channel 类型.
> var _ = (chan int)(nil) == (chan<- int)(nil)
> var _ = (chan int)(nil) == (<-chan int)(nil)
> ```

### 相同类型的两个 nil 值可能也不可比较 {#相同类型的两个-nil-值可能也不可比较}

在 Go 中, map, slice, function 类型不支持比较. 所以比较两个不可比较类型的两个 nil 是非法的

> ```
> // 以下代码行无法通过编译.
> var _ = ([]int)(nil) == ([]int)(nil)
> var _ = (map[string]int)(nil) == (map[string]int)(nil)
> var _ = (func())(nil) == (func())(nil)
> ```

但是, 上述无法比较的类型的任何值都可以与裸 nil 标识符进行比较. \(_译注：_这个地方有点难理解, 转化后的 nil 本质上已经具有类型了, 所以相当于两种具有相同类型的 nil 在比较, 在这里不可比较, 编译报错, 而一个零值类型与 nil 的比较是单纯的值比较\)

### 两个 nil 值可能不相等 {#两个-nil-值可能不相等}

如果两个要比较的 nil 值之一是一个接口值, 另一个不是, 假设它们是可比较的, 那么比较结果总是 false 的. 原因是进行比较之前, 非接口值将被转换为接口值的类型. 转换的接口值具有一个具体的动态类型, 但其他接口值没有. 这就是为什么比较结果总是 false 的原因. \(_译注_: 这个其实说的是比较的时候 \(\*int\)\(nil\) 会转化为一个 interface {} 类型, 这样这个接口就具有了明确的类型和值 nil, 我们知道 go 的接口实现是类型和值两个部分, \(interface{}\)\(nil\) 这个的类型和值都是 nil, 所以比较总是 false\)

例如:

> ```
> fmt.Println((interface{})(nil) == (*int)(nil)) // false
> ```

### 从 nil map 检索元素不会 panic {#从-nil-map-检索元素不会-panic}

从 nil map 检索元素总是会返回元素类型的零值. \(_译注_: map 是引用类型, nil map 不指向一个初始化的 map, 所以内存读会是零值, 写入一个 nil map 会 panic\)

例如:

> ```
> fmt.Println((map[string]int)(nil)["key"]) // 0
> fmt.Println((map[int]bool)(nil)[123])     // false
> fmt.Println((map[int]*int64)(nil)[123])   // <nil>
> ```

### 遍历 nil channel, map, slice 和 array 指针是合法的 {#遍历-nil-channel-map-slice-和-array-指针是合法的}

遍历 nil map 和 slice 的循环次数是 0.

遍历一个 nil array 指针的循环次数是它对应数组类型的长度. \(但是, 如果相应数组类型的长度不为 0, 并且迭代的第二个变量既不忽略也不省略, 那么迭代在运行时将会 panic._译注_: 这个可以看下面的增加注解代码进行理解\)

遍历一个 nil channel 将永久 block.

例如, 以下的代码将打印出 0, 1, 2, 3, 和 4, 然后将永远 block. Hello, word 和 Bye 不会打印出来.

> ```
> for range []int(nil) {
>     fmt.Println("Hello")
> }
>
> for range map[string]string(nil) {
>     fmt.Println("world")
> }
>
> for i,_ := range (*[5]int)(nil) {
>     fmt.Println(i)
> }
>
> // 译加: 以下代码将 panic
> // for i, v := range (*[5]int)(nil) {
> //     fmt.Println(i, v)
> // }
>
> for range chan bool(nil) { // 这里 block
>     fmt.Println("Bye")
> }
> ```

### 通过非接口 nil 参数调用方法将不会 panic {#通过非接口-nil-参数调用方法将不会-panic}

> ```
> package main
>
> type Slice []bool
>
> func (s Slice) Length() int {
>     return len(s)
> }
>
> func (s Slice) Modify(i int, x bool) {
>     s[i] = x // panic if s is nil
> }
>
> func (p *Slice) DoNothing() {
> }
>
> func (p *Slice) Append(x bool) {
>     *p = append(*p, x) // panic if p is nil
> }
>
> func main() {
>     // 如下选择器将不会 panic.
>     _ = ((Slice)(nil)).Length
>     _ = ((Slice)(nil)).Modify
>     _ = ((*Slice)(nil)).DoNothing
>     _ = ((*Slice)(nil)).Append
>
>     // 以下两行将不会 panic.
>     _ = ((Slice)(nil)).Length()
>     ((*Slice)(nil)).DoNothing()
>
>     // 如下两行将会 panic. 但是 panics 在调用方法时不会触发. 将在方法体内触发.
>     /*
>         ((Slice)(nil)).Modify(0, true)
>         ((*Slice)(nil)).Append(true)
>     */
> }
> ```

### 如果类型 T 的零值可以表示为 nil, 那么 \*new\(T\) 和 nil 相等 {#如果类型-T-的零值可以表示为-nil-那么-new-T-和-nil-相等}

> ```
> package main
>
> import "fmt"
>
> func main() {
>     fmt.Println(*new(*int) == nil)         // true
>     fmt.Println(*new([]int) == nil)        // true
>     fmt.Println(*new(map[int]bool) == nil) // true
>     fmt.Println(*new(chan string) == nil)  // true
>     fmt.Println(*new(func()) == nil)       // true
>     fmt.Println(*new(interface{}) == nil)  // true
> }
> ```

### nil channels

> 每个写过 Go 的人都知道 channels. 我们大多数人也知道 channels 的默认值是 nil. 但是我们很多人都不知道这个 nil 值是有用的. \(译注: 老实说读到这篇文章之前我也认为 nil channels 没什么用, 不仅没用, 我还觉得对 nil channels 的写入操作应该 panic 才对, 就像对 nil map 的写入是 panic 一样, 甚至认为这是 Go 的设计不一致问题\)
>
> 给定一个 nil channel c:
>
> ```
> 1：<-c 从 c 接收将永远阻塞
> 2：c <- v 发送值到 c 会永远阻塞
> 3：close(c) 关闭 c 引发 panic
> ```

* ##### 合并 channels

> 如果你选择接受它, 你的任务是编写一个函数, 给定两个 channels a 和 b 返回一个相同类型的 channel c. a 或 b 中收到的每个元素都将发送给 c, 并且一旦 a 和 b 都关闭, c 也将被关闭.
>
> 一个辅助函数：
>
> 在我们开始之前, 让我们编写一个函数来帮助我们测试我们的解决方案. 此函数返回一个 channel , 该 channel 最终将随机接收所有给定的值并通过关闭完成.
>
> ```
> func asChan(vs ...int) <-chan int {
>     c := make(chan int)
>     go func() {
>         for _, v := range vs {
>             c <- v
>             time.Sleep(time.Duration(rand.Intn(1000)) * time.Millisecond)
>         }
>         close(c)
>     }()
>     return c
> }
> ```
>
> 此函数创建一个 channel c, 启动一个新的 goroutine, 将值发送到创建的 channel c, 最后返回 channel c.在处理 channels 时这是很常见的模式, 因此在继续阅读之前, 请确保你了解它的工作原理.由于我们没有对 a 或 b 的偏好, 所以我们将避免通过选择我们应该首先 range 哪个 channel 来创建偏好.相反我们会保持这种对称性, 并且使用一个无限循环来 select 这两个 channels.
>
> ```
> func merge(a, b <-chan int) <-chan int {
>     c := make(chan int)
>     go func() {
>         for {
>             select {
>             case v := <-a:
>                 c <- v
>             case v := <-b:
>                 c <- v
>             }
>         }
>     }()
>     return c
> }
> func main() {
>     a := asChan(1, 3, 5, 7)
>     b := asChan(2, 4, 6, 8)
>     c := merge(a, b)
>     for v := range c {
>         fmt.Println(v)
>     }
> }
> ```
>
> 这应该以某种顺序打印 1 到 8 并成功结束. 让我们看看发生了什么.
>
> ```
> > go run main.go
> 1
> 2
> 3
> 4
> 5
> 6
> 7
> 8
> 0
> 0
> 0
> 0
> 0
> 0
> 0
> //好吧, 很明显这不好, 因为该程序没有结束. 一旦它打印出从 1 到 8 的值, 它将开始永远打印 0
> ```

* ##### 处理关闭的 channels

> 如果我们从一个关闭的 channel 接收会发生什么 ? 我们会得到 channel 类型的默认值. 在我们的例子中, 类型是 int, 所以值是 0.
>
> 我们可以通过与 0 比较来检查是否 channel 已经关闭, 但是如果我们接收到的其中一个值是 0, 会怎么样 ? 相反, 我们可以使用 “v, ok” 语法:
>
> ```
> v, ok := <- c
> ```
>
> 当使用这个语法时, ok 是一个布尔值, 只要 channel 是开着的, 它就是 true. 知道这一点, 我们可以避免将多余的 0 发送给 c .
>
> 在某一点上我们也应该停止迭代, 所以让我们也跟踪两个 channels 何时关闭.
>
> ```
> func merge(a, b <-chan int) <-chan int {
>     c := make(chan int)
>     go func() {
>         adone, bdone := false, false
>         for !adone || !bdone {
>             select {
>             case v, ok := <-a:
>                 if !ok {
>                     adone = true
>                     continue
>                 }
>                 c <- v
>             case v, ok := <-b:
>                 if !ok {
>                     bdone = true
>                     continue
>                 }
>                 c <- v
>             }
>         }
>     }()
>     return c
> }
> > go run main.go
> 1
> 2
> 3
> 4
> 5
> 6
> 7
> 8
> fatal error: all goroutines are asleep - deadlock!
> goroutine 1 [chan receive]:
> main.main()
>         /Users/francesc/src/github.com/campoy/campoy.cat/site/static/code/nilchans/main.go:13 +0x186
> exit status 2
> ```
>
> 可以看到只有一个 goroutine 在运行, 并且它阻塞在第 13 行
>
> ```
> for v := range c {
> ```
>
> 能看出问题是什么吗 ? range 语句迭代 channel 中的所有值直到 channel 关闭. 但是谁关闭了这个 channel ?应该在 goroutine 中添加 defer 语句, 以确保该 channel 最终关闭.
>
> ```
> func merge(a, b <-chan int) <-chan int {
>     c := make(chan int)
>     go func() {
>         defer close(c)
>         adone, bdone := false, false
>         for !adone || !bdone {
>             select {
>             case v, ok := <-a:
>                 if !ok {
>                     adone = true
>                     continue
>                 }
>                 c <- v
>             case v, ok := <-b:
>                 if !ok {
>                     bdone = true
>                     continue
>                 }
>                 c <- v
>             }
>         }
>     }()
>     return c
> }
> ```
>
> 请注意, defer 语句位于新的 goroutine 中调用的匿名函数中, 而不是在 merge 中. 否则, 只要我们退出 merage, c 就会被关闭, 那么发送一个值给它将引发 panic.
>
> 运行它, 看看会发生什么.
>
> ```
> > go run main.go
> 1
> 2
> 3
> 4
> 5
> 6
> 7
> 8
> ```

* ##### 繁忙的循环

> 为了显示问题所在, 添加一些日志记录.
>
> ```
> func merge(a, b <-chan int) <-chan int {
>     c := make(chan int)
>     go func() {
>         defer close(c)
>         adone, bdone := false, false
>         for !adone || !bdone {
>             select {
>             case v, ok := <-a:
>                 if !ok {
>                     log.Println("a is done")
>                     adone = true
>                     continue
>                 }
>                 c <- v
>             case v, ok := <-b:
>                 if !ok {
>                     log.Println("b is done")
>                     bdone = true
>                     continue
>                 }
>                 c <- v
>             }
>         }
>     }()
>     return c
> }
> ```
>
> 从一个关闭的 channel 读取从不阻塞.
>
> 因此, 只要两个 channels 都处于打开状态, select 语句将会阻塞, 直到新元素准备就绪, 但是一旦其中一个关闭, 我们将迭代并浪费 CPU. 这也被称为繁忙的循环, 并不好.

* ##### 在 select 语句中禁用一个 case

> 为了避免之前描述的繁忙循环, 希望禁用 select 语句的一部分. 具体来说, 当 a 关闭的时候想移除 \(case v, ok := &lt;- a\), b 也一样. 但是怎么做呢 ?
>
> 正如在开头提到的那样, 从 nil channels 接收数据会永远阻塞. 所以为了禁用一个从 channel 接收数据的 case, 可以简单将 channel 设置为 nil !
>
> 然后, 可以停止使用 adone 和 bdone, 而是检查 a 和 b 是否为 nil .
>
> ```
> func merge(a, b <-chan int) <-chan int {
>     c := make(chan int)
>     go func() {
>         defer close(c)
>         for a != nil || b != nil {
>             select {
>             case v, ok := <-a:
>                 if !ok {
>                     fmt.Println("a is done")
>                     a = nil
>                     continue
>                 }
>                 c <- v
>             case v, ok := <-b:
>                 if !ok {
>                     fmt.Println("b is done")
>                     b = nil
>                     continue
>                 }
>                 c <- v
>             }
>         }
>     }()
>     return c
> }
> > go run main.go
> 2
> 1
> 4
> 3
> 6
> 5
> 8
> 7
> b is done
> a is done
> ```



