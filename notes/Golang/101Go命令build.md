title: 101.Go命令之go build
date: 2020-04-08
tags: Go命令
categories: Go命令
layout: post

------

摘要：本节介绍`Go`命令中最常用的`go build`编译命令...

<!-- more -->

## 用途

`go build`编译指定的源文件、代码包以及它们的依赖包

### 源文件

命令格式

```bash
$ go build 包名/文件1.go 包名/文件2.go ...
```

可以一次编译多个文件，根据执行命令所在的目录确定是否需要**包名**

注意：编译指定源文件时，所有的源文件**只能在一个包中**

```bash
# 正确的命令（同一个包中的多个文件）
$ go build logging/base.go logging/tag.go
# 错误的命令（不同包中的文件）
$ go build logging/base.go format/log.go
```

### 代码包

命令格式

```bash
$ go build [包名]
```
说明：

- 包名作为参数，则可以在任何目录下执行。具体编译的包是 `$GOPATH/src/包名`
- 没有任何参数，需要在包所在的目录下执行。

```bash
# 指定包名，可以任何位置执行，编译包：$GOPATH/src/net/tcp
$ go build net/tcp
# 在目录中，编译目录中的包：logging
~/users/go/src/logging$ go build
```

注意：与编译源文件一样，不能同时编译多个包

### 依赖包

先明确两个概念，什么是依赖包和触发包？

如果包A依赖包B，则包B称为包A的依赖包，包A称为包B的触发包

```go
package app

import (
    "logging"
    "basic"
)
```

- 包`app`是`logging`和`basic`的触发包
- 包`logging`是包`app`的依赖包
- 包`basic`是包`app`的依赖包

我们知道，编译命令**不能**同时编译多个包，如果一个包一个包编译会非常麻烦

因此，在编译时，被编译包依赖其它包时，编译器将先编译依赖包，再编译当前包

举个例子：要编译包`app`，该包依赖`logging`和`basic`

```go
package app

import (
	"logging"
    "basic"
)
```

执行编译命令

```bash
$ go build app
```
则编译的顺序是

- 编译依赖包`logging`
- 编译依赖包`basic`
- 编译触发包`app`

注意：包间不能存在循环依赖，也就是说：包A依赖包B，包B依赖包A。如果存在循环依赖，则编译时会报错误

## 输出

编译代码是为了检查代码并输出结果，如：可执行文件、静态链接库、动态链接库等

我们知道，`Go`有三种类型的源文件：命令源文件、库源文件、测试源文件。编译命令只对前两种起作用，只有编译命令源文件时才会输出可执行文件

### 库源文件/库包

执行编译命令时，不会有任何的输出结果（其实结果保存在临时文件中），`Go`记录该文件/包已经编译以及编译的状态

### 命令源文件/main包

我们知道，`Go`的每个可执行程序都包含一个命名为`main`的包，包中包含`main()`函数的源文件称为命令源文件

当编译命令包/命令源文件时，编译通过后会在当前目录下生成可执行文件

## 编译参数

### `-o`指定输出文件名称

编译命令源文件/命令包时，在执行编译命令所在目录生成可执行文件，默认情况，该文件名与命令源文件名/命令包名相同。使用`-o`参数，可以设置输出可执行文件的名称

```bash
$ go build -o collsrv.exe app 
```

注意：在`windows`平台下，需要指定文件的扩展名

### `-v`显示构建包名

```bash
$ go build -v hello.go
command-line-arguments
```

这里显示的并不是它们所属的代码包的导入路径`other`。这是因为，命令程序在分析参数的时候如果发现第一个参数是`Go`源码文件而不是代码包，则会在内部生成一个虚拟代码包。这个虚拟代码包的导入路径和名称都会是`command-line-arguments`

### `-i`安装依赖包

在编译时，自动安装编译目标依赖的且未被安装的代码包。这里的安装意味着产生与代码包对应的归档文件，并将其放置到当前工作区目录的`pkg`子目录的相应子目录中。**在默认情况下，这些代码包是不会被安装的**。

工作目录如下

```bash
├── hello
│   ├── other
│   │   ├── other.go
│   ├── hello.go
```

`hello.go`源文件

```go
package main

import(
    "fmt"
    "hello/other"
)
```

`other.go`源文件

```go
package other

const (
	Monday  = iota //星期一
	Tuesday        //星期二
)
```

执行`go build -i`后，`pkg`中发生了一些变化

```bash
├── pkg
│   ├── mod
│   ├── submod
│   ├── windows_amd64 # 新增目录
│   │   ├── hello
│   │   |   ├── other.a # 安装的other包

```

### `-work`显示编译时临时工作目录

默认情况下，编译器会在编译开始时创建临时目录，并且在编译结束后删除该临时目录

该参数在命令行中打印输出创建的临时目录，并且在编译结束后保留该临时目录

```bash
$ go build -v -work logging
WORK=/tmp/go-build888760008
logging
```

### `-a`强制重新编译涉及到的所有代码包

默认情况下，编译通过的库代码包，在没有改动时不会再次编译

该参数强制对涉及到的所有代码包进行重新编译，并在命令行输出这些包名

```bash
$ go build -a -v -work hello
WORK=C:\Users\lijie\AppData\Local\Temp\go-build131882478
runtime/internal/sys
math/bits
runtime/internal/atomic
internal/cpu
runtime/internal/math
unicode/utf8
internal/race
sync/atomic
internal/bytealg
math
unicode
internal/syscall/windows/sysdll
unicode/utf16
internal/testlog
runtime
hello/other
internal/reflectlite
sync
errors
sort
internal/oserror
io
strconv
syscall
reflect
internal/syscall/windows
internal/syscall/windows/registry
time
internal/fmtsort
internal/poll
os
fmt
hello
```

