---
Title: 向NPM仓库发布自己的Package
Description: 阐述如何使用npm发布自己的Package
---
阐述如何使用npm发布自己的Package
# Publish

参考[官方指导](https://docs.npmjs.com/getting-started/publishing-npm-packages)

# 问题收集

## 问题1 Incorrect username or password
https://github.com/npm/npm/issues/6545
### workaround
删除文件.npmrc 后重试
> 不同系统的文件位置不同， windows系统在当前用户的目录下，比如：`C:\Users\Administrator\`

## 问题2 you must verify your email before publishing a new package
也许当时注册时候没有验证邮箱， 你需要重新验证下，去npm官网发一个验证到邮箱， 然后照着做