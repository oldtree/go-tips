### go语言中的一些小case

* **源文件字符集和字符集编码**

> Go源码文件默认采用Unicode字符集Unicode**码点\(code point\)**和内存中**字节序列（byte sequence）**的变换实现使用了UTF-8：一种变长多字节编码，同时也是一种事实字符集编码标准
>
> 在Go中对于字符串可以有三种写法
>
> 1. 字面值 ： `var s = "程序员"`
> 2. 码点表示法： `var s1 = "\u4e2d\u56fd\u4eba" `或者 ：`var s2 = "\U00004e2d\U000056fd\U00004eba"`
>
> 3. 字节序列表示法（二进制表示法）：`var s3 = "\xe4\xb8\xad\xe5\x9b\xbd\xe4\xba\xba"`

>



