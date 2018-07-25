# The Go Memory Model

`https://golang.org/ref/mem`

不同协程之间的变量是可以互相访问的，不论是写还是读

这样在多个协程操作一个数据时，需要根据他们所访问的顺序来对数据进行操作，可以使用[`sync`](https://golang.org/pkg/sync/)和[`sync/atomic`](https://golang.org/pkg/sync/atomic/) 来进行数据保护，或者使用一个`go channel`来对数据的操作请求进行排队并取出处理

## Happens Before的理解 {#tmp_2}

在一个单一的协程中，对数据的读和写都是按照实际执行顺序来发生

1. **如果事件1在事件2之前发生，则称之为事件2在事件1之后才发生**
2. **如果事件1没有在事件2之前发生，也没有在事件2之后发生，则称之为事件2事件2并行发生**

这个定义并不是多么的高深，更多是一种限先置的声明和定义

程序只运行一个goroutine时，这个goroutine启动了一个或者多个新的goroutine,则这些goroutine是并行的

如果package A import package B,那么B package的init函数先于A package启动，main.main会在所import包的init函数执行结束之后才会执行

---

```
var c = make(chan int, 10)
var a string

func f() {
    a = "hello, world"
    c <- 0
}

func main() {
    go f()
    <-c
    print(a)
}
```

对一个`channel`使用`close()`函数，会使所有阻塞在这个`channel`的`goroutine`不阻塞恢复执行,如上所示，在f\(\)函数中`close(c)`对`main()`来说有相同的行为

---

```
var limit = make(chan int, 3)
func main() {
    for _, w := range work {
        go func(w func()) {
            limit <- 1
            w()
            <-limit
        }(w)
    }
    select{}
}
```

拥有缓冲池的协程，可以用来做限制访问的limit,当buffer channel被填满，就会阻塞在这里，如上所示，可以用来做请求的限流

---

```
var l sync.Mutex
var a string

func f() {
    a = "hello, world"
    l.Unlock()
}

func main() {
    l.Lock()
    go f()
    l.Lock()
    print(a)
}
```

也可以使用sync包来实现happends before的规则，在等待执行的函数中，使用`Unlock`操作来等待`Lock`的发生，然后在重入加锁以保证顺序

---

```
var a string
var once sync.Once

func setup() {
    a = "hello, world"
}

func doprint() {
    once.Do(setup)
    print(a)
}

func twoprint() {
    go doprint()
    go doprint()
}
```

once类似于java或者C++中的静态成员，只执行一次所注册的函数，如上述代码：`doprint`只会被执行一次，另一个调用将会被阻塞直至先调用的`doprint`返回

