title: 3.Git-远程分支
date: 2019-02-20
tags: GIT
categories: 配置管理
layout: draft

------

摘要：文本是使用Git进行代码管理的第三篇文章，重点介绍：如何使用远程仓库、远程分支进行代码的共享与集成。

<!-- more -->

## 回顾

上文重点介绍了Git分支，包括提交、提交链、分支的概念和作用，如何创建分支、切换分支、合并分支以及删除分支。并结合`VS Code`开发工具，介绍在本地Git仓库使用分支的操作方法和注意事项。

**提交**。是Git进行版本控制的基本单位，可以理解为每个提交是一个版本的快照。

**提交链**。提交通过`parent`属性建立与父提交的联系组成一条提交链。通过这种关联可以反向遍历提交历史。

**分支**。可以理解为一个动态的指针，其总是指向某条提交链中最后的提交。使用分支可以方便的切换工作场景（如：功能开发、代码重构、Bug修复…）而不用过多的考虑不同工作场景下代码的版本。

## 远程仓库

在开始介绍**远程分支**之前，先聊聊**远程仓库**。远程仓库是指托管在因特网或其他网络中的版本库。作为一款分布式版本控制系统，理论上可以有好几个远程仓库，通常有些仓库对你只读，有些则可以读写。

### 用途

远程仓库主要的用途就是为了共享，包括：与他人协作以及本地仓库的备份等。

分布式版本管理系统是去中心化的，也就是说不依赖于一个所谓的中心仓库就可以开展工作。原则上，每个分布式节点通过配置`https`或`ssh`并设置用户的访问权限，就可以与他人进行仓库的共享。

不过，为了方便协作和数据备份，一般情况下我们会部署一个专门供大家共享的仓库。这个仓库与我们使用的本地仓库有一些不同。

### 裸仓库

作为共享用途的仓库，通过分支的推送和拉取实现与本地仓库的数据交换，原则上不需要直接对远程仓库进行代码的编辑和提交，因此远程仓库不需要创建工作区，我们称这样的仓库为**裸仓库**。

```bash
# 我们可以在本地创建一个裸仓库
# 裸仓库没有工作区，因此仓库名称采用[项目名.git]的形式
# 我们不能对裸仓库进行编辑和提交操作，如git add、git commit等
$ git init test.git --bare
```

### 常用远程仓库

作为当今最受欢迎的分布式版本控制系统，Git 的生态非常完善。不需要我们自行安装和配置相关的工具、协议、权限信息来实现远程仓库的部署，基于 Git 的可视化托管平台可以简化这个过程。

**Github**。是目前最大的开源代码托管平台，有1亿用户和超过1亿个代码库，所有知名的开源项目都托管在这个平台上。2018年微软收购 Github 后，允许免费用户创建私有化仓库。

**Gitlab**。仅次于 Github 的代码托管平台，提供的社区版本 Gitab CE 允许私有化部署，也是很多软件公司使用的内部代码管理平台，提供容器方式部署。 

**国内托管平台**。如 Coding 和 OSChina等。优势是相对于 Github 访问速度更快。 

### 准备工作

Git 远程仓库支持`https`和`ssh`加密协议进行数据的交换。而`ssh`协议通过公钥进行身份认证，这样就不用每次访问都输入用户名和密码，因此推荐采用这种方式。下面以 Github 为例介绍如何配置`ssh`，其他托管平台查看相关的配置说明。

**注册用户**。具体的操作步骤就不再赘述了，需要注意的是：建议使用邮箱作为用户名进行注册。

**安装Git**。具体的操作步骤见第一篇文章。安装 Git 后，使用`Git config`配置用户名和邮箱地址。**注意：用户名与注册用户一致**。

**生成ssh key**。在本机的命令行输入如下命令创建`ssh key`

```bash
# 按三次回车键即可
$ ssh-keygen -t rsa -C "用户名"
# 示例
$ ssh-keygen -t rsa -C "lleej@qq.com"
```

Windows 系统，在`C:\Users\[登录用户名]\.ssh`目录中创建两个文件：私钥文件`id_rsa`和公钥文件`id_rsa.pub`

**设置ssh**。打开`github`的`ssh`设置页面，将公钥文件`id_rsa.pub`的内容复制并创建一个新的`ssh key`。

![设置github的ssh](./assets/remote-github-ssh.png)

**测试连接**。设置完成后，通过以下指令测试是否设置成功。

```bash
# 测试是否成功
$ ssh git@github.com
# 出现以下提示，表示设置成功
Hi lleej! You ve successfully authenticated, but GitHub does not provide shell access.
```

