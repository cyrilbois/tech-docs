---
title:  Java Centos 环境
...

```
#### 1-jdk11  linux版本 jdk-11.0.7_linux-x64_bin.tar.gz
https://code.aliyun.com/kar/oracle-jdk/raw/3c932f02aa11e79dc39e4a68f5b0483ec1d32abe/jdk-11.0.7_linux-x64_bin.tar.gz

#### 2-jdk11 mac jdk-11.0.7_osx-x64_bin.tar.gz
https://code.aliyun.com/kar/oracle-jdk/raw/3c932f02aa11e79dc39e4a68f5b0483ec1d32abe/jdk-11.0.7_osx-x64_bin.tar.gz

#### 3-jdk11 windows jdk-11.0.7_windows-x64_bin.zip
https://code.aliyun.com/kar/oracle-jdk/raw/3c932f02aa11e79dc39e4a68f5b0483ec1d32abe/jdk-11.0.7_windows-x64_bin.zip

#### 4-jdk8 linux jdk-8u251-linux-x64.tar.gz
https://code.aliyun.com/kar/oracle-jdk/raw/3c932f02aa11e79dc39e4a68f5b0483ec1d32abe/jdk-8u251-linux-x64.tar.gz
#### 5-jdk8 mac  jdk-8u251-macosx-x64.dmg
https://code.aliyun.com/kar/oracle-jdk/raw/3c932f02aa11e79dc39e4a68f5b0483ec1d32abe/jdk-8u251-macosx-x64.dmg
#### 5-jdk8 windows jdk-8u251-windows-x64.exe
https://code.aliyun.com/kar/oracle-jdk/raw/3c932f02aa11e79dc39e4a68f5b0483ec1d32abe/jdk-8u251-windows-x64.exe
```


## 卸载OpenJDK
```
yum list java*
sudo yum -y remove java*
java -version
```

## 安装JDK
```
#### 创建安装目录
mkdir /usr/local/java/
#### 解压至安装目录
tar -zxvf jdk-8u361-linux-x64.tar.gz -C /usr/local/java/
```

### 设置环境变量

vim /etc/profile
在末尾添加
```
export JAVA_HOME=/usr/local/java/jdk1.8.0_361
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```
使环境变量生效`source /etc/profile`

添加软链接`ln -s /usr/local/java/jdk1.8.0_361/bin/java /usr/bin/java`

检查`java -version`


## 安装Maven
```
#### 创建安装目录
mkdir /usr/local/maven/
#### 解压至安装目录
tar -zxvf apache-maven-3.8.4-bin.tar.gz -C /usr/local/maven/
```
​
### 设置环境变量
​
vim /etc/profile
在末尾添加
```
export MVN_HOME=/usr/local/maven/apache-maven-3.8.4
export PATH=$PATH:$MVN_HOME/bin
```



