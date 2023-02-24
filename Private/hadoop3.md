---
title:  Hadoop 3 笔记
showOnHome: false
...

## 问题收集
### CDH安装后，`su hdfs` 命令报错
报错`This account is currently not available.`， 原因是 hdfs 用户的shell是/sbin/nologin需修改成/bin/bash。 

```
[root@bigdata-3 ~]# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
...
hdfs:x:994:990:Hadoop HDFS:/var/lib/hadoop-hdfs:/sbin/nologin
...
```
编辑/etc/passwd 改成 `hdfs:x:994:990:Hadoop HDFS:/var/lib/hadoop-hdfs:/bin/bash` 保存退出重新登录即可。

### CDH 安装后的hadoop的jar的目录

`/opt/cloudera/parcels/CDH/jars`