* ### 不要通过共享内存进行通信, 通过通信共享内存 \([Don’t communicate by sharing memory, share memory by communicating](https://www.bilibili.com/video/av18889438/?t=2m48s)\) {#不要通过共享内存进行通信-通过通信共享内存-Don’t-communicate-by-sharing-memory-share-memory-by-communicating}

> 传统的线程模型（通常在编写 Java, C++ 和 Python 程序时使用）要求程序员使用共享内存在线程之间进行通信. 通常, 共享数据结构受锁保护, 线程将争夺这些锁访问数据, 在某些情况下, 通过使用 Python 的 Queue 等线程安全的数据结构可以使这变得更容易.
>
> Go 的并发原语 \(goroutines 和 channels\) 为构造并发软件提供了一种优雅而独特的手段. \(这些概念有一个有趣的历史, 要从 C.A.R.Hoare 的通信顺序进程说起.\) Go 鼓励使用 channels 在 goroutines 之间传递对数据的引用, 而不是显式地使用锁来调解对共享数据的访问. 这种方法确保只有一个 goroutine 可以在给定的时间访问数据. 这个概念总结在 Effective Go 文档中 \(任何 Go 程序员都必须阅读\).

* ### 并发不是并行 \([Concurrency is not parallelism](https://www.bilibili.com/video/av18889438/?t=3m42s)\) {#并发不是并行-Concurrency-is-not-parallelism}

> 当人们听到 _**并发** _这个词的时候, 他们经常会想到并行, 这是一个相关的, 但非常独特的概念. 在编程中, 并发是独立执行的进程的组成, 而并行则是 \(可能相关的\) 计算的同时执行. 并发是一次处理很多事情. 并行是一次做很多事情.

* ### Channels 重排序; 互斥量串行化 \([Channels orchestrate; mutexes serialize](https://www.bilibili.com/video/av18889438/?t=4m20s)\) {#Channels-重排序-互斥量串行化-Channels-orchestrate-mutexes-serialize}

> ```
> package main
>
> import "fmt"
>
> func sum(s []int, c chan int) {
>     sum := 0
>     for _, v := range s {
>         sum += v
>     }
>     c <- sum // 此处如果改成互斥量一样可以做到
> }
>
> func main() {
>     s := []int{7, 2, 8, -9, 4, 0}
>
>     c := make(chan int)
>     go sum(s[:len(s)/2], c)
>     go sum(s[len(s)/2:], c)
>     x, y := <-c, <-c
>
>     fmt.Println(x, y, x+y)
> }
> ```

* ### 接口越大, 抽象越弱 \([The bigger the interface, the weaker the abstraction](https://www.bilibili.com/video/av18889438/?t=5m17s)\) {#接口越大-抽象越弱-The-bigger-the-interface-the-weaker-the-abstraction}

> 接口背后的概念是通过将对象的行为抽象为简单的契约来允许重用性. 虽然接口不是 Go 专有的, 但由于 Go 接口通常趋向于小型化, Go 程序员才广泛使用它们. 通常情况下, 一个接口只限于一到两个方法.
>
> Go io 包接口就是典型的例子.

* ### 充分利用零值 \([Make the zero value useful](https://www.bilibili.com/video/av18889438/?t=6m25s)\) {#充分利用零值-Make-the-zero-value-useful}

> 零值的典型例子如 bytes.Buffer 和 sync.Mutex,以及nil的channel

* ### interface{} 言之无物 \([interface{} says nothing](https://www.bilibili.com/video/av18889438/?t=7m36s)\) {#interface-言之无物-interface-says-nothing}

> 该谚语不是说 interface {} 不代表任何东西, 而是说该类型无静态检查以及调用时保证, 比如你的 func 接收一个 interface{} 类型, 你写的时候是可用的, 但是某个时间你进行了代码重构可能坏掉了.

* ### Gofmt 的风格没有人喜欢, 但是 gofmt 是每个人的最爱 \([Gofmt’s style is no one’s favorite, yet gofmt is everyone’s favorite](https://www.bilibili.com/video/av18889438/?t=8m43s)\) {#Gofmt-的风格没有人喜欢-但是-gofmt-是每个人的最爱-Gofmt’s-style-is-no-one’s-favorite-yet-gofmt-is-everyone’s-favorite}

> 有统一的格式比没有格式好，即使统一的格式并不那么好看

* ### 小复制好过小依赖 \([A little copying is better than a little dependency](https://www.bilibili.com/video/av18889438/?t=9m28s)\) {#小复制好过小依赖-A-little-copying-is-better-than-a-little-dependency}

* ### 系统调用必须始终使用构建标签保证 \([Syscall must always be guarded with build tags](https://www.bilibili.com/video/av18889438/?t=11m10s)\) {#系统调用必须始终使用构建标签保证-Syscall-must-always-be-guarded-with-build-tags}

