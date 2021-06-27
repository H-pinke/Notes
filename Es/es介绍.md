

#### 一. ElasticSearch 介绍

##### 1.1引言

```
1.在海量数据中执行搜索功能时，如果使用MYSQL，效率太低
2.如果关键字输入的不准确，一样可以搜索到想要的数据
3.将搜索关键字，以红色字体展示。
```

##### 1.2Es介绍

```
Es是一个使用JAVA语言并且 基于Lucene 编写的搜索引擎框架，他提供了分布式的全文搜索功能，提供了一个统一的基于R estful 风格的web 接口，官方客户端也对多种语言提供了相应的API

Lucene:本身就是一个搜索引擎的底层
分布式：es主要是为了突出他的横向扩展能力
全文检索： 将一段词语进行分词，并且将分出的单个词语统一的放到一个分词库中，在搜索时，根据关键字去分词库中检索，找到匹配的内容（倒排索引）

Restful 风格的web接口：操作Es很简单，只需要发送一个HTTP请求，并且根据请求方式不同，携带参数

```

##### 1.4

```
1. solr 在查询死数据时，速度相对Es更快一些。但是数据如果是实时改变的，solr的查询速度会降低很多，Es的查询的效率基本没有变化
2. solr搭建基于需要依赖 zookeeper来帮助管理。Es本身就支持集群的搭建，不需要第三方的介入。
3. 最开始sorl的社区可以说是非常火爆，针对国内的文档并不是很多。在Es出现之后，Es的社区火爆程度直线上升，Es的文档非常健全。
4. Es对现在云计算和大数据支持的特别好
```

##### 1.5 倒排索引

```
将存放的数据，以一定的方式进行分词，并且将分词的内容存放到一个单独的分词库中。

当用户去查询数据时，会将用户的查询关键字进行分词。

然后去分词库中匹配内容，最终得到数据的ID 标识

根据ID 标识去存放数据的位置 拉取到指定的数据
```

##### 1.6 es和mysql 

```
es: es 会对索引进行分片，还会对索引进行备份。
		es5.x 版本中，一个index下可以创建多个Type
		es6.x 版本中，一个index下可以创建一个Type
		es7.x 版本中，一个index下没有type
		
mysql :一个mysql服务中有多个数据库，一个库中有多张表，在每一张表中有多行数据
```

es服务--->Book索引--->ITtype---->document1->fields

mysql服务---->数据库---->表---->数据结构-->列



###### 3.1.1索引index

```
ES的服务中，可以创建多个索引。
每一个索引默认被分成5片存储
每一个分片都会存在至少一个备份分片
备份分片默认不会帮助检索数据，当es检索压力特别大的时候，备份分片才会帮助检索数据。
备份的分片必须放在不同的服务器中
```

#### 3.2操作es的 restful 语法

```
GET请求
http://xxxx/index  ：查询索引信息
http://xxxx/index/type/doc_id/ ：查询指定文档信息

POST请求
http://xxxx/index/type/_search : 查询文档，可以在请求体中添加 json 字符串来代表 查询条件

http://xxxx/index/doc_id/_update: 修改文档，在请求体中指定 json字符串代表修改的具体信息

PUT请求
http://xxxx/index 创建一个索引
http://xxxx/index/type/_mappings: 代表创建索引时，指定文档存储的属性信息。

DELETE请求：
http://xxxxx/index :删除跑路
http://xxxxx/index/type/doc_id :删除指定的文档


```

##### 3.1.2 数据同步

```
POST  _reindex
{
  
  "source": {
    "index": "person"
  },
  "dest": {
    "index": "person1"
  }
}

curl -u elastic:elastic -X POST "192.168.1.85:9200/_reindex" -H 'Content-Type: application/json' -d'
{
  "source": {
    "index": "db_customer"
  },
  "dest": {
    "index": "copy01_db_customer"
  }
}'
```

##### 3.4Es中可以指定的类型

```
字符串类型：
	text: 一般用于全文检索。将当前Field进行分词
	keyword： 当前的field 不会被分词
```

