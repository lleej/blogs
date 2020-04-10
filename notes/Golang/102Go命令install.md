title: 102.Go命令之go install
date: 2020-04-09
tags: Go命令
categories: Go命令
layout: post

------

摘要：本节介绍`Go`命令中常用的`go install`安装命令...

<!-- more -->

## 用途

用于编译并安装指定的代码包及它们的依赖包。只比`go build`命令多做了一件事，即：安装编译后的结果文件到指定目录，也就是将静态链接库`.a`文件输出到工作区的`pkg`目录中（`go build`保存在临时目录中）

命令参数只接收**导入路径**形式的代码包

```bash
$ go install hello/other
```

## 输出

### 库代码包

当命令参数是库代码包时，编译库代码包，并将编译后的静态链接文件（或称为归档文件）保存到工作目录中的`pkg`子目录中。

- 创建平台相关目录，目录名为`${GOOS}_${GOARCH}`
- 创建归档文件`.a`，并将文件保存到平台相关目录中
- 若库代码包有层次结构，则在平台相关目录中创建子目录，如：`go install x/y/z`则创建x/y目录，归档文件为`z.a`

```bash
$ go install hello/other

# 输出后的目录结构
├── pkg
│   ├── mod
│   ├── submod
│   ├── windows_amd64 # 平台相关目录
│   │   ├── hello
│   │   |   ├── other.a # 安装的other包

```

### 命令代码包

我们知道命令源文件是包含`main()`入口函数的文件，编译并链接输出可执行文件

与`go build`命令的执行效果是相同的，唯一的不同之处在于：

- `go build`将可执行文件保存在执行命令的目录中
- `go install`将可执行文件保存在`$GOBIN`所在的目录下(或保存在`$GOPATH`中的`bin`子目录下)

注意：当`$GOPATH`设置多个目录时，必须设置`$GOBIN`环境变量，这样`go install`才能将可执行文件保存到正确的位置，否则无法知道将可执行文件输出到哪里，会提示`GOBIN not set`错误