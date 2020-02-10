# Elasticsearch ---查询

**模拟数据创建**

首先利用head差检查创建book索引

然后修改mappings配置

http方法:  **put**

链接地址:  127.0.0.1:9200/book

```json
{

    "properties": {

      "word_count": {

        "type": "integer"

      },

      "author": {

        "type": "keyword"

      },

      "title": {

        "type": "text"

      },

      "publish_date": {

        "type": "date",

        "format": "yyyy-MM-dd HH:mm:ss || yyyy-MM-dd || epoch_millis"

      }

    }

}
```

数据插入

第一条数据

http方法: put

连接地址: 127.0.0.1:9200/book/_doc/1

请求参数

```json
{

  "author": "张三",

  "title": "移魂大法",

  "word_count": 1000,

  "publish_date": "2000-10-01"

}

{

  "author": "李三",

  "title": "JAVA入门",

  "word_count": 2000,

  "publish_date": "2010-10-01"

}

{

  "author": "张四",

  "title": "Python入门",

  "word_count": 2000,

  "publish_date": "2005-10-01"

}

{

  "author": "李四",

  "title": "Elasticsearch大法好",

  "word_count": 1000,

  "publish_date": "2017-08-01"

}

{

  "author": "王五",

  "title": "菜谱",

  "word_count": 5000,

  "publish_date": "2002-10-01"

}

{

  "author": "赵六",

  "title": "剑谱",

  "word_count": 10000,

  "publish_date": "1997-01-01"

}

{

  "author": "张三丰",

  "title": "太极拳",

  "word_count": 1000,

  "publish_date": "1997-01-01"

}

{

  "author": "瓦力",

  "title": "Elasticsearch入门",

  "word_count": 3000,

  "publish_date": "2017-08-20"

}

{

  "author": "很胖的瓦力",

  "title": "Elasticsearch精通",

  "word_count": 3000,

  "publish_date": "2017-08-15"

}

{

  "author": "牛魔王",

  "title": "芭蕉扇",

  "word_count": 1000,

  "publish_date": "2000-10-01"

}

{

  "author": "孙悟空",

  "title": "七十二变",

  "word_count": 1000,

  "publish_date": "2000-10-01"

}
```



### 3.1 简单查询

http方法: GET

http地址: 127.0.0.1:9200/book/_doc/1

book: 索引名称

novel: type名称

