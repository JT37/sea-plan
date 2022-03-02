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

# 多fields
GET /movies/_search
{
    "profile": true,
    "query":{
        "query_string":{
            "fields":[
            "title",
                "year"
            ],
            "query": "2012"
        }
    }
}

GET /movies/_search
{
    "profile":true,
    "query":{
        "simple_query_string":{
            "query":"Beautiful+mind",
            "fields":["title"]
        }
    }
}

```

## Dynamic Mapping 和常见字段类型

### 什么是 Mapping

- `Mappping` 类似数据库中的 `schema` 的定义，作用如下
  - 定义索引中字段的名称
  - 定义字段的数据类型，例如字符串，数字，布尔...
  - 字段，倒排索引的相关配置（`Analyzed or Not Analyzed, Analyzer`）
- `Mapping` 会把 `JSON` 文档映射成 `Lucene` 所需要的扁平格式
- 一个 `Mapping` 属于一个索引的 `Type`
  - 每个文档都属于一个 `Type`
  - 一个 `Type` 有一个 `Mapping` 定义
  - `7.0` 开始，不需要在 `Mapping` 定义中指定 `Type` 信息

### 字段的数据类型

- 简单类型
  - `Text / Keyword`
  - `Date`
  - `Integer / Floating`
  - `Boolean`
  - `IPv4 & IPv6`
- 复杂类型：对象和嵌套对象
  - 对象类型 / 嵌套类型
- 特殊类型
  - `geo_point & geo_shape / percolator`

### 什么是 Dynamic Mapping

- 在写入文档时候，如果索引不存在，会自动创建索引
- `Dynamic Mapping` 的机制，使得我们无需手动定义 `Mappings`。`Elasticsearch` 会自动根据文档信息，推算出字段的类型
- 但是有时候推算的不对，例如地理位置信息
- 当类型如果设置不对时，会导致一些功能无法正常运行，例如 `Range` 查询

![类型的自动识别](./images/类型的自动识别.png)

```curl
#写入文档，查看 Mapping
PUT mapping_test/_doc/1
{
  "firstName":"Chan",
  "lastName": "Jackie",
  "loginDate":"2018-07-24T10:29:48.103Z"
}

#查看 Mapping文件
GET mapping_test/_mapping


#Delete index
DELETE mapping_test

#dynamic mapping，推断字段的类型
PUT mapping_test/_doc/1
{
    "uid" : "123",
    "isVip" : false,
    "isAdmin": "true",
    "age":19,
    "heigh":180
}

#查看 Dynamic
GET mapping_test/_mapping
```

### 能否更改 Mapping 的字段类型

- 两种情况
  - 新增加字段
    - `Dynamic` 设为 `True` 时，一旦有新增字段的文档写入， `Mapping` 也同时被更新
    - `Dynamic` 设为 `False`，`Mapping` 不会被更新，新增字段的数据无法索引，但是信息会出现在 `_source` 中
  - 对已有字段，一旦已经有数据写入，就不再支持修改字段定义
    - `Lucene` 实现的倒排索引，一旦生成后，就不允许修改
  - 如果希望改编字段类型，必须 `Reindex API`，重建索引
- 原因
  - 如果修改了字段的数据类型，会导致已被索引的属于无法被搜索
  - 但是如果是增加新的字段，就不会有这样的影响

```curl
# 设置逻辑演示代码
#默认Mapping支持dynamic，写入的文档中加入新的字段
PUT dynamic_mapping_test/_doc/1
{
  "newField":"someValue"
}

#该字段可以被搜索，数据也在_source中出现
POST dynamic_mapping_test/_search
{
  "query":{
    "match":{
      "newField":"someValue"
    }
  }
}

#修改为dynamic false
PUT dynamic_mapping_test/_mapping
{
  "dynamic": false
}

#新增 anotherField
PUT dynamic_mapping_test/_doc/10
{
  "anotherField":"someValue"
}

#该字段不可以被搜索，因为dynamic已经被设置为false
POST dynamic_mapping_test/_search
{
  "query":{
    "match":{
      "anotherField":"someValue"
    }
  }
}
# ID 可以查到
GET dynamic_mapping_test/_doc/10

#修改为strict
PUT dynamic_mapping_test/_mapping
{
  "dynamic": "strict"
}

#写入数据出错，HTTP Code 400
PUT dynamic_mapping_test/_doc/12
{
  "lastField":"value"
}

DELETE dynamic_mapping_test
```