> 不同的系统 \(\*NIX, Windows\) 调用导致你同一个 func \(实现并不一样\) 可能需要在不同的系统上构建才能得到你想要的结果. 简单说就是系统调用不可移植才这么干. 示例可参见 Go 标准库 syscall.

* ### Cgo 必须始终使用构建标签保证 \([Cgo must always be guarded with build tags](https://www.bilibili.com/video/av18889438/?t=11m53s)\) {#Cgo-必须始终使用构建标签保证-Cgo-must-always-be-guarded-with-build-tags}

* ### Cgo 不是 Go \([Cgo is not Go](https://www.bilibili.com/video/av18889438/?t=12m37s)\) {#Cgo-不是-Go-Cgo-is-not-Go}

* ### unsafe 包无保证 \(With the unsafe package there are no guarantees\) {#unsafe-包无保证-With-the-unsafe-package-there-are-no-guarantees}

> 包如其名, 不安全. 你可以使用 unsafe 包如果你准备好了有一天它会坏掉.

* ### 清晰好过聪明 \([Clear is better than clever](https://www.bilibili.com/video/av18889438/?t=14m35s)\) {#清晰好过聪明-Clear-is-better-than-clever}

> Rob Pike 在他与别人合著的 &lt;&lt;[程序设计实践](https://item.jd.com/11836053.html)&gt;&gt; 中写到: “写清晰的代码, 不要写聪明的代码”.

* ### 反射永远不是清晰的 \([Reflection is never clear](https://www.bilibili.com/video/av18889438/?t=15m22s)\) {#反射永远不是清晰的-Reflection-is-never-clear}

> 很多人在 Stackoverflow 上抱怨 Go 的反射不工作, 因为那不是为你准备的😂! 只有很少很少的人应该用反射这个非常强大而又非常难的特性. 新手应该远离反射和 interface{}.

* ### 错误也是一种值 \([Errors are values](https://www.bilibili.com/video/av18889438/?t=16m13s)\) {#错误也是一种值-Errors-are-values}

> 值可以被编程, 并且由于错误是值, 所以错误可以被编程. Go 官方博客有对此的 [解读](https://blog.golang.org/errors-are-values)

* ### 不要止步于检查错误而要优雅的处理 \([Don’t just check errors, handle them gracefully](https://www.bilibili.com/video/av18889438/?t=17m25s)\) {#不要止步于检查错误而要优雅的处理-Don’t-just-check-errors-handle-them-gracefully}

* ### 设计架构, 命名组件, 记录细节 \([Design the architecture, name the components, document the details](https://www.bilibili.com/video/av18889438/?t=18m09s)\) {#设计架构-命名组件-记录细节-Design-the-architecture-name-the-components-document-the-details}

> 当你写一个大型系统的时候, 你把它设计成一种结构化的东西. 想象组件的每一个部分并行工作, 为不同的组件起好的名字, 因为这些名字会出现在稿纸上.
>
> 拿 Go 程序来说, 如果名字不错, 组件就好理解, 那么程序的结构设计就会清晰, 程序会感觉很自然.
>
> 但是还有很多东西你需要解释, 所以这些是你需要解释的细节. 但是命名会帮助你解释很大一部分设计. 细节只是填补材料的缺口可能用来为用户打印工程图解文档.

* ### 文档是针对用户的 \([Documentation is for users](https://www.bilibili.com/video/av18889438/?t=19m07s)\) {#文档是针对用户的-Documentation-is-for-users}

> 很多人写文档表明某个 func 是做什么的, 但是他们不想想这个 func 是为谁而写. 这有很大的不同. 你知道这个 func 返回什么是对的, 但是它为什么返回了你使用的时候不一样的结果?
>
> 把自己当成使用者而不是写它的人, 那么 godoc 上的文档就是对用户有用的. 这对于其他语言一样适用.

* ### 不要慌 \([Don’t panic](https://github.com/golang/go/wiki/CodeReviewComments#dont-panic)\) {#不要慌-Don’t-panic}

> 不要使用 panic 进行正常的错误处理. 使用错误 \(error\) 和多个返回值.

* ### Go 没有引用变量, 所以 Go 没有传递引用函数的调用语义.

> 在像 C++ 这样的语言中, 你可以声明一个别名, 或者一个现有变量的替代名称. 这被称为引用变量.
>
> ```
> #include <stdio.h>
> int main() {
>     int a = 10;
>     int &b = a;
>     int &c = b;
>     printf("%p %p %p\n", &a, &b, &c); // 0x7ffe114f0b14 0x7ffe114f0b14 0x7ffe114f0b14
>     return 0;
> }
> ```
>
> 你可以看到 a, b 和 c 都指向相同的内存位置. 写入 a 会改变 b 和 c 的内容. 当你想在函数调用不同的作用域声明引用变量时, 这是很有用的.
>
> ### Go 没有引用变量 {#Go-没有引用变量}
>
> 与 C++ 不同, Go 程序中定义的每个变量都占用一个唯一的内存位置.
>
> ```
> package main
>
> import "fmt"
>
> func main() {
>         var a, b, c int
>         fmt.Println(&a, &b, &c) // 0x1040a124 0x1040a128 0x1040a12c
> }
> ```
>
> 创建一个两个变量在内存中共享相同的存储位置的 Go 程序是不可能的. 可以创建两个变量, 其内容指向相同的存储位置, 但与共享相同存储位置的两个变量不同.
>
> ```
> package main
>
> import "fmt"
>
> func main() {
>         var a int
>         var b, c = &a, &a
>         fmt.Println(b, c)   // 0x1040a124 0x1040a124
>         fmt.Println(&b, &c) // 0x1040c108 0x1040c110
> }
> ```
>
> 在这个例子中, b 和 c 保持相同的值, 即 a 的地址. 但是, b 和 c 本身存储在不同的位置. 更新 b 的内容对 c 没有影响.
>
> ### 但是 map 和 channel 是引用吗 ? {#但是-map-和-channel-是引用吗}
>
> map 和 channel 不是引用. 如果他们是, 那么下面这个程序将打印 false
>
> ```
> package main
>
> import "fmt"
>
> func fn(m map[int]int) {
>         m = make(map[int]int)
> }
>
> func main() {
>         var m map[int]int
>         fn(m)
>         fmt.Println(m == nil)
> }
> ```
>
> 如果map m 是一个 C++ 风格的引用变量, 则在 main 中声明的 m 和 在 fn 中声明的 m 将在内存中占据相同的存储位置. 但是, 因为对 fn 内的 m 的赋值对 m 的值没有影响, 所以我们可以看到 m 不是引用变量.
>
> ### Go 没有传递引用语义, 因为 Go 没有引用变量

* ### Go 的值拷贝代价

> 值拷贝在 Go 语言编程中普遍发生. 赋值, 参数传递, channel 值发送以及接收操作都会涉及值拷贝. 本文将讨论 Go 中的值拷贝代价.
>
> * ##### 值大小
>
> | 类型 | Go 1.10 的值大小 | Go 规范要求 |
> | :--- | :--- | :--- |
> | bool | 1 字节 | 未明确 |
> | int8, uint8\(byte\) | 1 字节 | 1 字节 |
> | int16, uint16 | 2 字节 | 2 字节 |
> | int32 \(rune\), uint32, float32 | 4 字节 | 4 字节 |
> | int64, uint64, float64, complex64 | 8 字节 | 8 字节 |
> | complex128 | 16 字节 | 16 字节 |
> | int, uint | 1 字 | 体系结构相关, 4 或 8 个字节 |
> | uintptr | 1 字 | 足够大以存储指针值的未解释位 |
> | string | 2 字 | 未明确 |
> | pointer | 1 字 | 未明确 |
> | slice | 3 字 | 未明确 |
> | map | 1 字 | 未明确 |
> | channel | 1 字 | 未明确 |
> | function | 1 字 | 未明确 |
> | interface | 2 字 | 未明确 |
> | struct | 所有字段的大小总和 + 填充字节数 | 如果结构体类型不包含大于零的字段, 则其大小为零 |
> | array | \(元素值大小\) \* \(数组长度\) | 如果数组的元素类型大小为零, 则其大小为零 |

> * ##### 值拷贝代价
>
> 一般来说, 拷贝值的成本与值的大小成正比. 但是, 值的大小并不是计算值拷贝的唯一因素. 不同的 CPU 体系结构可能会针对具有特定大小的值专门优化值拷贝. 在实践中, 我们可以将大小小于四个本地字的值视为小值. 拷贝小值的成本很小.
>
> 除了大型结构和数组类型的值\(对于标准的 Go 编译器\), Go 中的大部分值都是小值.
>
> 为避免参数传递和 channel 值发送和接收操作中的大值复制成本, 我们应该尽量避免使用大型结构和数组类型作为函数和方法参数类型\(包括方法接收器类型\) 并避免使用大型结构和数组类型作为 channel 元素类型. 我们可以使用基类型为大型结构和数组类型的指针类型来应对这种情况.
>
> 另一方面, 我们还应该考虑在运行时使用太多指针造成的垃圾回收的负面影响. 因此, 是否应该使用大型结构体和数组类型或其相应的指针类型取决于特定的场景.
>
> 如果这些基本类型的值拷贝的成本很小.我们不应该使用基类型为 slice, map, channel map, function, 字符串和接口类型的指针类型.

> 如果元素类型是大值类型, 我们还应该尝试避免使用两次迭代变量形式来迭代数组和切片元素, 因为每个元素值都将被复制到迭代过程中的第二个迭代变量\(译注: for range 中的 v\).



