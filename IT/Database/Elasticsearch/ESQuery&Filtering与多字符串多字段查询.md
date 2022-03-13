# ES Query&Filtering与多字符串多字段查询

## Query Context & Filter Context

- 高级搜索的功能：支持多项文本输入，针对多个字段进行搜索
- 搜索引擎一般也是提供基于时间，价格等条件的过滤
- 在 `ES` 中，有 `Query` 和 `Filter` 两种不同的 Context
  - `Query Context`：相关性算分
  - `Filter Context`：不需要算分（`Yes or No`），可以利用 `Cache`，获得更好的查询性能

## Bool 查询

- 一个 `bool` 查询，是一个或者多个查询子句的组合
  - 总共包括 `4` 种子句。其中 `2` 种影响算分，`2` 中不影响算分。
- 相关性并不只是全文本检索的专利。也适用于 `yes | no` 的子句，匹配的子句越多，相关性评分越高。如果多条查询子句被合并为一条复合查询语句，比如 `bool` 查询，则每个子句计算得出的评分会被合并到总的相关性评分中。
  - `must`：必须匹配。贡献算分
  - `should`：选择性匹配。贡献算分
  - `must_not`：`Filter Context` 查询子句，必须不能匹配
  - `fiter`：`Filter Context` 必须匹配，但是不贡献算分
- bool 查询语法
  - 子查询可以任意顺序出现
  - 可以嵌套多个查询
  - 如果你的 `bool` 查询中，没有 `must` 条件，`should` 中必须至少满足一个查询

