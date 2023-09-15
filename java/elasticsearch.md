# ElasticSearch
Elaticsearch，简称为 ES，ES 是一个开源的高扩展的分布式全文搜索引擎，是整个 Elastic Stack 技术栈的核心。

The Elastic Stack, 包括 Elasticsearch、Kibana、Beats 和 Logstash(也称为 ELK Stack)。能够安全可靠地获取任何来源、任何格式的数据，然后实时地对数据进行搜索、分析和可视化。

## 全文搜索引擎
全文搜索引擎指的是目前广泛应用的主流搜索引擎。它的工作原理是计算机索引程序通过扫描文章中的每一个词，对每一个词建立一个索引，指明该词在文章中出现的次数和位置，当用户查询时，检索程序就根据事先建立的索引进行查找，并将查找的结果反馈给用户的检索方式。

## HTTP API
PUT幂等，POST非幂等。
### 索引
* 创建索引：**PUT**请求 :http://地址:端口/索引名
* 查看所有索引：**GET**请求 :http://地址:端口/_cat/indices?v
  * health：当前服务器健康状态: green(集群完整) yellow(单点正常、集群不完整) red(单点不正常)
  * status：索引打开、关闭状态
  * index：索引名
  * uuid：索引统一编号
  * pri：主分片数量
  * rep：副本数量
  * docs.count：可用文档数量
  * docs.deleted：文档删除状态(逻辑删除)
  * store.size：主分片和副分片整体占空间大小
  * pri.store.size：主分片占空间大小
* 查看单个索引：**GET**请求 :http://地址:端口/索引名
* 删除索引：**DELETE**请求 :http://地址:端口/索引名
* 关闭索引：**POST**请求 http://地址:端口/索引名/_close
* 打开索引：**POST**请求 http://地址:端口/索引名/_open 

#### 数据导入
将索引*index1*的数据导入到索引*index2*中：
```
POST http://地址:端口/_reindex
```
```JSON
{
    "source": {
        "index": "index1"
    },
    "dest": {
        "index": "index2"
    }
}
```

#### 别名操作
##### 添加别名
```
POST http://地址:端口/_aliases
```
```JSON
{
    "actions": [{
        "add": {
            "index": "person",
            "alias": "person_index"
        }
    }]
}
```

##### 删除别名
```
POST http://地址:端口/_aliases
```
```JSON
{
    "actions": [{
        "remove": {
            "index": "person",
            "alias": "person_index"
        }
    }]
}
```

### 文档
数字`1`为主键ID。

* 创建文档(不指定id)：**POST**请求 :http://地址:端口/索引名/_doc。 请求体的类型为JSON，请求体的内容为要插入的文档。由于没有指定数据唯一性标识(ID)，默认情况下，ES服务器会随机生成一个。
* 创建文档(指定id)：**POST**请求 :http://地址:端口/索引名/_doc/1。 请求体的类型为JSON，请求体的内容为要插入的文档。明确指定数据唯一性标识(ID)，请求方式可以为**PUT**。
* 根据id查询文档：**GET**请求 :http://地址:端口/索引名/_doc/1。
* 根据id修改文档：**POST**请求 :http://地址:端口/索引名/_doc/1。请求体的类型为JSON，请求体的内容为修改后的文档。
* 修改字段：**POST**请求 :http://地址:端口/索引名/_update/1。请求体的类型为JSON，请求体的内容为要修改字段的键值对。
* 根据id删除文档：**DELETE**请求 :http://地址:端口/索引名/_doc/1。删除一个文档不会立即从磁盘上移除，它只是被标记成已删除(逻辑删除)。
* 条件删除文档：**POST**请求 :http://地址:端口/索引名/_delete_by_query。请求体的类型为JSON，请求体的内容参考下面的[查询API](#查询)。

### 映射
映射类似于数据库(database)中的表结构(table)。创建数据库表需要设置字段名称，类型，长度，约束等。索引库也一样，需要知道这个类型下有哪些字段，每个字段有哪些约束信息，这就叫做映射(mapping)。

