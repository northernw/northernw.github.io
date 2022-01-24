---
title: ES入门
tags:
  - ES
categories:
  - 学习笔记
date: 2022-01-24 18:25:25
---



比较完整的入门文档：https://juejin.cn/post/6844904181325627400



## 简介

开源、全文搜索引擎、首选。

快速地存储、搜索和分析海量数据。

底层是开源库Lucene。Elastic是Lucene的封装，提供rest api调用。

7.x+ 移除了类型的概念，单类型索引

8.x+ URL中不支持type参数了 【待验证】

6.x 多类型索引



## 基本概念

### index 索引

1. 动词：插入
2. 名词：相当于一个database

### type 类型

1. 一个索引可以有多个类型，相当于table
2. 7.x+ 不再允许创建多个type了

### document 文档

1. JSON格式，相当于table里的一行数据
2. 倒排索引机制



## 集群操作命令

_cat

1. get /_cat/nodes 查看所有节点

2. get /_cat/health 查看健康状况

3. get /_cat/indices 查看所有索引

4. 相当于 show databases

   

## 索引

1. 创建索引
2. 删除索引
   1. delete ${indexName}

## 类型

1. 删除类型——不支持

   

## 映射

映射用来定义一个文档，以及它所包含的字段是如何存储和索引的

### 查询映射

get ${indexName}/_mapping

### 创建映射

创建索引的同时创建映射

```json
put  ${indexName}
{
  "mappings": {//映射规则
    "properties": {
      "age":    { "type": "integer" },  
      "email":  { "type": "keyword"  },//keyword不会进行全文检索 
      "name":   { "type": "text"  }//text保存的时候进行分词，搜索的时候进行全文检索   
    }
  }
}
```



### 修改映射

只能添加新字段

不能修改原有字段 【只能创建新索引、迁移数据、删除原索引】

参数从properties开始

```
put ${indexName}/_mapping
{
  "properties": {
    "employee-id": {
      "type": "keyword",
      "index": false//索引选项控制是否对字段值建立索引。 它接受true或false，默认为true。未索引的字段不可查询。
    }
  }
}
```



### 数据迁移

创建一个新索引

迁移数据

1. 7.x 后写法：

   ```json
   POST _reindex	//固定写法
   {
     "source": {	//老索引
       "index": "twitter"
     },
     "dest": {		//目标索引
       "index": "new_twitter"
     }
   }
   ```

   

2. 7.x 前写法：

   ```json
   POST _reindex	//固定写法
   {
     "source": {	
       "index": "twitter", //老索引
       "type": "twitter",  //老类型
     },
     "dest": {		//目标索引
       "index": "new_twitter"
     }
   }
   ```

   

### 字段类型

#### 基础类型

1. 字符串 text keyword
2. 数字 long integer short byte double float half_float scaled_float
3. 日期 date
4. 布尔 boolean
5. 二进制 binary

#### 复合类型

1. 数组 array 不针对特定类型
2. 对象 object 用于json对象
3. 嵌套 nested  用于json对象

#### 地理类型

1. 地理坐标 geo_point 经度纬度坐标
2. 地理形状 geo_shape 用于多边形等复杂形状

#### 特定类型

1. IP类型 ip

2. 补全类型 completion 提供自动完成提示？

3. 令牌计数类型 token_count 用来统计字符串中词条的数量

4. 附件类型 attachment 参考 mapper-attachments 插件，支持将附件例如Microsoft Office格式，open document格式，ePub，HTML等索引为 attachment 数据类型

5. 抽取类型 （Percolator） 接受来自领域特定语言（query-dsl）的查询

   

## 文档基础

### 创建or更新文档

1. 普通创建or更新
   1. put/post  ${indexName}/${typeName}/#{id}
      1. put post都可以
      2. put 可以新增也可以修改，必须指定id —— 一般用put进行修改
      3. post 指定id为更新，不指定id为新增，id若不存在也是新增 —— 用post进行新增
2. 使用乐观锁进行更新
   1. put/post ${indexName}/${typeName}/#{id}?if_seq_no=1&if_primary_term=2
      1. 如果数据中的_seq_no和_primary_term和请求中的不相等，则返回409，更新失败
      2. 相同则返回200，更新成功