```curl
POST /products/_bulk
{ "index": { "_id": 1 }}
{ "price" : 10,"avaliable":true,"date":"2018-01-01", "productID" : "XHDK-A-1293-#fJ3" }
{ "index": { "_id": 2 }}
{ "price" : 20,"avaliable":true,"date":"2019-01-01", "productID" : "KDKE-B-9947-#kL5" }
{ "index": { "_id": 3 }}
{ "price" : 30,"avaliable":true, "productID" : "JODL-X-1937-#pV7" }
{ "index": { "_id": 4 }}
{ "price" : 30,"avaliable":false, "productID" : "QQPX-R-3956-#aD8" }



#基本语法
POST /products/_search
{
  "query": {
    "bool" : {
      "must" : {
        "term" : { "price" : "30" }
      },
      "filter": {
        "term" : { "avaliable" : "true" }
      },
      "must_not" : {
        "range" : {
          "price" : { "lte" : 10 }
        }
      },
      "should" : [
        { "term" : { "productID.keyword" : "JODL-X-1937-#pV7" } },
        { "term" : { "productID.keyword" : "XHDK-A-1293-#fJ3" } }
      ],
      "minimum_should_match" :1
    }
  }
}

#改变数据模型，增加字段。解决数组包含而不是精确匹配的问题
POST /newmovies/_bulk
{ "index": { "_id": 1 }}
{ "title" : "Father of the Bridge Part II","year":1995, "genre":"Comedy","genre_count":1 }
{ "index": { "_id": 2 }}
{ "title" : "Dave","year":1993,"genre":["Comedy","Romance"],"genre_count":2 }

#must，有算分
POST /newmovies/_search
{
  "query": {
    "bool": {
      "must": [
        {"term": {"genre.keyword": {"value": "Comedy"}}},
        {"term": {"genre_count": {"value": 1}}}

      ]
    }
  }
}

#Filter。不参与算分，结果的score是0
POST /newmovies/_search
{
  "query": {
    "bool": {
      "filter": [
        {"term": {"genre.keyword": {"value": "Comedy"}}},
        {"term": {"genre_count": {"value": 1}}}
        ]

    }
  }
}


#Filtering Context
POST _search
{
  "query": {
    "bool" : {

      "filter": {
        "term" : { "avaliable" : "true" }
      },
      "must_not" : {
        "range" : {
          "price" : { "lte" : 10 }
        }
      }
    }
  }
}


#Query Context
POST /products/_bulk
{ "index": { "_id": 1 }}
{ "price" : 10,"avaliable":true,"date":"2018-01-01", "productID" : "XHDK-A-1293-#fJ3" }
{ "index": { "_id": 2 }}
{ "price" : 20,"avaliable":true,"date":"2019-01-01", "productID" : "KDKE-B-9947-#kL5" }
{ "index": { "_id": 3 }}
{ "price" : 30,"avaliable":true, "productID" : "JODL-X-1937-#pV7" }
{ "index": { "_id": 4 }}
{ "price" : 30,"avaliable":false, "productID" : "QQPX-R-3956-#aD8" }


POST /products/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "productID.keyword": {
              "value": "JODL-X-1937-#pV7"}}
        },
        {"term": {"avaliable": {"value": true}}
        }
      ]
    }
  }
}


#嵌套，实现了 should not 逻辑
POST /products/_search
{
  "query": {
    "bool": {
      "must": {
        "term": {
          "price": "30"
        }
      },
      "should": [
        {
          "bool": {
            "must_not": {
              "term": {
                "avaliable": "false"
              }
            }
          }
        }
      ],
      "minimum_should_match": 1
    }
  }
}


#Controll the Precision
POST _search
{
  "query": {
    "bool" : {
      "must" : {
        "term" : { "price" : "30" }
      },
      "filter": {
        "term" : { "avaliable" : "true" }
      },
      "must_not" : {
        "range" : {
          "price" : { "lte" : 10 }
        }
      },
      "should" : [
        { "term" : { "productID.keyword" : "JODL-X-1937-#pV7" } },
        { "term" : { "productID.keyword" : "XHDK-A-1293-#fJ3" } }
      ],
      "minimum_should_match" :2
    }
  }
}



POST /animals/_search
{
  "query": {
    "bool": {
      "should": [
        { "term": { "text": "brown" }},
        { "term": { "text": "red" }},
        { "term": { "text": "quick"   }},
        { "term": { "text": "dog"   }}
      ]
    }
  }
}

POST /animals/_search
{
  "query": {
    "bool": {
      "should": [
        { "term": { "text": "quick" }},
        { "term": { "text": "dog"   }},
        {
          "bool":{
            "should":[
               { "term": { "text": "brown" }},
                 { "term": { "text": "brown" }},
            ]
          }

        }
      ]
    }
  }
}


DELETE blogs
POST /blogs/_bulk
{ "index": { "_id": 1 }}
{"title":"Apple iPad", "content":"Apple iPad,Apple iPad" }
{ "index": { "_id": 2 }}
{"title":"Apple iPad,Apple iPad", "content":"Apple iPad" }


POST blogs/_search
{
  "query": {
    "bool": {
      "should": [
        {"match": {
          "title": {
            "query": "apple,ipad",
            "boost": 1.1
          }
        }},

        {"match": {
          "content": {
            "query": "apple,ipad",
            "boost":2
          }
        }}
      ]
    }
  }
}

DELETE news
POST /news/_bulk
{ "index": { "_id": 1 }}
{ "content":"Apple Mac" }
{ "index": { "_id": 2 }}
{ "content":"Apple iPad" }
{ "index": { "_id": 3 }}
{ "content":"Apple employee like Apple Pie and Apple Juice" }


POST news/_search
{
  "query": {
    "bool": {
      "must": {
        "match":{"content":"apple"}
      }
    }
  }
}

POST news/_search
{
  "query": {
    "bool": {
      "must": {
        "match":{"content":"apple"}
      },
      "must_not": {
        "match":{"content":"pie"}
      }
    }
  }
}

# 通过 boosting Query 提高准确度
POST news/_search
{
  "query": {
    "boosting": {
      "positive": {
        "match": {
          "content": "apple"
        }
      },
      "negative": {
        "match": {
          "content": "pie"
        }
      },
      "negative_boost": 0.5
    }
  }
}

```
