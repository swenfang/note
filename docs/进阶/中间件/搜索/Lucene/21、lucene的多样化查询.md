---
tags:
	- lucene
categories: lucene
title: lucene的多样化查询
---
# lucene（21）---lucene的多样化查询

| 查询类            | 说明                                   |
| ----------------- | -------------------------------------- |
| TermQuery         | 通过项进行搜索                         |
| TermRangeQuery    | 在指定的项范围内进行搜索               |
| PrefixQuery       | 通过字符串搜索                         |
| BooleanQuery      | 组合查询                               |
| PhraseQuery       | 通过短语搜索                           |
| WildcardQuery     | 通配符查询                             |
| FuzzyQuery        | 搜索类似项                             |
| MatchAllDocsQuery | 匹配所有文档                           |
| MatchNoDocsQuery  | 不用匹配文档                           |
| QueryParser       | 解析查询表达式                         |
| MultiPhraseQuery  | 多短语查询                             |
| NumericRangeQuery | 数字范围查询，一般在价格、时间域的查询 |

<!--more-->

在 Lucene4 以后，组合查询只有一个构造方法，并没有无参构造方法，而是多了一个静态内部类 Builder。所以组合查如下：

```java
Query booleanQuery = new BooleanQuery.Builder().add(query1,BooleanClause.Occur.MUST).add(query1,BooleanClause.Occur.MUST).build();
```

BooleanClause.Occur 提供了一下四种：

| 名称     | 作用                |
| -------- | ------------------- |
| MUST     | 相当于 SQL 中的 and |
| FILTER   |                     |
| SHOULD   | 相当于 SQL 中的 in  |
| MUST_NOT |                     |

当它们同事使用的情况：

## 高级搜索

lucene 包含了 一个建立在 SpanQuery 类基础上的整套查询体系，大致反映了 Lucene 的 Query 类体系。SpanQuery 是指域中的起始词汇单元和终止词汇单元的位置。SpanQuery 有一些常用的子类，如下所示：

### FieldMaskingSpanQuery

用于在多个域之间查询，即把另一个域看成某个域，从而看起来像是在同个域中查询，因为 Lucene 默认某个条件只能作用在单个域上，不支持跨域查询，只能在同一个域中查询，所以有了FieldMaskingSpanQuery。

### SpanTermQuery

和其他跨度查询类型结合使用，单独使用时，相当于 Term,slop 为跨度因子，用来限制两个 Trem 之间的最大跨度。还有一个 inOrder 参数，它用来设置是否允许尽心倒叙跨度。即 TremA 到 TramB 不一定从左到右去匹配也可以从右到左，从右到左就是倒叙，inOrder 为 true 即表示 order (顺序) 很重要不能倒叙去匹配必须正向去匹配，false 返之。注意，停用词不在 slop 统计范围内

### SpanFirstQuery

表示对出现在一个域中的 [0,n] 范围内的 Term 项进行的匹配查询，关键是 n 指定了查询的 term 出现范围的上限。

### SpanContainingQuery

返回在另一个范围内的查询匹配结果，big 和 little 的句子可以是任何 span 类型查询。在包含 little 匹配中从 big 匹配跨度返回。例如 "a beautiful and boring world" , big 查询是 SpanNearQuery(SpanTermQuery("beautiful "),SpanTermQuery("world")).setSlop(2),而 little 查询是 SpanTermQuery("boring") ，则该 Doc 命中，并 big 匹配跨度返回，即 big 优先级高。 





