# Linux 常用命令

## 1. 常用

<details class="detail">
<summary class="title"><span class="d-marker">&nbsp;</span></summary>

**<summary>**

### ls
显示指定工作目录下之内容
```bash
ls -a # 显示所有文件及目录（含隐藏文件）
ls -l # 除文件名称外，亦将文件型态、权限、拥有者、文件大小等资讯详细列出
```

### wc
用于计算字数
```bash
wc [-clw][filename]
-c # 字节
-l # 行数
-w # 字数
# 默认的情况下，wc将计算指定文件的行数、字数，以及字节数。
$ wc README.md
    29      56     366 README.md
```

### cat
用于连接文件并打印到标准输出设备上
```bash
cat [filename]
```

### more
类似 `cat` ，不过会以一页一页的形式显示，更方便使用者逐页阅读
```bash
more [filename]
```

### less
`less` 与 `more` 类似，但使用 `less` 可以随意浏览文件，而 `more` 仅能向前移动，却不能向后移动，而且 `less` 在查看之前不会加载整个文件
```bash
less [filename]
```

### cd
用于切换当前工作目录
```bash
# 跳转到 Home 目录
cd ~
# 跳转到指定路径
cd /usr/bin
# 跳转到上一级
cd ..
```

### cp
主要用于复制文件或目录
```bash
cp [filename] [filepath] # 默认复制文件
# 复制文件夹要带上 -r
$ cp –r test/ newtest # 将当前目录 test/ 下的所有文件复制到新目录 newtest 下
```

### mv
用来为文件或目录改名、或将文件或目录移入其它位置
```bash
mv [filename] [filepath]

# 目标目录与原目录一致，指定了新文件名，效果就是仅仅重命名
$ mv  /home/a.txt   /home/b.txt
# 目标目录与原目录不一致，没有指定新文件名，效果就是仅仅移动
$ mv  /home/a.txt   /home/test/
# 目标目录与原目录一致, 指定了新文件名，效果就是：移动 + 重命名
$ mv  /home/a.txt   /home/test/c.txt
```


### top
用于实时显示进程的动态
```bash
top # 显示进程信息
```

### rm
用于删除一个文件或者目录
```bash
rm [filename/path]
-f # 即使原档案属性设为唯读，亦直接删除，无需逐一确认
-r # 将目录及以下之档案亦逐一删除
# 传说中的删库跑路
rm -rf [filepath]
```

### pwd
用于显示工作目录

```bash
$ pwd
/Users/bin/Code/blog
```

### mkdir
用于创建目录
```bash
mkdir [-p] [dirpath]
-p # 确保目录名称存在，不存在的就建一个
# 本例若不加 -p 参数，且原本 temp 目录不存在，则产生错误
$ mkdir -p /home/temp/test
```

### ps
用于显示当前进程的状态
```bash
$ ps -ef # 显示所有命令，连带命令行
```

### grep
用于查找文件里符合条件的字符串
```bash
grep [pattern] [filename]
# 查找后缀有 file 字样的文件中包含 test 字符串的文件并打印出该字符串的行
$ grep test *file 
```

- `ps` 搭配 `grep`，有一个很常用的命令

根据`进程名/进程id`搜索进程状态：

```bash
ps -ef | grep [进程名/进程id]
```

### kill
关闭执行中的程序或工作
```bash
$ kill -9 [pid] # 彻底杀死进程
```

### chmod
控制用户对文件的权限的命令
```bash
chmod [-cfvR] [filename]
# 有时我们创建了脚本，发现 ./run.sh 无法执行，这时就需要添加可执行权限
$ chmod +x [filename] 
```

</details>

## 2. 难点
### sed

1. 输出`hello.txt`的第`3`行到最后一行：

```bash
sed -n '3,$'p hello.txt
```

2. 匹配以`10`开头的行，并替换为`yes`，并输出

```bash
sed -n 's/^10/yes/p' hello.txt
```

3. 删除匹配`100`的行：

```bash
sed '/100/'d hello.txt
```


> sed: https://mp.weixin.qq.com/s/29iMrw1CFpmLiW36p40szw

### awk

逐行处理文件中的数据

1. 输出`hello.txt`文件中第`3`到`5`行：

```bash
awk 'NR == 3, NR == 5{print;}' hello.txt
```

2. 输出`hello.txt`第`3`列：

```bash
awk '{print $3}' hello.txt
```

3. 输出`hello.txt`第`2`行、第`3`列：

```bash
sed -n "2p" hello.txt | awk '{print $3}'
```

4. 输出`hello.txt`中，正则匹配`hello`的行

```bash
awk '/hello/' hello.txt
```

> awk: https://mp.weixin.qq.com/s/ofvB31w5rRywIdUUtOjskw

## 3. 参考
菜鸟教程`Linux`命令大全：https://www.runoob.com/linux/linux-command-manual.html
