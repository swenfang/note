---
tags:
	- lucene
categories: lucene
title: lucene搜索之facet查询查询示例（2）
---
# lucene（16）---lucene搜索之facet查询查询示例（2）

lucene（14）---lucene搜索之facet索引原理和facet查询实例，上篇主要是统计facet的dim和每个种类对应的数量，个人感觉这个跟lucene的group不同的在于facet的存储类似于hash（key-field-value）形式的，而group则是单一的map（key-value）形式的，虽然都可以统计某一品类的数量，显然facet更具扩展性。

<!--more-->

## key-field-value查询

facet可以对某一个维度的满足某个条件的结果进行统计，如下：

```java
 @Test
 public void testDrillDownSlide(){
  try {
   DirectoryReader indexReader = DirectoryReader.open(directory);
   IndexSearcher searcher = new IndexSearcher(indexReader);
   DrillSideways ds = new DrillSideways(searcher, config, taxoReader);
   DrillDownQuery ddq = new DrillDownQuery(config);
   ddq.add("filePath", "ik");
   DrillSidewaysResult r = ds.search(ddq, 10);
   TopDocs hits = r.hits;
   for (ScoreDoc scoreDoc : hits.scoreDocs) {
    Document doc = searcher.doc(scoreDoc.doc);
    System.out.println(doc.get("path"));
   }
  } catch (Exception e) {
   e.printStackTrace();
  }
 }
```

这里我们搜索的dim是filePath，查找的范围是ik相关联的数据，对应的查询结果就是所有包含在IK文件夹下的数据

```
C:\Users\lenovo\Desktop\lucene\IK-Analyzer-2012FF\dist\IKAnalyzer.cfg.xml
C:\Users\lenovo\Desktop\lucene\IK-Analyzer-2012FF\dist\IKAnalyzer2012FF_u1.jar
C:\Users\lenovo\Desktop\lucene\IK-Analyzer-2012FF\dist\IKAnalyzer2015.jar
C:\Users\lenovo\Desktop\lucene\IK-Analyzer-2012FF\dist\LICENSE.txt
C:\Users\lenovo\Desktop\lucene\IK-Analyzer-2012FF\dist\NOTICE.txt
C:\Users\lenovo\Desktop\lucene\IK-Analyzer-2012FF\dist\stopword.dic
C:\Users\lenovo\Desktop\lucene\IK-Analyzer-2012FF\doc\allclasses-frame.html
C:\Users\lenovo\Desktop\lucene\IK-Analyzer-2012FF\doc\allclasses-noframe.html
C:\Users\lenovo\Desktop\lucene\IK-Analyzer-2012FF\doc\constant-values.html
C:\Users\lenovo\Desktop\lucene\IK-Analyzer-2012FF\doc\deprecated-list.html
```

## range查询

facet还支持range查询，range查询的类型包括DoubleRange和LongRange；其对应的Facets为DoubleRangeFacets和LongRangeFacets;

以LongRangeFacetCounts为例，LongRangeFacetCounts可以对long类型的数值进行整理查询

这里我们对每个文档的单词数量进行区间的分组，range查询示例如下：

```java
	@Test
	public void testOverlappedEndStart(){
		try {
			IndexReader reader = DirectoryReader.open(directory);
			FacetsCollector fc = new FacetsCollector();
		    IndexSearcher s = new IndexSearcher(reader);
		    s.search(new MatchAllDocsQuery(), fc);
		    Facets facets = new LongRangeFacetCounts("contentLength", fc,
		            new LongRange("0-100", 0L, true, 100L, true),
		            new LongRange("100-200", 100L, true, 200L, true),
		            new LongRange("200-300", 200L, true, 300L, true),
		            new LongRange("300-400", 300L, true, 400L, true));
		    FacetResult result = facets.getTopChildren(10, "contentLength");
		    System.out.println(result.toString());
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
```

其执行结果为：

```
dim=contentLength path=[] value=22 childCount=4
  0-100 (7)
  100-200 (9)
  200-300 (3)
  300-400 (3)
```

## 多个dim查询

facet里DrillSideways可以定义多个facetCount的查询，这时返回的结果为各个facet对应的统计数

