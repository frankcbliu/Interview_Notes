# Java 高频考点梳理

## 基础
- `String`的不变性如何理解
- 实现`String`的`equals`方法
- `StringBuilder`和`StringBuffer`，哪个是线程安全的，如何实现线程安全的？
- `Long`和`Integer`的缓冲机制
- 自动拆箱/自动装箱
- `volatile`修饰有什么用
- 反射

## 集合类
- **`HashMap`的源码实现**（`1.7/1.8`都要看，差别比较）
- **`HashMap`的`put`方法、扩容方法**
- **`HashMap`的初始容量为什么要是`2`的幂？**
- `HashMap`如何解决哈希冲突？
- **`ConcunrrentHashMap`的源码实现**（`1.7/1.8`都要看，同时要跟`HashMap`比较）
- 快速失败/安全失败
- `ArrayList`的源码分析，重要方法的实现步骤
- `LinkedList`的源码分析，重要方法的实现步骤

## 并发
- **乐观锁与悲观锁的区别**
- `Java`中如何实现乐观锁/悲观锁？
- `CAS`实现
- `ABA`问题如何解决？
- `AQS`
- `CountDownLatch`、`CyclicBarrier`、`Semaphore`
- `ThreadLocal`
- `ReentrantLock`底层实现
- `synchronized`原理
- 锁优化

## 线程池
- 如何实现一个线程池？（也就是线程池底层是如何实现的）
- 核心线程数、最大线程数的区别
- 线程池的拒绝策略

## JVM
- `JAVA`运行时数据区如何划分？
- 双亲委派模型
- 类加载机制
- 垃圾回收算法有哪些？
- 垃圾回收器有哪些？

## 去哪找答案？
- [《Java核心技术·卷 I》](https://book.douban.com/subject/26880667/)
- [《Java编程思想》](https://book.douban.com/subject/2130190/)
- [《深入理解JAVA虚拟机》——周志明](https://book.douban.com/subject/34907497/)
- [《实战Java高并发程序设计》](https://book.douban.com/subject/26663605/)
- [Google](http://www.google.com)、[博客园](https://zzk.cnblogs.com/s?w=)、[公众号](https://weixin.sogou.com/)、[CSDN](https://www.csdn.net/)

## 拓展

### Spring
- **`IOC`、`AOP`原理**
- `Bean`的创建方式、生命周期
- Spring 框架中用到了哪些设计模式
- 事务的实现方式和实现原理

### 分布式
- **`CAP`原理和`BASE`理论**
- 分布式一致性算法(`Raft`、`Paxos`等)
- 分布式事务
- 本地消息表
- 分布式锁

## 推荐书籍
- [Spring实战](https://book.douban.com/subject/26767354/)