---
title:  Restful URL 规范
...

很早的时候接触Restful的时候，觉得其标准很不错， 但是在实际项目中发现， 大部分程序员都没有类似的意识，甚至很资深的工程师。 后来发现可能是受国内大厂影响。 很多大厂的开放API的接口都没有遵循Restful的标准，甚至很多都将接口的URL定义为一个，通过POST的请求体不同来区分不同的接口。

但是针对新的开发项目来讲，一套标准的规范还是能增加可读性，提供协同效率。

## 资源路径: URL

> 一个好的URL具有很强可读性的，具有自描述性

```
/版本号/资源路径
/v1/users/{userId}  //URL 可以定位到具体的用户
/v1/users?[&keyword=xxx][&enable=1][&offset=0][&limit=20]  // URL 可以定位到一个用户集合
```

#### 版本号
命名版本号可以解决版本不兼容问题，在设计 RESTful API 的一种实用的做法是使用版本号。一般情况下，我们会在 url 中保留旧版本号，并同时兼容多个版本

```
【GET】 /v1/users/{userId} // 版本 v1 的获取id为{userId}的用户
【GET】 /v2/users/{user_id} // 版本 v2 的获取id为{userId}的用户
```
#### 资源路径
URI 严格意义不能包含动词，只能是名词（命名名词的时候，要使用小写、数字及下划线来区分多个单词）。

资源的路径应该从根到子依次如下:

/{resources}/{resource_id}/{sub_resources}/{sub_resource_id}/{sub_resource_property}

【POST】 /v1/users/{user_id}/roles/{role_id} // 添加用户的角色

有的时候，当一个资源变化难以使用标准的 RESTful API 来命名，可以考虑使用一些特殊的 actions 命名。

/{resources}/{resource_id}/actions/{action}

【PUT】 /v1/users/{user_id}/password/actions/modify // 密码修改

3、请求方式
【GET】 /users # 查询用户信息列表

【GET】 /users/1001 # 查看某个用户信息

【POST】 /users # 新建用户信息

【PUT】 /users/1001 # 更新用户信息(全部字段)

【PATCH】 /users/1001 # 更新用户信息(部分字段)

【DELETE】 /users/1001 # 删除用户信息

【PATCH】一般不用，用【PUT】

4、查询参数
RESTful API 接口应该提供参数，过滤返回结果。

【GET】 /{version}/{resources}/{resource_id}?offset=0&limit=20

5、响应参数
JSON格式（code、data、msg）

6、状态码
使用适合的状态码很重要，而不应该全部都返回状态码 200

状态码，可根据以下标准按照项目扩展自身状态码：

200~299段 表示操作成功：

200 操作成功，正常返回

201 操作成功，已经正在处理该请求

300~399段 表示参数方面的异常

300 参数类型错误

301 参数格式错误

302 参数超出正常取值范围

303 token过期

304 token无效

400~499段 表示请求地址方面的异常：

400 找不到地址

500~599段 表示内部代码异常：

500 服务器代码异常

作者：灰色的秩序
链接：https://www.jianshu.com/p/a6d9cf64e954
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。


