# Suggester API

## ES Suggester API

- 搜索引擎中类似的功能，在 `ES` 中是通过 `Suggester API` 实现的
- 原理：将输入的文本分解为 `Token`，然后在索引的字典里查找相似的 `Term` 并返回
- 根据不同的场景，ES 设计了 `4` 种类别的 `Suggesters`
  - `Term & Phrase Suggester`
  - `Complete & Context Suggester`

## Term Suggester

- `Suggester` 就是一种特殊类型的搜索。`text` 里是调用时候提供的文本，通常来自于用户界面上用户输入的内容
- 用户输入的 `Lucen` 是一个错误的拼写
- 会到指定的字段 `body` 上搜索，当无法搜索到结果时（`missing`），返回建议的词
- 几种 Suggestion Mode
  - Missing：如索引中存在，就不提供建议
  - Popular：推荐出现频率更加高的词
  - Always：无论是否存在，都提供建议

```curl

DELETE articles

POST articles/_bulk
{ "index" : { } }
{ "body": "lucene is very cool"}
{ "index" : { } }
{ "body": "Elasticsearch builds on top of lucene"}
{ "index" : { } }
{ "body": "Elasticsearch rocks"}
{ "index" : { } }
{ "body": "elastic is the company behind ELK stack"}
{ "index" : { } }
{ "body": "Elk stack rocks"}
{ "index" : {} }
{  "body": "elasticsearch is rock solid"}
{ "index" : {} }
{  "body": "elasticsearch is rocke solid1"}
{ "index" : {} }
{  "body": "elasticsearch is rocke solid2"}
{ "index" : {} }
{  "body": "elasticsearch is rocke solid2"}
POST _analyze
{
  "analyzer": "standard",
  "text": ["Elk stack  rocks rock"]
}

POST /articles/_search
{
  "size": 1,
  "query": {
    "match": {
      "body": "lucen rock"
    }
  },
  "suggest": {
    "term-suggestion": {
      "text": "lucen rock",
      "term": {
        "suggest_mode": "missing",
        "field": "body"
      }
    }
  }
}


POST /articles/_search
{

  "suggest": {
    "term-suggestion": {
      "text": "lucen rock",
      "term": {
        "suggest_mode": "popular",
        "field": "body"
      }
    }
  }
}


POST /articles/_search
{
  "suggest": {
    "term-suggestion": {
      "text": "lucen rock",
      "term": {
        "suggest_mode": "always",
        "field": "body"
      }
    }
  }
}


POST /articles/_search
{

  "suggest": {
    "term-suggestion": {
      "text": "lucen hocks",
      "term": {
        "suggest_mode": "always",
        "field": "body",
        "prefix_length":0,
        "sort": "frequency"
      }
    }
  }
}

POST /articles/_search
{

  "suggest": {
    "term-suggestion": {
      "text": "lucen hocke",
      "term": {
        "suggest_mode": "always",
        "field": "body",
        "prefix_length":0,
        "sort": "frequency"
      }
    }
  }
}
```

## Phrase Suggester

- `Phrase Suggester` 在 `Term Suggester` 上增加了一些额外的逻辑
- 一些参数
  - `Suggest Mode`: `missing,popular,always`
  - `Max Errors`: 最多可以拼错的 `terms` 数
  - `Confidence`: 限制返回结果数，默认为 `1`

```curl
# confidence 限制返回结果数
POST /articles/_search
{
  "suggest": {
    "my-suggestion": {
      "text": "lucne and elasticsear rock hello world ",
      "phrase": {
        "field": "body",
        "max_errors":2,
        "confidence":2,
        "direct_generator":[{
          "field":"body",
          "suggest_mode":"always"
        }],
        "highlight": {
          "pre_tag": "<em>",
          "post_tag": "</em>"
        }
      }
    }
  }
}

POST /articles/_search
{
  "suggest": {
    "my-suggestion": {
      "text": "lucne and elasticsear rock hello world ",
      "phrase": {
        "field": "body",
        "max_errors":2,
        "confidence":0,
        "direct_generator":[{
          "field":"body",
          "suggest_mode":"always"
        }],
        "highlight": {
          "pre_tag": "<em>",
          "post_tag": "</em>"
        }
      }
    }
  }
}
```

## 自动补全：The Completion Suggester

- `Completion Suggester` 提供了 自动完成（`Auto Complete`）的功能，用户每输入一个字符，就需要即时发送一个查询请求到后端查找匹配项
- 对性能要求比较苛刻。`ES` 采用了不同的数据结构，并非通过倒排索引来完成。而是将 `Analyze` 的数据编码成 `FST` 和索引一起存放。`FST` 会被 `ES` 整个加载进内存，速度很快
- `FST` 只能用于前缀查找

### 使用 Completion Suggester 的一些步骤

- 定义 `Mapping`，使用 `“completion” type`
- 索引数据
- 运行 `suggest` 查询，得到搜索建议

```curl
DELETE articles
PUT articles
{
  "mappings": {
    "properties": {
      "title_completion":{
        "type": "completion"
      }
    }
  }
}

POST articles/_bulk
{ "index" : { } }
{ "title_completion": "lucene is very cool"}
{ "index" : { } }
{ "title_completion": "Elasticsearch builds on top of lucene"}
{ "index" : { } }
{ "title_completion": "Elasticsearch rocks"}
{ "index" : { } }
{ "title_completion": "elastic is the company behind ELK stack"}
{ "index" : { } }
{ "title_completion": "Elk stack rocks"}
{ "index" : {} }

# prefix 切换值 el 、elk
POST articles/_search?pretty
{
  "size": 0,
  "suggest": {
    "article-suggester": {
      "prefix": "el ",
      "completion": {
        "field": "title_completion"
      }
    }
  }
}
```

## 基于上下文的提示：Context Suggester

- 上下文感知
- `Completion Suggester` 的扩展
- 可以在搜索中加入更多的上下文信息，例如，输入“`star`”
  - 咖啡相关：建议 “`starbucks`”
  - 电影相关：建议 “`star wars`”

### 实现 Context Suggester

- 可以定义两种类型的 `Context`
  - `Category`：任意的字符串
  - `GEO`：地理位置信息
- 实现 `Context Suggester` 的具体步骤
  - 定制一个 `Mapping`
  - 索引数据，并且为每个文档加入 `Context` 信息
  - 结合 `Context` 进行 `Suggestion` 查询

```curl
DELETE comments
PUT comments
PUT comments/_mapping
{
  "properties": {
    "comment_autocomplete":{
      "type": "completion",
      "contexts":[{
        "type":"category",
        "name":"comment_category"
      }]
    }
  }
}

POST comments/_doc
{
  "comment":"I love the star war movies",
  "comment_autocomplete":{
    "input":["star wars"],
    "contexts":{
      "comment_category":"movies"
    }
  }
}

POST comments/_doc
{
  "comment":"Where can I find a Starbucks",
  "comment_autocomplete":{
    "input":["starbucks"],
    "contexts":{
      "comment_category":"coffee"
    }
  }
}

# 调整comment_category的值：coffee/movies，查看搜索结果
POST comments/_search
{
  "suggest": {
    "MY_SUGGESTION": {
      "prefix": "sta",
      "completion":{
        "field":"comment_autocomplete",
        "contexts":{
          "comment_category":"coffee"
        }
      }
    }
  }
}
```

## 精准度和召回率

- 精准度
  - `Completion > Phrase > Term`
- 召回率
  - `Term > Phrase > Completion`
- 性能
  - `Completion > Phrase > Term`

## 官方文档

<https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters.html>
