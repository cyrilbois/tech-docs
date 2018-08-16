---
title:  Elastcisearch Restful API
description: Elastcisearch 常用的 Restful API
...

# Elastcisearch

## 索引API
### 创建索引
```
curl -XPUT "http://192.168.1.99:9200/test" //创建test的索引
```
```
{
   "ok": true,
   "acknowledged": true
}
```
### 删除索引
```
curl -XDELETE "http://192.168.1.99:9200/test" // 删除索引
```
```
{
   "ok": true,
   "acknowledged": true
}
```
### 创建索引并设置Type和Mapping
-------------------


## 快捷键

- Cmd-' 引用
- Cmd-B	加粗
- Cmd-E	 清除Block
- Cmd-H	 标题Header变小
- Cmd-I	   斜体
- Cmd-K	  链接
- Cmd-L	 无序列表
- Cmd-P	 Preview
- Cmd-Alt-C	 代码块
- Cmd-Alt-I	 插入图片
- Cmd-Alt-L	有序列表
- Shift-Cmd-H  标题Header变大
- F9	 窗口拆分
- F11	全屏


