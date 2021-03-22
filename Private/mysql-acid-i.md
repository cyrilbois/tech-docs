---
title:  Mysql 之事务隔离级别
...

**ACID**

* A - Atomicity  原子性  （通过事务实现）
* C - Consistency 一致性
* I -  Isolation   隔离性   （用于隔离并发的事务）
* D - Durability  持久性

### 四种隔离级别
* 读未提交(Read Uncommitted) 
* 读已提交(Read Committed) 
* 可重复读(Repeatable Read)  mysql默认事务隔离级别
* 串行化(Serializable)。 
 
隔离性逐步加强。


### 可重复读
可重复读的事务隔离级别， 事务启动的时候数据库会创建独立的视图，这个事务的操作都在这个视图中进行。

MVCC (Multi-Version Concurrency Control) 多版本并发控制。 

> 可重复读保证了一个事物中读到的数据的一致性，但是引入了视图为回滚带来了复杂。回滚日志必须在确定了没有更早的read-view的时候才可以删除。为了防止太多的回滚日志占用过多的空间，要避免长事务。



