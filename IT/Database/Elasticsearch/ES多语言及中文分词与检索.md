# ES 多语言及中文分词与检索

## 自然语言与查询 Recall

- 当处理人类自然语言时，有些情况，尽管搜索和原文不完全匹配，但是希望搜到一些内容
  - Quick brown fox 和 fast brown fox / Jumping fox 和 Jumped foxes
- 一些可采取的变化
  - 归一化词元：清除变音符号
  - 抽取词根：清除单复数和时态的差异
  - 包含同义词
  - 拼写错误或者同音异形词

## 混合多语言的挑战

- 一些具体的多语言场景
  - 不同的索引使用不同的语言 / 同一个索引中，不同的字段使用不同的语言 / 一个文档的一个字段内混合不同的语言
- 混合语言存在的一些挑战
  - 词干提取：以色列文档，包含了希伯来语，阿拉伯语，俄语和英文
  - 不正确的文档频率：英文为主的文章中，德文算分高（稀有）
  - 需要判断用户搜索时使用的语言，语言识别（`Compact Language Detector`）
    - 例如：根据语言，查询不同的索引

## 分词的挑战

- 英文分词：例如 `You're` 分成一个还是多个
- 中文分词：
  - 分词标准，具体情况需定制不同的标准
  - 歧义

## 中文分词器现状

- 中文分词器以统计语言为基础，经过几十年的发展，今天基本已经可以看作是一个已经解决的问题
- 不同分词器的好坏，主要差别在于数据的使用和工程使用的精度
- 常见的分词器都是使用机器学习算法和词典相结合，一方面能够提高分词准确率，另一方面能够改善领域适应性。

## 一些中文分词器

- `HanLP`：面向生产环境自然语言处理工具包
  - <https://github.com/hankcs/HanLP>
- `IK` 分词器
  - <https://github.com/medcl/elasticsearch-analysis-ik>
- `pinyin` 分词器
  - <https://github.com/medcl/elasticsearch-analysis-pinyin>

## 相关阅读

- 分词算法综述：<https://zhuanlan.zhihu.com/p/50444885>

- 中科院计算所NLPIR <http://ictclas.nlpir.org/nlpir/>
- ansj分词器 <https://github.com/NLPchina/ansj_seg>
- 哈工大的LTP <https://github.com/HIT-SCIR/ltp>
- 清华大学THULAC <https://github.com/thunlp/THULAC>
- 斯坦福分词器 <https://nlp.stanford.edu/software/segmenter.shtml>
- Hanlp分词器 <https://github.com/hankcs/HanLP>
- 结巴分词 <https://github.com/yanyiwu/cppjieba>
- KCWS分词器(字嵌入+Bi-LSTM+CRF) <https://github.com/koth/kcws>
- ZPar <https://github.com/frcchang/zpar/releases>
- IKAnalyzer <https://github.com/wks/ik-analyzer>
