> 上一篇文章我们了解了`Git`的常用命令，这一篇文章我们将来了解这些常用命令的工作原理，以便更好的掌握这些命令。

在开始本篇文章之前，读者可以先试着回答以下几个问题：

- 是否了解`工作区`、`暂存区`、`仓库`之间的区别？
- `Git`的常用命令在三大区域中是如何工作的？
- 分支是如何合并的？原理是什么？
- 分支合并中`rebase`和`merge`的区别？

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


<p align="center">
<img  src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghxbnyjbbyj316e0ne77f.jpg" height="400px"></img>
</p>

- `git diff`，不加任何参数，将工作区（未`add`的内容）和暂存区进行比较；
- `git diff HEAD`，将工作区与`HEAD`指针指向的`commit`进行比较，一般来说我们当前的改动就是在`HEAD`指向的`commit`的基础上进行改动；
- `git diff --cached`，将暂存区与当前`commit`进行比较；
- `git diff dev`，将工作区与目标分支的最新`commit`进行比较；
- `git diff [commitId_1] [commitId_2]`，将两个`commit`进行比较。

### commit

前面我们说了，`commit`会在暂存区生成快照，然后推到本地仓库，这里我们考虑三种情况下的提交：

- 当前`HEAD`指向末尾的`commit`：


<p align="center">
<img  src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghxcep8a22j317m0m441n.jpg" height="400px"></img>
</p>

- 当前`HEAD`指向中间的`commit`，此时提交就会再分离出一条新的路线，因此后续的分支合并就不可避免地要派上用场。


<p align="center">
<img  src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghxce8tyfnj30zq0p6juo.jpg" height="400px"></img>
</p>

- 希望用新提交覆盖前一个提交：`git commit --amend`：


<p align="center">
<img  src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghxclor0hjj313a0lk0wd.jpg" height="400px"></img>
</p>

这个使用场景也非常广泛，比如我们`git commit`后才发现漏改了点东西，这个时候如果再改再提交，就会导致对一个错误的修改用了两个`commit`，在`git log`上看将会非常丑，对于我们自己做小`demo`时可能无所谓，对于一些大项目或者开源项目，本来`commit`就很多，这样胡乱地增加`commit`必然是不能接受的。

如上图所示，我们新增的`commit`会代替原来的`commit`的位置，而旧`commit`则被抛弃掉。

### checkout

当我们使用`git checkout [branch_name]`切换分支时，如下图所示：


<p align="center">
<img  src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghxcrvk13rj30zq0komzm.jpg" height="400px"></img>
</p>

`dev`分支会把其中的内容复制到暂存区和工作区中，覆盖掉`master`的版本，而只存在于`master`的文件则会被删除。

### reset
下图展示了回滚的情况，具体的三种情况请仔细看下方的描述：


<p align="center">
<img  src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghxd30i7y1j30y80kigol.jpg" height="400px"></img>
</p>

- `git reset [commitId] --sort`，这是最弱的回滚方式，只改变`commit`信息，不影响**暂存区**和**工作区**；
- `git reset [commitId]`，不携带参数时，默认只回滚**暂存区**，也就是把`dks8v`所在的信息复制到**暂存区**，但是不影响**工作区**；
- `git reset [commitId] --hard`，这种方式则能回滚**工作区**和**暂存区**。

### merge

`Git`的合并有许多策略，默认情况下`Git`会帮助我们挑选合适的策略，当然如果我们需要手动指定，可以使用：`git merge -s [策略名称]`，了解 `Git` 合并策略的原理可以使你对合并结果有一个准确的预期。

####  Fast-forward

`Fast-forward`是最简单的一种合并策略，如我们前面示例的图所示，`dev`分支是`master`分支的祖先节点，那么合并`git merge dev`的话，只会将`dev`指向`master`当前位置，`Fast-forward`是`Git`合并两个**没有分叉**的分支时的默认行为。

#### Recursive

`Recursive`是`Git`在合并两个**有分叉**的分支时的默认行为，简单的说，是递归的进行三路合并。


<p align="center">
<img  src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghyh8c69f7j318u0oyn1n.jpg" height="400px"></img>
</p>

这里出现了一个新名词——`三路合并（three-way merge）`，也是我们接下来讲解的重点。我们先搞清楚合并的整体链路。

