---
title:  六大设计原则
...


### 依赖倒置原则
#### 原文定义：
 > High-level modules should not depend on low-level modules. Both should depend on abstractions.  Abstractions should not depend on details. Details should depend on abstractions.


#### 个人体会
依赖倒置原则本质也是为了降低软件耦合度；其独特点在于：
1. 开发顺序开发由上至下；由上层模块定义接口(抽象)，底层模块实现接口
2. 高层被重用
3. 底层具体实现一般是通过框架注入到上层； （这也是依赖反转Ioc的一种实现方式）


依赖倒置原则有时候被称为好莱坞原则，那是因为依赖倒置都是依赖框架来注入具体的实现， 有框架来调用底层实现，底层实现不需要主动和上层打交道。


### 接口隔离原则
接口隔离原则保证调用方只看到自己需要的方法。

#### 案例优化
cache 实现类中有四个方法，其中 put get delete 方法是需要暴露给应用程序的，rebuild 方法是需要暴露给系统进行远程调用的。如果将 rebuild 暴露给应用程序，应用程序可能会错误调用 rebuild 方法，导致 cache 服务失效。按照接口隔离原则：不应该强迫客户程序依赖它们不需要的方法。也就是说，应该使 cache 类实现两个接口，一个接口包含 get put delete 暴露给应用程序，一个接口包含 rebuild 暴露给系统远程调用。从而实现接口隔离，使应用程序看不到 rebuild 方法。

![interface-segregation-principle.png](http://tech.jiu-shu.com/Software-Design/interface-segregation-principle.png)

#### 符合接口隔离原则的类图

![interface-segregation-principle-cache.png](http://tech.jiu-shu.com/Software-Design/interface-segregation-principle-cache.png)


