---
Title: 旧书 Tech. Markdown 文档编辑规范及约定
Description: 旧书 Tech. Markdown 文档编辑规范及约定
ShowOnHome: false
---
在线查阅请访问 [旧书 Tech. Docs 官网](http://tech.jiu-shu.com)
# 文档注意事项
- 文件名不要出现‘.’
- 尽量使用英文文件夹；尽量使用英文文件名
- 文件夹和文件名避免使用空格，可以使用-作为连接符
- 文件的最前端定义的meta信息必须遵循如下格式

## 文档meta信息的格式
```
---
Title: Tech Docs
Description: Tech Docs collects all documents written by Joe.
ShowOnHome: false // 只用于根目录下面的文件，文件夹中的文件不能加这个属性。
---
```
# GIT相关问题应对
在linux上clone后，如果通过了chmod来修改文件夹及子目录的权限后，所有目录的文件git状态变成modified； 如果尝试用 `git diff` 查看会发现类似如下信息：

```shell
diff --git a/Dev-Ops/Centos-Common-Commands.md b/Dev-Ops/Centos-Common-Commands.md
old mode 100644
new mode 10075
```


以上是权限模型改变导致的，可以通过设置 `git config core.filemode false` 绕过这个问题：

# GITHUB Webhooks 使用
通过github的webhooks，可以对github的仓库在服务端的事件进行监听并且发送post请求。 可以在我们的web server端来接受post请求做出相应的操作。本repo设置了webhooks的请求，相应的push事件会触发[旧书 Tech. Docs 官网](http://tech.jiu-shu.com)上的文档的更新。
> webhooks最好使用了secret保护; 参考文档： https://developer.github.com/webhooks/securing/。