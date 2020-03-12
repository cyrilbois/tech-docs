---
title:  docker 使用
...

## docker

## docker-compose安装
https://docs.docker.com/compose/install/

```
sudo curl -L "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose
```
> 可能出现网速太慢的情况，手动下载sftp上去, `mv docker-compose-Linux-x86_64 /usr/local/bin/docker-compose`
> 
 
删除`sudo rm /usr/local/bin/docker-compose`


## docker-compose 开机启动
一般来说设置`restart: always`即可。
https://stackoverflow.com/questions/43671482/how-to-run-docker-compose-up-d-at-system-start-up
 


