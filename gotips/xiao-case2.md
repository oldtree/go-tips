### [go语言中的一些小case](https://tonybai.com/2015/09/17/7-things-you-may-not-pay-attation-to-in-go/)

* **源文件字符集和字符集编码**

> Go源码文件默认采用Unicode字符集Unicode**码点\(code point\)**和内存中**字节序列（byte sequence）**的变换实现使用了UTF-8：一种变长多字节编码，同时也是一种事实字符集编码标准
>
> 在Go中对于字符串可以有三种写法
>
> 1. 字面值 ： `var s = "中国人"`
> 2. 码点表示法： `var s1 = "\u4e2d\u56fd\u4eba"`或者 ：`var s2 = "\U00004e2d\U000056fd\U00004eba`
> 3. 字节序列表示法（二进制表示法）：`var s3 = "\xe4\xb8\xad\xe5\x9b\xbd\xe4\xba\xba"`
>
> 这三种表示法中，除字面值转换为字节序列存储时根据编辑器保存的源码文件编码格式之外，其他两种均不受编码格式影响。我们可以通过逐字节输出来查 看字节序列的内容
>
> ```
> fmt.Println("s byte sequence:")
> for i := 0; i < len(s); i++ {
>     fmt.Printf("0x%x ", s[i])
> }
> fmt.Println("")
> ```

* **续行**

> 良好的代码style一般会要求代码中不能有太long的代码行，否则会影响代码阅读者的体验。在C中有续行符"\"专门用于代码续行处理；但在 Go中没有专属续行符，如何续行需要依据Go的语法规则（参见[Go spec](https://golang.org/ref/spec)）；Go与C一样，都是以分号\(";"\)作为语句结束的标识。不过大多数情况下，分号无需程序员手工输入，而是由编译器自动识别语句结束位置，并插入 分号。
>
> ```
> var (
>     s = "This is an example about code newline," +
>         "for string as right value"
>     d = 5 + 4 + 7 +
>         4
>     a = [...]int{5, 6, 7,
>         8}
>     m = make(map[string]int,
>         100)
>     c struct {
>         m1     string
>         m2, m3 int
>         m4     *float64
>     }
>
>     f func(int,
>         float32) (int,
>         error)
> )
>
> func foo(int, int) (string, error) {
>     return "",
>         nil
> }
>
> func main() {
>     if i := d; i >
>         100 {
>     }
>
>     var sum int
>     for i := 0; i < 100; i = i +
>         1 {
>         sum += i
>     }
>
>     foo(1,
>         6)
>
>     var i int
>     fmt.Printf("%s, %d\n",
>         "this is a demo"+
>             " of fmt Printf",
>         i)
> }
> ```
>
> 实际编码中，我们可能经常遇到的是fmt.Printf系列方法中format string太长的情况，但由于Go不支持相邻字符串自动连接\(concatenate\)，只能通过+来连接fmt字符串，且+必须放在前一行末尾。

* **Method Set**

