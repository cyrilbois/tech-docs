---
title:  Nodejs 一些积累
...

## 如何在Linux上安装Node.js
转自： [https://blog.csdn.net/wh211212/article/details/53039286](https://blog.csdn.net/wh211212/article/details/53039286)

装Nodejs之前推荐优先安装nvm
参考上面链接， 去https://github.com/creationix/nvm 找到最新的版本。



## npm设置代理
```
npm config set registry https://registry.npm.taobao.org
npm config list #查看npm当前配置
```

## 后台运行
针对一些没有提供后台运行支持的工具， 可以使用forever来运行， 比如用来mock 接口的json-server
```
forever start /root/.nvm/versions/node/v10.12.0/bin/json-server --watch /home/mockdata/db.json --routes /home/mockdata/routes.json --port 3001 --host 0.0.0.0
```