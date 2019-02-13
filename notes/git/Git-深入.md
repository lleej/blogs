title: 3.Git-深入
date: 2019-02-13
tags: GIT
categories: 配置管理
layout: draft

------

摘要：文本对Git的实现原理进行深入的剖析，重点介绍：Git如何使用blob、tree、commit进行文件存储和版本管理，如何使用branch、tag、HEAD进行高效的分支管理。

<!-- more -->



## BLOB-TREE-COMMIT

在深入研究Git前，首先需要明白Git的几个核心概念，这样便于后面的学习和理解

下图是其中最主要的三个对象类型`commit` `tree` `blob`之间的关系。

![image-20181213145842415](/Users/lijie/Work/blogs/notes/git/assets/blob-tree-commit.png)

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

![commits-and-parents](/Users/lijie/Work/blogs/notes/git/assets/commits-and-parents.png)





## Branch-Tag-HEAD

![Commit Branch Tag HEAD关系](./assets/branch-and-history.png)

# 核心概念（三个指针）

上一节介绍了`git`的三个重要的对象（`commit`、`tree`、`blob`），这三个对象是`git`的存储对象，是实现版本管理的核心和基础。本节来说一说`git`的上层建筑，`HEAD`、`tag`、`branch`这三驾马车是实现`git`灵活、高效的核心。

> 后续章节会有关于`HEAD`和`branch`的更详细的介绍

![Commit Branch Tag HEAD关系](/Users/lijie/Work/blogs/notes/git/assets/branch-and-history.png)

## branch

如果说`commit`实现了对数据文件的版本管理，那么利用`branch`可以实现对多个版本的并行工作而之间互不影响。

### 是什么？

1. 本质上是指向提交对象的**可变**指针

   在当前`branch`进行提交操作后，会移动到新的`commit`对象

2. 是一个文件（存储在`.git/refs/heads/`目录）

   文件内容是其指向的`commit id`

3. 默认`branch`名字是 `master`

   在初始化仓库时由 Git 自动创建的

**注：新创建的配置库(没有提交过)，并没有`master`分支对象(在`.git/refs/heads/`目录中不存在)**

### 作用

在`Git`版本控制中，起到了重要的作用

1. 多个任务并行开展互不影响（如：当前发布版本、下一版本功能、问题修复、优化重构等）
2. 通过`branch`快速切换工作任务

分支实质上仅是包含所指对象校验和（长度为 40 的 SHA-1 值字符串）的文件，所以它的创建和销毁都异常高效。 创建一个新分支就相当于往一个文件中写入 41 个字节（40 个字符和 1 个换行符）如此的简单能不快吗？



### 创建

创建`branch`非常简单和快速，因为`git`中创建`branch`并不需要复制一份仓库文件的副本（其他版本控制需要）

![创建分支](/Users/lijie/Work/blogs/notes/git/assets/head-to-master.png)

使用`git branch <分支名称>`指令，该指令会在当前`branch`上，创建一个新的`branch`，如上图所示

该指令不会切换当前的`branch`，如果需要切换到新创建的`branch`，使用切换指令

**注：当前`branch`是`HEAD`指向的`branch`或者`commit id`**

```shell
# 创建一个新的分支
$ git branch testing

# 查看分支
# git branch -v
  testing   46b5bd7 Add three.md
* master 	46b5bd7 Add three.md # 最前面的 * 号，表示当前工作在这个分支上
```



### 切换

切换操作将执行两个任务

1. 将`HEAD`指针指向目标分支
2. 将工作目录和暂存区恢复成目标分支指向的快照（即`commit`）
   - 切换前 工作目录 如果有 未跟踪 文件，切换不影响该文件（继续存在）。
   - 切换前 暂存区 如果有 未提交 文件，切换后自动将这些文件添加到 分支中（将影响分支）。
3. 在切换前，需要清理暂存区，保持暂存区是干净的。

![切换分支](/Users/lijie/Work/blogs/notes/git/assets/head-to-testing.png)

