---
title: Shell 脚本学习笔记
description: Shell 学习笔记
...

## 杂记
### 括号的使用说明
参考：[Double parenthesis with and without dollar](http://stackoverflow.com/questions/31255699/double-parenthesis-with-and-without-dollar)

 - $(...) means execute the command in the parens and return its stdout. 

```
$ echo "The current date is $(date)"
The current date is Mon Jul  6 14:27:59 PDT 2015
```
- (...) means run the commands listed in the parens in a subshell. Example:

```
$ a=1; (a=2; echo "inside: a=$a"); echo "outside: a=$a"
inside: a=2
outside: a=1
```

- $((...)) means perform arithmetic and return the result of the calculation. Example:

```
$ a=$((2+3)); echo "a=$a"
a=5
```

- ((...)) means perform arithmetic, possibly changing the values of shell variables, but don't return its result. Example:

```
$ ((a=2+3)); echo "a=$a"
a=5
```

- ${...} means return the value of the shell variable named in the braces. Example:

```
$ echo ${SHELL}
/bin/bash
```

- {...} means execute the commands in the braces as a group. Example:

```
$ false || { echo "We failed"; exit 1; }
We failed
```


## 有用脚本收集

### 文件读取
读取文件目录的所有文件，按行读取每个文件，判断行文字是否包含特定字符串；如果包含，通过特殊字符来split并输出想要的值。

```
#!/bin/sh
for filename in /home/okchem/mysqlbackup/*.sql; do
    while IFS= read line
	do
		# display $line or do somthing with $line
		if [[ $line == *"/ocf/"* ]]; then
			SUBSTRING=$(echo $line| cut -d'`' -f 2) # 用'`'来拆分,输出数组第二个
			echo $SUBSTRING
		fi
	done <"${filename}.sql"
done
```

### 自动build 发布到远程机器
```
#！ /bin/bash

currentPath=$(pwd)
echo "当前文件夹路径: $currentPath"


# 拉去git上最新数据
echo "start pull updates from remote"
git pull

echo "Start Building......"
npm run build

sftp root@10.3.69.41 <<EOT
put -r $currentPath/dist /home/nantong/0_125_Umi/
quit
EOT
```

### 简易发布Java Maven工程
```
#！ /bin/bash

## 修改这一个变量即可，修改为代码clone的那个文件夹
NAME=websocket-service

# 找出进程ID 
tokill=`ps -ef | grep java | grep ${NAME} | awk '{print $2}'`
kill -9 $tokill

## 拉取代码、构建、复制jar、运行
cd ${NAME}
git pull
mvn clean package -Dmaven.test.skip=true
cd ..
mv ${NAME}/target/*.jar ./${NAME}.jar
nohup java -Dspring.profiles.active=dev -jar ${NAME}.jar > ./app.log 2>&1 &
```

### 远程发布脚本

##### 远程运行脚本
`deploy.sh`

```
#！ /bin/bash

## 修改这一个变量即可，修改为代码clone的那个文件夹
NAME=websocket-service

# 找出进程ID 
tokill=`ps -ef | grep java | grep ${NAME} | awk '{print $2}'`
kill -9 $tokill

# 定位到当前脚本所在的目录
cd "$(dirname "$0")"
nohup java -Xms256m -Xmx512m  -jar ${NAME}.jar > ./app.log 2>&1 &
```
##### 本地脚本
`remote-deploy.sh`
```
#！ /bin/bash

## 修改这一个变量即可，修改为代码clone的那个文件夹
NAME=websocket-service
scp websocket-service.jar root@remoteserver:/opt/icviews/websocket/ 
ssh root@remoteserver '/opt/icviews/websocket/deploy.sh'
```

## 参考网站
https://www.shellscript.sh/
https://www.shellscript.sh/quickref.html
