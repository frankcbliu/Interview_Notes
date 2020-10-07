# MySQL 高频考点梳理

> 一般考察 `Innodb`引擎。

## 原理
- `MySQL`三范式
- **`MySQL`索引的底层结构**
- `B+`树和`B`树的区别，为什么`Innodb`用`B+`树？
- `MySQL`有几种存储引擎，有什么区别？
- 缓冲池/写缓冲（拓展，可不看）
- 当我们输入一条 `SQL` 查询语句时，发生了什么？

## 事务
- 事务的`ACID`分别是什么？
- `MySQL`事务有几种隔离级别，分别是解决什么问题的？
- `MySQL`默认使用哪种隔离级别？是否解决了幻读问题，如何解决的？
- `MySQL`是如何实现事务的？

## 应用
- 锁（行锁，表锁，页级锁，意向锁，读锁，写锁，悲观锁，乐观锁，以及加锁的`sql`语句）
- 索引的优化方式有哪些？（联合索引、最左匹配原则、覆盖索引、索引下推）（这几个都好好看）
- SQL如何进行性能优化？（`explain`、慢查询日志）
- 分库分表（水平切分、垂直切分）
- `MySQL`主从复制（问得相对少一些）

## 去哪找答案？

- [《MySQL技术内幕:InnoDB存储引擎》——姜承尧](https://book.douban.com/subject/24708143/)
- [【专栏】《MySQL实战45讲》——林晓斌](https://time.geekbang.org/column/intro/139)
- [《高性能MySQL》](https://book.douban.com/subject/23008813/)
- [Google](http://www.google.com)、[博客园](https://zzk.cnblogs.com/s?w=)、[公众号](https://weixin.sogou.com/)、[CSDN](https://www.csdn.net/)
