title: 2.Git-分支
date: 2019-01-25
tags: GIT
categories: 配置管理
layout: draft

------

摘要：文本是使用Git进行代码管理的第二篇文章，重点介绍：Git“必杀技”分支的原理、用途、用法以及指令，通过一个示例演示如何利用分支进行多版本管理。

<!-- more -->

回顾：上篇介绍了Git的一些基本概念和日常用法，包括Git的特点、工作区域和文件状态的概念；如何安装Git环境、配置用户信息；如何在本地创建版本库，使用`VS Code`操作版本库等。

你也许会有这种感觉，这跟之前使用的版本管理系统没有什么区别呀？而且，这个也解决不了快速迭代和并行开发的问题呀？那是因为，我们还没有使用Git“必杀技”分支。本文就讲讲分支的妙用。

## 前言

当创建一个版本库时，GIT默认创建一个名为`master`的分支，使用`git commit`命令提交到版本库中的版本信息作用在这个分支上。为什么会有默认`master`分支呢？这是因为GIT的版本控制依赖于分支。

以一张图可以清晰的了解分支以及分支和提交(版本)的关系（github中仓库的Insights/Network页面）

![github-network](./assets/github-network.png)

上图三条颜色的线代表三个分支，线上的圆点代表提交(版本)

- `master`默认分支。在该分支上一共产生了4次提交

- `beijing`分支。在`master`分支第3次提交后创建，在该分支产生了3次提交

- `shanghai`分支。在`master`分支第3次提交后创建，在该分支也产生了3次提交

从上图可以看出

**分支相互独立**。基于某一次提交(版本)可以创建多个分支，之后在不同分支上的提交不受其他分支的影响。

**分支是具名的**。创建的分支名称是语义化的，默认`master`分支代表主分支，`beijing`和`shanghai`两个分支可以理解为在北京和上海的两个团队。当然，分支也可以按任务创建，如：`feature_login`、`fix_231`等，可根据团队的开发流程和习惯自行定义。

**提交依赖分支**。每次提交操作都发生在某个分支上（**没有分支的提交是危险的，会被GIT垃圾回收**），因此在提交前需要确认当前的工作分支。

以上是对分支概念的简单介绍，在正式开始之前还需要对上文提到的“提交”和“分支”以及之间关系做进一步的解释，看看GIT是如何实现的，这将有助于对分支的深入理解。

## 理解分支和提交

为了避免理解上的分歧，我们统称GIT中“版本”为“提交”英文名“commit”。也就是说，每次`git commit`都会在GIT版本库中创建一个新的版本，我们称这个“版本”为“提交”，这样就与`Vx.xx`版本区别开。

### 提交

在GIT中，版本的差异是通过“提交”进行管理的，每次通过`git commit`都会创建一个新的“提交对象”，以文件形式存储在GIT版本库中，记录版本间的差异。

下面以一个网页项目为例，详细介绍“提交对象”。

**工作区内容**

```bash
# test仓库中的文件结构
# tree命令查看结果
├── images
│   └── logo.png
├── index.html
├── readme
└── styles
    └── main.css
```

将工作区内的代码提交到版本库后，在GIT版本库中存储的对象如下图所示

![blob-tree-commit](./assets/blob-tree-commit.png)

在深入研究Git前，首先需要明白Git的几个核心概念，这样便于后面的学习和理解

下图是其中最主要的三个对象类型`commit` `tree` `blob`之间的关系。

![image-20181213145842415](D:/blogs/notes/git/assets/blob-tree-commit.png)

## commit

`commit`是存储在`git`仓库中的快照，每次执行`git commit`指令后，在`git`仓库中都会创建一个新的`commit`对象

### 是什么？

1. `git`内置的对象类型
2. 表现为`SHA-1`哈希值（长度40）
3. 一次提交的快照
4. 每次提交操作将产生一个新的`commit id`

### 在哪里？

1. 存储在`.git/objects/`目录中
2. 名称为`./前2位/后38位`

### 有什么？

1. `tree`：树对象名称
   - 一个`commit`对象中只有一个`tree`对象
   - 能够找到该`commit`包含的所有文件快照
