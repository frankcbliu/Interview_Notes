> 上一篇文章我们了解了`Git`的常用命令，这一篇文章我们将来了解这些常用命令的工作原理，以便更好的掌握这些命令。

在开始本篇文章之前，读者可以先试着回答以下几个问题：

- 是否了解`工作区`、`暂存区`、`仓库`之间的区别？
- `Git`的常用命令在三大区域中是如何工作的？
- 分支是如何合并的？原理是什么？
- 分支合并中`rebase`和`fast-forwad`的区别？

如果有回答不出的，那么建议还是往下仔细看看文章吧~

## 三大分区

我们首先用一张图来理解工作区、暂存区和仓库的位置：

<p align="center">
<img  src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghxahaqaouj30qw0qmwgy.jpg" height="600px"></img>
</p>

我们先看由下而上的路径，首先**工作区**就是我们当前的文件目录，我们改完代码，用`git add`命令把当前文件加入**暂存区**，然后`git commit`把**暂存区**生成的快照提交到**本地仓库**，最后再用`git push`命令把**本地仓库**的提交复制到**远程仓库**，也就是`Github`之类的在线仓库。

而由上到下的路径其实也很好理解，`git pull`用来将**远程仓库**的最新提交拉取到**本地仓库**，`git reset -- files` 用来撤销最后一次`git add files`，也就是撤销`commit`，这是我们前面提到的回滚的一种办法；`git checkout -- files`则是把文件从**暂存区**复制到工作区，用来丢弃本地修改（也就是覆盖掉还未`add`到暂存区的改动）。

## 常用命令的工作原理

先来个开胃小菜：

### diff

上一篇文章中我们讲了`git diff`可以直观的看到**工作区**和**暂存区**的差异，这里我们画图演示下不同的`diff`是如何比较的：

![image-20200820154529572](https://tva1.sinaimg.cn/large/007S8ZIlgy1ghxbnyjbbyj316e0ne77f.jpg)

- `git diff`，不加任何参数，将工作区（未`add`的内容）和暂存区进行比较；
- `git diff HEAD`，将工作区与`HEAD`指针指向的`commit`进行比较，一般来说我们当前的改动就是在`HEAD`指向的`commit`的基础上进行改动；
- `git diff --cached`，将暂存区与当前`commit`进行比较；
- `git diff dev`，将工作区与目标分支的最新`commit`进行比较；
- `git diff [commitId_1] [commitId_2]`，将两个`commit`进行比较。

### commit

前面我们说了，`commit`会在暂存区生成快照，然后推到本地仓库，这里我们考虑三种情况下的提交：

- 当前`HEAD`指向末尾的`commit`：

![image-20200820161112096](https://tva1.sinaimg.cn/large/007S8ZIlgy1ghxcep8a22j317m0m441n.jpg)

- 当前`HEAD`指向中间的`commit`，此时提交就会再分离出一条新的路线，因此后续的分支合并就不可避免地要派上用场。

![image-20200820161045817](https://tva1.sinaimg.cn/large/007S8ZIlgy1ghxce8tyfnj30zq0p6juo.jpg)

- 希望用新提交覆盖前一个提交：`git commit --amend`：

![image-20200820161754744](https://tva1.sinaimg.cn/large/007S8ZIlgy1ghxclor0hjj313a0lk0wd.jpg)

这个使用场景也非常广泛，比如我们`git commit`后才发现漏改了点东西，这个时候如果再改再提交，就会导致对一个错误的修改用了两个`commit`，在`git log`上看将会非常丑，对于我们自己做小`demo`时可能无所谓，对于一些大项目或者开源项目，本来`commit`就很多，这样胡乱地增加`commit`必然是不能接受的。

如上图所示，我们新增的`commit`会代替原来的`commit`的位置，而旧`commit`则被抛弃掉。

### checkout

当我们使用`git checkout [branch_name]`切换分支时，如下图所示：

![image-20200820162351757](https://tva1.sinaimg.cn/large/007S8ZIlgy1ghxcrvk13rj30zq0komzm.jpg)

`dev`分支会把其中的内容复制到暂存区和工作区中，覆盖掉`master`的版本，而只存在于`master`的文件则会被删除。

### reset
下图展示了回滚的情况，具体的三种情况请仔细看下方的描述：

![image-20200820163431733](https://tva1.sinaimg.cn/large/007S8ZIlgy1ghxd30i7y1j30y80kigol.jpg)

- `git reset [commitId] --sort`，这是最弱的回滚方式，只改变`commit`信息，不影响**暂存区**和**工作区**；
- `git reset [commitId]`，不携带参数时，默认只回滚**暂存区**，也就是把`dks8v`所在的信息复制到**暂存区**，但是不影响**工作区**；
- `git reset [commitId] --hard`，这种方式则能回滚**工作区**和**暂存区**。

### merge





## 参考资料
图解Git：  https://marklodato.github.io/visual-git-guide/index-zh-cn.html
Pro Git：https://bingohuang.gitbooks.io/progit2/content/