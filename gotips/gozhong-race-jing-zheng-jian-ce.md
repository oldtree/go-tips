### [Go Race Detector](https://blog.golang.org/race-detector)

[https://blog.golang.org/race-detector](https://blog.golang.org/race-detector)

比如在如下程序中，recover 不能恢复所有的 panic, 比如如下的程序:

> ```
> package main
>
> import (
>     "fmt"
>     "sync"
> )
>
> func main() {
>     defer func() {
>         if err := recover(); err != nil {
>             fmt.Println("recovered")
>         }
>     }()
>
>     m := map[int]int{}
>     var wg sync.WaitGroup
>
>     for i := 0; i < 10; i++ {
>         wg.Add(1)
>         v := i
>         go func() {
>             defer wg.Done()
>             m[v] = v
>         }()
>     }
>     wg.Wait()
> }
> ```

如果运行该程序, 那么会得到如下的输出:

> ```
> fatal error: concurrent map writes
> fatal error: concurrent map writes
> ```

_其实这个不叫 panic,_ 按官方文档的说法叫: 崩溃\(crash\) 和 内存腐败\(memory corruption\). 由于程序无视 [Go 内存模型](https://golang.org/ref/mem/) 而导致的数据竞争\(data race\).

**为了检测和预防这种崩溃, Go 内建了数据竞争探测器\(data race detector\), 可以通过传递 flag: -race 给 go 命令来启用:**

> ```
> $ go test -race mypkg
> $ go run -race mysrc.go
> $ go build -race mycmd
> $ go install -race mypkg
> ```

我们再次对上面的程序使用命令: go run -race main.go, 那么数据探测器可以检测到:

> ```
> ==================
> WARNING: DATA RACE
> Write at 0x00c420090180 by goroutine 6:
>   runtime.mapassign_faststr()
>       /usr/local/Cellar/go/1.10/libexec/src/runtime/hashmap_fast.go:694 +0x0
>   main.main.func1()
>       /Users/a/.go/src/github.com/gorocks/snippets/go/cmd/gosnippets/main.go:9 +0x5d
>
> Previous write at 0x00c420090180 by main goroutine:
>   runtime.mapassign_faststr()
>       /usr/local/Cellar/go/1.10/libexec/src/runtime/hashmap_fast.go:694 +0x0
>   main.main()
>       /Users/a/.go/src/github.com/gorocks/snippets/go/cmd/gosnippets/main.go:12 +0xc9
>
> Goroutine 6 (running) created at:
>   main.main()
>       /Users/a/.go/src/github.com/gorocks/snippets/go/cmd/gosnippets/main.go:8 +0x9a
> ==================
> 1 a
> 2 b
> Found 1 data race(s)
> exit status 66
> ```