```
“name”: {
	"type" : "text",
	"index": true 指定当前可以作为查询的条件
	”store“: false 是否需要额外存储
}
```

#### es各种查询

##### 6.1 term&terms查询

###### 6.1.1 term 查询

```
term的查询是代表完全匹配，搜索之前不会对你的关键字进行分词，对你的关键字去文档分词库中去匹配内容。
//⚠️ term 查询只对keyword 类型有用，text 类型无用
```

###### 6.1.2 terms查询

```
terms 和term的查询机制一样，都不会将指定的查询关键字进行分词，直接去分词库中匹配，找到相应文档内容。

terms是在针对一个字段包含 多个值的时候使用
```

##### 6.2 match查询

```
match 查询属于高层查询，他会根据你的查询的字段类型不一样，采用不同的查询方式

1. 查询的是日期或者是数值的话，他会将你基于的字符串查询查询内容转换为日期或者数值对待。
2. 如果查询的内容是一个可以被分词的内容（text）match会将你指定的查询内容根据一定的方式去分词，去分词库中匹配指定的内容。
3. 如果查询的内容是一个不能被分词的内容（keyword）match 查询不对对你指定的查询关键字进行分词。

match查询，实际底层就是多个term 查询，将多个 term查询的结果给你封装到了一起。
```

###### 6.2.1 match_all

```
查询全部内容，不指定任何查询条件
GET /book/_search
{
   "query": {
     "match_all": {
      
     }
  }
}
```

###### 6.2.2 mach bool查询

```
#基于一个Field 匹配的内容，采用and 或者 or的方式连接
POST /book/_search
{
  "query": {
    "match": {
      "name": {
        "query": "中国 健康",
        "operator": "and" #内容即包含中国也包含健康
      }
    }
  }
}

POST /book/_search
{
  "query": {
    "match": {
      "name": {
        "query": "中国 健康",
        "operator": "or" #内容中包含健康活着中国
      }
    }
  }
}
```

###### 6.2.3 multi_match查询

```
match 针对一个field做检索，multi_match针对多个field 进行检索，多个field 对应一个text
POST /book/_search
{
  "query": {
   "multi_match": {
     "query": "哈哈",
     "fields": ["mobile", "mobile_en"]
   }
  }
}

```

###### 6.3.2 ids查询

```
根据多个id查询，类似Mysql中的 where id in (id1，id2, id3)
POST /book/_search
{
  "query": {
    "ids": {
      "values": ["z9gVzXkByZ6c6ddFP_em", "zfn07XkBOrx-IiiAUEhF"]
    }
  }
}
```

###### 6.3.3 prefix

```
前缀查询，可以通过一个关键字去指定一个Field 的前缀，从而查询到指定的文档
POST /book/_search
{
  "query": {
    "prefix": {
      "name": {
        "value": "sd"
      }
    }
  }
}
```

###### 6.3.4 fuzzy

```
模糊查询，我们输入字符串的大概，ES就可以去根据输入的内容大概去匹配一下结果
#fuzzy 模糊查询
POST /book/_search
{
  "query": {
    "fuzzy": {
      "name": {
        "value": "sd",
        "prefix_length": 2   #指定前面几个字符是不允许出现错误的
      }
    }
  }
}
```

###### 6.3.5 wildcard 查询

```
通配查询，和Mysql 中的like 是一个套路，可以在查询时，在字符串中指定通配符* 和 占位符

POST /book/_search
{
  "query": {
    "wildcard": {
      "name": {
        "value": "时代的*" # 可以使用*和？指定通配符和占位符
      }
    }
  }
}
```

###### 6.3.6 range 查询

```
范围查询，只针对数值类型，对某一个Field进行大于或者小于范围指定。
POST /book/_search
{
  "query": {
    "range": {
      "FIELD": {
        "gte": 10,
        "lte": 20
        # 可以使用 gt :>  gte: >= lt :< lte :< =
      }
    }
  }
}
```

###### 6.3.7 regexp 查询

