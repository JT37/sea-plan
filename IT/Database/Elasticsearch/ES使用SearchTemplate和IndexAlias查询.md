# 使用 SearchTemplate 和 IndexAlias 查询

## Search Template：解耦程序 & 搜索DSL

- ES 的查询语句
  - 对相关性算分 / 查询性能都至关重要
- 在开发初期，虽然可以明确查询参数，但是往往还不能最终定义查询的 DSL 的具体结构
  - 通过 Search Template 定义一个 Contract
- 各司其职，解耦
  - 开发人员、搜索工程师、性能工程师

<https://www.elastic.co/guide/en/elasticsearch/reference/current/search-template.html>

## Index Alias 实现零停机运维

- 为索引指定一个别名
- 通过别名读写数据

<https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-aliases.html>

```curl
PUT movies-2019/_doc/1
{
  "name":"the matrix",
  "rating":5
}

PUT movies-2019/_doc/2
{
  "name":"Speed",
  "rating":3
}

POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "movies-2019",
        "alias": "movies-latest"
      }
    }
  ]
}

POST movies-latest/_search
{
  "query": {
    "match_all": {}
  }
}

POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "movies-2019",
        "alias": "movies-lastest-highrate",
        "filter": {
          "range": {
            "rating": {
              "gte": 4
            }
          }
        }
      }
    }
  ]
}

POST movies-lastest-highrate/_search
{
  "query": {
    "match_all": {}
  }
}


PUT movies-lastest-highrate/_doc/3
{
  "name":"gogo",
  "rating":6
}

PUT movies-2020/_doc/1
{
  "name":"Speed111",
  "rating":3
}

PUT movies-2020/_doc/2
{
  "name":"Speed222",
  "rating":7
}


POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "movies-2020",
        "alias": "movies-latest"
      }
    }
  ]
}


POST movies-latest/_search
{
  "query": {
    "match_all": {}
  }
}

# 两个 index 同一个别名，都可以查询，但插入需要设置参数 is_write_index
PUT movies-latest/_doc/1
{
  "name":"Speed333",
  "rating":9
}
```