```java
	@Test
	public void testMixedRangeAndNonRangeTaxonomy(){
		try {
			IndexReader reader = DirectoryReader.open(directory);
		    IndexSearcher s = new IndexSearcher(reader);
		    DrillSideways ds = new DrillSideways(s, config, taxoReader){
		        @Override
		        protected Facets buildFacetsResult(FacetsCollector drillDowns, FacetsCollector[] drillSideways, String[] drillSidewaysDims) throws IOException {        
		          FacetsCollector fieldFC = drillDowns;
		          FacetsCollector dimFC = drillDowns;
		          if (drillSideways != null) {
		            for(int i=0;i<drillSideways.length;i++) {
		              String dim = drillSidewaysDims[i];
		              if (dim.equals("contentLength")) {
		                fieldFC = drillSideways[i];
		              } else {
		            	dimFC = drillSideways[i];
		              }
		            }
		          }
		          Map<String,Facets> byDim = new HashMap<String,Facets>();
		          byDim.put("contentLength",new LongRangeFacetCounts("contentLength", fieldFC,
		                          new LongRange("less than 100", 0L, true, 100L, false),
		                          new LongRange("between 100 and 200", 100L, true, 200L, false),
		                          new LongRange("over 200", 200L, true, Integer.MAX_VALUE, false)));
		          byDim.put("dim", new FastTaxonomyFacetCounts(taxoReader, config, dimFC));
		          return new MultiFacets(byDim);
		        }
		        
		        @Override
		        protected boolean scoreSubDocsAtOnce() {
		          return false;
		        }
		    };
		    DrillDownQuery ddq = new DrillDownQuery(config);
		    DrillSidewaysResult dsr = ds.search(ddq, 10);
		    Facets facet = dsr.facets;
		    List<FacetResult> results = facet.getAllDims(reader.maxDoc());
		    for (FacetResult facetResult : results) {
				System.out.println(facetResult.dim);
				LabelAndValue[] values = facetResult.labelValues;
				for (LabelAndValue labelAndValue : values) {
					System.out.println("\t"+labelAndValue.label +"       "+labelAndValue.value);
				}
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
```

程序运行结果如下：

```
dim
	odd       126
	even       119
contentLength
	less than 100       7
	between 100 and 200       9
	over 200       229  
```

## 对单个range的列表查询支持

facet支持单个range的区间查询，这样可以查询出此range对饮的TopDocs列表，等同于返回了document对象列表；

这里我们查询内容长度在0到100之间的数据

```java
	@Test
	public void testDrillDownQueryWithRange(){
		try {
			IndexReader reader = DirectoryReader.open(directory);
		    IndexSearcher s = new IndexSearcher(reader);
		    DrillDownQuery ddq = new DrillDownQuery(config);
		    ddq.add("contentLength", NumericRangeQuery.newLongRange("contentLength", 0l, 100l, true, false));//;
		    TopDocs docs = s.search(ddq, reader.maxDoc());
		    System.out.println("查询到的数据总数："+docs.totalHits);
		    for (ScoreDoc scoreDoc : docs.scoreDocs) {
				System.out.println(s.doc(scoreDoc.doc).get("path"));
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
```

其运行结果如下：

```
查询到的数据总数：7
C:\Users\lenovo\Desktop\lucene\jcseg\DONATE.txt
C:\Users\lenovo\Desktop\lucene\jcseg\jcseg-elasticsearch\src\main\resources\es-plugin.properties
C:\Users\lenovo\Desktop\lucene\jcseg\lexicon\lex-autoload.todo
C:\Users\lenovo\Desktop\lucene\jcseg\lexicon\lex-en-pun.lex
C:\Users\lenovo\Desktop\lucene\jcseg\lexicon\lex-ln-adorn.lex
C:\Users\lenovo\Desktop\lucene\IK-Analyzer-2012FF\doc\resources\inherit.gif
C:\Users\lenovo\Desktop\lucene\IK-Analyzer-2012FF\src\ext.dic
```

本节内容都是示例，个人觉得这种会比较直观些，facet涉及的面比较广，这里没有facet的sort和其他相关操作，会在后续补上，希望大家持续关注。﻿﻿ 