使用`git checkout <分支名称>` 指令，该指令可以切换`branch`，还可以将创建和切换合二为一

```shell
# 切换到 testing 分支上
$ git checkout testing
Switched to branch testing

# 切换并创建分支 testing，使用 -b 参数开关
# git checkout -b testing
Switched to a new branch testing
```



### 删除

```shell
# 查看分支
$ git branch -v
  dev     46b5bd7 Add three.md
* master  46b5bd7 Add three.md
  testing 46b5bd7 Add three.md

# 删除分支 testing
$ git branch -d testing
Deleted branch testing (was 46b5bd7).

# 如果删除当前工作分支，git 会提示如下错误
error: Cannot delete branch testing checked out at <工作目录>
```

**注：思考一下，为什么不能删除当前工作分支呢？**

答案是：`HEAD`指针丢失，而`HEAD`指针必须指向一个`branch`或者`commit`

### 合并

```shell
On branch master
You have unmerged paths.
  (fix conflicts and run "git commit")
  (use "git merge --abort" to abort the merge)

Unmerged paths:
  (use "git add <file>..." to mark resolution)

	both modified:   three.md

no changes added to commit (use "git add" and/or "git commit -a")
```





## tag

标签本质上是指向提交对象的**固定**指针。通过给重要的提交 `commit` 添加标签(语义化)，来方便定位。比较代表性的使用场景是用来标记发布节点 ( 如 V1.0 等)

- 标签可以看做是`commit`的语义化描述
- 标签主要用于只读场景，即通过标签获得仓库中的数据文件
- 标签较多用来标记发布节点

```shell
# 添加标签
$ git tag v1.0 -m'v1.0'

# 查看标签
$ git tag
v1.0

# 标签存放在refs/tags目录下
$ cd refs/tags
$ ls -l
total 8
-rw-r--r--  1 lijie  staff  41 12 13 18:24 v1.0

# 标签文件中存放的内容是 tag对象
$ cat v1.0
0c2bfdbd52d00bb2da5f7b3b5b2047a412301d5f
$ git cat-file -t 0c2bfdbd
tag

# tag对象中存放内容
$ git cat-file -p 0c2bfdbd
object 72931be9bab7ba7e5c58ae1d80bdf2b959eda48d # commit对象
type commit
tag v1.0
tagger lijiemac <lijie@boco.com.cn> 1544696684 +0800 # 谁打的标签

v1.0  # 标签备注

```



## HEAD

`HEAD`是当前操作`branch`或者`commit`的指针。

- `git`存在多个`branch`，为了记录当前操作的`branch`，使用`HEAD`
- 每个仓库`repo`只有一个`HEAD`
- 在`.git/`目录中存储

## 目录结构

> 配置库所有的信息都存储在`<workdir>/.git`目录中

```shell
$ ls -al
total 24
-rw-r--r--   1 lijie  staff    9 12 12 09:14 COMMIT_EDITMSG		# 最近一次提交时的消息内容
-rw-r--r--   1 lijie  staff   23 12 12 09:14 HEAD				# HEAD头指向的branch/commit
drwxr-xr-x   2 lijie  staff   64 12 12 09:14 branches			# branch存放的目录
-rw-r--r--   1 lijie  staff  144 12 12 09:24 config				# local配置信息
-rw-r--r--   1 lijie  staff   73 12 12 09:14 description		# 
drwxr-xr-x  12 lijie  staff  384 12 12 09:14 hooks
-rw-r--r--   1 lijie  staff  118 12 12 09:14 index				# 暂存区
drwxr-xr-x   3 lijie  staff   96 12 12 09:14 info				# 可以存放项目的忽略文件配置
drwxr-xr-x   4 lijie  staff  128 12 12 09:14 logs				# 操作日志
drwxr-xr-x   4 lijie  staff  128 12 12 09:14 objects			# 文件快照存储的目录
drwxr-xr-x   4 lijie  staff  128 12 12 09:14 refs				# heads和tags存放的目录
```