![](https://github.com/jigexio/Elasticsearch/blob/master/10.png)

### 3.2 条件查询

http方法: POST

http地址: 127.0.0.1:9200/book/_doc/_search

查询所有数据

```json
{

    "query": {

        "match_all": {}

    }

}
```

Query: 为查询关键字

![](https://github.com/jigexio/Elasticsearch/blob/master/11.png)

添加起始条数和记录数

```json
{

  "query": {

    "match_all": {}

  },

  "from": 1,

  "size": 1

}
```

 ![](https://github.com/jigexio/Elasticsearch/blob/master/12.png)

关键词查询

```json
{

  "query": {

    "match": {

      "title": "Elasticsearch"

    }

  },

  "sort": [

    {

      "publish_date": {

        "order": "desc"

      }

    }

  ]

}
```



![](https://github.com/jigexio/Elasticsearch/blob/master/13.png)

Match中为匹配的字段和值

Sort中为排序字段



### 3.3 聚合查询

关键词:aggs

http方法: POST

http地址: 127.0.0.1:9200/book/_doc/_search

```json
{

  "aggs": {

    "group_by_word_count": {

      "terms": {

        "field": "word_count"

      }

    },

    "group_by_publish_date": {

      "terms": {

        "field": "publish_date"

      }

    }

  }

}
```

 ![](https://github.com/jigexio/Elasticsearch/blob/master/14.png)

![](https://github.com/jigexio/Elasticsearch/blob/master/15.png)



对指定的字段进行计算

```json
{

  "aggs": {

    "grades_word_count": {

      "stats": {

        "field": "word_count"

      }

    }

  }

}
```

![](https://github.com/jigexio/Elasticsearch/blob/master/16.png)

```json
{

  "aggs": {

    "grades_word_count": {

      "min": {

        "field": "word_count"

      }

    }

  }

}
```

![](https://github.com/jigexio/Elasticsearch/blob/master/17.png)



### 3.4 高级查询

#### 3.4.1 子条件查询-- Query context

特定字段查询所指特定值

Query context

在查询过程中, 除了判断文档是否满足查询条件之外, ES还会计算一个_score来标识匹配的程度, 旨在判断目标文档和查询条件匹配的有多好.

Query context常用的查询有**全文本查询**和**字段级别的查询**

##### 1. 全文本查询: 针对文本类型数据

###### 1) 模糊匹配

http方法: post

http地址: 127.0.0.1:9200/book/novel/_search

关键词: query(查询关键词), match(模糊匹配关键词)

请求参数:

```json
{

  "query": {

    "match": {

      "author": "瓦力"

    }

  }

}
```

![](https://github.com/jigexio/Elasticsearch/blob/master/18.png)

或者

```json
{

  "query": {

    "match": {

      "title": "Elasticsearch入门"

    }

  }

}
```

![](https://github.com/jigexio/Elasticsearch/blob/master/19.png)

**注：**模糊匹配会把查询的字段进行拆分, 如title中的”Elasticsearch入门”, 会查询”Elasticsearch”和”入门”两个词语所匹配的内容

###### 2) 习语匹配

http方法: post

http地址: 127.0.0.1:9200/book/novel/_search

关键词: query(查询关键词), match_phrase(习语匹配关键词)

请求参数:

```json
{

  "query": {

    "match_phrase": {

      "title": "Elasticsearch入门"

    }

  }

}
```

![](https://github.com/jigexio/Elasticsearch/blob/master/20.png)

该查询条件会将”title”字段中与”Elasticsearch入门”相匹配的查出来

###### 3) 多个字段的匹配查询

http方法: post

http地址: 127.0.0.1:9200/book/novel/_search

关键词: query(查询关键词), multi_match(多字段匹配关键词)

请求参数:

```json
{

  "query": {

    "multi_match": {

      "query": "瓦力",

      "fields": [

        "author",

        "title"

      ]

    }

  }

}
```

![](https://github.com/jigexio/Elasticsearch/blob/master/21.png)

该查询条件会将”author”或者是”title”字段中包含”瓦力”的都查询出来

###### 4) 语法查询(queryString)

支持通配符, 范围查询, 布尔查询, 也可以用在正则表达式中,

http方法: post

http地址: 127.0.0.1:9200/book/novel/_search

关键词: query(查询关键词), query_string(语法查询关键词)

请求参数

查询” Elasticsearch”和” 大法”关键词

```json
{

  "query": {

    "query_string": {

      "query": "Elasticsearch AND 大法"

    }

  }

}
```

关键词OR的使用

```json
{

  "query": {

    "query_string": {

      "query": "(Elasticsearch AND 大法) OR Python"

    }

  }

}
```

![](https://github.com/jigexio/Elasticsearch/blob/master/22.png)

指定字段和条件查询

```json
{

  "query": {

    "query_string": {

      "query": "瓦力 OR Elasticsearch",

      "fields": [

        "author",

        "title"

      ]

    }

  }

}
```

![](https://github.com/jigexio/Elasticsearch/blob/master/23.png)

##### 2. 字段级别的查询: 针对结构化数据, 如数字, 日期等.

结构化数据的查询

http方法: post

http地址: 127.0.0.1:9200/book/novel/_search

关键词: query(查询关键词), term(具体项关键词)

查询单词数”word_count”为1000的数据

```json
{

  "query": {

    "term": {

      "word_count": 1000

    }

  }

}
```

![](https://github.com/jigexio/Elasticsearch/blob/master/24.pngg)



查询字数范围在1000-2000之间的图书

关键词: range, gte大于等于, lte小于等于,gt大于,lt小于

```json
{

  "query": {

    "range": {

     "word_count": {

        "gte": 1000,

        "lte": 2000

      }

    }

  }

}
```

![](https://github.com/jigexio/Elasticsearch/blob/master/25.png)



查询出版日期是今年初到现在的书籍(now关键字代表现在时间)

```json
{

  "query": {

    "range": {

      "publish_date": {

        "gte": "2017-01-01",

        "lte": "now"

      }

    }

  }

}
```

![](https://github.com/jigexio/Elasticsearch/blob/master/26.png)



#### 3.4.2 子条件查询--Filter context

在查询过程中，只判断该文档是否满足条件，只有YES或NO两种回答方式

http方法: post

http地址: 127.0.0.1:9200/book/novel/_search

关键词: query(查询关键词), bool, filter

```json
{

  "query": {

    "bool": {

      "filter": {

        "term": {

          "word_count": 1000

        }

      }

    }

  }

}
```

![](https://github.com/jigexio/Elasticsearch/blob/master/27.png)

Filter context相对于query context, 用于数据过滤, es会对其查询结果缓存, 查询速率会更快一些, filter要结合bool一起使用.



#### 3.4.3 复合条件查询

以一定的逻辑组合子条件查询

##### 固定分数查询

http方法: post

http地址: 127.0.0.1:9200/_search (全文搜索)

关键词: query(查询关键词), filter, boost关键词, 固定分数

```json
{

  "query": {

    "constant_score": {

      "filter": {

        "match": {

          "title": "Elasticsearch"

        }

      },

      "boost": 2

    }

  }

}
```

固定了_score为2. 而在模糊查询中, _score为变化的

![](https://github.com/jigexio/Elasticsearch/blob/master/28.png)

##### 布尔查询

http方法: post

http地址: 127.0.0.1:9200/_search (全文搜索)

关键词: query(查询关键词), bool, should(应该满足), must(必须满足), must_not(一定不能满足的条件)

```json
{

  "query": {

    "bool": {

      "should": [

        {

          "match": {

            "author": "瓦力"

          }

        },

        {

          "match": {

            "title": "Elasticsearch"

          }

        }

      ]

    }

  }

}
```

should相当于“或”OR

![](https://github.com/jigexio/Elasticsearch/blob/master/29.pngg)

```json
{

  "query": {

    "bool": {

      "must": [

        {

          "match": {

            "author": "瓦力"

          }

        },

        {

          "match": {

            "title": "Elasticsearch"

          }

        }

      ]

    }

  }

}
```

must相当于“和”AND

![](https://github.com/jigexio/Elasticsearch/blob/master/30.png)



Bool和filter混合使用

```json
{

  "query": {

    "bool": {

      "must": [

        {

          "match": {

            "author": "瓦力"

          }

        },

        {

          "match": {

            "title": "Elasticsearch"

          }

        }

      ],

      "filter": [

        {

          "term": {

            "word_count": 3000

          }

        }

      ]

    }

  }

}
```

 

Must_not的使用

```json
{

  "query": {

    "bool": {

      "must_not": {

        "term": {

          "author": "瓦力"

        }

      }

    }

  }

}
```

在查询的author中不包含瓦力