#### 添加映射
已有索引并添加映射，也可使用该种方式给索引添加新字段
```
PUT http://地址:端口/索引名/_mapping
```
```JSON
{
    "properties": {
        "name":{
            "type": "text",
            "index": true
        }, 
        "sex":{
            "type": "text",
            "index": false
        },
        "age":{
            "type": "long",
            "index": false
        } 
    }
} 
```
映射数据说明:
* 字段名: 任意填写，下面指定许多属性
* type: 类型，Elasticsearch中支持的数据类型非常丰富，说几个关键的:
  * String类型，又分两种:
    * text: 可分词
    * keyword: 不可分词，数据会作为完整字段进行匹配
  * Numerical: 数值类型，分两类
    * 基本数据类型: long、integer、short、byte、double、float、half_float
    * 浮点数的高精度类型: scaled_float
  * Date: 日期类型
  * Array: 数组类型
  * Object: 对象
* index: 是否索引，默认为true
  * true: 字段会被索引，则可以用来进行搜索
  * false: 字段不会被索引，不能用来搜索
* store: 是否将数据进行独立存储，默认为false。原始的文本会存储在_source里面，默认情况下其他提取出来的字段都不是独立存储的，是从_source里面提取出来的。当然你也可以独立的存储某个字段，只要设置"store"为true即可，获取独立存储的字段要比从_source中解析快得多，但是也会占用更多的空间，所以要根据实际业务需求来设置。
* analyzer: 分词器

#### 查看映射
**GET**请求 :http://地址:端口/索引名/_mapping

#### 索引映射关联
```
PUT http://地址:端口/索引名
```
```JSON
{
    "settings": {},
    "mappings": {
        "properties": {
            "name":{
                "type": "text",
                "index": true
            }, 
            "sex":{
                "type": "text",
                "index": false
            },
            "age":{
                "type": "long",
                "index": false
            }
        }
    }
}
```

### 查询
`student`为索引名，定义数据格式如下：
```JSON
{
    "name":"zhangsan",
    "nickname":"zhangsan",
    "sex":"男",
    "age":30
}
```

查询结果通常格式如下：
```JSON
{
    "took":2,
    "timed_out":false,
    "_shards":{"total":5,"successful":5,"failed":0},
    "hits":{
        "total":1,
        "max_score":1.0,
        "hits":[
            {
                "_index":"student",
                "_type":"_doc",
                "_id":"AV3qGfrC6jMbsbXb6k1p",
                "_score":1.0,
                "_source": {
                    "name":"zhangsan",
                    "nickname":"zhangsan",
                    "sex":"男",
                    "age":30
                }
            }
        ]
    }
}
```
返回结果的took字段表示该操作的耗时（单位为毫秒），timed_out字段表示是否超时，hits字段表示命中的记录，里面子字段的含义如下:
* total：返回记录数，本例是1条。
* max_score：最高的匹配程度，本例是1.0。
* hits：返回的记录组成的数组。

在返回的记录中，每条记录都有一个_score字段，表示匹配的程序，默认是按照这个字段降序排列。

#### 查询所有
```
GET http://地址:端口/student/_search
```
```JSON
{
    "query": {
        "match_all": {}
    }
}
```
* "query":这里的query代表一个查询对象，里面可以有不同的查询属性
* "match_all":查询类型，例如:match_all(代表查询所有)，match，term，range等等
* {查询条件}:查询条件会根据类型的不同，写法也有差异

#### 匹配查询
```
GET http://地址:端口/student/_search
```

```JSON
{
    "query": {
        "match": {
            "name":"zhangsan"
        }
    } 
}

{
    "query": {
        "match": {
            "name": {
                "query": "zhangsan",
                "operator": "and"
            }
        }
    } 
}
```
match匹配类型查询，会把查询条件进行分词，然后进行查询，多个词条之间默认是 or 的关系，也可以指定 and 的关系。

#### 字段匹配查询
```
GET http://地址:端口/student/_search
```
```JSON
{
    "query": {
        "multi_match": {
            "query": "zhangsan",
            "fields": ["name","nickname"]
        }
    }
}

{
    "query": {
        "query_string": {
            "query": "zhangsan"
        }
    }
}
```
multi_match(同query_string) 与 match 类似，不同的是它可以在多个字段中查询，可以指定字段的也可以不指定字段。