新创建的空配置库中，目录结构是完整的，但没有任何的`branch`、`tag`、`commit`、`tree`、`blob`文件

## COMMIT_EDITMSG文件

初始化的配置库中，不存在该文件。

在本地使用`git commit -m'消息内容'`后，保存最后一次提交的消息内容

## HEAD文件

HEAD 保存当前操作的`branch`或`commit`，内容如下

```shell
$ cat HEAD
ref: refs/heads/master
```

注：当前指向的是`master`分支

如果`HEAD`指向的是一个`commit id`而不是`branch`，则系统会提示`detached HEAD`警告。

## branches目录



## config文件

配置库私有的配置文件。文件内容如下：

```shell
$ cat config
[core]
	repositoryformatversion = 0
	filemode = true
	bare = false
	logallrefupdates = true
	ignorecase = true
	precomposeunicode = true
[user]
	name = lijie
	email = lijie@boco.com.cn
```

注：由于设置了库作用域的用户信息`git config --local`，因此`[user]`中有相关的信息。



## Index 文件

`index`指的是`git`的暂存区，用于存放数据文件的索引。

- 所有`git add `指令添加/删除/更改的文件，都存放在暂存区中
- 通过`git clone`指令复制的仓库，暂存区中保存的是`HEAD`指针指向`branch`的所有文件

```shell
# 使用 git ls-files -s 查看暂存区中的文件
$ git ls-files -s
100644 ca028fbbadceb5991f1d554e3d521baac5977f78 0	1.md
100644 9a61029c3d46030edb0c85a6e99e98ffd510fcc8 0	second.md
100644 8672c74c0922bc3fbb5e4743ddc98f761ef6d26e 0	three.md
```



## objects目录

存放`object`对象的目录，`object`对象包括`commit`、`tree`、`blob`。其中的`blob`对象存储的就是压缩的文件快照。

注：`object`文件的文件名全部采用`SHA-1`计算的哈希值，长度为 40 。

```shell
$ ls -l
total 0
drwxr-xr-x  3 lijie  staff  96 12 12 16:09 15
drwxr-xr-x  3 lijie  staff  96 12 12 17:56 5d
drwxr-xr-x  3 lijie  staff  96 12 12 16:09 6d
drwxr-xr-x  3 lijie  staff  96 12 12 16:04 80
drwxr-xr-x  3 lijie  staff  96 12 12 16:07 e6
drwxr-xr-x  2 lijie  staff  64 12 12 09:14 info
drwxr-xr-x  2 lijie  staff  64 12 12 09:14 pack
```

### pack目录

当存储`object`对象的目录过于凌乱时，会将部分目录进行打包处理。处理后放在此目录中

### info目录

### 其他目录

用于存放`commit`、`tree`、`blob`类型的`git`文件

- 目录名：文件名的前 2 位
- 文件名：文件名后 38 位

因此，对文件进行处理时，需要将两部分拼接后才是真正的文件名。

```shell
# 进入 15 目录
$ cd 15

# 查看文件
$ ls
6aa60400c19d00a4d8bcef1ad0ed5a846e9849
```



## refs目录

存放仓库使用的`branch`和`tag`信息

```shell
$ ls -l
total 0
drwxr-xr-x  3 lijie  staff  96 12 12 16:09 heads
drwxr-xr-x  2 lijie  staff  64 12 12 09:14 tags
```

### heads目录

存放分支信息

```shell
$ ls -l
total 8
-rw-r--r--  1 lijie  staff  41 12 12 16:09 master
```

### tags目录

存放标签信息

```shell
$ ls -l
total 8
-rw-r--r--  1 lijie  staff  41 12 13 18:24 v1.0
```



## 哈希计算

# 计算原理

> 数据对象、树对象、提交对象

## 定义

1. 数据对象

   指`Git`管理的所有文件

   类型：`blob`

2. 树对象

   可以理解为目录，根树对象(`commit`中)是一个归集

   类型：`tree`

