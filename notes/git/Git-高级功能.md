title: 5.Git-高级功能
date: 2019-03-14
tags: GIT
categories: 配置管理
layout: draft

------

摘要：文本是使用Git进行代码管理的第五篇文章，重点介绍：Git的一些高级用法，包括：分离头指针、变基、Cherry-pick、回退、提交合并、修改提交消息、强制移动分支、垃圾回收等。这些高级用法让我们可以更加自如的应对出现的各种问题，例如：提交到了错误的分支上、修改已提交的消息、从不同分支中提取部分提交到新的分支上、自由移动分支的指针、让提交历史更简洁等。

<!-- more -->

我们之前了解了Git的基本用法，包括：初始化仓库、克隆远程仓库、添加文件到暂存区、提交、创建分支、切换分支、删除分支、推动到远程仓库、从远程仓库拉取等等。掌握了这些用法基本满足日常使用的需要，但有时在使用时会出现一些误操作，或者导致分支提交历史的混乱，这就需要使用Git的高级用法来解决。

## 头指针

在分支部分介绍过头指针`HEAD`，头指针是指向当前分支（更准确的说是提交）的引用。我们在切换工作分支时，头指针指向当前工作分支，工作区的内容切换为分支指向的提交版本。

![Git分支初始状态](./assets/branchs-ops-init.png)

使用检出指令`git checkout [分支名称] | [提交哈希]`，将改变头指针的指向。

### 分离头指针

分离头指针是头指针的一种非正常状态`detached HEAD`。当头指针指向的不是本地分支而是某个提交时（即使是分支指向的提交），就处于分离头指针状态。

![分离头指针](./assets/branch-detached.png)

由于远程跟踪分支是只读副本，无法进行提交操作。因此，如果检出一个远程跟踪分支也会处于分离头指针状态。

```bash
# 检出远程跟踪分支
$ git checkout origin/master
$ git status
# 输出
HEAD detached at origin/master

# 检出提交哈希
$ git checkout 7c9ca
$ git status
# 输出
HEAD detached at 7c9ca
```

当处于分离头指针状态时，添加暂存区和提交操作能够正常执行。但是，由于新的提交没有关联的分支，当切换到其他分支后，在分离头指针状态下的提交会被 Git 垃圾回收。

### 移动头指针

正常情况下，头指针的指向由 Git 负责维护。在切换分支、在分支上进行提交、分离头指针状态下的提交、回退历史等操作时，头指针都会自动进行移动。

当我们需要将头指针移动到指定位置（提交）时，通常情况下使用提交的哈希值，而哈希值的查找和记忆比较不方便，因此 Git 提供了一种简写方式，可以在分支名或`HEAD`后使用`^`和`~n`快速定位提交。

在分支章节我们介绍过，分支是一个单向的提交链，可以从分支的尾部向头部单方向遍历，这两个符号就是从当前分支指向的提交向上定位第几个提交。

`^`。一个符号代表向上一级

`~n`。数字n表示向上的级数

![移动头指针](./assets/branch-move-head.png)

```bash
# 定位到 b83c0 提交
$ git checkout b83c0
$ git checkout HEAD^
$ git checkout HEAD~1

# 定位到 7c9ca 提交
$ git checkout 7c9ca
$ git checkout HEAD^^^
$ git checkout HEAD~3
```

这种简写方式，在 Git 高级用法中会经常用到，需要掌握。

## 变基

变基`rebase`是分支合并的另一种方法。在分支章节，介绍了使用`git merge`进行分支合并的方法。

### 什么是变基

变基是将一系列提交按照原有次序依次应用到另一分支上，就好像在另一个分支上"重新播放"一样。

1. 变基用于分支的合并
2. 与`merge`不同的是，变基不会生成合并提交，而是将历次提交一次应用到目标分支上
3. `merge`是将分支合并到当前分支，而`rebase`是将当前分支合并到目标分支
4. 变基将保持提交树的整洁，不会出现分叉情况，就像串行工作一样

### 变基操作

下面通过一个例子，介绍变基的操作。

下图是 Git 的分支和提交情况，有两个分支分别为`experiment`和`master`，两个分支有一个共同的祖先`C2`。

![变基-源分支](./assets/basic-rebase-1.png)

如果使用`merge`进行合并，则合并操作将采用第三方合并的方式，创建一个合并提交`C5`，提交树出现分叉。

```bash
# 切换到 master 分支
$ git checkout master
# 合并 experiment 分支
$ git merge experiment
```

合并操作后，`master`分支指向合并提交`C5`，而`experiment`分支保持不变

![变基操作-merge](./assets/basic-rebase-2.png)

如果采用`rebase`进行合并，则合并操作将创建与`C4`提交相同的`C4'`提交并指向`C3`，而`C4`提交被废弃，提交树不会出现分叉。

```bash
# 切换到 experiment 分支
$ git checkout experiment
# 变基到 master 分支
$ git rebase master
```

合并操作后，`experiment`分支指向提交`C4'`，而`master`分支保持不变。

![变基操作-rebase](./assets/basic-rebase-3.png)

如果要移动`master`分支指向合并后的提交`C4'`，则需要执行快速合并操作`merge`

```bash
# 切换到 master 分支
$ git checkout master
# 合并 experiment 分支
$ git merge experiment
```

![变基操作-merge](./assets/basic-rebase-4.png)

### 变基操作原理

1. 找到两个分支（即当前分支 `experiment`、目标基底分支 `master`）的最近共同祖先 `C2`
2. 对比当前分支相对于该祖先`C2`的历次提交，提取相应的修改并存为临时文件
3. 将当前分支指向目标基底 `C3`
4. 将之前另存为临时文件的修改依序应用在当前分支

### 变基的用途

确保向远程分支推送时，保持提交历史的整洁。

### 变基的几种用法

假设 Git 仓库中有三个分支

1. 将当前工作分支变基到目标分支

```bash
# 切换 工作分支
$ git checkout experiment
# 变基到 主分支
$ git rebase master
```

2. 将特性分支变基到目标分支

```bash
$ git rebase [目标分支] [变基分支]
$ git rebase master experiment
```

3. 将两个分支的差异变基到目标分支

![变基操作-三分支](./assets/interesting-rebase-1.png)

```bash
# 只将 client 分支的变化(C9、C9) 变基到 master
$ git checkout client
$ git rebase --onto master server client
```

![变基操作-三分支](./assets/interesting-rebase-2.png)

## 交互式变基

