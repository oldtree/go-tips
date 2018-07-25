# 语法介绍

* [https://blog.golang.org/go-maps-in-action](https://blog.golang.org/go-maps-in-action)

go语言提供了原生的内建的hash table，使用时声明如下：

```
map[KeyType]ValueType
```

在这里，map中的key必须是可比较类型的，指针是可以比较的，但引用类型不是，所以`map,channel,slice`在这里是不可用的，但是value的值是可以任意类型的，包括`map,interface{},channel`

在go语言中，hash是一个引用类型（Map types are reference types, like pointers or slices, and so the value of`m`above is`nil`），初始化时是一个`nil`值,并不会指向一个已经分配好的`map`,一个nil没有初始化的`map`是可读的，但是却是不可写的，任何的对`nil map`的写操作都会`panic`

初始化一个`map:`

```
m = make(map[string]int)
```

这条语句会初始化一个`map`的结构，并返回一个指向它的引用指针

可用于一个`map`的操作：

```
m["route"] = 66
i := m["route"]
j := m["root"]
// j == 0 j not exist
n := len(m)
delete(m, "route")
i, ok := m["route"]
//如果存在key-pair,那么ok=true,i有非nil值，否则，ok=false,i=nil
_, ok := m["route"]
m = map[string]int{}
//初始化一个空的map
var counter = struct{
    sync.RWMutex
    m map[string]int
}{m: make(map[string]int)}
```

对一个`map`进行遍历：

```
for key, value := range m {
    fmt.Println("Key:", key, "Value:", value)
}
```

`map`并不是并行安全的需要使用`sync.Mutex`或者`sync.Map`,或者使用`channel`来实现一个

`map`的遍历是随机的，如果需要有序的访问结果，只能通过自建有序`key`来进行遍历

```
import "sort"
var m map[int]string
var keys []int
for k := range m {
    keys = append(keys, k)
}
sort.Ints(keys)
for _, k := range keys {
    fmt.Println("Key:", k, "Value:", m[k])
}
```