3. 提交对象

   当前仓库管理的版本的一个快照

   类型：`commit`

## 公式

```shell
header = <object type> + 空格 + <content length> + \0
hash = sha1(header + content)
```

- 计算哈希值的内容由两部分组成：头信息(header)和内容信息(content)

- 头信息由对象类型、内容长度和空字符组成

  - 对象类型 = [`blobl` | `tree` | `commit`]，明确对象
  - 内容长度 = 字节长度，不是字符串长度（中文的长度 !== 字节长度）
  - 空字符 = `\0`字符，起到分隔作用

- 内容信息

  不同类型对象，其内容信息不同

## 数据对象

**数据对象的哈希计算，不包含文件名**

```shell
header = blob <content length>\0
hash = sha1(<header><conent>)
```

示例如下：

```shell
# 计算内容的长度
$ echo -n "您好" | wc -c
  6
# 使用 git 的 hash 计算
$ echo -n "您好" | git hash-object --stdin
08c34184856086e2b1a02e81250bec00dd55e2ea
# 使用openssl 的 hash 计算
$ echo -n -e "blob 6\0您好" | openssl sha1
08c34184856086e2b1a02e81250bec00dd55e2ea

# 证明计算的格式是正确的
```



## 树对象

```shell
header = tree <content length>\0
content = <file mode> <filename>\0<item sha二进制>...
hash = sha1(<header><conent>)
```

- `file mode`
- `item sha` 是二进制的`sha`码

```shell
# 查看根树的内容
$ git cat-file -p 8c3d2
040000 tree 7cd194af54b759f0949bf26e7bbdf4c9325f1c29	1
040000 tree 7cd194af54b759f0949bf26e7bbdf4c9325f1c29	2
100644 blob 1bccab5e6f5a1222ae039f0df19f9a66a1c0e558	test.txt

# 查看其中 树7cd19 的内容
$ git cat-file -p 7cd19
100644 blob 1bccab5e6f5a1222ae039f0df19f9a66a1c0e558	test.txt

# 计算 树7cd19 中包含内容的哈希值的 二进制码
$ echo -n 1bccab5e6f5a1222ae039f0df19f9a66a1c0e558 | xxd -r -p >sha1.txt
# 加入 <file mode> 和 <filename>
$ echo -n -e "100644 test.txt\0" | cat - sha1.txt > content.txt 
# 计算 内容 的长度
$ cat content.txt | wc -c
  36
# 加入 header 部分 并计算哈希值
$ echo -n -e "tree 36\0" | cat - content.txt | openssl sha1
7cd194af54b759f0949bf26e7bbdf4c9325f1c29

## 结论：通过自行计算的哈希值 与 git计算的哈希值一致
```



## 提交对象

```shell
header = commit <content length>\0
content = tree <tree sha>
parent <parent sha>
[parent <parent sha>]
author <author name> <author email> <timestamp> <timezone>
committer <commiter name> <committer email> <timestamp> <timezone>

<commit message>
hash = sha1(<header><conent>)
```

- 内容部分每一项后加入`\n`换行符，`<commit message>`前再加入一个换行符
- 内容部分涉及`sha`值不需要转换为二进制了
- 如果有多个`parent`，则都需要列出来

```shell
# git中 commit 6ea06 的内容
$ git cat-file -p 6ea06
tree 8c3d22921e28aed901bb57bd7c3cf2be06b85619
author lijiemac <lijie@boco.com.cn> 1545703889 +0800
committer lijiemac <lijie@boco.com.cn> 1545703889 +0800

aaa

# 计算内容长度
$ echo -n -e "tree 8c3d22921e28aed901bb57bd7c3cf2be06b85619\nauthor lijiemac <lijie@boco.com.cn> 1545703889 +0800\ncommitter lijiemac <lijie@boco.com.cn> 1545703889 +0800\n\naaa\n" | wc -c
  160

# 加入 header 部分 并计算哈希值
$ echo -n -e "commit 160\0tree 8c3d22921e28aed901bb57bd7c3cf2be06b85619\nauthor lijiemac <lijie@boco.com.cn> 1545703889 +0800\ncommitter lijiemac <lijie@boco.com.cn> 1545703889 +0800\n\naaa\n" | openssl sha1
6ea063d24ed546cd9c75c16989d5c04774459f09

## 结论：通过自行计算的哈希值 与 git计算的哈希值一致
```



