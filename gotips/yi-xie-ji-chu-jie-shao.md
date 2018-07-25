#### go语言的一些简单介绍

执行main函数时的两种方式，实际上都是执行实际的二进制的可执行文件：

```
go run main.go  有将编译好的文件放在一个临时的目录下
go build main.go ----> ./main  编译好可运行文件之后并直接执行
```

```
GOARCH="amd64"
GOBIN="/usr/local/go/bin"
GOCACHE="/Users/a123/Library/Caches/go-build"
GOEXE=""
GOHOSTARCH="amd64"
GOHOSTOS="darwin"
GOOS="darwin"
GOPATH="/Users/a123/XXXX"
GORACE=""
GOROOT="/usr/local/go"
GOTMPDIR=""
GOTOOLDIR="/usr/local/go/pkg/tool/darwin_amd64"
GCCGO="gccgo"
CC="clang"
CXX="clang++"
CGO_ENABLED="1"
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
PKG_CONFIG="pkg-config"
GOGCCFLAGS="-fPIC -m64 -pthread -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fdebug-prefix-map=/var/folders/lx/k582ndw52gx114q1rj3430r00000gn/T/go-build965121195=/tmp/go-build -gno-record-gcc-switches -fno-common"
```

一个可以编辑go代码的环境，包括主要的`goroot,gopath,cgo`优化，临时文件目录，还有没有写上去的`GOGC`

* go的每个包的可导出类型，都是开头大写字母的，但是无论别的什么开头，都不可以导出了，中文或者其他都是
* 每个目录下的文件拥有统一的一个包名称
* 一个结构体中的成员变量和成员函数是不能重名的
* gopath可以有多个，在使用时会依次遍历

---

#### **控制结构**

```
if true{
}else{
}

if 1!=2{
}

if index,val:=array[0];val < 10{
}

if ok:=hell.(Heaven);ok{
}

if _,ok:= <-chan;ok{
}
```

1. if可以使用纯粹的bool

2. 可以使用三段式，

3. 可以使用类型断言或者判断channel的关闭状态

---

```
swaitch char {
    case 'a':
    case 'b':
    case 'c':
    case 'd':
    case 'e','f','g':
}

var i = 0
var a = 10
var b = 20
switch {
    case i < 10:
    case i > 100：
    case a > b:
}

switch hell.(type){
    case Heaven:
    case China:
    case NorthKorea:
    case *int:
    case *string:
}
```

1. switch可以用来根据给定的条件来判断执行
2. swicth可以根据所满足的bool条件来执行，从上到下
3. switch可以根据类型判断来执行

---

```
for index:=0;index < 100;index ++{
}

for {
}

for true{
}

for key,val:= range map[string]Hell{
}

for index ,value := range sliceA{

}

for val := range channelA{

} 

for val := <-channelB{
}
```

1. for循环用于遍历数组，

2. for循环用于遍历slice切片

3. 默认的空循环

4. for循环用于遍历一个map

5. for循环用于遍历一个channel,当channel关闭时，循环也结束

---

```
select{
}

select{
    case <-time.After(time.Second * 10):
    case <-closeChan:
}
```

1. `select`只用于`channel`操作
2. 或者一个类似的阻塞空的`select`
3. `select`中的`default`总是最后执行的

---

```
func CAll(){
}

type FF func()func()int
type FFF func() func()func()int
type F = FF
var future = func()int{
}
```

1. 在go语言中，func函数类似于int是第一类型，可以赋值，可以取址，可以定义新的类型，也可以作为参数，但是不可比较
2. 在go语言中，所有的函数参数都是传值调用，即使是`map,channel`也是传值，值得注意的是，数组是传值（数组的所有的信息都会被拷贝一份），slice也是传值（但是传的是一个包裹`len,cap,addr`的结构体，这也是为什么一些初学者或者对这里不清楚的使用者的迷惑之一，底层的address是同样的值）
3. 函数使用变参时，只能对最后一个函数使用变参类型
4. 汇编上来说，函数的参数和返回值都在函数的地址之上（`stack`）
5. 函数可以先命名函数的返回变量，并且在函数体中使用这个变量，defer函数也可以操作，这个变量根据类型（指针，非指针，引用，非引用），再加上defer函数的使用，会有不同的效果，一般情况下，这种使用方式不怎么推荐，可理解的代码是长久的，诗才是朗朗上口永存的
6. 结构体的成员函数也遵守上面的特性

