title: 104.Go命令之go clean
date: 2020-04-10
tags: Go命令
categories: Go命令
layout: post

------

摘要：本节介绍`Go`命令中的`go clean`清理命令...

<!-- more -->

## 用途

删除掉执行其它命令时产生的一些文件和目录

- 使用`go build`创建的可执行文件

  代码包所在目录中的可执行文件（其他目录不会清除）

- 使用`go build`创建的临时文件（`-work`参数）

  临时目录以`go-build`为前缀

- 使用`go install`安装的文件（包括归档文件和可执行文件）

  归档文件在`pkg`子目录下，可执行文件在`bin`子目录下

- 使用`go test`创建的测试文件

  代码包所在目录中的测试文件（其他目录不会清除）

概括的讲，就是清除使用`go`命令创建的文件，非项目源文件

## 命令标记

### `-x`查看执行命令

```bash
$ go clean -x hello
cd D:\Users\lijie\go\src\hello # 只清除代码包所在目录
rm -f hello hello.exe hello.test hello.test.exe hello hello.exe # 可执行文件、测试文件
```

### `-i`清除`go install`安装的文件

```bash
$ go clean -x -i hello
cd D:\Users\lijie\go\src\hello
rm -f hello hello.exe hello.test hello.test.exe hello hello.exe
rm -f D:\Users\lijie\go\bin\hello.exe # 命令源文件 安装到 bin 子目录下
```
