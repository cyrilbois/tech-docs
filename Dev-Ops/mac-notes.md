---
title:  macOS  系统使用参考
...


## Java 环境
这里找`/Library/Java/JavaVirtualMachines/` 参考：https://blog.csdn.net/a158123/article/details/79684499。  

## 环境变量配置

设置JAVA_HOME
记得切换成root用户(sudo -i)或者给指令添加sudo

#### 临时有效（重启后失效）
编辑.bash_profile文件：vim ~/.bash_profile
添加以下内容：
```
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_162.jdk/Contents/Home
export PATH=$JAVA_HOME/bin:$PATH
```

使修改的文件生效：source ~/.bash_profile
#### 永久有效
修改文件操作权限：chmod 773 /etc/profile
编辑/ect/profile文件：vim /etc/profile
添加以下内容：
```
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_162.jdk/Contents/Home
export PATH=$JAVA_HOME/bin:$PATH
```
使修改的文件生效：source /etc/profile 

## 查看端口占用
```
lsof -nP -i4TCP:$PORT | grep LISTEN
```

## 停掉 apache2
Mac 自带了apache2

## 控制自动启动项
使用[launchrocket](https://github.com/jimbojsb/launchrocket)控制自动启动项。安装之后就可以移步系统设置页面，找到launchrocket。 

## brew 使用
用来管理各种服务的启停
* 安装    `brew install mysql`
* 启动并且注册开机自启`brew services start mysql`
* 启动，不注册自启`brew services run mysql`
* 停止，已经注册过的话会取消`brew services stop mysql`
* 重启，并且注册开机自启`brew services restart mysql`
* 查看brew安装的服务状态 `brew services list`
* 清除已经卸载应用的无用配置`brew services cleanup`
* 查看安装的服务的信息 `brew info nginx`
* 注册服务
注册开机自启后，会创建.plist文件，该文件包含版本信息、编码、安装路径、启动位置、日志路径等信息，取消自启动后会自动删除，执行 `brew services list` 可以看到各个服务该文件的存放位置
* .plist存放目录  开机自启存放目录`/Library/LaunchDaemons/`
* 用户登录后自启存放目录`~/Library/LaunchDaemons/`

## brew update
Homebrew 镜像使用帮助
注:该镜像是 Homebrew 的 formula 索引的镜像（即 brew update 时所更新内容）。本镜像站同时提供 Homebrew 二进制预编译包的镜像，请参考 Homebrew bottles 镜像使用帮助。

### 替换现有上游
```
git -C "$(brew --repo)" remote set-url origin https://mirrors.ustc.edu.cn/brew.git

git -C "$(brew --repo homebrew/core)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-core.git

git -C "$(brew --repo homebrew/cask)" remote set-url origin https://mirrors.ustc.edu.cn/homebrew-cask.git

brew update
```
### 复原
(感谢Snowonion Lee提供说明)

```
git -C "$(brew --repo)" remote set-url origin https://github.com/Homebrew/brew.git

git -C "$(brew --repo homebrew/core)" remote set-url origin https://github.com/Homebrew/homebrew-core.git

git -C "$(brew --repo homebrew/cask)" remote set-url origin https://github.com/Homebrew/homebrew-cask.git

brew update
```


### Excel 乱码处理
```
iconv -f UTF8 -t GB18030 20220908.csv > 20220908-1.csv
```

## 网络代理 抓包工具

Charles

>抓包https网站需要设置 help/SSL Proxying/Install Charles Root Certificate,   Proxy/ SSL Proxying Setting， 参考https://www.youtube.com/watch?v=g1IQTuiPWZQ