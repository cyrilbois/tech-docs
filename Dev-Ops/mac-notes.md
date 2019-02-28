---
title:  macOS High Sierra 系统使用参考
...

## 查看端口占用
```
lsof -nP -i4TCP:$PORT | grep LISTEN
```

## 停掉 apache2
Mac 自带了apache2

