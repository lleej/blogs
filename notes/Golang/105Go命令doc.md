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

**基本规范**

可以给类型（`Type`）、变量（`Variable`）、常量（`Constans`）、函数（`Function`）甚至包（`Package`）添加注释，在其声明之前直接写一个常规注释即可。**注意：声明和注释中间不要插入空白行**

为了规范，注释以其描述的元素名称开头：

- 变量、常量、函数、类型

  ```go
  //Calc 计算年龄的函数
  func Calc(year: int) int {
    ...
  }
  ```

- 包注释比较特殊，以`Package [包名]`开头。注意第一个字母要大写

  ```go
  //Package other 其它辅助函数
  package other
  ...
  ```

**特殊注释**

- 标题注释

  可以在注释中书写段落标题，以区分不同段落。标题注释必须满足以下三个条件

  - 首字母大写开头
  - 结尾没有标点符号
  - 该段注释前不紧邻标题注释

  示例：标题注释（`Basic`）

  ```go
  
  Basics //标题注释
  
  A stream of gobs is self-describing. Each data item in the stream is preceded by
  ...
  ```

  示例：非标题注释

  ```go
  basic //非标题注释
  
  Basic. //非标题注释
  
  Basic One //标题注释
  
  Basic Two //非标题注释
  ```

- 段落注释

  以空行分隔的注释自成一个段落

  ```go
  /*
  这是第一段的第一行
  这是第一段的第二行
  
  这是第二段的第一行
  ...
  */
  ```

- 代码注释

  预格式化的文本必须相对于周围的注释文本缩进

  ```go
  can be sent from or received into any of these Go types:
  
  		struct { A, B int }	// the same
  		*struct { A, B int }	// extra indirection of the struct
  ```

- 错误注释

  使用`BUG(who) xxx`，标记是已知的错误，在包文档的“Bugs”部分单独显示

  ```go
  //BUG(lleej): 计算日期没有考虑闰年
  func Calc(year: int) int {
    ...
  }
  ```

- 弃用注释

  有时，结构、函数、类型甚至整个包都变得多余或不必要，但必须保留这些结构以与现有程序兼容，则在注释中添加` Deprecated: xxx`。启用注释不会体现在文档中的独立部分，但是在编辑工具中会提示使用者，该标识符被启用

  ```go
  //Deprecated: 1.2.0
  func Calc(year: int) int {
    ...
  }
  ```

**注释文件**

如果包注释超过3行（单个源文件或者包中多个文件都添加了包注释的情况下），建议统一放到`doc.go`注释文件中，该文件仅包含那些注释和`package`子句

```go
/*
Package xxx ...
...
...
*/
package xxx
```

**示例文件**

便于其他用户能够快速的使用你开发的包，建议添加有包使用`demo`的示例文件，文件名格式：`example_名称_test.go`

`go`源代码`GOROOT/src/encoding/gob/`目录中的示例文件`example_encdec_test.go`

```go
package gob_test
...
```

## 用法

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

## 工具godoc

相比于`go doc`在命令行中输出包/实体的注释文档，`godoc`工具输出在网页中查看的`HTML`格式的体验更好。但是，默认安装`golang`并没有安装这个工具（注：版本`1.14.1`）

### 安装步骤

需要手动下载`godoc`命令包，由于`golang.org`在国内无法正常访问，也就无法通过`go get`指令下载，只能通过手动下载、安装的方式

**下载包**

从`github`上下载`tools`项目

```bash
$ git clone git@github.com:golang/tools.git
```

在`GOPATH/src`目录下，手动创建`golang.org/x`目录

将下载的项目目录拷贝到该目录中

```bash
├── src
│   ├── golang.org
│   │   ├── x
│   │   |   ├── tools #拷贝到该目录下
```

由于`tools`安装时需要依赖包`net`和`xerrors`，所以这些包也需要手动下载，并拷贝到`golang.org/x`目录下

```bash
$ git clone git@github.com:golang/xerrors.git
$ git clone git@github.com:golang/net.git
```

最终的目录结构如下

```bash
├── src
│   ├── golang.org
│   │   ├── x
│   │   |   ├── tools 	#godoc包项目目录
│   │   |   ├── net 		
│   │   |   ├── xerrors 
```

执行安装指令

```bash
$ go install golang.org/x/tools/cmd/godoc
```

将`godoc`可执行文件安装到`GOPATH/bin`目录下

### 如何使用

在命令行工具中启动`godoc`服务器

```bash
$ godoc -http=:6060
```

在浏览器中访问：`http://localhost:6060`即可查看`golang`文档

查看第三方/项目包文档，在`URL`中加入导入路径即可，如：`http://localhost:6060/pkg/hello/other`







