title: 110.Go命令之go mod
date: 2020-04-15
tags: Go命令
categories: Go命令
layout: post

------

摘要：本节介绍`Go`命令中的`go mod`依赖包管理命令...

<!-- more -->

## 简介

> `Go module`是用来解决`Go`语言开发过程中，各种库代码包的依赖管理问题。类似于`pip`和`npm`

在`go module`推出之前，使用`GOPATH`进行项目中各种包的依赖管理，这部分前面章节均有介绍。使用第三方的各种库代码包时，我们通常情况下使用`go get`指令，将代码包下载到`GOPATH`工作目录中的`src`和`pkg`子目录下。

`Go`语言官方推荐每个项目配置独立的`GOPATH`使用，但这样一来有些重复使用的代码包就会重复下载，并且每个项目使用的代码包的版本也各不相同，给后期的维护工作带来不小的麻烦。

当然也有解决方案，给`GOPATH`配置多个工作目录，第一个工作目录用来保存第三方下载的库代码包。这样可以解决很多项目公用库代码包的问题，但也同时带来其他问题，比如不同版本的兼容性问题等。

还有一些大牛推荐的替代方案`govener`，将依赖包保存到项目中的`vender`目录中，可以独立维护不同项目的依赖包

到了`Go V1.11`版本时，官方推出了`go module`来解决依赖管理的问题。该方案可以将工作目录从`GOPATH`中脱离出来，`GOPATH`只负责保存依赖包，而项目目录可以根据需要自行定义。

## 启用

默认情况下，`Go`使用`GOPATH`进行依赖管理，要启用`Go module`需要手动开启环境变量

```bash
$ go env -w GO111MODULE=on
```

由于国内访问`golang.org`被墙，需要使用国内的镜像代理

```bash
$ go env -w GOPROXY=https://goproxy.cn,direct
```

可以使用三个常用的镜像代理站点

- `goproxy.io`，速度较慢
- `goproxy.cn`，七牛云提供代理
- `mirrors.aliyun.com/goproxy`，阿里云提供代理

注1：只有开启`go module`，才能使用`GOPROXY`代理模式进行下载

注2：开启`GO module`模式后，将自动关闭`GOPATH`模式

## 指令

开启`Go module`模式后，就可以使用`go mod [命令]`命令进行依赖管理，下面列出常用指令

| 命令       | 描述                                                   |
| ---------- | ------------------------------------------------------ |
| `download` | 下载依赖的`module`到本地`cache`                        |
| `edit`     | 编辑`go.mod`文件                                       |
| `graph`    | 打印模块依赖图                                         |
| `init`     | 在当前文件夹下初始化一个新的`module`, 创建`go.mod`文件 |
| `tidy`     | 增加丢失的`module`，去掉未使用的`module`               |
| `vendor`   | 将依赖复制到`vendor`下，也称为打包                     |
| `verify`   | 校验依赖                                               |
| `why`      | 解释为什么需要依赖                                     |

## 初始化

启用操作完成后，就可以正式使用`go module`进行依赖管理了，为了区别于`GOPATH`，请将当前项目从`GOPATH/src`目录中移出，放到任何一个工作目录中，本例就放在了`/usrs/lleej/work/go/hello`目录下

在当前项目的根目录中（注意：必须是项目根目录），初始化当前项目的`mod`管理

```bash
$ go mod init [模块名称]
```

如果项目是从`github`等远程代码托管平台下载（`clone`），则不需要提供模块名称，自动根据`git`库地址生成

```bash
$ go mod init #从github Clone的项目，不需要指定模块名称
$ go mod init myProject #myProject 就是模块的名称
```

初始化操作执行后，在当前项目的根目录中创建`go.mod`依赖包管理文件

注：一个项目只需要一个`go.mod`文件，子目录中不要进行该操作

### 依赖描述

`go.mod`文件是`Go`项目的依赖描述文件，该文件主要用来描述两件事

- 当前项目(`module`)的名称。每个项目都应该设置一个名称，当前项目中的包(`package`)使用该名称相互调用
- 依赖的第三方包名称。项目运行时会(`go build`)自动分析项目中的代码依赖，生成`go.sum`依赖分析结果，`go`编译器会去下载这些第三方包，然后再编译运行

```go
module github.com/lleej/helloworld //模块名称

go 1.14 //go 版本

require ( //依赖包及版本
    github.com/elastic/go-elasticsearch v0.0.0
    github.com/gorilla/mux v1.7.2
    github.com/gosoon/glog v0.0.0-20180521124921-a5fbfb162a81
)
```

 `go.sum`依赖分析文件，记录每个依赖库的版本和哈希值

```go
github.com/elastic/go-elasticsearch v0.0.0 h1:Pd5fqOuBxKxv83b0+xOAJDAkziWYwFinWnBO0y+TZaA=
github.com/elastic/go-elasticsearch v0.0.0/go.mod h1:TkBSJBuTyFdBnrNqoPc54FN0vKf5c04IdM4zuStJ7xg=
github.com/gorilla/mux v1.7.2 h1:zoNxOV7WjqXptQOVngLmcSQgXmgk4NMz1HibBchjl/I=
github.com/gorilla/mux v1.7.2/go.mod h1:1lud6UwP+6orDFRuTfBEV8e9/aOM/c4fVVCaMa2zaAs=
github.com/gosoon/glog v0.0.0-20180521124921-a5fbfb162a81 h1:JP0LU0ajeawW2xySrbhDqtSUfVWohZ505Q4LXo+hCmg=
github.com/gosoon/glog v0.0.0-20180521124921-a5fbfb162a81/go.mod h1:1e0N9vBl2wPF6qYa+JCRNIZnhxSkXkOJfD2iFw3eOfg=
```

`go`命令使用`go.sum`文件确保这些模块未来下载的版本与第一次下载相同，确保项目所依赖的模块不会出现意外更改，无论是出于恶意、意外还是其他原因。 

注：`go.mod`和`go.sum`都应检入版本控制。`go.sum` 不建议手工维护。

### 创建依赖

`go.mod`文件一旦创建后，它的内容将会被`go toolchain`全面掌控。`go toolchain`会在各类命令，如：`go get`、`go build`、`go mod`执行时，修改和维护`go.mod`文件。

#### 方式一

执行`go build`构建包时，自动根据包中`import`导入的依赖包创建依赖关系，并更新`go.mod`文件

```bash
$ go build
go: finding module for package github.com/gorilla/mux
go: found github.com/gorilla/mux in github.com/gorilla/mux v1.7.4
# hello
./main.go:5:2: imported and not used: "github.com/gorilla/mux"
```

#### 方式二

执行`go get ./...`指令，自动根据包中`import`导入的依赖包创建依赖关系，并更新`go.mod`文件

更新后的`go.mod`文件是这样的

```go
module hello

go 1.14

require github.com/gorilla/mux v1.7.4
```

注：使用`VS Code`进行开发时（添加插件），当添加`import`导入第三方包，会自动更新`go.mod`并生成`go.sum`文件

## 打包

将项目依赖的包下载到项目目录中的`vendor`子目录中，与项目源代码打包到一起

执行 `go mod vendor` 将项目所有的依赖下载到本地 `vendor` 目录中然后进行编译

## 下载

将项目的依赖包下载到`$GOPATH/pkg/mod`目录中，多项目可以共享缓存`mod`目录中的依赖包

```bash
$ go mod download
```