```
正则查询，通过你编写的正则表达式去匹配内容
ps: 
prefix, fuzzy, wildcard和regexp 查询效率相对比较低，要求效率比较高时，避免去使用
POST /book/_search
{
  "query": {
    "regexp": {
      "mobile": "J-[a-z]"#编写正则
    }
  }
}
```

###### 6.3.8 深分页面scroll

```
ES 对from + size 是有限制的， from和size 二者只和不能超过 1W
原理
from+size在ES查询数据的方式
1. 第一步现将用户指定的关键进行分词。
2. 第二步将词汇去分词库中进行检索，得到多个文档的ID
3. 第三步去各个分片中去拉取指定的数据。耗时较长
4. 第四步将数据根据score进行排序，耗时较长
5. 第五步根据from的值，将查询到的数据舍弃一部分
6. 第六步返回结果

scroll在Es查询数据的方式
1. 第一步现将用户指定的关键词进行分词
2. 第二步将词汇去分词库中进行检索，得到多个文档的ID
3. 第三步将文档的ID存放在一个Es的上下文中。
4. 第四步根据你指定的size的个数去Es中检索指定个数的数据，拿完数据的文档ID，会从上下文中移除。
5. 第五步如果需要下一页数据，直接去Es中的上下文中，找后续内容
6. 第六步循环第四步和第五步

scroll查询方式，不适合做实时的查询


#scroll
# 执行scroll查询，返回第一页数据，并且将文档的ID信息放在ES上下文中，指定生存时间
POST /book/_search?scroll=1m
{
  "query": {
    "match_all": {}
  },
  "size": 2,
  "sort": [
    {
      "free": {
        "order": "desc"
      }
    }
  ]
}

#下一页
POST /_search/scroll
{
#根据第一步得到的scroll_id去指定  "scroll_id":"FGluY2x1ZGVfY29udGV4dF91dWlkDnF1ZXJ5VGhlbkZldGNoBRQtX2lKUUhvQkZSRUtDLTNpTVdCUAAAAAAAAAqLFkZuRl9TdnFRVFNLdHpLR3duSWdSNUEUX1BpSlFIb0JGUkVLQy0zaU1XQlAAAAAAAAAKjBZGbkZfU3ZxUVRTS3R6S0d3bklnUjVBFF9maUpRSG9CRlJFS0MtM2lNV0JQAAAAAAAACo0WRm5GX1N2cVFUU0t0ektHd25JZ1I1QRRfdmlKUUhvQkZSRUtDLTNpTVdCUAAAAAAAAAqOFkZuRl9TdnFRVFNLdHpLR3duSWdSNUEUX19pSlFIb0JGUkVLQy0zaU1XQlAAAAAAAAAKjxZGbkZfU3ZxUVRTS3R6S0d3bklnUjVB",
  "scroll" : "1m" #scroll信息的生存时间
}

DELETE /_search/scroll/scroll_id
```

###### 6.3.9 delete-by-query

```
根据term match等查询方式去删除大量的文档
PS：如果你需要删除的内容，是index下的大部分数据，推荐创建一个全新的index，将保留的文档内容，添加到全新的索引

#delete-by-query  删除指定的查询条件
POST /book/_delete_by_query
{
    "query": {
    "ids": {
      "values": ["z9gVzXkByZ6c6ddFP_em", "zfn07XkBOrx-IiiAUEhF"]
    }
  }
}
```

##### 7 复合查询

###### 7.1.1 bool查询

```
复合过滤器，将你的多个查询条件，以一定的逻辑组合在一起
1. must:所有的条件，用must组合在一起，表示And的意思
2. must_not: 将must_not中的条件，全部都不能匹配，标示Not的意思
3. should: 所有的条件，用should组合在一起，表示Or的意思

#bool复合查询
POST /book/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "name": {
              "value": "sd时代的哈哈"
            }
          }
        },
        {
          "term": {
            "mobile_count": {
              "value": "123"
            }
          }
        }
        
      ],
      "must": [
        {
          "match": {
            "free": "5"
          }
        }
      ],
      "must_not": [
        {
          "match": {
            "mobile_count": "1234"
          }
        }
      ]
     
    }
  }
}
```