- 首先`dev`分支的`c5k8x`与`HEAD`指向的`sf22x`，再加上它们的最近公共祖先`a23c4`先进行一次三路合并；
- 然后将合并后的结果拷贝到**暂存区**和**工作区**；
- 再然后产生一次新的提交，该提交的祖先为`dev`和`原master`；

## 分支合并的原理

首先，我们来看看两个文件如何合并：

下图所示为`test.py`中某一行的代码，如果我们要将`A/B`两个版本合并，就需要确定是`A`修改了`B`，还是`B`修改了`A`，亦或者两者都修改了，显然这种情况下分辨不出来。


<p align="center">
<img  src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghyhs6cue4j30ka08w74o.jpg"></img>
</p>

因此，为了实现两个文件的合并，我们引入**三路合并**：

如下图所示，很显然`A`与`Base`版本相同，`B`版本的修改比`A`版本新，因此将`A/B`合并后，得到的就是`B`版本。


<p align="center">
<img  src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghyhw6316fj30jq0emq3p.jpg" height="400px"></img>
</p>

聪明的读者看完上面的例子，就会想到，要是`A/B`和`Base`都不一样怎么办？这就是接下来要讲的问题了。

### 冲突

当出现下图这种情况时，一般就需要我们手动解决冲突了。


<p align="center">
<img  src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghyhz5vbohj30ji0e6mxx.jpg" height="400px"></img>
</p>

也就是我们在合并代码时往往会看到的一种情况：

```python
<<<<<<< HEAD
print("hello")
=======
print("fxxk")
>>>>>>> B
```

对于新手而言，看到这个箭头可能有点摸不着头脑，到底哪个是哪个呢？其实分辨起来很简单，中间的`=======`是分隔符，到最上方的`<<<<<<`之间的内容，是`HEAD`版本，也就是当前的`master`分支，而到最下方`>>>>>>`之间的内容，则是分支`B`的，我们只需要删除箭头，保留所需要的版本即可：

```python
print("hello")
```

最终合并结果：


<p align="center">
<img  src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghyi62do05j30w80dkmyc.jpg" height="400px"></img>
</p>

### 递归三路合并

在实际的生产环境中，`Git`的分支往往非常繁杂，会导致合并`A/B`时，能找到多个`A/B`的共同祖先，而所谓的递归三路合并就是，对它们的共同祖先继续找共同祖先，直到找到唯一一个共同祖先为止，这样可以减少冲突的概率。


<p align="center">
<img  src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghyiesx9xnj30sg0bumz2.jpg" width="600px"></img>
</p>

如上图所示，我们要合并`5`和`6`，就需要先找到`5/6`的共同祖先——`2`和`3`，然后再继续找共同祖先——`1`，当我们找到唯一祖先时，开始递归三路合并，先对`1、2、3`进行三路合并，得到临时节点`2'/B`：


<p align="center">
<img  src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghyikyugzyj30zi0be0ux.jpg" width="600px"></img>
</p>

接下来继续对`2、5、6`进行三路合并，得到`7/C`：


<p align="center">
<img  src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghyimzenpkj30w40bowge.jpg" width="600px"></img>
</p>

## rebase

当我们处于`dev`分支，然后使用`git rebase master`时，可以理解为把`dev`分支上的部分在`master`分支后面重新提交了一遍（重演），具体看下图：


<p align="center">
<img  src="https://tva1.sinaimg.cn/large/007S8ZIlgy1ghyk3uajtij31dw0oo79j.jpg" width="800px"></img>
</p>

首先找到`dev`分支和`master`分支的祖先`a23c4`，然后从`a23c4`到`dev`所在路径上的节点，都通过回放的方式插入到`master`之后，注意，这里“复制”的过程中，`commitId`是会改变的。同时，`dev`旧分支上的节点因为没有了引用则会被丢弃。

## 总结

回顾开头的问题，相信仔细阅读完本篇文章的你已经可以解答了。本篇文章更多聚焦在`Git`的工作原理上，但对于`底层原理`还未展开叙述，下一篇我们会对`Git`底层到底是如何存储文件，如何实现进行讲解，敬请期待。

## 参考资料

图解Git：  https://marklodato.github.io/visual-git-guide/index-zh-cn.html

Pro Git：https://bingohuang.gitbooks.io/progit2/content/