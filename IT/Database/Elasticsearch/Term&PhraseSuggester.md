# Term&Phrase Suggester

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

