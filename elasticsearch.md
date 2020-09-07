# ElasticSearch

### **1.Restful**

软件架构风格，遵从rest设计原则

rest:资源的表现层状态转化

### 2.全文检索

计算机程序通过扫描文章中的每一个词，建立索引，指明该词在文章中出现的次数和位置。当用户查询时根据建立的索引查找。

只处理文本，不处理语义，英文不区分大小写，结果列表有相关度排序。

### 3.ElasticSearch

开源，最流行的企业级搜索引擎

Elasticsearch 是基于 Lucene 实现的分布式搜索引擎，提供了海量数据实时检索和分析能力。

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

### 11.elasticsearch原理

【https://www.jianshu.com/p/b50d7fdbe544】

**near realtime（NRT）**，从写入数据到数据可以被搜索有一个延迟事件时间，约1秒。

elasticsearch使用倒排索引，也就是根据索引内容查找索引对应的文档id，然后根据文档id取出文档。

**term（单词），**文本经过分析器分析之后得到的一系列单词，每个单词就叫做term。

**posting list（倒排列表），**存储包含改对应term的文档id的集合，除此之外还包括词频、偏移量等信息。

**term dictionary（单词字典）**  ，所有term的集合。Elasticsearch为了能快速找到某个term，将所有的term排个序，二分法查找term，logN的查找效率，就像通过字典查找一样，这就是**Term Dictionary**。

**term index（单词索引），**为单词建立索引，以快速检索单词字典。term index是包含term一些前缀的树，通过term index可以快速定位到term dictionary的某个offset，然后从这个位置再往后顺序查找。

![img](https://static001.infoq.cn/resource/image/e4/26/e4599b618e270df9b64a75eb77bfb326.jpg)

**term index 在内存中是以 FST（finite state transducers）的形式保存的，其特点是非常节省内存。Term dictionary 在磁盘上是以分 block 的方式保存的，一个 block 内部利用公共前缀压缩，比如都是 Ab 开头的单词就可以把 Ab 省去。这样 term dictionary 可以比 b-tree 更节约磁盘空间。**

**索引原理**

https://www.cnblogs.com/dreamroute/p/8484457.html

https://blog.csdn.net/lixiangchibang/article/details/84793901

https://www.infoq.cn/article/database-timestamp-02/?utm_source=infoq&utm_medium=related_content_link&utm_campaign=relatedContent_articles_clk

**联合索引查询**

对各条件索引得到的posting list 取并集

**使用skip list数据结构，同时遍历posting list，互相skip**

**使用bitset数据结构，分别求bitset，然后做and**

Elasticsearch 支持以上两种的联合索引方式，如果查询的 filter 缓存到了内存中（以 bitset 的形式），那么合并就是两个 bitset 的 AND。如果查询的 filter 没有缓存，那么就用 skip list 的方式去遍历两个 on disk 的 posting list。

Lucene使用 Roaring Bitmap

![img](https://static001.infoq.cn/resource/image/94/7e/9482b84c4aa3fb77a959c1ead553037e.png)

（postgreSQL从8.4开始通过bitmap联合使用两个索引）

**FST**

FST, 全称Finite State Transducer, 中文翻译: 有限状态转换器或有限状态传感器。

**变种字典树，字典树只共享了前缀，FST共享前缀和后缀，更加节约空间**

FST最重要的功能是可以实现Key到Value的映射，相当于HashMap<Key,Value>。FST的内存消耗要比HashMap少很多，
但FST的查询速度比HashMap要慢。FST在Lucene中被大量使用，例如：倒排索引的存储，同义词词典的存储，搜索关键字建议等。



cat-5 deep-10 do-15 dog-2 dogs-8

构造的FST如下

![img](https://upload-images.jianshu.io/upload_images/17052744-c67fb1c45f75799b.png?imageMogr2/auto-orient/strip|imageView2/2/w/680/format/webp)

https://blog.csdn.net/vivian_ll/article/details/95049652?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-1.channel_param

### 12 词典的完整结构

![img](https://upload-images.jianshu.io/upload_images/17052744-d455fdbe9da8cb08.png?imageMogr2/auto-orient/strip|imageView2/2/w/891/format/webp)

Lucene 的tip文件即为 Term Index 结构，tim文件即为 Term Dictionary 结构。由图可视，tip中存储的就是多个FST，
 FST中存储的是<单词前缀，以该前缀开头的所有Term的压缩块在磁盘中的位置>。即为前文提到的从 term index 查到对应的 term dictionary 的 block 位置之后，再去磁盘上找 term，大大减少了磁盘的 random access 次数。

可以形象地理解为，Term Dictionary 就是新华字典的正文部分包含了所有的词汇，Term Index 就是新华字典前面的索引页，用于表明词汇在哪一页。

但是 FST 即不能知道某个Term在Dictionary(.tim)文件上具体的位置，也不能仅通过FST就能确切的知道Term是否真实存在。它只能告诉你，查询的Term可能在这些Blocks上，到底存不存在FST并不能给出确切的答案，因为FST是通过Dictionary的每个Block的前缀构成，所以通过FST只可以直接找到这个Block在.tim文件上具体的File Pointer，并无法直接找到Terms。