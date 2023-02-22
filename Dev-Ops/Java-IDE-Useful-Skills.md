---
title:  Java 开发工具使用技巧收集
description: Java 开发工具使用技巧收集， 让开发效率更高一些
...



# Eclipse
## Eclipse 提速
https://blog.csdn.net/leolu007/article/details/53541641

Remote System Explorer Operation卡死
第一步：Eclipse -> Preferences -> General -> Startup and Shutdown.不要勾选 RSE UI. 
第二步：Eclipse -> Preferences -> Remote Systems. 取消勾选 Re-open Remote Systems view to previous state.
 
## 在Eclispe中使用Git 命令
参考： https://blog.csdn.net/wu_cai_/article/details/71637199  （建议选择git-bash）
直接在eclipse中使用并不是很方便，但是可以快速打开git-bash. 配置方式参考上面。

## Eclipse maven spring boot 项目的诡异问题 unkown error
降低spring boot版本
https://www.youtube.com/watch?v=BLkU9Fr741E


## 调试
### 断点出用于打印信息的代码收集
#### Request的Header信息
```
java.util.Enumeration headerNames = req.getHeaderNames();
while(headerNames.hasMoreElements()) {
  String headerName = (String)headerNames.nextElement();
  System.out.println("Header Name - " + headerName + ", Value - " + req.getHeader(headerName));
} 
```


## IntelliJ IDEA  代码错误
通过`Analyze -> Inspect Code` 来分析代码的错误， 这个在用JPA 的Query的时候非常用用， 方便代码重构。






