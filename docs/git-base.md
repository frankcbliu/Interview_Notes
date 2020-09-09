
在开始本篇文章之前，读者可以先试着回答以下几个问题：
- `Git`是什么？和`Github`的区别与联系
- `Git`有哪些命令，你能数出`10`个来么？
- `CR`、`MR`、`CC`分别是啥意思？

如果有回答不出的，那么建议还是往下仔细看看文章吧~

## 基本概念

首先需要明白`Git`是一款软件，它跟你用的 QQ、微信没有本质的区别，其次，它是用于在软件开发过程中进行版本控制的软件。
- 使用`C`开发，性能好
- 分布式版本控制，与集中式版本控制的`SVN`等不同
- 创造之初是为了高效管理`Linux`内核开源项目

了解了`Git`，再来说下`Github`。`Github`实际上是一个在现代码托管平台，说人话就是远程仓库。与我们在本地使用 `Git` 时产生的本地仓库相对应。没有`Github`，我们依然可以在本地操作`Git`去管理我们的代码。除了`Github`以外，还有`Gitlab`，国内的码云`Gitee`，都是远程仓库。

## 常用的 Git 命令

### 获取 Git 仓库

在现有目录下初始化：
```shell
$ git init
```

从远程仓库克隆：
```shell
$ git clone [project_url]
```

### 查看当前状态

```shell
$ git status   
On branch master
Your branch is up to date with 'origin/master'.

Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        deleted:    docs/helloworld.md

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        docs/git.md

no changes added to commit (use "git add" and/or "git commit -a")
```
可以看到当前我们处于`master`分支，然后有一个未跟踪的文件`git.md`，有一个被删除的文件`helloworld.md`。
熟练的话可以使用`git status -s`，可以得到更加紧凑的输出。

> 这里看起来多了几个文件是为了下面展示不同的标记，所以又改的。

```shell
$ git status -s
M  docs/code.md
 D docs/helloworld.md
 M index.html
?? docs/git.md
```

- `??`: 新添加的未跟踪文件
-  `A`: 新添加到暂存区中的文件
-  `M`: 左边的 M 表示该文件被修改了并放入了暂存区
-  `M`: 右边的 M 表示该文件被修改了但是还没放入暂存区
-  `D`: 被删除的文件

### 跟踪新的文件（加入暂存区）

```shell
$ git add [file_name]
```

按照上面的例子，我们可以使用`git add README`来把`README`文件加入`git`的跟踪范围。
如果我们有多个文件，甚至还有一堆目录，总不能一个一个加的吧，这时我们可以：
```shell
$ git add .
```
就可以添加当前目录下的所有文件/文件夹了。

### 比较未暂存区和暂存区的代码差异

```shell
$ git diff
```

有实习过的同学往往会听到大佬们说把结果`diff`一下，也是类似的意思，就是比较两者之间的差异。用`git diff`可以很直观的看到这次修改做了哪些改动，建议大家写`add`前也养成先`diff`下的习惯。

### 提交更新到仓库

```shell
$ git commit -m '此次提交的修改信息'
```

这里网上也有一些规范，比如`Angular`的提交规范，有兴趣的同学可以自己搜下。
到这里，其实修改已经提交到本地仓库了，接下来，我们还需要将代码同步到远程仓库。

### 本地仓库与远程仓库的同步

将本地代码提交到远程仓库的`master`分支：
```shell
$ git push origin master
```

往往我们是多人合作开发，远程仓库可能还有其他人的改动，那如何从远程仓库拉取其他人的更新到本地仓库呢？
```shell
$ git pull origin master
```
当然，往往直接`git pull`即可。

> 以为到这里就结束了？不不不，这才刚刚开始。

### 代码冲突怎么办？
你吭哧吭哧写了几百行代码，提交的时候发现已经有人先你一步提交了代码，此时你没办法提交代码上去，咋办？
有`聪明`的小伙伴说，这个简单，我先把自己的代码备份下，然后重新拉仓库代码下来，然后再把代码丢进去，再提交就好啦。

> 不得不说，这种暴力的方法，在我一开始不怎么会用`Git`时确实蛮好用的，但是这样一方面效率差，另一方面一直以解决问题就行的态度对自身提高有很大阻碍。

所以我们需要更聪明的办法来解决这个问题。

方法1：
```shell
$ git stash
```
先将本地的修改保存的栈中，然后拉取远程仓库代码：
```shell
$ git pull
```
再把刚刚修改的代码放出来：
```shell
$ git stash pop
```
如果存在没法自动合并的部分，代码中会出现

```
<<<< HEAD
// 原来的代码
-----
// 你改的代码
>>>> 一大串字符串
```

你只要保留你想要的代码，然后把无关的符号(`<<<`、`----`、`>>>>`)删除：
```
// 只保留你改的代码
```
然后重新`add -> commit -> push`即可。

### 怎么代码回滚？
写了一个 BUG，上线后才发现，这时候咋办呢？一般来说保存现场，然后立马回滚，那么怎么回滚代码呢？
一般来说有**两种**办法：`revert`和`reset`，以及暴力物理回滚（把上个版本代码的备份下，然后修改现有代码重新提交。）

而在回滚之前我们必须的操作就是看看提交历史：
```shell
$ git log

commit d69ca4e2aafxxx6a357701be960c6a80491917xx (HEAD -> master, origin/master, origin/HEAD)
Author: xxxx 此处打码了 <xxx@xxx.com>
Date:   Thu Jul 23 01:02:30 2020 +0800

    feat:add buildTree.md

commit da8fcec8d4xxxf96f2b492e200c0aa2849fdb1xx
Author: xxxx 此处打码了 <xxx@xxx.com>
Date:   Thu Jul 23 00:56:28 2020 +0800

    feat:add xxxxx
```
可以看到每个`commit`都有一大串的字符串作为`commitID`，比如：`da8fcec8d4xxxf96f2b492e200c0aa2849fdb1xx`

那么我们要回滚到下方那个`da8f`版本咋办呢？

```shell
$ git reset -hard da8fcec8d4xxxf96f2b492e200c0aa2849fdb1xx
```

然后强制推上`Github`：

```shell
$ git push -f origin master
```
这里为啥又要强制呢？展开来又是另外一个故事了，总而言之，这里介绍的回滚方法适合你自己做的小项目，代码丢失不会造成太大损失，那么放心使用。至于在公司我们该怎样回滚比较合适，后续会再写一篇来讲解。

## 了解 Git 相关的行业术语

有些术语其实就是缩写，但是你不知道就是不知道，早点了解不会有坏处。
- `CR`: `Code Review`的缩写，意思是代码评审，在流程规范的公司开发，代码是需要经过评审才能合入主干的
- `MR`: `Merge Request`的缩写，意思是合并代码请求，在流程严格的公司开发，代码合并时可能还需要主管同意，也就是需要主管审批后你的代码才能合入主干
- `CC`: `Code Check`的缩写，意思是代码检查，有些公司会有自动化检查，当你把代码合入仓库时，就会触发代码检查，如果代码检查不通过，那么你的代码就无法合入主干

说完这几个术语，我们再来看看一般公司的代码开发流程：
```
开发代码 -> 提交代码 -> 代码评审 -> 主管审批-> 代码检查
```
大家自己和同学一起开发项目时也可以尝试下这样的规范，提前熟悉。

# 总结
希望读者们看完文章后可以回答出开篇的问题，那么这篇文章也就没白写。