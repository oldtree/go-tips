### panic,recover,error

`Panic`的使用

`panic`在被触发时，会返回整个`goroutine`的运行`stack`,但是函数已经注册的`defer`函数还是会被执行

`Recover`的使用

`Recover`也是一个内建函数，提供控制一个`panic`的函数，`Recover`只在一个defer函数中运行，然后将一个已经`panic`的函数从`paniking`中恢复

`error`建议使用全局的`error`定义，必要时可以继承标准库的`error`定义，添加错误码，并且改写`String()`函数

一些关于`error`和`panic,recove`r的建议：

1. 在小型项目中，error定义全局变量即可，单独使用一个`.go`文件来管理
2. 在大型项目中，共享一个统一的error定义是可以的，这样在不同的模块判断错误的消耗会少一些；但是这样组织代码并不方便。在不同的模块（功能平级而不是上下级的关系）之间共享错误定义是不好的，应该在全局定义一个可以注册`error`的接口,error之间的判断需要使用自定义的`errorCode`这样，模块的`错误注册`由模块自己来做，这样可以集成进`etcd`等的可以用来做服务法相的组件中
3. `error`属于可控的程序错误，程序开发者会根据错误的信息来判定下一步的处理流程，`error`是程序的一部分
4. `panic`会导致运行stack信息丢失，属于**最边缘型**的错误处理,可以进行主动`panic`的流程
5. `recover`过程可以用来恢复业务的状态值，可以保证一些重要的流程即使`panic`也是可以再重建起来的，比如说一个不断处理任务的`for`循环，如果需要不断地去运行，则需要在`panic`时可以`recover`到正常的流程

---

### Go errors 堆栈

* ##### Errors 也是值

