title: 105.Go命令之go doc
date: 2020-04-10
tags: Go命令
categories: Go命令
layout: post

------

摘要：本节介绍`Go`命令中的`go doc`打印文档命令...

<!-- more -->

## 注释规范

在介绍`go doc`命令前，先简单介绍`go`代码中的注释规范

https://blog.csdn.net/mostone/article/details/97766858

## 作用

打印附于`Go`程序实体上的文档。我们可以通过把程序实体的标识符作为该命令的参数来达到查看其文档的目的。

通俗的讲，就是使用命令将`包/实体/标识符`中的文档输出到命令行工具上，方便使用者快速了解其用法

默认情况下，只能输出`可导出`实体的文档（即首字母大写的），也可以通过添加`-u`命令行参数输出全部实体的文档



下面以`Go`内置的`fmt`包举例说明。注：`Go`版本为`1.14.1`
```bash
$ go doc fmt
package fmt // import "fmt"
Package fmt implements formatted I/O with functions analogous to C's printf
and scanf. The format 'verbs' are derived from C's but are simpler.
... ... # 省略号省略了其中大段的注释内容
type State interface{ ... }
type Stringer interface{
```

### 包文档

输出的包文档包括两部分内容

- 代码中的注释
- 可导出的变量/常量/接口/结构/函数/方法



- 省略号前面的，是在`doc.go`中的大段注释
- 省略号后面的，是多个文档中可导出的接口/函数

建议：如果包中有多个源文件，建议增加`doc.go`源文件对包进行统一说明

问题：为什么文件中的第一段注释：`copyright`部分没有添加进来呢？是第一段都不会添加还是`copyright`是关键字

### 实体/标识符文档

## 参数

### `-u`输出全部文档

