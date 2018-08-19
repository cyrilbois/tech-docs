---
title:  Elastcisearch 6.2 Restful API 
description: Elastcisearch 常用的 Restful API
...

# Elastcisearch
详细的API请参考官方网站： https://www.elastic.co/guide/en/elasticsearch/reference/current/index.html 这里只列举常用的方式。

## 索引API
官方链接： [https://www.elastic.co/guide/en/elasticsearch/reference/6.2/indices.html](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/indices.html)
### 创建索引
```
PUT /news
```
创建名为test的索引，没有创建任何对应的Type
```
{
  "acknowledged": true,
  "shards_acknowledged": true,
  "index": "news"
}
```
### 查看索引
```
GET /news
```
```
{
  "test": {
    "aliases": {},
    "mappings": {},
    "settings": {
      "index": {
        "creation_date": "1534644406300",
        "number_of_shards": "5",
        "number_of_replicas": "1",
        "uuid": "EgrQnql6TjGVPXaW7a145Q",
        "version": {
          "created": "6020499"
        },
        "provided_name": "test"
      }
    }
  }
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


