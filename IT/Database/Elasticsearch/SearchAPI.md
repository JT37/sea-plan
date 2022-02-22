# Search API

## URI Search

- 在 `URL` 中使用查询参数
- 使用 “`q`”，指定查询字符串
- ”`query string syntax`“，`KV` 键值对
- `df` 默认字段，不指定时会对所有字段进行查询
- `sort` 排序，`from` 和 `size` 用于分页
- `profile` 可以查看查询时是如何被执行的
- `timeout` 查询超时时间
- 指定字段 vs 泛查询
  - q=title:2022 | q=2022
- 

```curl
curl -XGET "http://elasticsearch:9200/kibana_sample_data_ecommerce/_search?q=customer_first_name:Eddie"

#URI Query
GET kibana_sample_data_ecommerce/_search?q=customer_first_name:Eddie
GET kibana*/_search?q=customer_first_name:Eddie
GET /_all/_search?q=customer_first_name:Eddie
```

```curl
#基本查询
GET /movies/_search?q=2012&df=title&sort=year:desc&from=0&size=10&timeout=1s

#带profile
GET /movies/_search?q=2012&df=title
{
  "profile":"true"
}

#泛查询，不带df，对比查询详情
GET /movies/_search?q=2012
{
  "profile":"true"
}

#指定字段
GET /movies/_search?q=title:2012&sort=year:desc&from=0&size=10&timeout=1s
{
  "profile":"true"
}



```

## Request Body Search

- 使用 `ES` 提供的，基于 `JSON` 格式的更加完备的 `Query Domain Specific Language (DSL)`

```curl
# 支持 Get 和 POST
curl -XGET "http://elasticsearch:9200/kibana_sample_data_ecommerce/_search" -H 'Content-Type:application/json' -d '{"profile":true,"query":{"match_all":{}}}'

#REQUEST Body
POST kibana_sample_data_ecommerce/_search
{
	"profile": true,
	"query": {
		"match_all": {}
	}
}
```

## 指定查询的索引

|   语法   |   范围   |
| ---- | ---- |
|   /_search   |   集群上所有的索引   |
|   /index1/_search   |   index1    |
| /index1,index2/_search | index1 和 index2 |
| /index*/_search | 以index开头的索引 |

## 搜索的相关性 Relevance

- 搜索是用户和搜索引擎的对话
- 用户关心的是搜索结果的相关性
  - 是否可以找到所有相关的内容
  - 有多少不相关的内容被返回了
  - 文档的打分是否合理
  - 结合业务需求，平衡结果排名

## 衡量相关性

- `Information Retrieval`
  - `Precision`（查准率）：尽可能返回较少的无关文档
  - `Recall`（查全率）：尽量返回较多的相关文档
  - `Ranking`：是否能够按照相关度进行排序
