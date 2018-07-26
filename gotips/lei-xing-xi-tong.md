> # Go 类型系统概述

* ### 概念: 基本类型 {#概念-基本类型}

> Go 中内置的基本类型已经在[内置的基本类型和基本值字面量](https://go101.org/article/basic-types-and-value-literals.html)中引入
>
> ```
> 1：内建字符串类型: string.
> 2：内建布尔类型: bool.
> 3：内建数字类型:
>     1：int8, uint8 (byte), int16, uint16, int32 (rune), uint32, int64, uint64, int, uint, uinptr.
>     2：float32, float64.
>     3：complex64, complex128.
> ```

* ### 概念: 复合类型 {#概念-复合类型}

> [指针类型](https://go101.org/article/pointer.html)
>
> * 和 C 指针一样.
>
> [结构体类型](https://go101.org/article/struct.html)
>
> * 和 C 结构体一样.
>
> [函数类型](https://go101.org/article/function.html)
>
> * 函数是 Go 的一等类型.
>
> [容器类型](https://go101.org/article/container.html)
>
> * 数组类型 - 固定长度的容器类型.
> * 切片类型 - 动态长度和动态容量的容器类型.
> * 映射类型 - 映射是关联数组\(或字典\). 标准 Go 编译器将映射实现为哈希表.
>
> [管道类型](https://go101.org/article/channel.html)
>
> * 管道\(channels\) 用于在 goroutine \(Go 中的绿色线程\) 之间同步数据.
>
> [接口类型](https://go101.org/article/interface.html)
>
> * 接口在反射和多态中起着关键作用.
>
> 与基本类型不同, 复合类型可以表示为它们各自的类型字面量. 以下是各种复合类型的一些字面量表示示例.
>
> ```
> // Assume T is an arbitrary type and Tkey is
> // a type supporting comparison (== and !=).
>
> *T         // a pointer type
> [5]T       // an array type
> []T        // a slice type
> map[Tkey]T // a map type
>
> // a struct type
> struct {
>     name string
>     age  int
> }
>
> // a function type
> func(int) (bool, string)
>
> // an interface type
> interface {
>     Method0(string) int
>     Method1() (int, bool)
> }
>
> // some channel types
> chan T
> chan<- T
> <-chan T
> ```

* ### 类型的种类 {#事实-类型的种类}

> 上面提到的每种基本类型和复合类型都对应于一种类型. 除了这些类型, 引入 unsafe[标准库](https://golang.org/pkg/unsafe)的不安全类型 unsafe.Pointer 也是 Go 中的一种类型.
>
> 所以, 到现在为止\(Go 1.10\), Go 有 26 种类型.

* ### 类型定义 {#语法-类型定义}

> \(类型定义, 或者类型定义声明, 在 Go 1.9 之前称为**类型声明**. 从 Go 1.9 开始, 类型定义已经成为 Go 中两种类型声明中的一种.\)
>
> 在 Go 中, 我们可以使用以下语法来定义新类型. 在下面的例子中, type 是一个关键字.
>
> ```
>  // Define a solo new type.
> type NewTypeName SourceType
>
> // Define multiple new types together.
> type (
>     NewTypeName1 SourceType1
>     NewTypeName2 SourceType2
> )
> ```
>
> 新类型名称必须是标识符.
>
> ```
> 1:类型定义中的新定义类型及其各自的源类型是两种截然不同的类型.
> 2:两种不同的定义类型总是两种不同的类型.
> 3:新定义的类型和源类型将共享相同的底层类型(请参阅下面的底层类型), 并且它们的值可以相互转换.
> 4:类型可以在函数体中定义.
> ```
>
> 一些类型定义示例:
>
> ```
> // The following new defined and source types are all basic types.
> type (
>     MyInt int
>     Age   int
>     Text  string
> )
>
> // The following new defined and source types are all composite types.
> type IntPtr *int
> type Book struct{author, title string; pages int}
> type Convert func(in0 int, in1 bool)(out0 int, out1 string)
> type StringArray [5]string
> type StringSlice []string
>
> func f() {
>     // The three defined types can be only used within the function.
>     type PersonAge map[string]int
>     type MessageQueue chan string
>     type Reader interface{Read([]byte) int}
> }
> ```

* ### 类型别名声明 {#语法-类型别名声明}

> \(类型别名声明是 Go 1.9 引入的一种新的类型声明.\)
>
> 如上所述, Go 中只有两个内置类型别名, byte\(uint8 的别名\)和 rune\(int32 的别名\). 他们是 Go 1.9 之前的唯一两种别名.
>
> 从 Go 1.9 开始, 我们可以使用以下语法来声明自定义类型别名. 别名声明的语法与类型定义非常类似, 但请注意每种类型别名声明中都有一个_=_.
>
> ```
> type (
>     Name = string
>     Age  = int
> )
>
> type table = map[string]int
> type Table = map[Name]Age
> ```
>
> 类型别名必须是标识符. 和类型定义一样, 类型别名也可以在函数体中声明.
>
> 通过上述的声明, Name 是 string 的别名, 他们表示相同的类型. 其他三对类型也是同样的关系.
>
> ```
> 1:Age 和 int
> 2:table 和 map[string]int
> 3:Table 和 map[Name]Age
> ```
>
> 事实上, map\[string\]int 和 map\[Name\]Age 也表示相同的类型. 所以涉及最后两个类型别名声明的四种类型都表示相同的类型.
>
> 注意, 尽管 table 和 Table 表示相同的类型, Table 是一个可导出类型, 所以它能被其他包引用, 而 table 不能.

* ### 命名类型 vs 非命名类型 {#概念-命名类型-vs-非命名类型}

> 在 Go 中
>
> 1:如果一个类型有一个名称, 该名称必须是一个标识符, 并且它的名称不是空白标识符,那么这种类型称为命名类型. 所有基本类型都是命名类型.
>
> 2:如果一个类型不能用一个纯粹的标识符来表示, 那么这个类型就是一个未命名的类型. 在上面的例子中, 由它们各自的类型字面量表示的复合类型都是未命名的类型.
>
> 未命名类型必须是复合类型, 反之则不然, 因为复合类型可以是定义类型和别名类型.

* ### 定义类型 vs 非定义类型 {#概念-定义类型-vs-非定义类型}

> 在 Go 1.9 之前, 只有一种类型声明. 这是当前的类型定义. 所以非常清楚的是, 一个命名类型必须是一个定义类型, 并且定义类型和命名类型在 Go 1.9 之前几乎是相同的概念. 但是, 从 Go 1.9 开始, 由于引入了类型别名, 事情变得有点复杂. 现在, 一个命名类型也可以是一个类型别名, 而一个别名类型可能是一个定义的类型或不是.
>
> 为了让许多解释变得简单明了, Go 101 采用了一种新的术语, 即非定义类型, 来表示未命名类型和未命名类型的别名. 这个术语在 Go 规范中没有.
>
> 所有基本类型都已定义. 非定义类型必须是复合类型.
>
> 在下面的例子中. 别名类型 C 和类型字面量 \[\]string 都是非定义类型, 但类型 A 和别名类型 B 都是定义类型.
>
> ```
> type A []string
> type B = A
> type C = []string
> ```

* ### 底层类型 {#概念-底层类型}

> 在 Go 中, 每种类型都有一个底层类型. 规则是:
>
> * 对于内置的基本类型, 底层类型就是自身.
> * unsafe.Pointer 的底层类型是自身.
> * 未命名类型的底层类型\(它必须是复合类型\)是自身.
> * 在类型声明中, 新的声明类型和源类型具有相同的底层类型.
>
> ```
> // The underlying types of the following ones are both int.
> type (
>     MyInt int
>     Age   MyInt
> )
>
> // The following new types have different underlying types.
> type (
>     IntSlice   []int   // underlying type is []int
>     MyIntSlice []MyInt // underlying type is []MyInt
>     AgeSlice   []Age   // underlying type is []Age
> )
>
> // The underlying types of Ages and AgeSlice are both []Age.
> type Ages AgeSlice
> ```
>
> 如何跟踪给定用户定义类型的底层类型? 规则是, 当内建的基本类型, unsafe.Pointer 或未命名的类型被满足时, 跟踪将被停止. 以上面的类型声明为例, 我们来追踪一下它们的底层类型.
>
> ```
> MyInt → int
> Age → MyInt → int
> IntSlice → []int
> MyIntSlice → []MyInt → []int
> AgeSlice → []Age → []MyInt → []int
> Ages → AgeSlice → []Age → []MyInt → []int
> ```
>
> 在 Go 中
>
> ```
> 1:底层类型为 bool 的类型称为布尔类型;
> 2:底层类型是任何内置整数类型的类型称为整数类型;
> 3:底层类型为 float32 或 float64 的类型称为浮点类型;
> 4:底层类型为 complex64 或 complex128 的类型称为复数类型;
> 5:整数, 浮点和复数类型也称为数字类型;
> 6:底层类型为字符串的类型称为字符串类型.
> ```
>
> 底层类型的概念在[Go 的值转换, 赋值和比较](https://go101.org/article/value-conversions-assignments-and-comparisons.html)中扮演着重要的角色

* ### 值 {#概念-值}

> 一个类型的实例称为该类型的值. 一个类型可能有很多值, 但它只有一个零值. 相同类型的值共享一些共同属性.
>
> 每种类型都有一个零值, 可以将其视为该类型的默认值. 预定义的 nil 标识符可以用来表示切片, 映射, 函数, 管道, 指针\(包括类型不安全的指针\)和接口类型的零值. 关于 nil 的更多内容, 请稍后阅读[Go 中的 nil](https://lingchao.xin/post/go-nils.html).
>
> 代码中有几种值的表示形式, 包括[字面量](https://go101.org/article/basic-types-and-value-literals.html),[命名常量](https://go101.org/article/constants-and-variables.html#constant),[变量](https://go101.org/article/constants-and-variables.html#variable)和[表达式](https://go101.org/article/expressions-and-statements.html), 但前三者可以被视为后者的特例.
>
> 值可能是[有类型或无类型](https://go101.org/article/constants-and-variables.html#untyped-value)的. 除了预定义的 nil 之外, 复合类型的值都是有类型的. 无类型值都是基本值或预定义的 nil.
>
> 在文章[基本类型和基本值字面量](https://go101.org/article/basic-types-and-value-literals.html)中引入了所有类型的基本值字面量. 在 Go 中还有两种字面量, 复合字面量和函数字面量.
>
> 顾名思义, 函数字面量用于表示函数值. 实际上, 函数字面量并不陌生.[函数声明](https://go101.org/article/function-declarations-and-calls.html#declaration)由函数字面量和函数标识符组成. 换句话说, 函数声明只有一个部分, 即函数标识符部分, 而不是其对应的函数字面量部分. 每个函数字面量也是一个匿名函数. 函数字面量将在[Go 中的函数](https://go101.org/article/function.html)中正式引入.
>
> 复合字面量用于表示结构体类型和容器类型\(数组, 片和映射\)的值, 请阅读[Go中的结构体](https://go101.org/article/struct.html)和[Go 中的容器](https://go101.org/article/container.html)以获取详细信息.
>
> 没有字面量来表示指针, 管道和接口类型的值.

* ### 值部分 {#概念-值部分}

> 当运行的时候, 许多值都存储在内存中的某处. 在 Go 中, 每个这样的值都有一个直接的部分, 然而, 其中的一些都有一个或多个间接部分. 每个值部分都占用连续的内存段. 值的间接底层部分由其直接部分通过指针引用.
>
> Go 规范中未定义术语**值部分**. 它仅在 Go 101 中用于简化一些解释并帮助 Go 程序员更好地理解 Go 类型和值. 请阅读文章[值部分](https://go101.org/article/value-part.html)来了解更多关于值部分的信息.

* ### 值大小 {#概念-值大小}

> 当一个值存储在内存中时, 该值的直接部分占用的字节数称为该值的大小. 相同类型的所有值具有相同的值大小, 因此类型值的大小通常称为类型值大小的大小. 我们可以使用 unsafe 标准库中的 Sizeof 函数来获取任何值的大小.
>
> Go 规范没有规定非数字类型的值大小要求. 文章[基本类型和基本值字面量](https://go101.org/article/basic-types-and-value-literals.html)中列出了各种基本数值类型的值大小要求.

* ### 接口值的动态类型和动态值 {#概念-接口值的动态类型和动态值}

> 接口值是其类型为接口类型的值.
>
> 每个接口值都可以包含一个非接口值. 装入接口值中的值称为接口值的动态值. 动态值的类型称为接口值的动态类型. 接口值没有装任何东西是一个 nil 接口值.
>
> 关接口类型和值的更多信息, 请阅读[这篇文章](https://go101.org/article/interface.html).

* ### 函数类型签名 {#概念-函数类型签名}

> 函数类型的签名由函数的输入参数定义列表和输出结果定义列表组成.
>
> 函数名和函数体不是函数签名的组成部分. 参数和结果类型对函数签名很重要, 但参数和结果名称并不重要.
>
> 请阅读[Go 中的函数](https://go101.org/article/function.html)以获取有关函数类型和函数值的更多详细信息.

* ### 类型的方法和方法集 {#概念-类型的方法和方法集}

> 在 Go 中, 有些类型可以有[方法](https://go101.org/article/method.html). 方法也可以被称为成员函数.
>
> 类型的方法集由一个类型的所有方法组成. 如果一个类型的方法集是一个接口类型的方法集的超集, 我们说这个类型实现了该接口类型.

* ### 接口类型的字段 {#概念-接口类型的字段}

> 结构体类型由成员变量的集合组成. 每个成员变量都称为结构体类型的字段. 例如, 以下结构体类型 Book 有三个字段, author, title 和 pages.
>
> ```
> struct {
>     author string
>     title  string
>     pages  int
> }
> ```
>
> 有关结构类型和值的更多信息, 请阅读[Go 中的结构体](https://go101.org/article/struct.html).
>
> 在 Go 中, 我们可以通过将此类型[嵌入](https://go101.org/article/type-embedding.html)结构体类型来扩展满足某些条件的类型.

* ### 指针类型的基类型 {#概念-指针类型的基类型}

> 对于指针类型, 假设它的底层类型可以用字面表示为 \*T, 则 T 被称为指针类型的基类型.
>
> 有关指针类型和值的更多信息, 请阅读[Go 中的指针](https://go101.org/article/pointer.html).

* ### 容器类型 {#概念-容器类型}

> 数组, 切片和映射可以视为正式的内置容器类型.
>
> 非正式地, 字符串和管道类型也可以被视为容器类型.
>
> 容器类型的每个值都有一个长度, 容器类型要么是正式的, 要么是非正式的.
>
> 有关容器类型和值的更多信息, 请阅读[Go 中的容器](https://go101.org/article/container.html).

* ### 映射类型的键类型 {#概念-映射类型的键类型}

> 如果映射类型的底层类型可以表示为 map\[Tkey\]T, 那么 Tkey 被称为映射类型的键类型. Tkey 必须是可比较的类型.

* ### 容器类型的键类型和元素类型 {#概念-容器类型的键类型和元素类型}

> 存储在容器中的元素的类型必须相同. 元素的标识类型被称为容器值的类型的元素类型.
>
> * 对于数组类型, 如果其底层类型为 \[N\]T, 则其元素类型为 T.
> * 对于切片类型, 如果其底层类型为 \[\]T, 则其元素类型为 T.
> * 对于映射类型, 如果其底层类型是 map\[Tkey\]T, 则其元素类型是 T.
> * 对于管道类型, 如果其底层类型是 chan T, chan&lt;- T 或 &lt;-chan T, 则其元素类型是 T.
> * 任何字符串类型的元素类型总是 byte \(又名 uint8\).

* ### 管道类型的方向 {#概念-管道类型的方向}

> 管道值可以被视为同步的先进先出\(FIFO\)队列. 管道类型和值有方向.
>
> ```
> 1:可发送和可接收的管道值称为双向管道. 它的类型称为双向管道类型. 双向管道类型在字面量中表示为 chan T.
> 2:仅可发送的管道值被称为发送专用管道. 它的类型被称为发送管道类型. 只发送管道类型在字面量中表示为 chan<- T.
> 3:仅可接收的管道值被称为只收管道. 它的类型被称为只接收管道类型. 只接收管道类型在字面量中表示为 <-chan T.
> ```
>
> 有关管道类型和值的更多信息, 请阅读[Go 中的管道](https://go101.org/article/channel.html).

* ### 支持或不支持比较的类型 {#事实-支持或不支持比较的类型}

> 目前\(Go 1.10\), 以下类型不支持比较\(使用 == 和 != 运算符\):
>
> * 切片类型
> * 映射类型
> * 函数类型
> * 任何具有类型无法比较的字段的结构体类型以及任何元素类型无法比较的数组类型.
>
> 以上列出的类型称为无法比较的类型. 编译器禁止比较两个不可比较类型的值.
>
> 所有基本类型, 指针类型, 管道类型和接口类型都是可比的. 但是, 如果两个接口值的动态类型都是相同的不可比较类型, 那么比较两个接口值可能会在运行时出现 panic.
>
> 不可比的类型的值称为无法比较的值. 一些可比较类型的值也可能无法比较. 例如, 具有不可比较的动态类型的接口值是一个无法比较的值. 比较一个无法比较类型可比较的值会在运行时引发 panic.
>
> 我们可以从文章[Go 的值转换, 赋值和比较](https://go101.org/article/value-conversions-assignments-and-comparisons.html)中详细了解比较的详细规则.
>
> 任何映射类型的键类型必须是可比较类型. 因此, 上面列出的类型不能用作映射的键类型.

* ### Go 中的面向对象的编程 {#事实-Go-中的面向对象的编程}

> Go 不是一个全功能的面向对象编程语言, 但 Go 确实支持一些面向对象的编程风格. 有关详细信息, 请阅读以下列出的文章

> * [Go 中的方法](https://go101.org/article/method.html)
>   .
> * [Go 中的接口实现](https://go101.org/article/interface.html#implementation)
>   .
> * [Go 中的类型嵌入](https://go101.org/article/type-embedding.html)
>   .

* ### Go 中的范型 {#事实-Go-中的范型}

> Go 中的范型功能仅限于内置类型和函数. 截至目前\(v1.10\), Go 不支持自定义类型和自定义函数的范型. 有关详细信息, 请阅读
>
> [Go 中的内置范型](https://go101.org/article/generic.html)