#### 关键字精确查询
```
GET http://地址:端口/student/_search
```
```JSON
{
    "query": {
        "term": {
            "name": {
                "value": "zhangsan"
            }
        }
    }
}
```
term 查询，精确的关键词匹配查询，不对查询条件进行分词。

#### 多关键字精确查询
```
GET http://地址:端口/student/_search
```
```JSON
{
    "query": {
        "terms": {
            "name": ["zhangsan","lisi"]
        }
    }
}
```
terms 查询和 term 查询一样，但它允许你指定多值进行匹配。如果这个字段包含了指定值中的任何一个值，那么这个文档满足条件，类似于 mysql 的 in。


#### 指定查询字段
```
GET http://地址:端口/student/_search
```
```JSON
{
    "_source": ["name","nickname"],
    "query": {
        "terms": {
            "nickname": ["zhangsan"]
        }
    }
}
```
默认情况下，Elasticsearch在搜索的结果中，会把文档中保存在_source的所有字段都返回。如果我们只想获取其中的部分字段，我们可以添加_source的过滤。类似于mysql中通过select关键字指定要查询的字段。


#### 过滤字段
```
GET http://地址:端口/student/_search
```
```JSON
{
    "_source": {
        "includes": ["name","nickname"],
        "excludes": ["sex","age"]
    },
    "query": {
        "terms": {
            "nickname": ["zhangsan"]
        }
    }
}
```
我们也可以通过:
* includes:来指定想要显示的字段
* excludes:来指定不想要显示的字段

#### 组合/布尔查询
```
GET http://地址:端口/student/_search
```
```JSON
{
    "query": {
        "bool": {
            "must": [{
                "match": {
                    "name": "zhangsan"
                }
            }],
            "must_not": [{
                "match": {
                    "age": "40"
                }
            }],
            "should": [{
                "match": {
                    "sex": "男"
                }
            }]
        }
    }
}
```
`bool`把各种其它查询通过`must`(必须/与)、`must_not`(必须不/非)、`should`(应该/或)的方式进行组合。

可以用filter代替must，它性能比must高，但不会计算得分。

#### 范围查询
```
GET http://地址:端口/student/_search
```
```JSON
{
    "query": {
        "range": {
            "age": {
                "gte": 30,
                "lte": 35
            }
        }
    }
}
```
range查询找出那些落在指定区间内的数字或者时间。range查询允许以下字符：
* gt：大于>
* gte：大于等于>=
* lt：小于<
* lte：小于等于<=


#### 模糊查询
```
GET http://地址:端口/student/_search
```
```JSON
{
    "query": {
        "fuzzy": {
            "title": {
                "value": "zhangsan",
                "fuzziness": 2
            }
        }
    }
}
```
返回包含与搜索字词相似的字词的文档。

编辑距离是将一个术语转换为另一个术语所需的一个字符更改的次数。这些更改可以包括: 
* 更改字符(box → fox)
* 删除字符(black → lack)
* 插入字符(sic → sick)
* 转置两个相邻字符(act → cat)

为了找到相似的术语，fuzzy 查询会在指定的编辑距离内创建一组搜索词的所有可能的变体或扩展。然后查询返回每个扩展的完全匹配。  
通过 fuzziness 修改编辑距离。一般使用默认值AUTO（即不指定），根据术语的长度生成编辑距离。

可以使用通配符 ?（任意单个字符）和 *（0个或多个字符），注意不要在查询条件/查询字符的前面使用通配符，否则就和mysql数据库一样，索引失效了。

#### 单字段排序
```
GET http://地址:端口/student/_search
```
```JSON
{
    "query": {
        "match": {
            "name": "zhangsan"
        }
    },
    "sort": [{
        "age": {
            "order": "desc"
        }
    }]
}
```
sort 可以让我们按照不同的字段进行排序，并且通过 order 指定排序的方式。desc 降序，asc 升序。

#### 多字段排序
```
GET http://地址:端口/student/_search
```
```JSON
{
    "query": {
        "match_all": {}
    },
    "sort": [{
            "age": {
                "order": "desc"
            }
        },
        {
            "_score": {
                "order": "desc"
            }
        }
    ]
}
```