2. `parent`：在哪个`commit`对象基础上创建的
   - 形成`commit`的遍历关系
   - 第一个提交`parent`为空
   - 从多个分支合并时，有多个`parent`
3. `author`：该`commit`的作者
   - 该`commit`的创建人
   - 多人协作时，`author`可能不是自己
4. `committer`：该`commit`的提交人
   - 谁提交了
5. 提交注释

看一个`commit`的具体案例

```shell
# 查看 Git 对象的类型
$ git cat-file -t 1f414886a
commit

# 查看 Git 对象的内容
$ git cat-file -p 1f414886a
tree 124098142d34d43b4df63a4c3655f0112f7c0682 # 该 commit 中包含的目录树
parent 156aa60400c19d00a4d8bcef1ad0ed5a846e9849	# 从该 commit 创建的，因此parent 为该 commit
author lijie <lijie@boco.com.cn> 1544613332 +0800 # 作者
committer lijie <lijie@boco.com.cn> 1544613332 +0800 # 提交人

Add readme.md # 提交注释
```



## tree

在`git`中数据文件的存储没有采用类似操作系统的树状结构，而是用`tree`对象代替

### 是什么？

1. `git`内置对象类型
2. 模拟目录存储结构
3. 表现为`SHA-1`哈希值（长度40）

### 在哪里？

1. 存储在`.git/objects/`目录中
2. 名称为`./前2位/后38位`

### 有什么？

1. `tree`对象
   - 子树（目录）
2. `blob`对象
   - 数据文件（各种类型）
3. 每个对象一行描述（各属性用空格分隔）
   - 对象类型
   - 哈希文件名（ 仓库中存储的文件 ）
   - 真实文件/目录名（ 对应的工作目录中的名称 ）

```shell
# 看看 commit 中 tree 配置项对应对象的类型
$ git cat-file -t 124098142d34d43b4
tree

# tree 具体的内容
$ git cat-file -p 124098142d34d43b4
100644 blob 72943a16fb2c8f38f9dde202b7a70ccc19c52f34	aaa.txt
100644 blob e6076a05b53658ebd812398523da7f38fc552aa1	readme.md
100644 blob 5d308e1d060b0c387d452cf4747f89ecb9935851	test

# 使用 git ls-tree 直接查看分支/HEAD/commit中 tree 的内容
$ git ls-tree -r 1f414886a # -r 表示递归显示子树
100644 blob 72943a16fb2c8f38f9dde202b7a70ccc19c52f34	aaa.txt
100644 blob e6076a05b53658ebd812398523da7f38fc552aa1	readme.md
100644 blob 5d308e1d060b0c387d452cf4747f89ecb9935851	test
```



## blob

在`git`中多有的文件统一由`blob`对象表示

### 是什么？

1. `git`内置对象类型
2. 文件的存储实体
3. 表现为`SHA-1`哈希值（长度40）

### 在哪里？

1. 存储在`.git/objects/`目录中
2. 名称为`./前2位/后38位`

### 有什么？

1. 数据文件内容
2. 压缩存储

```shell
# 看看 blob 对象的类型
$ git cat-file -t e6076a0
blob

# 看看 blob 对象的内容
$ git cat-file -p e6076a0
你好

# 使用 git ls-files 列出 暂存区 或者 本地数据目录中的文件
# 常用参数 -s --with-tree=
$ git ls-files -s # 列出暂存区中文件
100644 blob 72943a16fb2c8f38f9dde202b7a70ccc19c52f34	aaa.txt
100644 blob e6076a05b53658ebd812398523da7f38fc552aa1	readme.md
100644 blob 5d308e1d060b0c387d452cf4747f89ecb9935851	test

$ git ls-files --with-tree=HEAD^ # 列出上一次提交的文件
100644 blob 72943a16fb2c8f38f9dde202b7a70ccc19c52f34	aaa.txt
100644 blob e6076a05b53658ebd812398523da7f38fc552aa1	readme.md
```





**提交对象**。保存一个“版本”的全部文件信息的**快照**，文件中保存如下信息：文件根Tree、`parent`提交对象、提交操作的作者、提交人、提交消息

**Tree对象**。可以理解为目录树、在GIT中目录用`tree对象`存储

**Blob对象**。

![commits-and-parents](./assets/commits-and-parents.png)

