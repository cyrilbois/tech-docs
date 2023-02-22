---
title:  构建上海时区JDK镜像
...

待完善


https://blog.csdn.net/wangooo/article/details/109148640

```
docker run -itd --name openjdk8-en openjdk:8-jdk-alpine
docker exec -it openjdk8-en sh
查看时间： date
```

```
docker run -itd --name openjdk8-zh openjdk8-zh:latest
docker exec -it openjdk8-zh sh
查看时间： date
```
