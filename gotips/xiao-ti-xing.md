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
>     defer func() {
>         r += n // modify the return value
>     }()
>
>     return n + n // <=> r = n + n; return
> }
>
> func main() {
>     fmt.Println(Triple(5)) // 15
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
>     i = n
>     for i = 0; i < len(s); i++ {}
>     return i
> }
>
> func fb(s []int, n int) int {
>     i = n
>     for i = range s {}
>     return i
> }
>
> func main() {
>     s := []int{2, 3, 5, 7, 11, 13}
>     fmt.Println(fa(s, -1), fb(s, -1)) // 6 5
>     s = nil
>     fmt.Println(fa(s, -1), fb(s, -1)) // 0 -1
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
>     go func() {
>         time.Sleep(time.Second)
>         os.Exit(1)
>     }()
>     select{}
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
>     c := make(chan int)
>     go func() {
>         defer func() {c <- 1}()
>         defer fmt.Println("Go")
>         func() {
>             defer fmt.Println("C")
>             runtime.Goexit()
>         }()
>         fmt.Println("Java")
>     }()
>     <-c
> }
> ```

* 指针

> 对于数值指针 p, 表达式\_p++ 相当于 \(\_p\)++. 如果 p 不是数值指针, 则 \*p++ 编译不过
>
> 在 Go 中, 指针不能进行算术运算. 对于指针值 p, 语句 p++ 在 Go 中是非法的. 如果 p 是一个指向数值的指针, 那么 Go 编译器会将 \*v++ 视为 \(\*v\)++.
>
> 例如:
>
> ```
> package main
>
> import "fmt"
>
> func main() {
>     a := int64(5)
>     p := &a
>
>     // The following two lines don't compile.
>     /*
>     p++
>     p = (&a) + 8
>     */
>
>     *p++
>     fmt.Println(*p, a)   // 6 6
>     fmt.Println(p == &a) // true
> }
> ```
>
> 如果两种类型的基本类型相同, 则两个命名指针类型的值可以相互转换. 两种指针类型的底层类型可能不同
>
> 给定非接口值 x 和非接口类型 T, 假定 x 的类型为 Tx
>
> * 如果 Tx 和 T 共享相同的底层类型\(忽略结构体 tags\), 那么 x 可以显示地转换为 T, 尤其当 Tx 或者 T 不是一个定义类型并且它们的底层类型相同\(考虑结构体 tags\), 那么 x 可以隐式地转换为 T.
> * 如果 Tx 和 T 具有不同的底层类型, 但 Tx 和 T 都是[非定义](https://go101.org/article/type-system-overview.html#non-defined-type)的指针类型, 并且它们的基类型共享相同的底层类型\(忽略结构体 tags\), 则 x 可以\(并且必须\)可以显示地转换为 T.
>
> 一般性例子:
>
>     package main
>
>     func main() {
>         // []int, IntSlice and MySlice share
>         // the same underlying type: []int
>         type IntSlice []int
>         type MySlice  []int
>
>         var s  = []int{}
>         var is = IntSlice{}
>         var ms = MySlice{}
>         var x struct{n int `foo`}
>         var y struct{n int `bar`}
>
>         // The two lines both fail to compile.
>         /*
>         is = ms
>         ms = is
>         */
>
>         // Must use explicit conversions here.
>         is = IntSlice(ms)
>         ms = MySlice(is)
>         x = struct{n int `foo`}(y)
>         y = struct{n int `bar`}(x)
>
>         // Implicit conversions are okay here.
>         s = is
>         is = s
>         s = ms
>         ms = s
>     }
>
> 指针的例子:
>
> ```
> package main
>
> func main() {
>     type MyInt int
>     type IntPtr *int
>     type MyIntPtr *MyInt
>
>     var pi = new(int) // type is *int
>     var ip IntPtr = pi // ok, same underlying type
>
>     // var _ *MyInt = pi // can't convert implicitly
>     var _ = (*MyInt)(pi) // ok, must explicitly
>
>     // var _ MyIntPtr = pi  // can't convert implicitly
>     // var _ = MyIntPtr(pi) // can't convert explicitly
>     var _ MyIntPtr = (*MyInt)(pi)  // ok
>     var _ = MyIntPtr((*MyInt)(pi)) // ok
>
>     // var _ MyIntPtr = ip  // can't convert implicitly
>     // var _ = MyIntPtr(ip) // can't convert explicitly
>     var _ MyIntPtr = (*MyInt)((*int)(ip))  // ok
>     var _ = MyIntPtr((*MyInt)((*int)(ip))) // ok
> }
> ```
>
> 不同零大小的值的地址可能相同也可能不同.
>
> 两个零大小值的地址是否相等取决于编译器和编译器版本.
>
> ```
> package main
>
> import "fmt"
>
> func main() {
>     a := struct{}{}
>     b := struct{}{}
>     x := struct{}{}
>     y := struct{}{}
>     m := [10]struct{}{}
>     n := [10]struct{}{}
>     o := [10]struct{}{}
>     p := [10]struct{}{}
>
>     fmt.Println(&x, &y, &o, &p)
>
>     // For the standard Go compiler (1.10),
>     // x, y, o and p escape to heap,
>     // but a, b, m and n are allocated on stack.
>
>     fmt.Println(&a == &b) // false
>     fmt.Println(&x == &y) // true
>     fmt.Println(&a == &x) // false
>
>     fmt.Println(&m == &n) // false
>     fmt.Println(&o == &p) // true
>     fmt.Println(&n == &p) // false
> }
> ```
>
> 上面代码中的输出用于标准 Go 编译器 1.10.
>
> 指针类型的基类型可能是指针类型本身
>
> ```
> package main
>
> func main() {
>     type P *P
>     var p P
>     p = &p
>     p = **************p
> }
> ```
>
> 同样,
>
> 1. 切片类型的元素类型可以是切片类型本身,
> 2. 映射类型的元素类型可以是映射类型本身,
> 3. 管道类型的元素类型可以是管道类型本身
> 4. 函数类型的参数和结果类型可以是函数类型本身.
>
> ```
> package main
>
> func main() {
>     type S []S
>     type M map[string]M
>     type C chan C
>     type F func(F) F
>
>     s := S{0:nil}
>     s[0] = s
>     m := M{"Go": nil}
>     m["Go"] = m
>     c := make(C, 3)
>     c <- c; c <- c; c <- c
>     var f F
>     f = func(F)F {return f}
>
>     _ = s[0][0][0][0][0][0][0][0]
>     _ = m["Go"]["Go"]["Go"]["Go"]
>     <-<-<-c
>     f(f(f(f(f))))
> }
> ```
>
> 关于选择器的细节
>
> 对于一个指针值, 不管其类型是否定义, 如果它的\(指针\)类型的基类型是一个结构体类型, 那么指针值可以访问它引用的结构体值的字段. 但是, 如果指针值的类型是定义的类型, 则该值无法访问其引用的值的方法.
>
> ```
> package main
>
> type T struct {
>     x int
> }
> func (T) m(){} // T has one method.
>
> type P *T  // a defined one-level pointer type.
> type PP *P // a defined two-level pointer type.
>
> func main() {
>     var t T
>     var tp = &t
>     var tpp = &tp
>     var p P = tp
>     var pp PP = &p
>     tp.x = 12  // okay
>     p.x = 34   // okay
>     pp.x = 56  // error: type PP has no field or method x
>     tpp.x = 78 // error: type **T has no field or method x)
>
>     tp.m()  // okay. Type *T also has a "m" method.
>     p.m()   // error: type P has no field or method m
>     pp.m()  // error: type PP has no field or method m
>     tpp.m() // error: type **T has no field or method m
> }
> ```

* 容器

> 嵌套复合字面量可以简化
>
> 如果复合字面量嵌套了许多其他复合字面量, 那么这些嵌套复合字面量可以简化为 {…} 形式.
>
> 例如, 切片值字面量:
>
> ```
> // A slcie value of a type whose element type is *[4]byte.
> // The element type is a pointer type whose base type is [4]byte.
> // The pointer base type is an array type whose element type is byte.
> var heads = []*[4]byte{
> 	&[4]byte{'P', 'N', 'G', ' '},
> 	&[4]byte{'G', 'I', 'F', ' '},
> 	&[4]byte{'J', 'P', 'E', 'G'},
> } 
> ```
>
> 可以简化为:
>
> ```
> var heads = []*[4]byte{
> 	{'P', 'N', 'G', ' '},
> 	{'G', 'I', 'F', ' '},
> 	{'J', 'P', 'E', 'G'},
> }
> ```

> 如下例子中的数组字面量:
>
> ```
> type language struct {
> 	name string
> 	year int
> }
>
> var _ = [...]language{
> 	language{"C", 1972},
> 	language{"Python", 1991},
> 	language{"Go", 2009},
> }
> ```

> 可以简化为:
>
> ```
> var _ = [...]language{
> 	{"C", 1972},
> 	{"Python", 1991},
> 	{"Go", 2009},
> }
> ```

> 如下是映射值字面量的一个例子:
>
> ```
> type LangCategory struct {
> 	dynamic bool
> 	strong  bool
> }
>
> // A value of map type whose key type is a struct type and
> // whose element type is another map type "map[string]int".
> var _ = map[LangCategory]map[string]int{
> 	LangCategory{true, true}: map[string]int{
> 		"Python": 1991,
> 		"Erlang": 1986,
> 	},
> 	LangCategory{true, false}: map[string]int{
> 		"JavaScript": 1995,
> 	},
> 	LangCategory{false, true}: map[string]int{
> 		"Go":   2009,
> 		"Rust": 2010,
> 	},
> 	LangCategory{false, false}: map[string]int{
> 		"C": 1972,
> 	},
> }
> ```

> 可以简化为:
>
> ```
> var _ = map[LangCategory]map[string]int{
> 	{true, true}: {
> 		"Python": 1991,
> 		"Erlang": 1986,
> 	},
> 	{true, false}: {
> 		"JavaScript": 1995,
> 	},
> 	{false, true}: {
> 		"Go":   2009,
> 		"Rust": 2010,
> 	},
> 	{false, false}: {
> 		"C": 1972,
> 	},
> }
> ```

> 在某些情况下, 可以使用数组指针作为数组
>
> 在许多情况下, 我们可以使用指向数组的指针作为数组.
>
> 我们可以通过指向数组的指针来迭代数组的元素. 对于长度较大的数组, 这种方法效率更高, 因为复制指针比复制大数组效率更高. 在下面的例子中, 两个循环块是等价的, 并且都是高效的.
>
> ```
> package main
>
> import "fmt"
>
> func main() {
> 	var a [100]int
>
> 	for i, n := range &a { // copying a pointer is cheap
> 		fmt.Println(i, n)
> 	}
>
> 	for i, n := range a[:] { // copying a slice is cheap
> 		fmt.Println(i, n)
> 	}
> }
> ```

> 如果第二个迭代参数既不被忽略\(用 \_ \)也不被省略\(不写\), 那么覆盖零数组指针的范围将会 panic. 在以下示例中, 前两个循环块中的每一个都会打印五个索引, 但最后一个会产生 panic.
>
> ```
> package main
>
> import "fmt"
>
> func main() {
> 	var p *[5]int // nil
>
> 	for i, _ := range p { // okay
> 		fmt.Println(i)
> 	}
>
> 	for i := range p { // okay
> 		fmt.Println(i)
> 	}
>
> 	for i, n := range p { // panic
> 		fmt.Println(i, n)
> 	}
> }
> ```

> 数组指针也可以用来索引数组元素. 通过 nil 数组指针索引数组元素会产生 panic.
>
> ```
> package main
>
> import "fmt"
>
> func main() {
> 	a := [5]int{2, 3, 5, 7, 11}
> 	p := &a
> 	p[0], p[1] = 17, 19
> 	fmt.Println(a) // [17 19 5 7 11]
> 	p = nil
> 	p[0] = 31 // panic
> }
> ```

> 我们也可以从数组指针派生切片. 从 nil 数组指针派生切片会产生 panic.
>
> ```
> package main
>
> import "fmt"
>
> func main() {
> 	pa := &[5]int{2, 3, 5, 7, 11}
> 	s := pa[1:3]
> 	fmt.Println(s) // [3 5]
> 	pa = nil
> 	s = pa[1:3] // panic
> }
>
> ```
>
> 我们也可以传递数组指针作为内置的 len 和 cap 函数的参数. 传入 nil 数组指针参数给这两个函数不会产生 panic.
>
> ```
> var pa *[5]int // nil
> fmt.Println(len(pa), cap(pa)) // 5 5
>
> ```



