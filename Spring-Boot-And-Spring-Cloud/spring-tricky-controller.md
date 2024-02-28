---
title:  Tricky Part Of Spring Mapping
description: Spring 中的Controller需要注意的地方
...

记录Spring应用中Controller容易出问题的地方

## RequestMapping 位置

![spring-controller-requestmapping-correct.jpg](http://tech.icoding.tech/Spring-Boot-And-Spring-Cloud/spring-controller-requestmapping-correct.jpg)

非简单的path，比如： robots.txt， test.html 注解在class层，无法正常工作。

![spring-controller-requestmapping-incorrect.jpg](http://tech.icoding.tech/Spring-Boot-And-Spring-Cloud/spring-controller-requestmapping-incorrect.jpg)

## 静态路径不适合restful接口
下面的restful的接口的路径加上后缀`.html`后就会报错：`Resolved exception caused by handler execution: org.springframework.web.HttpMediaTypeNotAcceptableException: Could not find acceptable representation` 

![mapping-path-tricky-rest.PNG](http://tech.icoding.tech/Spring-Boot-And-Spring-Cloud/mapping-path-tricky-rest.PNG)
