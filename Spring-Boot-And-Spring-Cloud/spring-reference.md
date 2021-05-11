---
title:  Spring  收集
...


## 任务
集群和多个实例情况下如何保证只有一个实例运行定时任务
https://www.baeldung.com/shedlock-spring 


Spring Boot Docker化 https://spring.io/guides/topicals/spring-boot-docker/


## 问题整理

#### Transactional 不生效原因
主要有两个主要原因：
 - @Transactional 声明不在公共方法上
 - 方法调用在同一个类实例

参考： https://ignaciosuay.com/why-is-spring-ignoring-transactional/






