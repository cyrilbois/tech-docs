---
title:  Java JVM
...

 

一些重要的JVM 参数https://www.baeldung.com/jvm-parameters


## 关于堆内存
* -Xms 最小堆内存
* -Xmx 最大堆内存 

启动时不显示设置的话，JVM 默认Xmx取最大系统内存的1/4。  JVM能识别容器的内存限制，如果部署容器中未配置， 则最大堆内存是容器最大内存的限制的1/4。 因此在容器化部署的时候应该明确设置这两个参数。 （基于JDK8 和 腾讯云TKE测试）

运行命令示例: `nohup java -Xms256m -Xmx512m -Dspring.profiles.active=prod -jar app.jar > ./app.log 2>&1 &`