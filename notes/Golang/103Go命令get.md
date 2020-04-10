title: 103.Go命令之go get
date: 2020-04-09
tags: Go命令
categories: Go命令
layout: post

------

摘要：本节介绍`Go`命令中常用的`go get`下载包命令...

<!-- more -->

## 用途

> 类似于`Node`的`npm`，`Python`的`pip`

从互联网下载/更新指定的代码包及其依赖包，对它们进行编译，并安装到环境变量`GOPATH`中包含的第一个工作目录中

通俗的讲就是做以下三件事

- 从互联网指定位置（支持的版本控制系统）检出代码包并下载到本地
- 编译代码包(等同于`go build`)
- 安装代码包到`$GOPATH`包含的的第一个工作区中(等同于`go install`)

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

支持的版本控制系统：

| 名称       | 主命令 | 说明                                                         |
| ---------- | ------ | ------------------------------------------------------------ |
| Mercurial  | hg     | 轻量级分布式版本控制系统，采用Python语言实现，易于学习和使用，扩展性强 |
| Git        | git    | 广泛使用。进行有效、高速的各种规模项目的版本管理             |
| Subversion | svn    | 第一个将分支概念和功能纳入到版本控制模型的系统               |
| Bazaar     | bzr    | 开源的分布式版本控制系统。用它来作为VCS的项目并不多          |

预置的代码托管站点：

| 名称                        | 主域名          | 支持的VCS                  | 代码包远程导入路径示例                                       |
| --------------------------- | --------------- | -------------------------- | ------------------------------------------------------------ |
| Bitbucket                   | bitbucket.org   | Git, Mercurial             | bitbucket.org/user/project bitbucket.org/user/project/sub/directory |
| GitHub                      | github.com      | Git                        | github.com/user/project github.com/user/project/sub/directory |
| Google Code Project Hosting | code.google.com | Git, Mercurial, Subversion | code.google.com/p/project code.google.com/p/project/sub/directory code.google.com/p/project.subrepository code.google.com/p/project.subrepository/sub/directory |
| Launchpad                   | launchpad.net   | Bazaar                     | launchpad.net/project launchpad.net/project/series launchpad.net/project/series/sub/directory launchpad.net/~~user/project/branch launchpad.net/~~user/project/branch/sub/directory |
| IBM DevOps Services         | hub.jazz.net    | Git                        | hub.jazz.net/git/user/project hub.jazz.net/git/user/project/sub/directory |

执行步骤：

- 首先，检查本地是否已经存在该代码包（根据代码包远程导入路径判断）。若代码包已存在，不会重复下载（除非使用命令参数`-u`强制下载）
- 分析代码包远程导入路径对应的版本控制系统和远程仓库的URL
  - 静态分析，与预置的代码托管站点的主域名进行比对
  - 动态分析，未知
- 根据分析结果（版本控制系统和远程仓库URL）检出代码包，并在检出目录中增加元数据目录（格式：`.{主命令}`，如:`.git`），以此来简化代码包更新时的分析过程

#### 自定义代码包远程导入路径

如果你想把你编写的（被托管在不同的代码托管网站上的）代码包的远程导入路径统一起来，或者不希望让你的代码包中夹杂某个代码托管网站的域名，那么你可以选择自定义你的代码包远程导入路径。

也就是说，在本地库中的代码包的导入路径是以自定义路径开头，而不是各个托管网站路径开头

实现方式：导入注释。具体写法如下

```go
package 包名 // import "自定义的导入路径"
```

举个例子：

代码包`logging`托管在`github`上，属于项目`collsrv`，项目网址是`https://github.com/lleej/collsrv`。标准检出的命令是

```bash
$ go get github.com/lleej/collsrv/logging
```

如果要使用自定义导入路径，需要做以下工作

- 修改代码包`logging`，添加导入注释

  ```go
  package logging // import "go2sts.com/collsrv/logging"
  ```

  - 使用注释`//`声明自定义的导入路径
  - 注释必须是源文件的第一行

  通常情况，替换的是导入路径的主域名和用户名ID部分

  ```go
  github.com/lleej/collsrv/logging // 标准的导入路径
  go2sts.com      /collsrv/logging // 导入注释中的导入路径 
  ```

- 在自定义路径中主域名的`Web`服务器上，部署可以处理`HTTP`请求的程序

  - 可以处理`go2sts.com/collsrv`这个请求路径，也就是代码库中项目的访问地址

  - 在收到该请求路径后，响应的`HTML`文件头中加入以下格式的内容

    ```html
    <meta name="go-import" content="import-prefix vcs repo-root">
    ```

    `import-prefix`：自定义的远程代码包导入路径的前缀，与处理程序关联的路径一致

    `vcs`：对应的版本控制系统的命令

    `repo-root`：托管代码库中项目的地址

    以本例来说，需要添加的响应头是这样的

    ```html
    <meta name="go-import" content="go2stc.cn/collsrv git https://github.com/lleej/collsrv">
    ```

  当使用自定义导入路径时，下载到本地的代码库的存储路径是自定义导入路径

### 安装位置

安装到环境变量`GOPATH`中包含的第一个工作目录中

为了将第三方代码包与个人项目代码包区分，建议在`GOPATH`中定义两个工作目录，第一个工作目录保存从远端安装的第三方代码包

## 输出

### 源代码

使用`go get`检出的代码库的源文件，存储在`GOPATH`中定义的第一个工作的`src`子目录下

### 归档文件

使用`go get`检出的代码库编译安装后的退档文件`.a`，存储在`GOPATH`中定义的第一个工作的`pkg`子目录下

## 参数

`go build`和`go install`中的参数都可以在`go get`中使用，这里就不再赘述了

| 标记名称  | 标记描述                                                     |
| --------- | ------------------------------------------------------------ |
| -d        | 让命令程序只执行下载动作，而不执行安装动作。                 |
| -f        | 仅在使用`-u`标记时才有效。该标记会让命令程序忽略掉对已下载代码包的导入路径的检查。<br />如果下载并安装的代码包所属的项目是你从别人那里`Fork`过来的，那么这样做就尤为重要了。 |
| -fix      | 让命令程序在下载代码包后先执行修正动作，而后再进行编译和安装。 |
| -insecure | 允许命令程序使用非安全的scheme（如HTTP）去下载指定的代码包。<br />如果你用的代码仓库（如公司内部的Gitlab）没有HTTPS支持，可以添加此标记。<br />请在确定安全的情况下使用它。 |
| -t        | 让命令程序同时下载并安装指定的代码包中的测试源码文件中依赖的代码包。 |
| -u        | 让命令利用网络来更新已有代码包及其依赖包。<br />默认情况下，该命令只会从网络上下载本地不存在的代码包，而不会更新已有的代码包。 |


### `-d`只下载不安装

只是将远程代码包下载到工作目录中的`src`子目录中，并不对代码包进行编译和安装操作，也就是说`pkg`子目录下没有更新

### `-fix`先修正再编译安装

主要是处理向后兼容的问题。如果检出代码包是使用老版本的`go`进行开发的，可能与当前的`go`版本在语法上存在不兼容的情况。

加入`-fix`参数，当检出代码包后，先对代码包中不符合`Go`当前版本语言规范的语法进行修正，再下载依赖包，最后进行编译和安装

### `-insecure`非安全通道

如果托管的代码库不支持`HTTPS`安全套接字连接，需要加入该参数，这样就可以使用`HTTP`连接下载


