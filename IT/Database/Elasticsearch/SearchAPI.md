# Search API

## URI Search

- 在 `URL` 中使用查询参数
- 使用 “`q`”，指定查询字符串
- ”`query string syntax`“，`KV` 键值对
- `df` 默认字段，不指定时会对所有字段进行查询
- `sort` 排序，`from` 和 `size` 用于分页
- `profile` 可以查看查询时是如何被执行的
- `timeout` 查询超时时间
- 指定字段 `vs` 泛查询
  - `q=title:2022 | q=2022`
- `Term vs Phrase`
  - `Beautiful Mind` 等效于 `Beautiful OR Mind`
  - `"Beautiful Mind"` 等效于 `Beautiful AND Mind`。`Phrase` 查询还要求前后顺序一致
- 分组与引号
  - `title:(Beautiful AND Mind)`
  - `title:"Beautiful Mind"`
- 布尔操作
  - `AND/OR/NOT` 或者  `&&/||/!`
  - `title:(matrix NOT reload)`
- 分组操作
  - `+` 表示 `must`
  - `-` 表示 `must_not`
  - title:(+matrix  -reload)
- 范围操作
  - 区间表示：`[]` 表示闭区间，`{}` 表示开区间
    - `year:{2019 TO 2018}`
    - `year:[* TO 2018]`
  - 算数符号
    - `year:>2010`
    - `year:(>2010 && <=2018)`
    - `year:(+>2010 +<=2018)`
- 通配符查询（查询效率低，占用内存大，不建议使用）
  - `?` 代表 `1` 个字符，`*` 号代表 `0` 或者多个字符
    - `title:m?nd`
    - `title:be*`
- 正则表达
  - `title:[bt]oy`
- 模糊匹配与近似查询
  - `title:beautifl~1`
  - `title:"long rings"~2`

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


#泛查询，正对_all,所有字段
GET /movies/_search?q=2012
{
  "profile":"true"
}

#指定字段
GET /movies/_search?q=title:2012&sort=year:desc&from=0&size=10&timeout=1s
{
  "profile":"true"
}


# 查找美丽心灵, Mind为泛查询
GET /movies/_search?q=title:Beautiful Mind
{
  "profile":"true"
}

# 泛查询
GET /movies/_search?q=title:2012
{
  "profile":"true"
}

#使用引号，Phrase查询
GET /movies/_search?q=title:"Beautiful Mind"
{
  "profile":"true"
}

#分组，Bool查询
GET /movies/_search?q=title:(Beautiful Mind)
{
  "profile":"true"
}


#布尔操作符
# 查找美丽心灵
GET /movies/_search?q=title:(Beautiful AND Mind)
{
  "profile":"true"
}

# 查找美丽心灵
GET /movies/_search?q=title:(Beautiful NOT Mind)
{
  "profile":"true"
}

# 查找美丽心灵
GET /movies/_search?q=title:(Beautiful %2BMind)
{
  "profile":"true"
}


#范围查询 ,区间写法
GET /movies/_search?q=title:beautiful AND year:[2002 TO 2018%7D
{
  "profile":"true"
}


#通配符查询
GET /movies/_search?q=title:b*
{
  "profile":"true"
}

#模糊匹配&近似度匹配
# 参数范围只有 0，1，2
GET /movies/_search?q=title:bsautifl~2
{
  "profile":"true"
}

GET /movies/_search?q=title:"Lord Rings"~2
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
