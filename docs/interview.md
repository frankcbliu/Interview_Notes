## 设计模式

### 懒汉式-线程不安全
没有用到该类，就不会实例化。

```java
public class Singleton {
    private static Singleton uniqueInstance;

    private Singleton() {

    }

    public static Singleton getUniqueInstance() {
        if (uniqueInstance == null) { // 多个线程到达这里，会导致多次初始化 uniqueInstance
            uniqueInstance = new Singleton();
        }
        return uniqueInstance;
    }
}
```

### 懒汉式-线程安全
```java
    // 加锁，保证每次只有一个线程进入
    public static synchronized Singleton getUniqueInstance() {
        if (uniqueInstance == null) {
            uniqueInstance = new Singleton();
        }
        return uniqueInstance;
    }
```

### 双重校验锁-线程安全

加锁只需对实例化部分代码进行

```java
public class Singleton {
    // 使用 volatile 禁止 JVM 的指令重排，保证多线程环境下也能正常运行
    private volatile static Singleton uniqueInstance;

    private Singleton() {

    }

    public static synchronized Singleton getUniqueInstance() {
        if (uniqueInstance == null) {
            synchronized (Singleton.class) {
                if (uniqueInstance == null) { // 双重校验，避免多个线程进入第一个 if 语句后，先后多次初始化
                    uniqueInstance = new Singleton();
                }
            }
        }
        return uniqueInstance;
    }
}
```

### 静态内部类

```java
public class Singleton {

    private Singleton() {

    }

    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getUniqueInstance() { // 只有当调用该方法时，才会触发初始化
        return SingletonHolder.INSTANCE;          // 同时由 JVM 提供了对线程安全的支持
    }
}
```

## Linux 常用命令

清单：
```bash
ls cat wc more less cd top cp mv rm pwd mkdir ps kill chmod grep
sed awk
```
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


## Question - shell
### Q1: 查找错误信息
用`shell`命令在日志文件里面查找错误信息

```bash
# -i 不区分大小写
cat [file.log] | grep -i --color error
```

### Q2: 查看机器资源
用`shell`命令查看机器资源使用情况

```bash
# -i 不展示闲置进程
top -i

# 查看磁盘
vmstat

# 查看监听端口
netstat -nltp

# 列出所有进程，一般搭配 `| grep` 使用
ps -ef
```

### Q3: 数据排序
用`shell`命令对文件里面的数据排序

```bash
# -r 倒序
sort [file]
```

## 面经参考

- https://www.nowcoder.com/discuss/192234?source_id=profile_create&channel=666 

- https://www.nowcoder.com/discuss/381058?type=all&order=time&pos=&page=1&channel=666&source_id=search_all

- https://www.nowcoder.com/discuss/477426?type=all&order=time&pos=&page=1&channel=666&source_id=search_all

- https://www.nowcoder.com/discuss/434392?type=all&order=time&pos=&page=1&channel=666&source_id=search_all

- https://www.nowcoder.com/discuss/382429?type=all&order=time&pos=&page=1&channel=666&source_id=search_all

- https://www.nowcoder.com/discuss/201562?type=post&order=time&pos=&page=1&channel=666&source_id=search_post

- https://www.nowcoder.com/discuss/410509?type=post&order=time&pos=&page=1&channel=666&source_id=search_post

- https://www.nowcoder.com/discuss/388680?type=post&order=time&pos=&page=1&channel=666&source_id=search_post

- https://www.nowcoder.com/discuss/417164?type=post&order=time&pos=&page=1&channel=666&source_id=search_post

- https://www.nowcoder.com/discuss/322029?type=post&order=time&pos=&page=1&channel=666&source_id=search_post

