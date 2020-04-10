title: 103.Go命令之go get
date: 2020-04-09
tags: Go命令
categories: Go命令
layout: post

------

摘要：本节介绍`Go`命令中常用的`go get`下载包命令...

<!-- more -->

## 用途

从互联网下载/更新指定的代码包及其依赖包，对它们进行编译，并安装到环境变量`GOPATH`中包含的第一个工作目录中

通俗的讲就是做以下三件事

- 从互联网指定位置（支持的版本控制系统）检出代码包并下载到本地
- 编译代码包
- 安装代码包到`$GOPATH`包含的的第一个工作区中

### 命令格式

```bash
$ go get 代码包导入路径
```

举个例子，从`github.com`版本控制系统获取代码包`github.com/hyper-carrot/go_lib/logging`

```bash
$ go get github.com/hyper-carrot/go_lib/logging
```

在`.go`文件中导入该代码包时，使用如下导入路径

```go
import "github.com/hyper-carrot/go_lib/logging"
```

### 代码包源

`go get`只能从特定的代码包源获取代码包，包括：支持的版本控制系统，以及根据接口规范开发的自定义`Web`系统

#### 支持的版本控制系统



#### 自定义`Web`系统

### 安装位置

https://hyper0x.github.io/go_command_tutorial/#/0.3





## 输出

## 参数

