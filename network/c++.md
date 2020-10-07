# C++高频考点梳理

## C++ 基础

- `const`、`define`的联系与区别
- 指针和引用的区别
- 堆和栈的区别
- 构造函数、析构函数
- `new、delete` 和 `malloc、free` 的区别
- **深拷贝和浅拷贝**
- 友元函数、友元类
- **`static`**的用法与意义
- 内联函数
- 继承、虚继承、钻石继承问题
- 同名覆盖问题
- 虚函数表、虚指针、虚函数（实现原理）、纯虚函数
- 接口（实现原理）、多态
- 重写、重载
- 函数重载、运算符重载
- 流类库和文件
- 为什么C++没有实现垃圾回收？

## 进阶
- 内存管理
- 函数模板、类模板
- C++ 中对于异常的处理
- 对于继承和多态底层的理解
- **对于 `virtual` 底层的理解**
- 智能指针有哪些？`<scoped_ptr/shared_ptr/weak_ptr>` 这三个是最核心的智能指针，理解清楚智能指针的本质是，内存的申请与释放全部交给了对象管理，以避免人为疏忽，造成内存泄露
- `C11`新特性
- `string`底层实现

## STL
- `unordered_map`和`map`的区别、底层实现
- `vector`的底层实现
- `set、map`和`vector`的插入复杂度
- 考察自动扩容的原理
- **对于迭代器、空间配置器的理解**，(比如：一级空间配置器、二级空间配置器的运用场合分别是什么？一二级空间配置器的本质是什么，如何用内存池去管理？所存在的问题又有哪些，源码又是如何实现的等等)

## 去哪找答案？
- [C++ Primer](https://book.douban.com/subject/25708312/)
- [Effective C++](https://book.douban.com/subject/5387403/)
- [More Effective C++](https://book.douban.com/subject/5908727/)
- [深度探索C++对象模型](https://book.douban.com/subject/10427315/)
- [C++ 沉思录](https://book.douban.com/subject/2970056/)
- [STL 源码剖析](https://book.douban.com/subject/1110934/)


## 拓展


### Linux（进阶）
> C++ 路线对于`Linux`的掌握程度要求更高

- Linux 进程环境：僵尸进程、孤儿进程、守护进程、进程组、会话、前台进程组、后台进程组
- Linux 进程七大通信方式：`signal、file、pipe、shm、sem、msg、socket`
- Linux 线程：互斥量、锁机制、条件变量、信号量、读写锁
- Linux 下并发模型：多进程、多线程、线程池
- **Linux 下 `I/O` 复用：`select、poll、epoll` 高并发**
- Linux 网络编程
- 静态库和动态库

### Linux 内核源码剖析（进阶）

> 对于 Linux 内核源码，可以先看 `Linux` 内核的设计与实现，了解清楚每部分的构造与原理，前期多看书、多看相关视频，对一些源码的解读，到一定程度，最好拿到 `Linux 2.6` 版本内核源码，我是用 `Source Insight` 工具辅助分析源码的。
> 
> 这个工具对于源码的分析特别友好，很快定位变量、追踪函数，其实重点应该放在内核文件系统与内核数据结构的实现上面，多看看源码是如何实现的，比如：内核链表的源码实现，真的是一种非常独特的思想，没有看的可以去看看。

### 推荐书籍

- [UNIX 环境高级编程](https://book.douban.com/subject/25900403/)
- [UNIX网络编程 卷1：套接字联网API](https://book.douban.com/subject/26434583/)
- [Linux 内核设计与实现](https://book.douban.com/subject/6097773/)
- [深入理解LINUX内核](https://book.douban.com/subject/2287506/)


###  参考文章：
- 编程剑谱：https://mp.weixin.qq.com/s/kxBpVoQVUCl04dh1tB9tzg