#### 高亮查询
```
GET http://地址:端口/student/_search
```

```JSON
{
    "query": {
        "match": {
            "name": "zhangsan"
        }
    },
    "highlight": {
        "pre_tags": "<font color='red'>",
        "post_tags": "</font>",
        "fields": {
            "name": {}
        }
    }
}
```
在进行关键字搜索时，搜索出的内容中的关键字会显示不同的颜色，称之为高亮。

Elasticsearch 可以对查询内容中的关键字部分，进行标签和样式(高亮)的设置。

在使用 match 查询的同时，加上一个 highlight 属性:
* pre_tags:前置标签
* post_tags:后置标签
* fields:需要高亮的字段

#### 分页查询
```
GET http://地址:端口/student/_search
```
```JSON
{
    "query": {
        "match_all": {}
    },
    "sort": [{
        "age": {
            "order": "desc"
        }
    }],
    "from": 0,
    "size": 2
}
```
* from:当前页的起始索引，默认从 0 开始。
* size:每页显示多少条

#### 指标聚合查询
```
GET http://地址:端口/student/_search
```
```JSON
{
    "aggs": {
        "max_age": {
            "max": {
                "field": "age"
            }
        }
    },
    "size": 0
}

{
    "aggs": {
        "min_age": {
            "min": {
                "field": "age"
            }
        }
    },
    "size": 0
}

{
    "aggs": {
        "sum_age": {
            "sum": {
                "field": "age"
            }
        }
    },
    "size": 0
}

{
    "aggs": {
        "avg_age": {
            "avg": {
                "field": "age"
            }
        }
    },
    "size": 0
}

{
    "aggs": {
        "distinct_age": {
            "cardinality": {
                "field": "age"
            }
        }
    },
    "size": 0
}

{
    "aggs": {
        "stats_age": {
            "stats": {
                "field": "age"
            }
        }
    },
    "size": 0
}
```

聚合允许使用者对 es 文档进行统计分析，类似与关系型数据库中的 group by，当然还有很多其他的聚合，例如取最大值、平均值等等。
* 对某个字段取最大值 max
* 对某个字段取最小值 min
* 对某个字段求和 sum
* 对某个字段取平均值 avg
* 对某个字段的值进行去重之后再取总数
* State 聚合：对某个字段一次性返回 count，max，min，avg 和 sum 五个指标


#### 桶聚合查询
```
GET http://地址:端口/student/_search
```
```JSON
{
    "aggs": {
        "age_groupby": {
            "terms": {
                "field": "age"
            }
        }
    },
    "size": 0
}
```
桶聚和相当于 sql 中的 group by 语句，terms 聚合，分组统计。

还可以在terms分组下再进行聚合:
```JSON
{
    "aggs": {
        "age_groupby": {
            "terms": {"field": "age"},

            "aggs": {
                "sum_age":{
                    "sum":{"field":"age"}
                }
            }

        }
    },
    "size": 0
}
```



## 核心概念
### 索引(Index)
一个索引就是一个拥有几分相似特征的文档的集合。一个索引由一个名字来标识(必须全部是小写字母)，并且当我们要对这个索引中的文档进行索引、搜索、更新和删除的时候，都要使用到这个名字。在一个集群中，可以定义任意多的索引。能搜索的数据必须索引。

Elasticsearch 索引的精髓:一切设计都是为了提高搜索的性能。

### ~~~类型(Type)~~~
在一个索引中，你可以定义一种或多种类型。一个类型是你的索引的一个逻辑上的分类/分区，其语义完全由你来定。通常，会为具有一组共同字段的文档定义一个类型。

### 文档(Document)
一个文档是一个可被索引的基础信息单元，也就是一条数据。文档以JSON(Javascript Object Notation)格式来表示，而 JSON 是一个到处存在的互联网数据交互格式。在一个 index/type 里面，你可以存储任意多的文档。

### 字段(Field)
相当于是数据表的字段，对文档数据根据不同属性进行的分类标识。