> Method Set是Go语法中一个重要的隐式概念，在为interface变量做动态类型赋值、embeding struct/interface、type alias、method expression时都会用到Method Set这个重要概念。
>
> * **interface的Method Set**
>
> > 根据Go spec，interface类型的Method Set就是其interface（An interface type specifies a method set called its interface）。
> >
> > ```
> > type I interface {
> >     Method1()
> >     Method2()
> > }
> > ```
> >
> > I的Method Set包含的就是其literal中的两个方法：Method1和Method2。我们可以通过reflect来获取interface类型的 Method Set：
> >
> > ```
> > package main
> >
> > import (
> >     "fmt"
> >     "reflect"
> > )
> >
> > type I interface {
> >     Method1()
> >     Method2()
> > }
> >
> > func main() {
> >     var i *I
> >     elemType := reflect.TypeOf(i).Elem()
> >     n := elemType.NumMethod()
> >     for i := 0; i < n; i++ {
> >         fmt.Println(elemType.Method(i).Name)
> >     }
> > }
> >
> > $go run interfacemethodset.go
> > Method1
> > Method2
> > ```
>
> * **除interface type外的类型的Method Set**
>
> > 对于非interface type的类型T，其Method Set为所有receiver为T类型的方法组成；而类型\*T的Method Set则包含所有receiver为T和\*T类型的方法。
> >
> > ```
> > package main
> >
> > import "./utils"
> >
> > type T struct {
> > }
> >
> > func (t T) Method1() {
> > }
> >
> > func (t *T) Method2() {
> > }
> >
> > func (t *T) Method3() {
> > }
> >
> > func main() {
> >     var t T
> >     utils.DumpMethodSet(&t)
> >
> >     var pt *T
> >     utils.DumpMethodSet(&pt)
> > }
> > ```
> >
> > 我们要dump出T和\*T各自的Method Set，运行结果如下：
> >
> > ```
> > $go run othertypemethodset.go
> > main.T's method sets:
> >      Method1
> >
> > *main.T's method sets:
> >      Method1
> >      Method2
> >      Method3
> > ```
> >
> > 可以看出类型T的Method set仅包含一个receiver类型为T的方法：Method1，而\*T的Method Set则包含了T的Method Set以及所有receiver类型为\*T的Method。
> >
> > 如果此时我们有一个interface type如下：
> >
> > ```
> > type I interface {
> >     Method1()
> >     Method2()
> > }
> > ```
> >
> > 那下面哪个赋值语句合法呢？合不合法完全依赖于右值类型是否实现了interface typeI的所有方法，即右值类型的Method Set是否包含了I的 所有方法.
> >
> > ```
> > var t T
> > var pt *T
> >
> > var i I = t
> >
> > or
> >
> > var i I = pt
> > ```
> >
> > 编译错误告诉我们：
> >
> > ```
> > var i I = t // cannot use t (type T) as type I in assignment:
> >                   T does not implement I (Method2 method has pointer receiver)
> > ```
> >
> > T的Method Set中只有Method1一个方法，没有实现I接口中的 Method2，因此不能用t赋值给i；而\*T实现了I的所有接口，赋值合 法。不过Method set校验仅限于在赋值给interface变量时进行，无论是T还是\*T类型的方法集中的方法，对于T或\*T类型变量都是可见且可以调用的，如下面代码 都是合法的：
> >
> > ```
> > pt.Method1()
> > t.Method3()
> > ```

> > 因为Go编译器会自动为你的代码做receiver转换
> >
> > ```
> > pt.Method1() <=> (*pt).Method1()
> > t.Method3() <=> (&t).Method3() 
> > ```
> >
> > 很多人纠结于method定义时receiver的类型（T or \*T），个人觉得有两点考虑
> >
> > * 效率:Go方法调用receiver是以传值的形式传入方法中的。如果类型size较大，以value形式传入消耗较大，这时指针类型就是首选
> >
> > * 是否赋值给interface变量、以什么形式赋值:就像本节所描述的，由于T和\*T的Method Set可能不同，我们在设计Method receiver type时需要考虑在interface赋值时通过对Method set的校验
>
> * **embeding type的Method Set**
>
> > **interface embeding**
> >
> > ```
> > package main
> >
> > import "./utils"
> >
> > type I1 interface {
> >     I1Method1()
> >     I1Method2()
> > }
> > type I2 interface {
> >     I2Method()
> > }
> >
> > type I3 interface {
> >     I1
> >     I2
> > }
> >
> > func main() {
> >     utils.DumpMethodSet((*I1)(nil))
> >     utils.DumpMethodSet((*I2)(nil))
> >     utils.DumpMethodSet((*I3)(nil))
> > }
> >
> > $go run embedinginterface.go
> > main.I1's method sets:
> >      I1Method1
> >      I1Method2
> >
> > main.I2's method sets:
> >      I2Method
> >
> > main.I3's method sets:
> >      I1Method1
> >      I1Method2
> >      I2Method
> > ```

> > 可以看出嵌入interface type的interface type I3 的Method Set包含了被嵌入的interface type：I1和I2

> > 的Method Set。很多情况下，我们Go的interface type中仅包含有少量方法，常常仅是一个Method，通过interface type embeding来定义一个新interface，这是Go的一个惯用法，比如我们常用的io包中的Reader, Writer以及ReadWriter接口
> >
> > ```
> > type Reader interface {
> >     Read(p []byte) (n int, err error)
> > }
> >
> > type Writer interface {
> >     Write(p []byte) (n int, err error)
> > }
> >
> > type ReadWriter interface {
> >     Reader
> >     Writer
> > }
> > ```