3. 带_update参数的更新 ——【不常使用】
   1. post ${indexName}/${typeName}/#{id}/_update
      1. 会检验参数和原数据是否相同，如果相同就什么都不做，不改变_version、_seq_no
      2. 不相同则进行更新
      3. 不带_update的更新，则每次都会改变_version、_seq_no 【存疑，待确认】
      4. 带与不带_update，参数格式有一些不同
         1. 带 { "doc": { "name": "John Nash", "age":40 } }
         2. 不带 { "name": "John Nash2", "age": 40}

### 查询文档

1. get ${indexName}/${typeName}/#{id}
   1. _index 索引
   2. _type 类型
   3. _id 记录ID
   4. _version 版本号
   5. _seq_no 并发控制字段，每更新一次+1，用来做乐观锁
   6. _primary_term 并发控制字段，如果重启会变化
   7. _source 数据内容

### 删除文档

1. delete ${indexName}/${typeName}/#{id}

### 批量操作

1. 指定索引
   1. post ${indexName}/${typeName}/bulk
   2. 参数格式：
      1. {“index”:{“_id”:”1"}}
      2. {“title”:”my title"}
      3. {“index”:{“_id”:”2"}}
      4. {“title”:”my title2"}
2. 不指定索引，在es全局执行
   1. post /bulk
   2. 参数格式：
      1. {"delete":{"_index":"website","_type":"blog","_id":"123"}}
      2. {"create":{"_index":"website","_type":"blog","_id":"123"}}
      3. {"title":"My first blog post"}
      4. {"index":{"_index":"website","_type":"blog"}}
      5. {"title":"My second blog post"}
      6. {"update":{"_index":"website","_type":"blog","_id":"123"}}
      7. {"doc":{"title":"My updated blog post"}}

## 文档进阶

### 检索

#### 支持两种方式的检索

1. request param: uri + 检索参数
   1. get ${indexName}/_search  检索所有信息，包括types和docs
   2. get ${indexName}/_search?q=*&sort=name:asc 请求参数
2. request body: uri + 请求体
   1. get  ${indexName}/search
   2. 请求体：——Query DSL（Domain Specific Language 领域特定语言，查询语言）
      1. {“query”{“match_all”:{}}}

#### 响应结果

1. took 检索耗时，毫秒
2. time_out 是否超时
3. _shards 多少个分片被检索了，并统计了成功/失败的分片
4. hits 检索结果
5. hits.total 检索的命中个数
6. hits.hits 实际的检索结果（默认为前10的文档）
7. sort 结果的排序键（没有按score排序）
8. score和max_score 相关性得分和最高得分

### DSL

典型结构 {QUERY_NAME:{FIELD_NAME:{ARGUMENT:VALUE, ARGUMENT:VALUE}}}

#### query

##### match_all 匹配所有

##### match  匹配查询

1. 基本类型，非字符串，精确匹配
   1. match:{account:20}
2. 字符串，分词+全文检索
   1. 参数为一个单词：包含这个单词的所有记录，并且计算记录的相关性得分
      1. match:{address:kings}
   2. 参数为多个单词：对参数进行分词，包含某个单词或者单词组合的记录，并且计算相关性得分
      1. match:{address:kings land}

##### match_phrase 短语匹配

1. 把参数当成整体，不进行分词
   1. match_phrase:{address:kings land}

##### multi_match 多字段匹配

1. address或state包含mill
2. 也会进行分词
3. multi_match:{query:mill,fields:[address,state]}

##### term 精确匹配

1. 类似match，匹配某个属性的值，但是不能并且避免用于text类型的字段
2. text的精确匹配，使用 match: {${field}.keyword:${value}}

##### bool 复合查询

1. must: 子句必须出现在文档中
2. should：子句应该出现在文档中
3. must_not：子句不应该出现在文档中
4. filter：子句必须出现在文档中，不计算相关性得分
5. 子句里面再使用match/term等查询

#### sort 排序

“sort”:{“field_name”:{“order”:”asc"}}

#### _source 查询指定的字段

“_source”:[“name”,”age"]

#### aggregations 聚合

1. aggregations: {<aggregation_name>: {<aggregation_type>:<aggregation_body>}}
2. ![在这里插入图片描述](/Users/wangyuanqing1/github/northernw.github.io/image/es-introduction/1728738964e5dd2b~tplv-t2oaga2asx-watermark.awebp.png)
3. 【还有其他复杂的聚合...】