###### 7.1.2  boosting 查询

```
boosting 查询可以帮助我们去影响查询后的score
1. positive: 只有匹配上positive的查询的内容，才会被放到返回到结果集中。
2. negative: 如果匹配上和positive并且也匹配上了negative,就可以降低这样的文档 score
3. negative_boost: 指定系数，必须小于1.0

关于查询时，分数时如何计算的：
1. 搜索的关键字在文档中出现的频次越高，分数就越高
2. 指定的文档内容越短，分数就越高
3. 我们在搜索时，指定的关键字也会被分词，这个被分词的内容，被分词库匹配的个数越多，分数越高

# boosting 查询
POST /book/_search
{
  "query": {
    "boosting": {
      "positive": {
        "match": {
          "name": "哈哈1"
        }
      },
      "negative": {
        "match": {
          "mobile_count": "123"
        }
      },
      "negative_boost": 0.2
    }
  }
}
```

###### 7.1.2  filter 查询

```
query : 根据你的查询条件，去计算文档的匹配度得到一个分数，并且根据分数进行排序，不会做缓存的。
filter: 根据你的查询条件去查询文档，不去计算分数，而且filter 会对经常被过滤掉数据进行缓存

# filter
POST /book/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "match": {
            "name": "1"
          }
        },
        {
          "term": {
            "mobile_count": "123"
          }
        }
       
      ]
    }
  }
}
```

###### 7.1.2 高亮查询

```
高亮查询就是你用户输入的关键字，以一定的特殊性展示给用户，让用户知道为什么这个结果被检索出来。
高亮展示的数据，本身就是文档中的一个Field，单独将Field以HighLight的形式返回给你

ES提供了一个hightlight属性。和query同级别
1. fragment_size: 指定高亮数据展示多少个字符回来
2. pre_tags: 指定前缀标签，举个栗子 <font color='red'>
3. post_tags: 指定后缀标签，举个栗子 </font>
4. fields: 指定哪几个Field以高亮形式返回

POST /book/_search
{
  "query": {
    "match": {
      "name": "哈哈1"
    }
  },
  "highlight": {
    "fields": {
      "name": {}
    },
    "pre_tags": "<font color='red'>",
    "post_tags": "</font>",
    "fragment_size": "2"
  }
}
```

##### 8 聚合查询

###### 8.1.1 去重计数查询

```
去重计数，即Cardinality,第一步先将返回的文档中的一个指定 field进行去重，统计一共多少条
#cardinality
POST /book/_search
{
  "aggs": {
    "agg": {
      "cardinality": {
        "field": "mobile_count"
      }
    }
  }
}

```

###### 8.1.2 范围统计

```
统计一定范围内出现的文档个数，比如，针对某一个Field的值在 0-100, 100-200, 200-300之间文档出现的个数分别是多少。

范围统计可以针对普通的数值，针对时间类型，针对IP类型都可以做相应的统计。

range, date_range, ip_range

#时间方式统计
POST /book/_search
{
  "aggs": {
    "agg": {
      "range": {
        "field": "free",
        "ranges": [
          {
            "to": 4 #小于4
          },
          {
            "from": 10,#大于10
            "to": 50 #小于50
          },
          {
            "from": 100 #大于等于100
          }
        ]
      }
    }
  }
}
```

###### 8.1.3 统计聚合查询

```
他可以帮你查询指定Field的的最大值，最小值，平均值，平方和。。。
使用 extended_stats

#统计聚合查询
POST /book/_search
{
  "aggs": {
    "agg": {
      "extended_stats": {
        "field": "free"
      }
    }
  }
}
```

###### 8.1.4 ES的地图检索方式

```
geo_distance :直线距离检索方式
geo_bounding_box: 以两个点确定一个矩形，获取在矩形内的全部数据
geo_polygon: 以多个点，确认一个多边形，获取多边形内的全部数据
```

