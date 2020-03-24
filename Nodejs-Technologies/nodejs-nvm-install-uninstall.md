---
title:  如何在Linux上安装Node.js
description: 如何在Linux上安装Node.js
...

转自： [https://blog.csdn.net/wh211212/article/details/53039286](https://blog.csdn.net/wh211212/article/details/53039286)

装Nodejs之前推荐优先安装nvm
参考上面链接， 去https://github.com/creationix/nvm 找到最新的版本。



## npm设置代理
```
npm config set registry https://registry.npm.taobao.org
npm config list #查看npm当前配置
```
