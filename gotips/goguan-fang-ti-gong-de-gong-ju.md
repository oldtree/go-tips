* ## **go tools**

介绍一些go tools 的使用方法

```
go tool fix -r context your/package
```

`go fix` 可以用来修复指定的代码包引入，比如在大版本升级后或者使用新的包之后

可是在多种运行模式下使用race参数：

```
go test -race mypkg
go run -race mysrc.go
go build -race mycmd
go install -race mypkg
```

```
go build
//编译代码包cnet/ctcp，只需要在任意目录下执行命令go build cnet/ctcp即可
```

```
go build logging/base.go logging/console_logger.go logging/log_manager.go logging/tag.go
//同时编译多个Go源码文件:.使用这种方法会有一个限制。作为参数的多个Go源码文件必须在同一个目录中
```

```
go build命令在编译只包含库源码文件的代码包（或者同时编译多个代码包）时，只会做检查性的编译，而不会输出任何结果文件。
```

```
go build命令编译单一命令源码文件。我们在执行命令时加入一个标记-v。这个标记的意义在于可以使命令把执行过程中构建的包名打印出来
```

```
go build会把编译命令源码文件后生成的结果文件存放到执行该命令时所在的目录下。这个所说的结果文件就是与命令源码文件对应的可执行文件。它的名称会与命令源码文件的主文件名相同
```

```
go build 使用-o标记可以指定输出文件（在这个示例中指的是可执行文件）的名称
```

```
go build -i会使go build命令安装那些编译目标依赖的且还未被安装的代码包。这里的安装意味着产生与代码包对应的归档文件，并将其放置到当前
工作区目录的pkg子目录的相应子目录中。在默认情况下，这些代码包是不会被安装的。
```

| go build 的命令参数 |
| :--- |


| 标记名称 | 标记描述 |
| :--- | :--- |
| -a | 强行对所有涉及到的代码包（包含标准库中的代码包）进行重新构建，即使它们已经是最新的了。 |
| -n | 打印编译期间所用到的其它命令，但是并不真正执行它们。 |
| -p n | 指定编译过程中执行各任务的并行数量（确切地说应该是并发数量）。在默认情况下，该数量等于CPU的逻辑核数。但是在`darwin/arm`平台（即iPhone和iPad所用的平台）下，该数量默认是`1`。 |
| -race | 开启竞态条件的检测。不过此标记目前仅在`linux/amd64`、`freebsd/amd64`、`darwin/amd64`和`windows/amd64`平台下受到支持。 |
| -v | 打印出那些被编译的代码包的名字。 |
| -work | 打印出编译时生成的临时工作目录的路径，并在编译结束时保留它。在默认情况下，编译结束时会删除该目录。 |
| -x | 打印编译期间所用到的其它命令。注意它与`-n`标记的区别。 |
| -asmflags | -D，-I ，-S 用于控制Go语言编译器编译汇编语言文件时的行为 |
|  |  |

* -buildmode

> ```
> 此标记用于指定编译模式，使用方式如-buildmode=default（这等同于默认情况下的设置）。此标记支持的编译模式目前有6种。
> 借此，我们可以控制编译器在编译完成后生成静态链接库（即.a文件，也就是我们之前说的归档文件）、动态链接库（即.so文件）
> 或/和可执行文件（在Windows下是.exe文件）
> ```

* `-compiler`

> ```
> 此标记用于指定当前使用的编译器的名称。其值可以为gc或gccgo。其中，gc编译器即为Go语言自带的编辑器，而gccgo编译器则为
> GCC提供的Go语言编译器
> ```

* `-gccgoflags`

> ```
> 此标记用于指定需要传递给gccgo编译器或链接器的标记的列表。
> ```

* `-gcflags`

> ```
> 此标记用于指定需要传递给go tool compile命令的标记的列表
> ```

* `-installsuffix`

> ```
> 为了使当前的输出目录与默认的编译输出目录分离，可以使用这个标记。此标记的值会作为结果文件的父目录名称的后缀。其实，
> 如果使用了-race标记，这个标记会被自动追加且其值会为race。如果我们同时使用了-race标记和-installsuffix，那么在
> -installsuffix标记的值的后面会再被追加_race，并以此来作为实际使用的后缀
> ```

* `-ldflags`

> ```
> 此标记用于指定需要传递给go tool link命令的标记的列表
> ```

* `-linkshared`

> ```
> 此标记用于与-buildmode=shared一同使用。后者会使作为编译目标的非main代码包都被合并到一个动态链接库文件中，而前者则会在此之上进行链接操作。
> ```

* `-pkgdir`

> ```
> 使用此标记可以指定一个目录。编译器会只从该目录中加载代码包的归档文件，并会把编译可能会生成的代码包归档文件放置在该目录下。
> ```

* `-tags`

> ```
> 此标记用于指定在实际编译期间需要受理的编译标签（也可被称为编译约束）的列表。这些编译标签一般会作为源码文件开始处的注释的一部分
> 例如，在$GOROOT/src/os/file_posix.go开始处的注释为:
> // Copyright 2009 The Go Authors. All rights reserved.
> // Use of this source code is governed by a BSD-style
> // license that can be found in the LICENSE file.
>
> // +build darwin dragonfly freebsd linux nacl netbsd openbsd solaris windows
> 最后一行注释即包含了与编译标签有关的内容。大家可以查看代码包go/build的文档已获得更多的关于编译标签的信息。
> ```

* `-toolexec`

> ```
> 此标记可以让我们去自定义在编译期间使用一些Go语言自带工具（如vet、asm等）的方式
> ```