### 映射(Mapping)
mapping是处理数据的方式和规则方面做一些限制，如:某个字段的数据类型、默认值、分析器、是否被索引等等。

### 分片(Shards)
一个索引可以存储超出单个节点硬件限制的大量数据。比如，一个具有 10 亿文档数据的索引占据 1TB 的磁盘空间，而任一节点都可能没有这样大的磁盘空间。或者单个节点处理搜索请求，响应太慢。为了解决这个问题，Elasticsearch 提供了将索引划分成多份的能力，每一份就称之为分片。当你创建一个索引的时候，你可以指定你想要的分片的数量。每个分片本身也是一个功能完善并且独立的“索引”，这个“索引”可以被放置到集群中的任何节点上。分片怎样分布，它的文档怎样聚合和搜索请求，是完全由 Elasticsearch 管理的，对用户透明。

#### 重要性
分片很重要，主要有两方面的原因:
1. 允许你水平分割/扩展你的内容容量。
2. 允许你在分片之上进行分布式的、并行的操作，进而提高性能/吞吐量。

### 副本(Replicas)
Elasticsearch允许你创建分片的一份或多份拷贝，这些拷贝叫做复制分片(副本)。

#### 重要性
复制分片之所以重要，有两个主要原因:
1. 在分片/节点失败的情况下，提供了高可用性。因为这个原因，注意到复制分片从不与原/主要(original/primary)分片置于同一节点上是非常重要的。
2. 扩展你的搜索量/吞吐量，因为搜索可以在所有的副本上并行运行。

### 索引 - 分片 - 副本
每个索引可以被分成多个分片，一个索引也可以被复制0次(意思是没有复制)或多次。一旦复制了，每个索引就有了主分片(作为复制源的原来的分片)和复制分片(主分片的拷贝)之别。分片和复制的数量可以在索引创建的时候指定。在索引创建之后，你可以在任何时候动态地改变复制的数量，但是你事后不能改变分片的数量。默认情况下，Elasticsearch中的每个索引被分片1个主分片和1个复制，这意味着，如果你的集群中至少有两个节点，你的索引将会有1个主分片和另外1个复制分片(1个完全拷贝)，这样的话每个索引总共就有2个分片，我们需要根据索引需要确定分片个数。

### 分配(Allocation)
将分片分配给某个节点的过程，包括分配主分片或者副本。如果是副本，还包含从主分片复制数据的过程。这个过程是由 master 节点完成的。

## 系统结构
一个运行中的 Elasticsearch 实例称为一个节点，而集群是由一个或者多个拥有相同 cluster.name 配置的节点组成，它们共同承担数据和负载的压力。当有节点加入集群中或者从集群中移除节点时，集群将会重新平均分布所有的数据。

当一个节点被选举成为主节点时，它将负责管理集群范围内的所有变更，例如增加、删除索引，或者增加、删除节点等。而主节点并不需要涉及到文档级别的变更和搜索等操作，所以当集群只拥有一个主节点的情况下，即使流量的增加它也不会成为瓶颈。任何节点都可以成为主节点。

作为用户，我们可以将请求发送到集群中的任何节点，包括主节点。每个节点都知道任意文档所处的位置，并且能够将我们的请求直接转发到存储我们所需文档的节点。无论我们将请求发送到哪个节点，它都能负责从各个包含我们所需文档的节点收集回数据，并将最终结果返回給客户端。 Elasticsearch 对这一切的管理都是透明的。


## 路由计算
`shard = hash(routing) % number_of_primary_shards`

当我们创建文档时，上面这个公式决定这个文档应当被存储的分片，这个决定的过程叫做路由。

routing 是一个可变值，默认是文档的 _id ，也可以设置成一个自定义的值。routing 通过 hash 函数生成一个数字，然后这个数字再除以 number_of_primary_shards (主分片的数量)后得到余数 。这个分布在 0 到 number_of_primary_shards-1 之间的余数，就是文档所在分片的位置。

## 文本分析
分析 包含下面的过程:
1. 将一块文本分成适合于倒排索引的独立的**词条**
2. 将这些词条统一化为标准格式以提高它们的“可搜索性”