如果我们需要通过在本机访问多个远程仓库，例如：个人的`github`、公司的`gitlab`...，需要进行如下操作。

**为每个远程仓库创建SSH key**。最好为不同的远程仓库创建不同的`ssh key`，即使用户名一样。由于默认情况下创建的密钥文件名称为`id_rsa`，因此需要在创建多个平台的`ssh key`时指定密钥文件名。

```bash
# 指定密钥文件
$ ssh-keygen -t rsa -C "用户名" -f ~/.ssh/[平台简称]_rsa
# 示例: 为 github 创建 ssh key
$ ssh-keygen -t rsa -C "lleej@qq.com" -f ~/.ssh/github_rsa
```

注意：使用`-f`参数指定文件名时，需要指定文件放置的目录，可以写成`C:\Users\[登录用户名]\.ssh`，也可以用示例中的`~/.ssh/`表示。

**添加配置文件**。在`C:\Users\[登录用户名]\.ssh`目录中添加`config`文件，在配置文件中可以设置访问不同远程仓库时使用不同的公钥文件进行认证。

```bash
# Gitlab Private Server
Host localhost
  Hostname localhost
  IdentityFile ~/.ssh/gitlab_rsa
  User xxx@boco.com.cn

# Github Server
Host github.com
  Hostname github.com
  IdentityFile ~/.ssh/github_rsa
  User xxx@qq.com

# Coding Server
Host git.dev.tencent.com
  Hostname git.dev.tencent.com
  IdentityFile ~/.ssh/coding_rsa
  User xxx@qq.com
```

**Host**。远程仓库的名称，通常情况下这个值与`Hostname`一致

**Hostname**。远程仓库的主机名称，通常情况下是仓库的域名或IP地址

**IdentityFile**。使用的公钥文件的名称（不包括扩展名）

**User**。登录远程仓库的用户名

## 管理远程仓库

> VS Code中只集成了部分远程仓库的管理功能（无添加和删除）
>
> 本章节重点介绍远程仓库的管理，涉及到远程分支的部分在后面章节说明

为了能够操作远程仓库，需要在本机 Git 中添加一个具名的指向远程仓库URL的引用，称为**远程仓库**。Git 中远程仓库的默认名称是`origin`（类似默认分支名称为`master`），在本地 Git 中远程仓库不能重名，但可以设置指向同一个远程仓库URL的多个远程仓库。

管理远程仓库包括：添加远程仓库、移除无效的远程仓库、管理不同的远程分支并定义它们是否被跟踪等等。

### 添加远程仓库

只是添加一个指向远程仓库URL的链接，远程仓库的创建需要在远程 Git 端操作。

**克隆仓库**

如果我们使用`git clone`从远程克隆仓库到本地，Git 会自动添加远程仓库，命名为`origin`。

```bash
# 通过克隆方式添加远程仓库
$ git clone [远程仓库名称]
# 示例
$ git clone git@github.com:lleej/test.git
```

注意：使用`ssh`协议时远程仓库的命名格式，`git@[版本库域名]:[用户名]/[版本库名]`

**手动添加**

如果需要将本地仓库与多个远程仓库建立联系，比如：公司的集成仓库和备份仓库。就需要通过手动方式添加远程仓库

```bash
# 手动添加远程仓库
$ git remote add [远程仓库名称] [远程仓库URL]
# 示例
$ git remote add origin git@github.com:lleej/test.git
```

添加远程仓库后，就可以在本地对远程仓库进行推送和抓取版本信息的操作。

### 删除远程仓库

只是删除指向远程仓库URL的链接，远程仓库的删除需要在 Git 服务端操作。

```bash
# 删除远程仓库
$ git remote rm [远程仓库名称]
# 示例
$ git remote rm origin
```

删除远程仓库并不会影响本地的已有版本也不会影响远程仓库的内容。

### 重命名远程仓库

只是对本地访问远程仓库的名称进行修改，并不会影响本地已有版本。

```bash
# 重命名远程仓库
$ git remote rename [OLD] [NEW]
# 示例
$ git remote rename origin ori
```

重命名后，需要使用新的远程仓库名称进行操作。

### 查看远程仓库

查看在本地 Git 仓库中连接了哪些远程仓库，以及这些仓库的 URL 地址和分支的跟踪情况。

**简要信息**

```bash
# 查看远程仓库名称
$ git remote
# 查看远程仓库名称和URL
$ git remote -v
```

**详细信息**

