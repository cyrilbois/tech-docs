---
title:  mysql 学习
showOnHome: false
...

这里是临时的学习比较的保存处；完成后可以移到对应的目录。

## 知识点
#### 查询缓存

查询缓存机制是key-value的模式， 查询语句为key，结果作为value存于内存，再次查询通过key来寻找， 但是实际的意义不是很大。 

mysql8已经没有了查询缓存。 大部分云上的数据库默认也禁用了查询缓存。  （因为对一个表的任意一行的更新都会导致缓存失效，所以大部分场景查询缓存并没有意义）

通过命令 `show variables where variable_name like 'query_cache%';` 可以查看`查询缓存`的设置。 query_cache_type为OFF的时候说明禁用了。




