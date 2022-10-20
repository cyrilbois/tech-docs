---
title:  Redis 命令
...


### Redis Cli命令
```
auth 密码 //密码登录
keys [pattern]  //列出键， 如： keys wx:session:* 列出 wx:session:开头的key
select [database] //选择库 ， 如： select 2
Flushdb  //清空当前的库
```


