# ES Query&Filtering与多字符串多字段查询

## Query Context & Filter Context

- 高级搜索的功能：支持多项文本输入，针对多个字段进行搜索
- 搜索引擎一般也是提供基于时间，价格等条件的过滤
- 在 `ES` 中，有 `Query` 和 `Filter` 两种不同的 Context
  - `Query Context`：相关性算分
  - `Filter Context`：不需要算分（`Yes or No`），可以利用 `Cache`，获得更好的性能
