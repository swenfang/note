---
tags:
	- lucene
categories: lucene
title: lucene搜索之分组处理group查询
---
# lucene（12）---lucene搜索之分组处理group查询

## grouping介绍

我们在做lucene搜索的时候，可能会用到对某个条件的数据进行统计，比如统计有多少个省份，在sql查询中我们可以用distinct来完成类似的功能，也可以用group by来对查询的列进行分组查询。在lucene中我们实现类似的功能怎么做呢，比较费时的做法时我们查询出所有的结果，然后对结果里边的省份对应的field查询出来，往set里边放，显然这种做法效率低，不可取；lucene为了解决上述问题，提供了用于分组操作的模块group，group主要用户处理不同lucene中含有某个相同field值的不同document的分组统计。

<!--more-->

Grouping可以接收如下参数：

- groupField：要分组的字段；比如我们对省份（province）进行分组，要传入对应的值为province，要注意的是如果groupField在document中不存在，会返回一个null的分组；
- groupSort：分组是怎么排序的，排序字段决定了分组内容展示的先后顺序；
- topNGroups：分组展示的数量，只计算0到topNGroup条记录；
- groupOffset：从第几个TopGroup开始算起，举例来说groupOffset为3的话，会展示从3到topNGroup对应的记录，此数值我们可以用于分页查询；
- withinGroupSort：每组内怎么排序；
- maxDocsPerGroup：每组处理多少个document；
- withinGroupOffset：每组显示的document初始位置；

group的实现需要两步：

- 第一步：利用TermFirstPassGroupingCollector来收集top groups；
- 第二步：用`TermSecondPassGroupingCollector处理每个group对应的documents`

`group模块定义了group和group的采集方式；所有的grouping colletor,所有的grouping collector都是抽象类并且提供了基于term的实现；`

`实现group的前提：`

- `要group的field必须是必须是SortedDocValuesField类型的；`
- `solr尽管也提供了grouping by的相关方法实现，但是对group的抽象实现还是由该模块实现；`
- `暂不支持sharding,我们需要自己提供groups和每个group的documents的合并`

## group示例

```java
package com.lucene.search;
 
import java.io.IOException;
 
import org.apache.lucene.search.IndexSearcher;
import org.apache.lucene.search.MatchAllDocsQuery;
import org.apache.lucene.search.Query;
import org.apache.lucene.search.Sort;
import org.apache.lucene.search.SortField;
import org.apache.lucene.search.grouping.GroupDocs;
import org.apache.lucene.search.grouping.GroupingSearch;
import org.apache.lucene.search.grouping.TopGroups;
import org.apache.lucene.util.BytesRef;
 
public class GroupSearchTest {
	public static void main(String[] args) {
		GroupingSearch groupingSearch = new GroupingSearch("province");
		SortField sortField = new SortField("city", SortField.Type.STRING_VAL);
		Sort sort = new Sort(sortField);
		groupingSearch.setGroupSort(sort);
		groupingSearch.setFillSortFields(true);
		groupingSearch.setCachingInMB(4.0, true);
		groupingSearch.setAllGroups(true);
		IndexSearcher searcher;
		try {
			searcher = SearchUtil.getIndexSearcherByIndexPath("index", null);
			Query query = new MatchAllDocsQuery();
			TopGroups<BytesRef> result = groupingSearch.search(searcher,query, 0, searcher.getIndexReader().maxDoc());
			// Render groupsResult...
			GroupDocs<BytesRef>[] docs = result.groups;
			for (GroupDocs<BytesRef> groupDocs : docs) {
				System.out.println(new String(groupDocs.groupValue.bytes));
			}
			int totalGroupCount = result.totalGroupCount;
			System.out.println(totalGroupCount);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
 
		
 
	}
 
}
```

利用BlockGroupingCollector

我们有时候想要在索引的时候就将group字段存入以方便search，我们可以在确保docs被索引的前提下，先查询出来每个要group的term对应的documents,然后在最后的document插入一个标记分组的field,我们可以如此做：