# 深入分析

## 内容相同的数据对象，保存为一个Blob文件

- 数据对象的哈希值计算 仅与其内容相关，与文件名称和所在位置无关

## 目录中文件相同（数量、名称、内容），保存为一个Tree文件

- 数对象的哈希值计算 仅与其中包含的 文件名 文件哈希值 有关，与其目录名称无关

## 演示

- 创建一个名称为`test.txt`文件，其内容为`您好，我是一个测试文件。`
- 将这个文件复制到 目录`1`和目录`2`下。这样在工作目录中就存在三个相同的`text.txt`文件和两个包含相同内容的目录`1`和`2`
- 将这三个文件提交后，查看仓库中如何存储这些文件的

### 第一步 创建文件和目录

```shell
# 创建一个文件
$ echo "您好，我是一个测试文件。" > test.txt
# 将文件拷贝到新创建的目录1中
$ mkdir 1
$ cp test.txt 1
# 将文件拷贝到新创建的目录2中
$ mkdir 2
$ cp test.txt 2
# 查看工作目录中的文件
$ ls -l
total 8
drwxr-xr-x  3 lijie  staff  96 12 25 10:08 1
drwxr-xr-x  3 lijie  staff  96 12 25 10:08 2
-rw-r--r--  1 lijie  staff  37 12 25 10:07 test.txt

# 查看仓库状态
$ git s
On branch master
No commits yet
Untracked files:
  (use git add <file>... to include in what will be committed)

	1/
	2/
	test.txt
nothing added to commit but untracked files present (use git add to track)
```



### 第二步 将文件提交到暂存区

```shell
# 提交到暂存区
$ git add .
$ git s
On branch master
No commits yet
Changes to be committed:
  (use git rm --cached <file>... to unstage)
	new file:   1/test.txt
	new file:   2/test.txt
	new file:   test.txt

# 提交到暂存区 已经在仓库中创建了 数据对象
# 只有一个数据文件
$ ls .git/objects
1b	info	pack
$ ls .git/objects/1b
ccab5e6f5a1222ae039f0df19f9a66a1c0e558
# 文件内容为
$ git cat-file -p 1bcca
您好，我是一个测试文件。
```



### 第三步 将暂存区文件提交到仓库中

```shell
# 提交到仓库中
$ git commit -m'aaa'
[master (root-commit) 6ea063d] aaa
 3 files changed, 3 insertions(+)
 create mode 100644 1/test.txt
 create mode 100644 2/test.txt
 create mode 100644 test.txt

# 查看仓库中存储的对象
# 有四个对象
$ ls .git/objects
1b	6e	7c	8c	info	pack

# 6e开头的是 commit对象 

# 1b开头的是 blob对象  
# 从这里看出，工作目录中三个文件，实际存储为1个对象
$ git cat-file -p 1bcca
您好，我是一个测试文件。

# 7c开头的是 tree对象
$ git cat-file -p 7cd19
100644 blob 1bccab5e6f5a1222ae039f0df19f9a66a1c0e558	test.txt

# 8c开头的是 tree对象
# 从这里可以看出 目录1 和 目录2 公用一个 tree对象
$ git cat-file -p 8c3d2
040000 tree 7cd194af54b759f0949bf26e7bbdf4c9325f1c29	1
040000 tree 7cd194af54b759f0949bf26e7bbdf4c9325f1c29	2
100644 blob 1bccab5e6f5a1222ae039f0df19f9a66a1c0e558	test.txt
```