看到没有，这一个长长的列表，只有几行代码的源文件，需要依赖这么多的库包

### `-p n`指定编译任务并行数量

指定编译过程中执行各任务的并行数量（确切地说应该是并发数量）。在默认情况下，该数量等于`CPU`的逻辑核数。但是在`darwin/arm`平台（即`iPhone`和`iPad`所用的平台）下，该数量默认是`1`

```bash
$ go build -a -v -work -p 1 hello
WORK=C:\Users\lijie\AppData\Local\Temp\go-build545214450
internal/cpu
internal/bytealg
runtime/internal/atomic
runtime/internal/sys
runtime/internal/math
runtime
internal/reflectlite
errors
math/bits
math
unicode/utf8
strconv
internal/race
sync/atomic
sync
unicode
reflect
sort
internal/fmtsort
io
internal/oserror
internal/syscall/windows/sysdll
unicode/utf16
syscall
internal/syscall/windows
internal/syscall/windows/registry
time
internal/poll
internal/testlog
os
fmt
hello/other
hello
```

注意：指定只有`1`个并发任务时，输出的包名顺序才是真正体现出包的依赖关系。这个顺序与前面的顺序是不同的

### 输出编译期间执行的指令`-n`与`-x`

在实际编译期间，编译器会指定一系列的命令，完成代码格式化、语法检查、编译、链接、输出可执行文件等

```bash
$ go build -x hello
WORK=C:\Users\lijie\AppData\Local\Temp\go-build404782846
mkdir -p $WORK\b001\
cat >$WORK\b001\importcfg.link << 'EOF' # internal
packagefile hello=C:\Users\lijie\AppData\Local\go-build\9a\9a35cb461fb41e0758c6b9e08aa353f612f4eac43e709dead41e54c3b7d970cc-d
packagefile fmt=c:\go\pkg\windows_amd64\fmt.a
packagefile hello/other=C:\Users\lijie\AppData\Local\go-build\d2\d22494281fc39ba581ff1c507e77f75043507d68feba7dbb8c87d288bdc701f1-d
packagefile runtime=c:\go\pkg\windows_amd64\runtime.a
packagefile errors=c:\go\pkg\windows_amd64\errors.a
packagefile internal/fmtsort=c:\go\pkg\windows_amd64\internal\fmtsort.a
packagefile io=c:\go\pkg\windows_amd64\io.a
packagefile math=c:\go\pkg\windows_amd64\math.a
packagefile os=c:\go\pkg\windows_amd64\os.a
packagefile reflect=c:\go\pkg\windows_amd64\reflect.a
packagefile strconv=c:\go\pkg\windows_amd64\strconv.a
packagefile sync=c:\go\pkg\windows_amd64\sync.a
packagefile unicode/utf8=c:\go\pkg\windows_amd64\unicode\utf8.a
packagefile internal/bytealg=c:\go\pkg\windows_amd64\internal\bytealg.a
packagefile internal/cpu=c:\go\pkg\windows_amd64\internal\cpu.a
packagefile runtime/internal/atomic=c:\go\pkg\windows_amd64\runtime\internal\atomic.a
packagefile runtime/internal/math=c:\go\pkg\windows_amd64\runtime\internal\math.a
packagefile runtime/internal/sys=c:\go\pkg\windows_amd64\runtime\internal\sys.a
packagefile internal/reflectlite=c:\go\pkg\windows_amd64\internal\reflectlite.a
packagefile sort=c:\go\pkg\windows_amd64\sort.a
packagefile math/bits=c:\go\pkg\windows_amd64\math\bits.a
packagefile internal/oserror=c:\go\pkg\windows_amd64\internal\oserror.a
packagefile internal/poll=c:\go\pkg\windows_amd64\internal\poll.a
packagefile internal/syscall/windows=c:\go\pkg\windows_amd64\internal\syscall\windows.a
packagefile internal/testlog=c:\go\pkg\windows_amd64\internal\testlog.a
packagefile sync/atomic=c:\go\pkg\windows_amd64\sync\atomic.a
packagefile syscall=c:\go\pkg\windows_amd64\syscall.a
packagefile time=c:\go\pkg\windows_amd64\time.a
packagefile unicode/utf16=c:\go\pkg\windows_amd64\unicode\utf16.a
packagefile unicode=c:\go\pkg\windows_amd64\unicode.a
packagefile internal/race=c:\go\pkg\windows_amd64\internal\race.a
packagefile internal/syscall/windows/sysdll=c:\go\pkg\windows_amd64\internal\syscall\windows\sysdll.a
packagefile internal/syscall/windows/registry=c:\go\pkg\windows_amd64\internal\syscall\windows\registry.a
EOF
mkdir -p $WORK\b001\exe\
cd .
"c:\\go\\pkg\\tool\\windows_amd64\\link.exe" -o "$WORK\\b001\\exe\\a.out.exe" -importcfg "$WORK\\b001\\importcfg.link" -buildmode=exe -buildid=hfsXM9iSACNXWQEA-5Ko/3u_Owsxy0L4Uf2HMk_Ao/8gIynBqIWz50y4H6oZx4/hfsXM9iSACNXWQEA-5Ko -extld=gcc "C:\\Users\\lijie\\AppData\\Local\\go-build\\9a\\9a35cb461fb41e0758c6b9e08aa353f612f4eac43e709dead41e54c3b7d970cc-d"
"c:\\go\\pkg\\tool\\windows_amd64\\buildid.exe" -w "$WORK\\b001\\exe\\a.out.exe" # internal
cp $WORK\b001\exe\a.out.exe hello.exe
rm -r $WORK\b001\
```