> > **struct embeding interface**
> >
> > 在struct中嵌入interface type后，struct的Method Set中将包含interface的Method Set
> >
> > ```
> > type T struct {
> >     I1
> > }
> >
> > func (T) Method1() {
> >
> > }
> >
> > ....
> >
> > func main() {
> >     … …
> >     var t T
> >     utils.DumpMethodSet(&t)
> >     var pt = &T{
> >         I1: I1Impl{},
> >     }
> >     utils.DumpMethodSet(&pt)
> > }    
> >
> > main.T's method sets:
> >      I1Method1
> >      I1Method2
> >      Method1
> >
> > *main.T's method sets:
> >      I1Method1
> >      I1Method2
> >      Method1
> > ```
> >
> > **struct embeding struct**
> >
> > 在struct中embeding struct提供了一种“继承”的手段，外部的Struct可以“继承”嵌入struct的所有方法（无论receiver是T还是\*T类型）实现，但 Method Set可能会略有不同。看下面例子
> >
> > ```
> > type T struct {
> > }
> >
> > func (T) InstMethod1OfT() {
> >
> > }
> >
> > func (T) InstMethod2OfT() {
> >
> > }
> >
> > func (*T) PtrMethodOfT() {
> >
> > }
> >
> > type S struct {
> > }
> >
> > func (S) InstMethodOfS() {
> >
> > }
> >
> > func (*S) PtrMethodOfS() {
> > }
> >
> > type C struct {
> >     T
> >     *S
> > }
> >
> > func main() {
> >     var c = C{S: &S{}}
> >     utils.DumpMethodSet(&c)
> >     var pc = &C{S: &S{}}
> >     utils.DumpMethodSet(&pc)
> >
> >     c.InstMethod1OfT()
> >     c.PtrMethodOfT()
> >     c.InstMethodOfS()
> >     c.PtrMethodOfS()
> >     pc.InstMethod1OfT()
> >     pc.PtrMethodOfT()
> >     pc.InstMethodOfS()
> >     pc.PtrMethodOfS()
> > }
> >
> > $go run embedingstructinstruct.go
> > main.C's method sets:
> >      InstMethod1OfT
> >      InstMethod2OfT
> >      InstMethodOfS
> >      PtrMethodOfS
> >
> > *main.C's method sets:
> >      InstMethod1OfT
> >      InstMethod2OfT
> >      InstMethodOfS
> >      PtrMethodOfS
> >      PtrMethodOfT
> > ```

> > 可以看出：

> > 类型C的Method Set = T的Method Set + \*S的Method Set

> > 类型\*C的Method Set = \*T的Method Set + \*S的Method Set
> >
> > 同时通过例子可以看出，无论是T还是\*S的方法，C或\*C类型变量均可调用（编译器甜头），不会被局限在Method Set中

> * **alias type的Method Set**

> > Go支持为已有类型定义alias type，如
> >
> > ```
> > type MyInterface I
> > type Mystruct T
> > ```
>
> > 对于alias type, Method Set是如何定义的呢？我们看下面例子
> >
> > ```
> > type I interface {
> >     IMethod1()
> >     IMethod2()
> > }
> >
> > type T struct {
> > }
> >
> > func (T) InstMethod() {
> >
> > }
> > func (*T) PtrMethod() {
> >
> > }
> >
> > type MyInterface I
> > type MyStruct T
> >
> > func main() {
> >     utils.DumpMethodSet((*I)(nil))
> >
> >     var t T
> >     utils.DumpMethodSet(&t)
> >     var pt = &T{}
> >     utils.DumpMethodSet(&pt)
> >
> >     utils.DumpMethodSet((*MyInterface)(nil))
> >
> >     var m MyStruct
> >     utils.DumpMethodSet(&m)
> >     var pm = &MyStruct{}
> >     utils.DumpMethodSet(&pm)
> > }
> >
> > $go run aliastypemethodset.go
> > main.I's method sets:
> >      IMethod1
> >      IMethod2
> >
> > main.T's method sets:
> >      InstMethod
> >
> > *main.T's method sets:
> >      InstMethod
> >      PtrMethod
> >
> > main.MyInterface's method sets:
> >      IMethod1
> >      IMethod2
> >
> > main.MyStruct's method set is empty!
> > *main.MyStruct's method set is empty!
> > ```
>
> > 从例子的结果上来看，Go对于interface和struct的alias type给出了“不一致”的结果：
> >
> > MyInterface的Method Set与接口类型I Method Set一致；  
> > 而MyStruct并未得到T的哪怕一个Method，MyStruct的Method Set为空。



