### [go语言中的一些小case](https://tonybai.com/2015/09/17/7-things-you-may-not-pay-attation-to-in-go/)

* **源文件字符集和字符集编码**

> Go源码文件默认采用Unicode字符集Unicode**码点\(code point\)**和内存中**字节序列（byte sequence）**的变换实现使用了UTF-8：一种变长多字节编码，同时也是一种事实字符集编码标准
>
> 在Go中对于字符串可以有三种写法
>
> 1. 字面值 ： `var s = "中国人"`
> 2. 码点表示法： `var s1 = "\u4e2d\u56fd\u4eba"`或者 ：`var s2 = "\U00004e2d\U000056fd\U00004eba`
> 3. 字节序列表示法（二进制表示法）：`var s3 = "\xe4\xb8\xad\xe5\x9b\xbd\xe4\xba\xba"`

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

> 实际编码中，我们可能经常遇到的是fmt.Printf系列方法中format string太长的情况，但由于Go不支持相邻字符串自动连接\(concatenate\)，只能通过+来连接fmt字符串，且+必须放在前一行末尾。



