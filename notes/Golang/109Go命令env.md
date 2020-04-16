title: 109.Go命令之go env
date: 2020-04-15
tags: Go命令
categories: Go命令
layout: post

------

摘要：本节介绍`Go`命令中的`go env`环境变量命令...

<!-- more -->

## 用途

维护、打印`Go`语言的环境信息

**命令格式**

```bash
$ go env [环境变量]
```

如果指定环境变量，则只输出环境变量值，否则输出所有环境变量信息

```bash
$ go env GOPATH
D:\Users\lijie\go
```

## 环境变量

介绍常用的环境变量

| 名称        | 说明                                       |
| ----------- | ------------------------------------------ |
| CGO_ENABLED | 指明`cgo`工具是否可用的标识                |
| GO111MODULE | 是否使用`go module`管理依赖                |
| GOARCH      | 程序**构建环境**的目标计算架构             |
| GOBIN       | 存放可执行文件的目录的绝对路径             |
| GOCHAR      | 程序**构建环境**的目标计算架构的单字符标识 |
| GOEXE       | 可执行文件的后缀                           |
| GOHOSTARCH  | 程序**运行环境**的目标计算架构             |
| GOOS        | 程序**构建环境**的目标操作系统             |
| GOHOSTOS    | 程序**运行环境**的目标操作系统             |
| GOPATH      | 工作区目录的绝对路径                       |
| GORACE      | 用于数据竞争检测的相关选项                 |
| GOROOT      | `Go`语言的安装目录的绝对路径               |
| GOTOOLDIR   | `Go`工具目录的绝对路径                     |

### CGO_ENABLED

是否可以使用`cgo`进行编译，`cgo`是基于`C`语言的编译工具，不支持交叉编译

注意：如果存在交叉编译的时候（即构建环境与运行环境的架构不同），必须禁用`cgo`

```bash
set CGO_ENABLED=1 #开启 
set CGO_ENABLED=0 #禁用
```

### GO111MODULE

启用`go module`进行包依赖管理，从`go v1.11`版本开始引入。可以替代之前的一些包依赖管理工具

```bash
set GO111MODULE=on #启用
set GO111MODULE=off #禁用
set GO111MODULE=auto | (空白) #自动 根据目录下是否有`go.mod`文件判断是否启用
```

### GOARCH

程序构建环境的目标计算架构的标识，也就是程序在构建或安装时所对应的计算架构的名称

| 名称  | 描述        |
| ----- | ----------- |
| 386   | 32位x86架构 |
| amd64 | 64位x86架构 |
| arm   | 32位arm架构 |
| arm64 | 64位arm架构 |

注：可以使用`go tool dist list`命令查看所有支持的`os`和`arch`值

### GOBIN

存放可执行文件的目录，是绝对路径。使用`go install`命令安装命令源码文件时生成的可执行文件会存放于这个目录中

默认是空白

### GOCHAR

程序构建环境的目标计算架构的单字符标识，与`GOARCH`值相对应

| GOARCH | GOCHAR |
| ------ | ------ |
| 386    | 8      |
| amd64  | 6      |
| arm    | 5      |

- 官方平台的相关工具的名称会以其值为前缀。如：在`amd64`计算架构下，用于编译`Go`语言代码的编译器名称是`6g`，链接器名称是`6l`。用于编译`C`语言代码的编译器名称是`6c`，编译汇编语言代码的编译器名称是`6a`
- 官方编译器生成的结果文件会以其值为扩展名。如：`Go`语言的官方编译器`6g`在对命令源码文件编译之后会把结果文件`go.6`存放到临时工作目录的相应位置中

### GOEXE

可执行文件的后缀。与`GOOS`的值存在一定关系，只有`GOOS`的值为“windows”时GOEXE的值才会是“.exe”，否则其值就为空字符串""

### GOHOSTARCH

程序**运行环境**的目标计算架构的标识。通常情况下，不需要被显式设置。因为用来安装`Go`语言的二进制分发文件和`MSI`软件包文件都是平台相关的。所以，对于不同计算架构的`Go`语言环境来说，它都会是一个常量

### GOOS

程序**构建环境**的目标操作系统的标识。与`GOARCH`类似，在不同操作系统下是固定不变的，不需要显式设置

### GOHOSTOS

程序**运行环境**的目标操作系统的标识。与`GOHOSTARCH`类似，在不同操作系统下是固定不变的，不需要显式设置

### GOPATH

工作区目录，其值是绝对路径。需要显式设置环境变量`GOPATH`。如果有多个工作区，那么多个工作区的绝对路径之间需要用分隔符分隔。`windows`操作系统下分隔符为";"，其它操作系统下分隔符为":"。

注意：`GOPATH`的值不能与`GOROOT`的值相同

### GORACE

### GOROOT

安装目录，其值是绝对路径。默认情况下，不需要手动设置其值

### GOTOOLPATH

工具目录，其值是绝对路径。根据`GOROOT`、`GOHOSTOS`和`GOHOSTARCH`来设置，其值为`$GOROOT/pkg/tool/$GOOS_$GOARCH`

### GOPROXY

是一个代理，让我们更方便的下载哪些由于墙的原因而导致无法下载的第三方包，比如`golang.org/x/`下的包

```bash
set GOPROXY=https://goproxy.io,direct
```

一般国内使用的代理有如下几个：

- `goproxy.io`，速度较慢
- `goproxy.cn`，七牛云提供代理
- `mirrors.aliyun.com/goproxy`，阿里云提供代理

## 命令标记

### `-w`写入环境变量

修改环境变量信息，并写入`go env`中

**标记格式**

```bash
$ go env -w 变量名称=变量值
```

该标记需要一个或多个`变量名称=变量值`参数对

```bash
$ go env -w GOPROXY=https://goproxy.io,direct
```

### `-json`输出格式

默认情况下，输出在命令行中的信息，使用标准文本输出格式

```bash
$ go env
set GO111MODULE=on
set GOARCH=amd64
set GOBIN=
...
```

该标记可以使输出格式转变为`json`格式

```bash
$ go env -json
{
        "AR": "ar",
        "CC": "gcc",
        "CGO_CFLAGS": "-g -O2",
        ...
```

