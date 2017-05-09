_原文地址：http://blog.csdn.net/yangwenbo214/article/details/54142786_

### 一、基本情况

前言：term query和match query牵扯的东西比较多，例如分词器、mapping、倒排索引等。我结合官方文档中的一个实例，谈谈自己对此处的理解

* string类型在es5.*分为text和keyword。text是要被分词的，整个字符串根据一定规则分解成一个个小写的term，keyword类似es2.3中not_analyzed的情况。
> string数据put到elasticsearch中，默认是text。
> NOTE:默认分词器为standard analyzer。”Quick Brown Fox!”会被分解成[quick,brown,fox]写入倒排索引
* term query会去倒排索引中寻找确切的term，它并不知道分词器的存在。这种查询适合keyword 、numeric、date
* match query知道分词器的存在。并且理解是如何被分词的

总的来说有如下： 
- term query 查询的是倒排索引中确切的term 
- match query 会对filed进行分词操作，然后在查询

### 二、测试（1）

* 准备数据：
```
POST /termtest/termtype/1
{
  "content":"Name"
}
```
```
POST /termtest/termtype/2
{
  "content":"name city"
}
```
* 查看数据是否导入
```
GET /termtest/_search
{
  "query":
  {
    "match_all": {}
  }
}
```
* 结果：
```
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 1,
    "hits": [
      {
        "_index": "termtest",
        "_type": "termtype",
        "_id": "2",
        "_score": 1,
        "_source": {
          "content": "name city"
        }
      },
      {
        "_index": "termtest",
        "_type": "termtype",
        "_id": "1",
        "_score": 1,
        "_source": {
          "content": "Name"
        }
      }
    ]
  }
}
```

`如上说明，数据已经被导入。该处字符串类型是text，也就是默认被分词了`

* 做如下查询：
```
POST /termtest/_search
{
  "query":{
    "term":{
      "content":"Name"
    }
  }
}
```
* 结果
```
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 0,
    "max_score": null,
    "hits": []
  }
}
```

`分析结果：因为是默认被standard analyzer分词器分词，大写字母全部转为了小写字母，并存入了倒排索引以供搜索。term是确切查询，必须要匹配到大写的Name。所以返回结果为空`

```
POST /termtest/_search
{
  "query":{
    "match":{
      "content":"Name"
    }
  }
}
```
* 结果
```
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 2,
    "max_score": 0.2876821,
    "hits": [
      {
        "_index": "termtest",
        "_type": "termtype",
        "_id": "1",
        "_score": 0.2876821,
        "_source": {
          "content": "Name"
        }
      },
      {
        "_index": "termtest",
        "_type": "termtype",
        "_id": "2",
        "_score": 0.25811607,
        "_source": {
          "content": "name city"
        }
      }
    ]
  }
}
```

```
分析结果: 原因（1）：默认被standard analyzer分词器分词，大写字母全部转为了小写字母，并存入了倒排索引以供搜索， 
原因（2）：match query先对filed进行分词，分词为”name”,再去匹配倒排索引中的term
```

### 三、测试（2）

下面是官网实例官网实例 

* 导入数据

```
PUT my_index
{
  "mappings": {
    "my_type": {
      "properties": {
        "full_text": {
          "type":  "text" 
        },
        "exact_value": {
          "type":  "keyword" 
        }
      }
    }
  }
}
```
```
PUT my_index/my_type/1
{
  "full_text":   "Quick Foxes!", 
  "exact_value": "Quick Foxes!"  
}
```

`先指定类型，再导入数据`

* full_text: 指定类型为text，是会被分词
* exact_value: 指定类型为keyword，不会被分词
* full_text： 会被standard analyzer分词为如下terms [quick,foxes],存入倒排索引
* exact_value： 只有[Quick Foxes!]这一个term会被存入倒排索引

* 做如下查询
```
GET my_index/my_type/_search
{
  "query": {
    "term": {
      "exact_value": "Quick Foxes!" 
    }
  }
}
```
* 结果：
```
{
  "took": 1,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.2876821,
    "hits": [
      {
        "_index": "my_index",
        "_type": "my_type",
        "_id": "1",
        "_score": 0.2876821,
        "_source": {
          "full_text": "Quick Foxes!",
          "exact_value": "Quick Foxes!"
        }
      }
    ]
  }
}
```

`exact_value包含了确切的Quick Foxes!，因此被查询到`

```
GET my_index/my_type/_search
{
  "query": {
    "term": {
      "full_text": "Quick Foxes!" 
    }
  }
}
```

* 结果：

```
{
  "took": 4,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 0,
    "max_score": null,
    "hits": []
  }
}
```

`full_text被分词了，倒排索引中只有quick和foxes。没有Quick Foxes!`

```
GET my_index/my_type/_search
{
  "query": {
    "term": {
      "full_text": "foxes" 
    }
  }
}
```

* 结果：

```
{
  "took": 2,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.25811607,
    "hits": [
      {
        "_index": "my_index",
        "_type": "my_type",
        "_id": "1",
        "_score": 0.25811607,
        "_source": {
          "full_text": "Quick Foxes!",
          "exact_value": "Quick Foxes!"
        }
      }
    ]
  }
}
```

`full_text被分词，倒排索引中只有quick和foxes，因此查询foxes能成功`

```
GET my_index/my_type/_search
{
  "query": {
    "match": {
      "full_text": "Quick Foxes!" 
    }
  }
}
```

* 结果：

```
{
  "took": 3,
  "timed_out": false,
  "_shards": {
    "total": 5,
    "successful": 5,
    "failed": 0
  },
  "hits": {
    "total": 1,
    "max_score": 0.51623213,
    "hits": [
      {
        "_index": "my_index",
        "_type": "my_type",
        "_id": "1",
        "_score": 0.51623213,
        "_source": {
          "full_text": "Quick Foxes!",
          "exact_value": "Quick Foxes!"
        }
      }
    ]
  }
}
```

`match query会先对自己的query string进行分词。也就是”Quick Foxes!”先分词为quick和foxes。然后在去倒排索引中查询，此处full_text是text类型，被分词为quick和foxes
因此能匹配上。`