---

```
func Call(){
    defer print("hello")
}

func Call()int{
    defer func(){
        in = 100
    }()
    var in = 10
    return in
}

func Call()(result int){
    defer func(){
        result = 10
    }
    retsult = 1
}

func Call()(result *int){
    defer func(){
        *result = 10
    }()
    i := 100
    return &i
}

func Call(){
    if true {
        defer func(){
            println("first")
        }()

    }else{
        defer func(){
            println("second")
        }()
    }
}

func TTTT(params int){
    println("fly : ",params)
}

func Call (){
    var ti = 10
    defer TTTT(ti)
    ti = 100
    println(ti)
}
```

1. defer函数类似于一般的析构函数，可以注册默认的行为
2. defer函数是函数块级别的，执行顺序为FIFO类型
3. defer函数可以修改已经声明的返回变量
4. defer注册的函数，在注册时会对参数预先求值

##### defer 语句会将函数调用放入一个列表. 当外围的函数执行返回后列表内保存的函数调用会被执行. defer 像 Python 的 with, finally 一样都具有类似资源情理的作用. 当然 defer 在这方面更灵活.defer 的行为遵循 3 个简单的原则:

* 一个 defer 函数的参数在放入待调用列表的时候被立即求值,这个例子中, @1 中的 i 会被立即求值, 所以 a 函数调用返回时, 会打印出 “0” 而不是 “1”.

> ```
> func a() {
>     i := 0
>     defer fmt.Println(i) // @1
>     i++
>     return
> }
> ```

* defer 函数在外围函数返回时遵循 **先进后出 **的调用顺序.b 函数将会打印出 “3210”.

> ```
> func b() {
>     for i := 0; i < 4; i++ {
>         defer fmt.Print(i)
>     }
> }
> ```

* defer 函数会读取并且赋值外围函数的命名返回值\(named return values\).这个例子中 i 的值会 在 defer 被调用时执行加 1 操作, 所以 c 将返回 2. 这对于修改一个函数的 error 返回值很方便.

> ```
> func c() (i int) {
>     defer func() { i++ }()
>     return 1
> }
> ```

---

panic 是一个内置的函数用来阻断一般的控制流程. Go spec 对于 panic 有一段 [描述](https://golang.org/ref/spec#Handling_panics):

> 当执行函数 F 时, 一个明确的 panic 调用或者 [运行时 panic](https://golang.org/ref/spec#Run_time_panics) \(数组越界访问等\)会终止 F 的执行. F 所属的任何 defer 函数继续执行. 紧接着, 任何 F 调用者的 defer 函数运行, 而且对于执行中的 goroutine 中的任何顶层 defer 函数也是如此. 到那时, 程序终止并且报错\(错误包含在 panic 参数中\). 该终止序列被称为 **panicking**
>
> .

---

recover 是一个内建函数用来恢复一个 panicking 的 goroutine. Go spec 对于 recover 有一段 [描述](https://golang.org/ref/spec#Handling_panics):

> 设想有一个函数 G, 有一个 defer 函数会调用 recover 并且和 G 在同一 goroutine 的函数发生了 panic. 当运行到 defer 函数 D 调用的时候, D 调用 recover 的返回值是 panic 调用时传入的值. 如果 D 正常返回, 没有开启一个新的 panic, panicking 序列会终止. 在这种情况下, G 和 panic 之间的函数调用状态被废弃, 并且正常的调用恢复. 然后 G 中任何在 D 之前的 defer 函数会执行, 并且 G 的执行通过返回到它的调用者而终止.

以下情况中 recover 的返回值为 nil:

> 1. panic 参数为 nil;
> 2. 当前 goroutine 没有发生 panicking;
> 3. recover 没有被一个 defer 函数直接调用.

#### _`Go 标准库中的约定是,`_`即使一个包在内部使用 panic, 其外部 API 仍会给出明确的错误返回值(真实的例子可以参见标准库(`[`json 包`](https://golang.org/src/pkg/encoding/json/decode.go)`). 这是很好的使用原则, 尤其当我们写自己库的时候, 难免需要处理来自第三方的包返回的各种 error, 如果不需要处理, 简单的一个 panic 会是很好的方案.`



