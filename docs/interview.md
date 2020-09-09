## 计算机网络

### HTTPS - 验证机制

[SSL/TLS协议四次握手](https://blog.csdn.net/odyyy/article/details/80256129)

![image-20200823155549071](https://tva1.sinaimg.cn/large/007S8ZIlgy1gi0sto1c02j30lf0dfajx.jpg)

1、首先什么是HTTP协议?

http协议是超文本传输协议，位于tcp/ip四层模型中的应用层；通过请求/响应的方式在客户端和服务器之间进行通信；但是缺少安全性，http协议信息传输是通过明文的方式传输，不做任何加密，相当于在网络上裸奔；容易被中间人恶意篡改，这种行为叫做中间人攻击；

2、加密通信：
为了安全性，双方可以使用对称加密的方式key进行信息交流，但是这种方式对称加密秘钥也会被拦截，也不够安全，进而还是存在被中间人攻击风险；
于是人们又想出来另外一种方式，使用非对称加密的方式；使用公钥/私钥加解密；通信方A发起通信并携带自己的公钥，接收方B通过公钥来加密对称秘钥；然后发送给发起方A；A通过私钥解密；双发接下来通过对称秘钥来进行加密通信；但是这种方式还是会存在一种安全性；中间人虽然不知道发起方A的私钥，但是可以做到偷天换日，将拦截发起方的公钥key;并将自己生成的一对公/私钥的公钥发送给B；接收方B并不知道公钥已经被偷偷换过；按照之前的流程，B通过公钥加密自己生成的对称加密秘钥key2;发送给A；
这次通信再次被中间人拦截，尽管后面的通信，两者还是用key2通信，但是中间人已经掌握了Key2;可以进行轻松的加解密；还是存在被中间人攻击风险；

3、解决困境：权威的证书颁发机构CA来解决；

- 制作证书：作为服务端的A，首先把自己的公钥key1发给证书颁发机构，向证书颁发机构进行申请证书；证书颁发机构有一套自己的公私钥，CA通过自己的私钥来加密key1,并且通过服务端网址等信息生成一个证书签名，证书签名同样使用机构的私钥进行加密；制作完成后，机构将证书发给A；

- 校验证书真伪：当B向服务端A发起请求通信的时候，A不再直接返回自己的公钥，而是返回一个证书；
说明：各大浏览器和操作系统已经维护了所有的权威证书机构的名称和公钥。B只需要知道是哪个权威机构发的证书，使用对应的机构公钥，就可以解密出证书签名；接下来，B使用同样的规则，生成自己的证书签名，如果两个签名是一致的，说明证书是有效的；
签名验证成功后，B就可以再次利用机构的公钥，解密出A的公钥key1;接下来的操作，就是和之前一样的流程了；

- 中间人是否会拦截发送假证书到B呢？
因为证书的签名是由服务器端网址等信息生成的，并且通过第三方机构的私钥加密中间人无法篡改； 所以最关键的问题是证书签名的真伪；

4、https主要的思想是在http基础上增加了ssl安全层，即以上认证过程；:

> https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/74


### IO模型

#### select

1. 每次调用，需要把`fd`数组从用户态拷贝到内存态，开销很大；
2. 同时需要在内存态遍历所有`fd`，当`fd`很多时开销也很大；
3. 文件描述符数量限制较大，默认是`1024`。

#### poll

1. `fd`部分与`select`类似；
2. 采用链表的形式，没有最大文件描述符数量的限制。


#### epoll

提供了三个函数：
- `epoll_create`：创建一个`epoll`句柄；
- `epoll_ctl`：注册要监听的事件类型；
- `epoll_wait`：等待事件的产生。

1. 在`epoll_ctl`函数中。每次注册新的事件到`epoll`句柄中时（在`epoll_ctl`中指定`EPOLL_CTL_ADD`），会把所有的`fd`拷贝进内核，而不是在`epoll_wait`的时候重复拷贝。`epoll`保证了每个`fd`在整个过程中只会拷贝一次。

2. `epoll`的解决方案不像`select/poll`一样每次都把`current`轮流加入`fd`对应的设备等待队列中，而只在`epoll_ctl`时把`current`挂一遍（这一遍必不可少）并为每个`fd`指定一个回调函数，当设备就绪，唤醒等待队列上的等待者时，就会调用这个回调函数，而这个回调函数会把就绪的`fd`加入一个就绪链表）。`epoll_wait`的工作实际上就是在这个就绪链表中查看有没有就绪的`fd`（利用`schedule_timeout()`实现睡一会，判断一会的效果，和`select`实现中的第7步是类似的）。

3. `epoll`所支持的`fd`上限是最大可以打开文件的数目，这个数字一般远大于`2048`,举个例子,在`1GB`内存的机器上大约是`10`万左右，具体数目可以`cat /proc/sys/fs/file-max`察看，一般来说这个数目和系统内存关系很大。

工作模式：

- `LT`：检测到描述符事件发生并且通知应用程序时，应用程序可以不立即处理，等下次调用还可以再次响应；

- `ET`：检测到描述事件发生并且通知应用程序时，应用程序必须立即处理，否则下次调用也不会响应。减少了`epoll`事件被重复触发的次数，效率比`LT`模式高。

> https://www.cnblogs.com/aspirant/p/9166944.html

## 数据结构与算法

### 堆排序

```java
public static void heapInsert(int[] arr, int i) { // 升序排序，最大堆
    // 插入到数组末尾，从下往上走，比父亲结点大就交换
    while (arr[i] > arr[(i - 1) / 2]) {
        swap(arr, i, (i - 1) / 2);
        i = (i - 1) / 2;
    }
}

// 将当前堆最大值取出后，重新调整成最大堆
public static void heapify(int[] arr, int size) {
    int cur = 0; // 根节点一定为 0
    int left = 2 * cur + 1;
    while (left < size) {
        // 判断左右孩子结点哪个大
        int max = (left + 1 < size && arr[left + 1] > arr[left]) ? left + 1 : left;
        // 判断最大的孩子结点和当前结点哪个大
        max = arr[cur] > arr[max] ? cur : max;
        if (max == cur) // 如果当前结点就是最大值，那么无需继续调整，终止即可
            return;
        swap(arr, cur, max);
        cur = max;
        left = 2 * cur + 1;
    }
}
```

### 快速排序

```java
private static void quickSort(int[] arr, int l, int r) {
    if (l < r) {
        // Math.random() [0,1) --> [0, r-l] + l --> [l, r]
        int random = (int) (Math.random() * (r - l + 1)) + l;
        swap(arr, random, r);
        int[] p = partition(arr, l, r);
        quickSort(arr, l, p[0]);
        quickSort(arr, p[1], r);
    }
}

private static int[] partition(int[] arr, int l, int r) {
    int less = l - 1;
    int more = r;
    while (l < more) {
        if (arr[l] < arr[r]) {
            swap(arr, ++less, l++);
        } else if (arr[l] > arr[r]) {
            swap(arr, --more, l);
        } else {
            l++;
        }
    }
    swap(arr, more, r);
    // 此时 more 的位置就是我们的锚点位置，返回其左右边界
    // 对于 2,1,4,5,3 而言，假设选取最后一位进行比较，那么比 3 小的都在左边，比 3 大的都在右边
    // 最终排成： ... less, 3 (more), more + 1, ... 
    return new int[]{less, more + 1};
}
```

## 分布式一致性

### 两阶段提交

- 准备阶段：协调者向所有参与者询问是否事务是否执行成功，而参与者返回执行结果给协调者；
- 提交阶段：如果所有参与者都执行成功了，那么协调者发送通知让参与者提交事务，否则回滚事务。

### 三阶段提交

// TODO

## 设计模式 — 单例模式

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