```java
/**带group的索引创建
  * @param writer
  * @param docs
  * @throws IOException 
  */
 public void indexDocsWithGroup(IndexWriter writer,String groupFieldName,String groupFieldValue,List<Document> docs) throws IOException{
   Field groupEndField = new Field(groupFieldName, groupFieldValue, Field.Store.NO, Field.Index.NOT_ANALYZED);
   docs.get(docs.size()-1).add(groupEndField);
   writer.updateDocuments(new Term(groupFieldName, groupFieldValue),docs);
   writer.commit();
   writer.close();
 }
```

在分组查询的时候，我们可以 

```java
/**group查询，适用于对group字段已经进行分段索引的情况
  * @param searcher
  * @param groupEndQuery
  * @param query
  * @param sort
  * @param withinGroupSort
  * @param groupOffset
  * @param topNGroups
  * @param needsScores
  * @param docOffset
  * @param docsPerGroup
  * @param fillFields
  * @return
  * @throws IOException
  */
 public static TopGroups<BytesRef> getTopGroupsByGroupTerm(IndexSearcher searcher,Query groupEndQuery,Query query,Sort sort,Sort withinGroupSort,int groupOffset,int topNGroups,boolean needsScores,int docOffset,int docsPerGroup,boolean fillFields) throws IOException{
  @SuppressWarnings("deprecation")
  Filter groupEndDocs = new CachingWrapperFilter(new QueryWrapperFilter(groupEndQuery));
  BlockGroupingCollector c = new BlockGroupingCollector(sort, groupOffset+topNGroups, needsScores, groupEndDocs);
  searcher.search(query, c);
  @SuppressWarnings("unchecked")
  TopGroups<BytesRef> groupsResult = (TopGroups<BytesRef>) c.getTopGroups(withinGroupSort, groupOffset, docOffset, docOffset+docsPerGroup, fillFields);
  return groupsResult;
 }
```

我们也可以直接进行group的查询，此为通用的实现

## 查询方法

```java
/**
	 * @param searcher
	 * @param query
	 * @param groupFieldName
	 * @param sort
	 * @param maxCacheRAMMB
	 * @param page
	 * @param perPage
	 * @return
	 * @throws IOException
	 */
	public static TopGroups<BytesRef> getTopGroups(IndexSearcher searcher,Query query,String groupFieldName,Sort sort,double maxCacheRAMMB,int page,int perPage) throws IOException{
		GroupingSearch groupingSearch = new GroupingSearch(groupFieldName);
		groupingSearch.setGroupSort(sort);
		groupingSearch.setFillSortFields(true);
		groupingSearch.setCachingInMB(maxCacheRAMMB, true);
		groupingSearch.setAllGroups(true);
		TopGroups<BytesRef> result = groupingSearch.search(searcher,query, (page-1)*perPage, page*perPage);
		return result;
	}
```

以下是查询的工具类

## 查询工具类