> [Errors are values](https://blog.golang.org/errors-are-values), 错误也是一种值, 就像其他任何类型的值一样
>
> ```
> // https://play.golang.org/p/VFVX0fRKBnS
> package main
>
> import (
>     "fmt"
>     "strconv"
> )
>
> func main() {
>     if _, err := strconv.ParseInt("abcd", 10, 64); err != nil {
>         fmt.Println(err)
>     }
> }
> // 输出
> // strconv.ParseInt: parsing "abcd": invalid syntax
> ```
>
> 看起来好像还不错, 起码我们知道 3 个内容:
>
> 1. 错误发生的 func: strconv.ParseInt
> 2. 发生错误 func 的参数: abcd
> 3. 错误的原因: invalid syntax
>
> 如果查看[`strconv.ParseInt`](https://golang.org/src/strconv/atoi.go?s=4086:4153#L146)` `的源码你会发现, 代码如标准库者依然没有使用最原始的 error 返回. 它之所以打印出上面的 3 个内容, 是因为使用了自定义的 NumError :
>
> ```
> // A NumError records a failed conversion.
> type NumError struct {
>     Func string // the failing function (ParseBool, ParseInt, ParseUint, ParseFloat)
>     Num  string // the input
>     Err  error  // the reason the conversion failed (e.g. ErrRange, ErrSyntax, etc.)
> }
>
> func (e *NumError) Error() string {
>     return "strconv." + e.Func + ": " + "parsing " + Quote(e.Num) + ": " + e.Err.Error()
> }
> ```
>
> 如果我们领会了 Errors are values 的精神, 基本上能写出这样的错误处理已经很符合[不要仅仅检查 errors, 优雅的处理它们](https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully)的宗旨了

* ##### Errors 堆栈跟踪

> 用 Go 强大的 runtime 包, 尝试自己实现一个简单的自定义 errors 堆栈跟踪:
>
> ```
> // https://play.golang.org/p/-tesfXuy9fc
> package main
>
> import (
> 	"fmt"
> 	"runtime"
> 	"strings"
> )
>
> func callers() []uintptr {
> 	var pcs [32]uintptr
> 	n := runtime.Callers(3, pcs[:])
> 	st := pcs[0:n]
> 	return st
> }
>
> type trace struct {
> 	m string
> 	s []uintptr
> }
>
> func (e *trace) Error() string {
> 	var b strings.Builder
> 	b.WriteString(e.m)
> 	b.WriteString("\n\n")
> 	b.WriteString("Traceback:")
> 	for _, pc := range e.s {
> 		fn := runtime.FuncForPC(pc)
> 		b.WriteString("\n")
> 		f, n := fn.FileLine(pc)
> 		b.WriteString(fmt.Sprintf("%s:%d", f, n))
> 	}
> 	return b.String()
> }
>
> // NewTrace creates a simple traceable error.
> func NewTrace(message string) error {
> 	return &trace{m: message, s: callers()}
> }
>
> func f() error {
> 	return NewTrace("ooops")
> }
>
> func main() {
> 	fmt.Println(f())
> }
>
> // goplay 输出
> // ooops
>
> // Traceback:
> // /tmp/sandbox315155193/main.go:41
> // /tmp/sandbox315155193/main.go:45
> // /usr/local/go/src/runtime/proc.go:207
> // /usr/local/go/src/runtime/asm_amd64p32.s:968
> ```

* ##### 社区实现\(pkg/errors\)

> 我们当然不打算在平常的代码中使用这样一个简单的实现, 我们这里介绍一下社区已经存在的[pkg/errors](https://github.com/pkg/errors)库.
>
> 对于标准库或者第三方库的 errors 返回我们需要一个简单的 wrap, 以便在同样使用 pkg/errors 的地方可以获取一致的 errors 体验:
>
> ```
> _, err := ioutil.ReadAll(r)
> if err != nil {
>         return errors.Wrap(err, "read failed")
> }
> ```

> 我们看看怎样使用 pkg/errors 得到的 errors:
>
> ```
> package main
>
> import (
> 	"fmt"
> 	"os"
>
> 	"github.com/pkg/errors"
> )
>
> func parseArgs(args []string) error {
> 	if len(args) < 3 {
> 		return errors.Errorf("not enough arguments, expected at least 3, got %d", len(args))
> 	}
> 	return nil
> }
>
> func main() {
> 	err := parseArgs(os.Args[1:])
> 	fmt.Printf("%v\n", err)
> }
>
> // 输出
> // not enough arguments, expected at least 3, got 0
> ```

> 没有堆栈信息打印,更改一下：
>
>     // ...
>     func main() {
>     	err := parseArgs(os.Args[1:])
>     	fmt.Printf("%+v\n", err) // 没错, 加个 `+` flag
>     }
>
>     // 输出
>     // not enough arguments, expected at least 3, got 0
>     // main.parseArgs
>     // 	/Users/xxx/.go/src/github.com/gorocks/snippets/go/cmd/gosnippets/main.go:12
>     // main.main
>     // 	/Users/xxx/.go/src/github.com/gorocks/snippets/go/cmd/gosnippets/main.go:18
>     // runtime.main
>     // 	/usr/local/Cellar/go/1.10.1/libexec/src/runtime/proc.go:198
>     // runtime.goexit
>     // 	/usr/local/Cellar/go/1.10.1/libexec/src/runtime/asm_amd64.s:2361

> 之所以会有这个效果是因为 pkg/errors 实现了 [Formatter](https://golang.org/pkg/fmt/#Formatter) 接口:
>
> ```
> func (f *fundamental) Format(s fmt.State, verb rune) {
> 	switch verb {
> 	case 'v':
> 		if s.Flag('+') {
> 			io.WriteString(s, f.msg)
> 			f.stack.Format(s, verb)
> 			return
> 		}
> 		fallthrough
> 	case 's':
> 		io.WriteString(s, f.msg)
> 	case 'q':
> 		fmt.Fprintf(s, "%q", f.msg)
> 	}
> }
> ```

> 实现 Formatter 的一个好处是, 我们可以如使用标准库般始终如一, 没有多余的任何 func 调用.
>
> pkg/errors 堆栈跟踪不是没有运行时开销的,[官方](https://github.com/pkg/errors/issues/72)给出的指标是每个操作大约 1000-3000 ns.
>
> 一般来说这不会构成性能忧虑. 如果超过了你的性能预期, 可以定制成调试模式启用.



