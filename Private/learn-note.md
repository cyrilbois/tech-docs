---
title:  mysql 学习
showOnHome: false
...

这里是临时的学习比较的保存处；完成后可以移到对应的目录。


## SQL 执行过程

连接器 -> 分析器 -> 优化器 -> 执行器

## 查询缓存

查询缓存机制是key-value的模式， 查询语句为key，结果作为value存于内存，再次查询通过key来寻找， 但是实际的意义不是很大。 

mysql8已经没有了查询缓存。 大部分云上的数据库默认也禁用了查询缓存。  （因为对一个表的任意一行的更新都会导致缓存失效，所以大部分场景查询缓存并没有意义）

通过命令 `show variables where variable_name like 'query_cache%';` 可以查看`查询缓存`的设置。 query_cache_type为OFF的时候说明禁用了。

## redo log 和 bin log

WAL - Write Ahead Logging  先写日志再写磁盘。 redo log是WAL思想的实现，其目的是为了提高性能及响应速速。 WAL是异步更新磁盘的机制，所以redo log必须做到crash-safe。 

bin log 用于误操作后的数据库恢复，也可用于主从复制。

> redo log 不是用来实现crash-safe，是用来实现WAL的，而WAL会带来crash-unsafe的隐患，所以redo log需要解决这个问题。


 - redo log 是 InnoDB 引擎特有的；binlog 是 MySQL 的 Server 层实现的，所有引擎都可以使用。

 - redo log 是物理日志，记录的是“在某个数据页上做了什么修改”；binlog 是逻辑日志，记录的是这个语句的原始逻辑，比如“给 ID=2 这一行的 c 字段加 1 ”。

 - redo log 是循环写的，空间固定会用完；binlog 是可以追加写入的。“追加写”是指 binlog 文件写到一定大小后会切换到下一个，并不会覆盖以前的日志。

```
   1. redo log的概念是什么? 为什么会存在.
   2. 什么是WAL(write-ahead log)机制, 好处是什么.
   3. redo log 为什么可以保证crash safe机制.
   4. binlog的概念是什么, 起到什么作用, 可以做crash safe吗?
   5. binlog和redolog的不同点有哪些?
   6. 物理一致性和逻辑一直性各应该怎么理解?
   7. 执行器和innoDB在执行update语句时候的流程是什么样的?
   8. 如果数据库误操作, 如何执行数据恢复?
   9. 什么是两阶段提交, 为什么需要两阶段提交, 两阶段提交怎么保证数据库中两份日志间的逻辑一致性(什么叫逻辑一致性)?
  10. 如果不是两阶段提交, 先写redo log和先写bin log两种情况各会遇到什么问题?
```

```
1. redo log是重做日志。主要用于MySQL异常重启后的一种数据恢复手段，确保了数据的一致性。归根到底是MySQL为了实现WAL机制的一种手段。因为MySQL进行更新操作，为了能够快速响应，所以采用了异步写回磁盘的技术，写入内存后就返回。但是会存在crash后内存数据丢失的隐患，而redo log具备crash safe能力。为了什么redo log 可以提高性能？ Redolog是顺序写，并且可以组提交，还有别的一些优化，收益最大是是这两个因素。 
2. WAL机制是写前日志，也就是MySQL更新操作后在真正把数据写入到磁盘前先记录日志。好处是不用每一次操作都实时把数据写盘，就算crash后也可以通过redo log重放恢复，所以能够实现快速响应SQL语句。
3. 因为redo log是每次更新操作完成后，就一定会写入的，如果写入失败，这说明此次操作失败，事务也不可能提交。redo log内部结构是基于页的，记录了这个页的字段值变化，只要crash后读取redo log进行重放就可以恢复数据。（因为redo log是循环写的，如果满了InnoDB就会执行真正写盘）
4. bin log是归档日志，属于MySQL Server层的日志。可以起到全量备份的作用。当需要恢复数据时，可以取出某个时间范围内的bin log进行重放恢复。但是bin log不可以做crash safe，因为crash之前，bin log可能没有写入完全MySQL就挂了。所以需要配合redo log才可以进行crash safe。
5. bin log是Server层，追加写，不会覆盖，记录了逻辑变化，是逻辑日志。redo log是存储引擎层，是InnoDB特有的。循环写，满了就覆盖从头写，记录的是基于页的物理变化，是物理日志，具备crash safe操作。
6. 前者是数据的一致性，后者是行为一致性。（不清楚）
7. 执行器在优化器选择了索引后，调用InnoDB读接口，读取要更新的行到内存中，执行SQL操作后，更新到内存，然后写redo log，写bin log，此时即为完成。后续InnoDB会在合适的时候把此次操作的结果写回到磁盘。
8. 数据库在某一天误操作，就可以找到距离误操作最近的时间节点前的bin log，重放到临时数据库里，然后选择当天误删的数据恢复到线上数据库。
9. 两阶段提交就是对于三步操作而言：1.prepare阶段 2. 写入bin log 3. commit
redo log在写入后，进入prepare状态，然后bin log写入后，进入commit状态，事务可以提交。
如果不用两阶段提交的话，可能会出现bin log写入之前，机器crash导致重启后redo log继续重放crash之前的操作，而当bin log后续需要作为备份恢复时，会出现数据不一致的情况。所以需要对redo log进行回滚。
如果是bin log commit之前crash，那么重启后，发现redo log是prepare状态且bin log完整（bin log写入成功后，redo log会有bin log的标记），就会自动commit，让存储引擎提交事务。
10.先写redo log，crash后bin log备份恢复时少了一次更新，与当前数据不一致。先写bin log，crash后，由于redo log没写入，事务无效，所以后续bin log备份恢复时，数据不一致。
```








