---
title:  macOS  系统使用参考
...

## 查看端口占用
```
lsof -nP -i4TCP:$PORT | grep LISTEN
```

## 停掉 apache2
Mac 自带了apache2

## 控制自动启动项
使用[launchrocket](https://github.com/jimbojsb/launchrocket)控制自动启动项。安装之后就可以移步系统设置页面，找到launchrocket。 

