---
title:  Spring Boot Cloud 之配置文件 application  vs  bootstrap
...

参考：
https://www.baeldung.com/spring-cloud-bootstrap-properties


## 术语
应用配置 - application
启动配置 - bootstrap


## 简介
application 是Spring Boot的配置文件； bootstrap 是spring cloud 引入的配置文件

bootstrap 配置文件优先加载， application优先级最低
 
 

## 什么时候应用配置

我们使用application.yml或application.properties 来配置应用程序上下文。

当Spring Boot应用程序启动时，它将创建不需要显式配置的应用程序上下文-它已经自动配置。但是，Spring Boot提供了不同的方法来覆盖这些属性。

我们可以在代码，命令行参数，ServletConfig初始化参数，ServletContext初始化参数，Java系统属性，操作系统变量和应用程序属性文件中覆盖这些内容。

要记住的重要一点是，与其他形式的覆盖应用程序上下文属性相比，这些应用程序属性文件的优先级最低。

我们倾向于对可以在应用程序上下文中覆盖的属性进行分组：

    核心属性（日志记录属性，线程属性）
    集成属性（RabbitMQ属性，ActiveMQ属性）
    Web属性（HTTP属性，MVC属性）
    安全属性（LDAP属性，OAuth2属性）
		
		
		
## 什么场景用启动配置文件 bootstrap
我们使用bootstrap.yml或bootstrap.properties 来配置bootstrap上下文。这样，我们将引导程序的外部配置和主上下文很好地分开了。

所述自举上下文负责加载从外部源配置属性和用于在本地外部配置文件解密属性。

当Spring Cloud应用程序启动时，它会创建一个引导 上下文。首先要记住的是，引导 上下文是主应用程序的父上下文。

要记住的另一个关键点是这两个上下文共享环境，这是任何Spring应用程序外部属性的来源。与应用程序上下文相反，引导程序上下文使用不同的约定来定位外部配置。

例如，配置文件的源可以是文件系统甚至是git存储库。这些服务使用它们的spring-cloud-config-client依赖性来访问配置服务器。

简而言之，配置服务器是我们访问应用程序上下文配置文件的起点。


## 总结
与Spring Boot应用程序相比，Spring Cloud应用程序具有引导程序上下文，该上下文是应用程序上下文的父级。尽管它们共享相同的Environment，但是它们具有不同的约定来查找外部配置文件。

引导上下文正在搜索bootstrap.properties或bootstrap.yaml文件，而应用程序上下文正在搜索application.properties或application.yaml文件。

并且，当然，引导上下文的配置属性在应用程序上下文的配置属性之前加载。