分析器执行上面的工作，分析器实际上是将三个功能封装到了一个包里:
1. 字符过滤器：首先，字符串按顺序通过每个字符过滤器。他们的任务是在分词前整理字符串。一个字符过滤器可以用来去掉HTML，或者将 & 转化成 and。
2. 分词器：其次，字符串被分词器分为单个的词条。一个简单的分词器遇到空格和标点的时候，可能会将文本拆分成词条。
3. 过滤器：最后，词条按顺序通过每个过滤器。这个过程可能会改变词条(例如，小写化 Quick )，删除词条(例如，像 a，and，the等无用词)，或者增加词条(例如，像 jump 和 leap 这种同义词)。

### 内置分析器
- Standard Analyzer（标准分析器/分词器） - 默认的分词器，它根据 [Unicode 联盟](http://www.unicode.org/reports/tr29/) 定义的 *单词边界* 划分文本。它删除了大多数标点符号，并支持删除停止词。最后，将词条小写。
- Simple Analyzer（简单分析器） - 在任何不是字母的地方分隔文本，将词条小写。
- Stop Analyzer - 它和simple analyzer 很像，唯一不同的是 stop 分析器增加了对删除停止词的支持，停止词过滤(the,a,is)。
- Whitespace Analyzer（空格分析器） - 按照空格切分，不转小写。
- Keyword Analyzer - 不分词，直接将输入当作输出。
- Patter Analyzer - 正则表达式，默认\W+(非字符分割) （中文会被去掉）。
- Language Analyzer - 特定语言分词器。

### 使用场景
当我们*索引*一个文档，它的全文域被分析成词条以用来创建倒排索引。当我们在全文域*搜索*的时候，我们需要将查询字符串通过相同的分析过程，以保证我们*搜索*的词条格式与*索引*中的词条格式一致。

### 测试分析器
为了理解分词的过程和实际被存储到索引中的词条，可以使用 analyze API 来看文本是如何被分析的。

在消息体里，指定分析器和要分析的文本：
```
GET http://localhost:9200/_analyze
{
    "analyzer": "standard",
    "text": "Text to analyze"
}
```
结果中每个元素代表一个单独的词条:
```
{
    "tokens": [{
        "token": "text",
        "start_offset": 0,
        "end_offset": 4,
        "type": "<ALPHANUM>",
        "position": 1
    }, {
        "token": "to",
        "start_offset": 5,
        "end_offset": 7,
        "type": "<ALPHANUM>",
        "position": 2
    }, {
        "token": "analyse",
        "start_offset": 8,
        "end_offset": 15,
        "type": "<ALPHANUM>",
        "position": 3
    }]
}
```
token 是实际存储到索引中的词条。position 指明词条在原始文本中出现的位置。start_offset 和 end_offset 指明字符在原始字符串中的位置。

### IK分析器
ElasticSearch 内置分析器对中文很不友好，因此对于中文，我们使用IK分析器。

IKAnalyzer是一个开源的，基于java语言开发的轻量级的中文分词工具包，是一个基于Maven构建的项目，具有60万字/秒的高速处理能力，并且支持用户词典扩展定义。

[下载](https://github.com/medcl/elasticsearch-analysis-ik)IK分词器，将解压后的后的文件夹放入 ES 根目录下的 plugins 目录下，重启 ES 即可。

#### demo
```
GET http://localhost:9200/_analyze
{
    "text":"测试单词",
    "analyzer":"ik_max_word"
}
```
analyzer可以设为：
* ik_max_word:会将文本做最细粒度的拆分
* ik_smart:会将文本做最粗粒度的拆分

```
{
    "tokens": [{
        "token": "测试",
        "start_offset": 0,
        "end_offset": 2,
        "type": "CN_WORD",
        "position": 0
    }, {
        "token": "单词",
        "start_offset": 2,
        "end_offset": 4,
        "type": "CN_WORD",
        "position": 1
    }]
}
```

## Kibana
Kibana 是一个免费且开放的用户界面，能够让你对 Elasticsearch 数据进行可视化，并让你在 Elastic Stack 中进行导航。



## 参考
* https://www.yiibai.com/elasticsearch
* https://www.bilibili.com/video/BV1hh411D7sb