# ES 复杂查询

## Query String Query

- 类似于 `URI Query`

```curl
PUT /users/_doc/1
{
  "name":"Allen Zhang",
  "about":"java, golang, node, swift, elasticsearch"
}

PUT /users/_doc/2
{
  "name":"Hope Zhang",
  "about":"Hadoop"
}

POST users/_search
{
  "query": {
    "query_string": {
      "default_field": "name",
      "query": "Allen AND Zhang"
    }
  }
}


POST users/_search
{
  "query": {
    "query_string": {
      "fields":["name","about"],
      "query": "(Allen AND Zhang) OR (Java AND Elasticsearch)"
    }
  }
}

```

## Simple Query String Query

- 类似 `Query String`，但是会忽略错误的语法，同时只支持部分查询语法
- 不支持 `AND OR NOT`，会当作字符串处理
- `Term` 之间默认的关系是 `OR`，可以指定 `Operator(default_operator)`
- 支持部分逻辑
  - `+` 替代 `AND`
  - `|` 替代 `OR`
  - `-` 替代 `NOT`

```curl
#Simple Query 默认的operator是 Or
POST users/_search
{
  "query": {
    "simple_query_string": {
      "query": "Allen AND Zhang",
      "fields": ["name"]
    }
  }
}

POST users/_search
{
  "query": {
    "simple_query_string": {
      "query": "Allen Zhang",
      "fields": ["name"],
      "default_operator": "AND"
    }
  }
}

```
