# ElasticSearch

### **1.Restful**

软件架构风格，遵从rest设计原则

rest:资源的表现层状态转化

### 2.全文检索

计算机程序通过扫描文章中的每一个词，建立索引，指明该词在文章中出现的次数和位置。当用户查询时根据建立的索引查找。

只处理文本，不处理语义，英文不区分大小写，结果列表有相关度排序。

### 3.ElasticSearch

开源，最流行的企业级搜索引擎

java编写，提供简单易用的restful api

ES栈

beats+elasticsearch+kibana+logstash

### 4.应用场景

ES以轻量级json作为数据存储格式，在统计、日志数据存储分析、可视化领先

，维基百科、Stack Overflow、GitHub、百度等.

### 5.安装

以普通用户身份安装

默认启动分配内存1G

默认占用端口号 9200(web访问) 9300 TCP端口

curl 模拟get请求 http://localhost:9200

es默认单个节点以集群方式启动

![image-20200811204451544](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200811204451544.png)

开启远程连接权限

 elasticsearch.yml修改network.host=0.0.0.0

![image-20200811205107264](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200811205107264.png)

root权限下修改 /etc/sysctl.conf vm.max_map_count=655360

### 6.ES基本概念

接近实时的搜索平台，通常是1s内

**索引**

用友几分相似特征的文档的集合，一个索引有一个名字来标志，必须全都是小写字母，对这个索引

中的文档进行索引搜索更新删除时，都要使用到这个名字。索引类似与关系型数据库汇总database的概念。

**类型**

索引中可以定义一种或多种数据类型，类型类似表的概念（6版本不建议多个数据类型，7以后只能存储一个数据类型）

**映射**

类似表的schema，用于定义一个索引中类型的数据结构。mapping包括字段名，字段数据类型，和字段索引类型。

**文档**

一个文档是一个可以被索引的基础信息单元，类似于表中一条记录

### **7.kibana**

需要和 版本对应，主要用来可视化数据

![image-20200811222528977](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200811222528977.png)

**指令**

PUT  /dangdang

![image-20200813213430447](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200813213430447.png)

**索引不能包含大写**

GET /_cat/indices            //查看所有索引

DELETE  /

**类型操作**

```
PUT /example1
{
"mappings": {
  "emp":{
    "properties":{
    "id":{"type":"keyword"},
    "name":{"type":"text"},
    "age":{"type":"integer"},
    "bir":{"type":"date"}
    }
  }
}  
}
```

emp类型下属性有id，name，age，bir

**文档操作**

插入文档，put /索引/类型/文档id

```
PUT /example1/emp/1
{
  "id":1,
  "name":"zhangsan",
  "age":32,
  "bir":"2020-02-02"
}
```

自动生成文档id需要 post

更新

POST更新(不加_update)，先删除原始文档，再创建新文档

```
POST /example1/emp/1
{
  "id":2
}
```

POST更新,可以添加新的属性

```
POST /example1/emp/1/_update
{
  "doc":{
    "id":2,
    "name":"zhang",
    "age":22
  }
}
```

脚本更新

```
POST /example1/emp/1/_update
{
  "script": "ctx._source.age+=3"
}
```

文档批量操作

![image-20200813222306766](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200813222306766.png)

### 8.ES中高级检索

1.通过URL参数进行搜索（GET /索引/类型/_search?参数）

2.通过DSL进行搜索，传递json作为请求体（GET /索引/类型/_search）





**基于关键词查询**

```
GET example1/emp/_search

{

"query":{

"term":{

"xxx":{

"value":"xxx"

}

}

}

}
```

在type中，text会分词。ES中默认使用的分词器是标准分词器（），中文是单字分词，英文是单词分词。

其他都不分词

**范围查询**

一般对integer类型进行范围查询

```
GET /user/user/_search
{
  "query": {
    "range": {
      "age": {
        "gte": 10,
        "lte": 22
      }
    }
  }
}
```

gte 大于等于，gt 大于

lte小于等于，lt小于

**前缀查询**

基于关键词前缀查询

### 9.ES底层存储原理

**索引库原理**

![image-20200815155013935](C:\Users\Administrator\AppData\Roaming\Typora\typora-user-images\image-20200815155013935.png)

底层分为索引区和元数据区，元数据区存放文档数据，索引区记录索引。搜索时，先去索引区找到文档id，然后根据文档id企业元数据区找到文档。对于分词的test类型，只能使用单字搜索，对于其他类型，不分词，要使用全部内容区搜索。

### 10.IK分词器

穷举各种可能，ik_max_word

智能分词，ik_smart