```bash
# 查看远程仓库详细信息
$ git remote show [远程仓库名称]
# 输出内容如下
* remote origin
  Fetch URL: git@github.com:lleej/test.git
  Push  URL: git@github.com:lleej/test.git
  HEAD branch: master
  Remote branch:
    from-remote new (next fetch will store in remotes/origin)
    master tracked
  Local branch configured for 'git pull':
    master merges with remote master
  Local ref configured for 'git push':
    master pushes to master (up to date)
```

**Fetch URL**。从远程仓库拉取版本信息的URL地址

**Push URL**。向远程仓库推送版本信息的URL地址

**HEAD branch**。本地当前工作分支，本例为`master`

**Remote branch**。远程仓库中的所有分支。标注为`tracked`的分支表示已经拉取到本地，标注为`new`的分支是没有拉取到本地的分支。

**Local branch configured for "git pull"**。为`git pull`操作配置的本地分支。如果存在则表示当执行拉取`git pull`操作时，能够从远程仓库拉取这些分支并与本地分支进行合并。不在此列的分支不能从远端拉取到本地，需要使用`git fetch`操作。

**Local branch configured for "git push"**。为`git push`操作配置的本地分支。如果存在则表示当执行推送`git push`操作时，能够将这些分支推动到远程仓库对应的分支中。如果推送的分支不在此列，则需要手动指定推动的分支名称。

**注：以上三部分的操作只能通过命令行方式或者使用 Git 客户端工具实现，`VS Code`中并没有提供相应的功能。**

### 从远程仓库拉取

在与他人协作时，远程仓库中的版本会被多人更新，导致个人本地的版本滞后于远程仓库（有时也会超前）。当我们需要使用最新版本时，就需要从远程仓库拉取到本地。

**拉取 Fetch**

使用`git fetch`指令，可以从远程仓库拉取**所有分支**的最新版本到本地 Git 仓库。拉取的分支并不会与本地分支进行合并，因此不会影响当前的工作。

```bash
# 拉取远程仓库 所有分支
$ git fetch [远程仓库名称]
# 示例
$ git fetch origin

# 拉取远程仓库 指定分支
$ git fetch [远程仓库名称] [远程分支名称]
# 示例
$ git fetch origin master
```

**抓取 Pull**

当本地的分支设置为跟踪远程仓库的分支（见 查看远程仓库章节）使用`git pull`指令，可以从远程仓库拉取分支到本地 Git 仓库，并与本地仓库中建立跟踪关系的分支进行合并。

注意：只能抓取设置为跟踪远程仓库的分支。否则只能使用`git fetch`进行拉取操作，并手动进行分支的合并。

```bash
# 抓取远程仓库
# 当前工作分支设置为跟踪远程仓库的分支
$ git pull [远程仓库名称]
# 示例
$ git pull origin

# 建立远程仓库分支与本地分支的跟踪关系
$ git branch --set-upstream-to=<remote>/<branch> <branch>
```

在`VS Code`中集成了远程仓库的抓取功能，使用的就是`git pull`指令，会自动与本地分支进行合并。

### 向远程仓库推送

在多人协作的场景中，当工作完成后需要将其推送到远程仓库，便于他人拉取到本地使用。

```bash
# 推送到远程仓库 
# 针对 为git push操作配置的分支 不用设置具体分支名称
$ git push [远程仓库名称]
# 示例
$ git push origin

# 指定本地分支
$ git push [远程仓库名称] [分支名称]
# 示例
$ git push origin master
```

### 小结

**远程仓库**。指向远端 Git 仓库的一个引用，通过这个引用可以拉取或推送版本信息

**远程仓库名称唯一**。为远程仓库设置的名称必须在本地仓库中唯一。

**任意推动本地分支**。本地仓库创建的分支均可以使用`git push`指令推动到远程仓库中。如果存在为`git push`设置的分支清单，则不用指定具体分支名称，将一次将清单中的分支全部推送到远端；否则，需要手动指定推动的分支，并且进行过推动操作的分支将自动添加到清单中。

**任意拉取远程分支**。使用`git fetch`操作可以一次拉取远程仓库中所有分支的更新，也可以拉取指定的远程分支，但拉取到本地后不会与本地分支自动进行合并操作。

**抓取跟踪分支**。使用`git pull`操作可以将远程仓库的分支拉取到本地并自动与本地分支进行合并，不过不是所有的分支都能奏效，必须是为`git pull`设置的分支才可以。

## 远程分支



## 远程分支操作



```bash
Note: checking out 'origin/master'.

You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by performing another checkout.

If you want to create a new branch to retain commits you create, you may
do so (now or later) by using -b with the checkout command again. Example:

  git checkout -b <new-branch-name>

HEAD is now at 25fb6af 合并fea_login分支，解决了readme.md文件的冲突
```