```java
package com.lucene.search;
 
import java.io.File;
import java.io.IOException;
import java.nio.file.Paths;
import java.util.Set;
import java.util.concurrent.ExecutorService;
 
import org.apache.lucene.analysis.Analyzer;
import org.apache.lucene.analysis.standard.StandardAnalyzer;
import org.apache.lucene.document.Document;
import org.apache.lucene.index.DirectoryReader;
import org.apache.lucene.index.IndexReader;
import org.apache.lucene.index.MultiReader;
import org.apache.lucene.index.Term;
import org.apache.lucene.queryparser.classic.ParseException;
import org.apache.lucene.queryparser.classic.QueryParser;
import org.apache.lucene.search.BooleanQuery;
import org.apache.lucene.search.CachingWrapperFilter;
import org.apache.lucene.search.Filter;
import org.apache.lucene.search.IndexSearcher;
import org.apache.lucene.search.MatchAllDocsQuery;
import org.apache.lucene.search.NumericRangeQuery;
import org.apache.lucene.search.Query;
import org.apache.lucene.search.QueryWrapperFilter;
import org.apache.lucene.search.ScoreDoc;
import org.apache.lucene.search.Sort;
import org.apache.lucene.search.SortField;
import org.apache.lucene.search.TermQuery;
import org.apache.lucene.search.TopDocs;
import org.apache.lucene.search.BooleanClause.Occur;
import org.apache.lucene.search.grouping.BlockGroupingCollector;
import org.apache.lucene.search.grouping.GroupDocs;
import org.apache.lucene.search.grouping.GroupingSearch;
import org.apache.lucene.search.grouping.TopGroups;
import org.apache.lucene.search.highlight.Highlighter;
import org.apache.lucene.search.highlight.InvalidTokenOffsetsException;
import org.apache.lucene.search.highlight.QueryScorer;
import org.apache.lucene.search.highlight.SimpleFragmenter;
import org.apache.lucene.search.highlight.SimpleHTMLFormatter;
import org.apache.lucene.store.FSDirectory;
import org.apache.lucene.util.BytesRef;
 
/**lucene索引查询工具类
 * @author lenovo
 *
 */
public class SearchUtil {
	/**获取IndexSearcher对象
	 * @param indexPath
	 * @param service
	 * @return
	 * @throws IOException
	 */
	public static IndexSearcher getIndexSearcherByParentPath(String parentPath,ExecutorService service) throws IOException{
		MultiReader reader = null;
		//设置
		try {
			File[] files = new File(parentPath).listFiles();
			IndexReader[] readers = new IndexReader[files.length];
			for (int i = 0 ; i < files.length ; i ++) {
				readers[i] = DirectoryReader.open(FSDirectory.open(Paths.get(files[i].getPath(), new String[0])));
			}
			reader = new MultiReader(readers);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		return new IndexSearcher(reader,service);
	}
	/**多目录多线程查询
	 * @param parentPath 父级索引目录
	 * @param service 多线程查询
	 * @return
	 * @throws IOException
	 */
	public static IndexSearcher getMultiSearcher(String parentPath,ExecutorService service) throws IOException{
		File file = new File(parentPath);
		File[] files = file.listFiles();
		IndexReader[] readers = new IndexReader[files.length];
		for (int i = 0 ; i < files.length ; i ++) {
			readers[i] = DirectoryReader.open(FSDirectory.open(Paths.get(files[i].getPath(), new String[0])));
		}
		MultiReader multiReader = new MultiReader(readers);
		IndexSearcher searcher = new IndexSearcher(multiReader,service);
		return searcher;
	}
	/**根据索引路径获取IndexReader
	 * @param indexPath
	 * @return
	 * @throws IOException
	 */
	public static DirectoryReader getIndexReader(String indexPath) throws IOException{
		return DirectoryReader.open(FSDirectory.open(Paths.get(indexPath, new String[0])));
	}
	/**根据索引路径获取IndexSearcher
	 * @param indexPath
	 * @param service
	 * @return
	 * @throws IOException
	 */
	public static IndexSearcher getIndexSearcherByIndexPath(String indexPath,ExecutorService service) throws IOException{
		IndexReader reader = getIndexReader(indexPath);
		return new IndexSearcher(reader,service);
	}
	
	/**如果索引目录会有变更用此方法获取新的IndexSearcher这种方式会占用较少的资源
	 * @param oldSearcher
	 * @param service
	 * @return
	 * @throws IOException
	 */
	public static IndexSearcher getIndexSearcherOpenIfChanged(IndexSearcher oldSearcher,ExecutorService service) throws IOException{
		DirectoryReader reader = (DirectoryReader) oldSearcher.getIndexReader();
		DirectoryReader newReader = DirectoryReader.openIfChanged(reader);
		return new IndexSearcher(newReader, service);
	}
	
	/**多条件查询类似于sql in
	 * @param querys
	 * @return
	 */
	public static Query getMultiQueryLikeSqlIn(Query ... querys){
		BooleanQuery query = new BooleanQuery();
		for (Query subQuery : querys) {
			query.add(subQuery,Occur.SHOULD);
		}
		return query;
	}
	
	/**多条件查询类似于sql and
	 * @param querys
	 * @return
	 */
	public static Query getMultiQueryLikeSqlAnd(Query ... querys){
		BooleanQuery query = new BooleanQuery();
		for (Query subQuery : querys) {
			query.add(subQuery,Occur.MUST);
		}
		return query;
	}
	/**从指定配置项中查询
	 * @return
	 * @param analyzer 分词器
	 * @param field 字段
	 * @param fieldType	字段类型
	 * @param queryStr 查询条件
	 * @param range 是否区间查询
	 * @return
	 */
	public static Query getQuery(String field,String fieldType,String queryStr,boolean range){
		Query q = null;
		try {
			if(queryStr != null && !"".equals(queryStr)){
				if(range){
					String[] strs = queryStr.split("\\|");
					if("int".equals(fieldType)){
						int min = new Integer(strs[0]);
						int max = new Integer(strs[1]);
						q = NumericRangeQuery.newIntRange(field, min, max, true, true);
					}else if("double".equals(fieldType)){
						Double min = new Double(strs[0]);
						Double max = new Double(strs[1]);
						q = NumericRangeQuery.newDoubleRange(field, min, max, true, true);
					}else if("float".equals(fieldType)){
						Float min = new Float(strs[0]);
						Float max = new Float(strs[1]);
						q = NumericRangeQuery.newFloatRange(field, min, max, true, true);
					}else if("long".equals(fieldType)){
						Long min = new Long(strs[0]);
						Long max = new Long(strs[1]);
						q = NumericRangeQuery.newLongRange(field, min, max, true, true);
					}
				}else{
					if("int".equals(fieldType)){
						q = NumericRangeQuery.newIntRange(field, new Integer(queryStr), new Integer(queryStr), true, true);
					}else if("double".equals(fieldType)){
						q = NumericRangeQuery.newDoubleRange(field, new Double(queryStr), new Double(queryStr), true, true);
					}else if("float".equals(fieldType)){
						q = NumericRangeQuery.newFloatRange(field, new Float(queryStr), new Float(queryStr), true, true);
					}else{
						Analyzer analyzer = new StandardAnalyzer();
						q = new QueryParser(field, analyzer).parse(queryStr);
					}
				}
			}else{
				q= new MatchAllDocsQuery();
			}
			
			System.out.println(q);
		} catch (ParseException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		return q;
	}
	/**根据field和值获取对应的内容
	 * @param fieldName
	 * @param fieldValue
	 * @return
	 */
	public static Query getQuery(String fieldName,Object fieldValue){
		Term term = new Term(fieldName, new BytesRef(fieldValue.toString()));
		return new TermQuery(term);
	}
	/**根据IndexSearcher和docID获取默认的document
	 * @param searcher
	 * @param docID
	 * @return
	 * @throws IOException
	 */
	public static Document getDefaultFullDocument(IndexSearcher searcher,int docID) throws IOException{
		return searcher.doc(docID);
	}
	/**根据IndexSearcher和docID
	 * @param searcher
	 * @param docID
	 * @param listField
	 * @return
	 * @throws IOException
	 */
	public static Document getDocumentByListField(IndexSearcher searcher,int docID,Set<String> listField) throws IOException{
		return searcher.doc(docID, listField);
	}
	
	/**分页查询
	 * @param page 当前页数
	 * @param perPage 每页显示条数
	 * @param searcher searcher查询器
	 * @param query 查询条件
	 * @return
	 * @throws IOException
	 */
	public static TopDocs getScoreDocsByPerPage(int page,int perPage,IndexSearcher searcher,Query query) throws IOException{
		TopDocs result = null;
		if(query == null){
			System.out.println(" Query is null return null ");
			return null;
		}
		ScoreDoc before = null;
		if(page != 1){
			TopDocs docsBefore = searcher.search(query, (page-1)*perPage);
			ScoreDoc[] scoreDocs = docsBefore.scoreDocs;
			if(scoreDocs.length > 0){
				before = scoreDocs[scoreDocs.length - 1];
			}
		}
		result = searcher.searchAfter(before, query, perPage);
		return result;
	}
	public static TopDocs getScoreDocs(IndexSearcher searcher,Query query) throws IOException{
		TopDocs docs = searcher.search(query, getMaxDocId(searcher));
		return docs;
	}
	/**高亮显示字段
	 * @param searcher
	 * @param field
	 * @param keyword
	 * @param preTag
	 * @param postTag
	 * @param fragmentSize
	 * @return
	 * @throws IOException 
	 * @throws InvalidTokenOffsetsException 
	 */
	public static String[] highlighter(IndexSearcher searcher,String field,String keyword,String preTag, String postTag,int fragmentSize) throws IOException, InvalidTokenOffsetsException{
		Term term = new Term("content",new BytesRef("lucene"));
        TermQuery termQuery = new TermQuery(term);
        TopDocs docs = getScoreDocs(searcher, termQuery);
        ScoreDoc[] hits = docs.scoreDocs;
        QueryScorer scorer = new QueryScorer(termQuery);
        SimpleHTMLFormatter simpleHtmlFormatter = new SimpleHTMLFormatter(preTag,postTag);//设定高亮显示的格式<B>keyword</B>,此为默认的格式  
        Highlighter highlighter = new Highlighter(simpleHtmlFormatter,scorer);   
        highlighter.setTextFragmenter(new SimpleFragmenter(fragmentSize));//设置每次返回的字符数
        Analyzer analyzer = new StandardAnalyzer();
        String[] result = new String[hits.length];
        for (int i = 0; i < result.length ; i++) {
			Document doc = searcher.doc(hits[i].doc);
			result[i] = highlighter.getBestFragment(analyzer, field, doc.get(field));
		}
        return result;
	}
	/**统计document的数量,此方法等同于matchAllDocsQuery查询
	 * @param searcher
	 * @return
	 */
	public static int getMaxDocId(IndexSearcher searcher){
		return searcher.getIndexReader().maxDoc();
	}
	
	/**group查询，适用于对group字段已经进行分段索引的情况
	 * @param searcher
	 * @param groupEndQuery
	 * @param query
	 * @param sort
	 * @param withinGroupSort
	 * @param groupOffset
	 * @param topNGroups
	 * @param needsScores
	 * @param docOffset
	 * @param docsPerGroup
	 * @param fillFields
	 * @return
	 * @throws IOException
	 */
	public static TopGroups<BytesRef> getTopGroupsByGroupTerm(IndexSearcher searcher,Query groupEndQuery,Query query,Sort sort,Sort withinGroupSort,int groupOffset,int topNGroups,boolean needsScores,int docOffset,int docsPerGroup,boolean fillFields) throws IOException{
		@SuppressWarnings("deprecation")
		Filter groupEndDocs = new CachingWrapperFilter(new QueryWrapperFilter(groupEndQuery));
		BlockGroupingCollector c = new BlockGroupingCollector(sort, groupOffset+topNGroups, needsScores, groupEndDocs);
		searcher.search(query, c);
		@SuppressWarnings("unchecked")
		TopGroups<BytesRef> groupsResult = (TopGroups<BytesRef>) c.getTopGroups(withinGroupSort, groupOffset, docOffset, docOffset+docsPerGroup, fillFields);
		return groupsResult;
	}
	/**通用的进行group查询
	 * @param searcher
	 * @param query
	 * @param groupFieldName
	 * @param sort
	 * @param maxCacheRAMMB
	 * @param page
	 * @param perPage
	 * @return
	 * @throws IOException
	 */
	public static TopGroups<BytesRef> getTopGroups(IndexSearcher searcher,Query query,String groupFieldName,Sort sort,double maxCacheRAMMB,int page,int perPage) throws IOException{
		GroupingSearch groupingSearch = new GroupingSearch(groupFieldName);
		groupingSearch.setGroupSort(sort);
		groupingSearch.setFillSortFields(true);
		groupingSearch.setCachingInMB(maxCacheRAMMB, true);
		groupingSearch.setAllGroups(true);
		TopGroups<BytesRef> result = groupingSearch.search(searcher,query, (page-1)*perPage, page*perPage);
		return result;
	}
}
```
