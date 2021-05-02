---
tags:
	- lucene
categories: lucene
title: lucene搜索之索引的查询原理和查询工具类（支持分页）示例
---
# lucene（8）---lucene搜索之索引的查询原理和查询工具类（支持分页）示例

## IndexSearcher常用方法

IndexSearcher提供了几个常用的方法：

<!--more-->

- IndexSearcher.doc(int docID)   获取索引文件中的第n个索引存储的相关字段，返回为Document类型，可以据此读取document中的Field.STORE.YES的字段；
- IndexSearcher.doc(int docID, StoredFieldVisitor fieldVisitor)  获取StoredFieldVisitor指定的字段的document，StoredFieldVisitor定义如下

```java
        StoredFieldVisitor visitor = new DocumentStoredFieldVisitor(String... fields);
```

- IndexSearcher.doc(int docID, Set<String> fieldsToLoad) 此方法同上边的IndexSearcher.doc(int docID, StoredFieldVisitor fieldVisitor) ，其实现如下图

![](http://blogimg.nos-eastchina1.126.net/shenwf20190316102208-992593.jpg)



- IndexSearcher.count(Query query) 统计符合query条件的document个数

- IndexSearcher.searchAfter(final ScoreDoc after, Query query, int numHits) 此方法会返回符合query查询条件的且在after之后的numHits条记录；

  其实现原理为：

  先读取当前索引文件的最大数据条数limit，然后判断after是否为空和after对应的document的下标是否超出limit的限制，如果超出的话抛出非法的参数异常；

  设置读取的条数为numHits和limit中最小的（因为有超出最大条数的可能，避免超出限制而造成的异常）

  接下来创建一个CollectorManager类型的对象，该对象定义了要返回的TopDocs的个数，上一页的document的结尾（after）,并且对查询结果进行分析合并

  最后调用search(query,manager)来查询结果

  ![](http://blogimg.nos-eastchina1.126.net/shenwf20190316104446-227930.jpg)

- IndexSearcher.search(Query query, int n) 查询符合query条件的前n个记录
- IndexSearcher.search(Query query, Collector results) 查询符合collector的记录，collector定义了分页等信息
- IndexSearcher.search(Query query, int n,Sort sort, boolean doDocScores, boolean doMaxScore) 实现任意排序的查询，同时控制是否计算hit score和max score是否被计算在内，查询前n条符合query条件的document;
- IndexSearcher.search(Query query, CollectorManager<C, T> collectorManager) 利用给定的collectorManager获取符合query条件的结果，其执行流程如下：

 先判断是否有ExecutorService执行查询的任务，如果没有executor，IndexSearcher会在单个任务下进行查询操作；

 如果IndexSearcher有executor，则会由每个线程控制一部分索引的读取，而且查询的过程中采用的是future机制，此种方式是边读边往结果集里边追加数据，这种异

 步的处理机制也提升了效率，其执行过程如下：

![](http://blogimg.nos-eastchina1.126.net/shenwf20190316104636-782439.jpg)

## 编码实践

我中午的时候写了一个SearchUtil的工具类，里边添加了多目录查询和分页查询的功能，经测试可用，工具类和测试的代码如下：

```java
package com.lucene.search.util;
import java.io.File;
import java.io.IOException;
import java.nio.file.Paths;
import java.util.Set;
import java.util.concurrent.ExecutorService;
import org.apache.lucene.document.Document;
import org.apache.lucene.index.DirectoryReader;
import org.apache.lucene.index.IndexReader;
import org.apache.lucene.index.MultiReader;
import org.apache.lucene.search.BooleanQuery;
import org.apache.lucene.search.IndexSearcher;
import org.apache.lucene.search.Query;
import org.apache.lucene.search.ScoreDoc;
import org.apache.lucene.search.TopDocs;
import org.apache.lucene.search.BooleanClause.Occur;
import org.apache.lucene.store.FSDirectory;
 
/**lucene索引查询工具类
 * @author lenovo
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
			e.printStackTrace();
		}
		return new IndexSearcher(reader,service);
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
	/**统计document的数量,此方法等同于matchAllDocsQuery查询
	 * @param searcher
	 * @return
	 */
	public static int getMaxDocId(IndexSearcher searcher){
		return searcher.getIndexReader().maxDoc();
	}
	
}
```

相关测试代码如下：

```java
package com.lucene.index.test;
import java.io.IOException;
import java.util.HashSet;
import java.util.Set;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import org.apache.lucene.document.Document;
import org.apache.lucene.index.Term;
import org.apache.lucene.search.IndexSearcher;
import org.apache.lucene.search.Query;
import org.apache.lucene.search.ScoreDoc;
import org.apache.lucene.search.TermQuery;
import org.apache.lucene.search.TopDocs;
import com.lucene.search.util.SearchUtil;
 
public class TestSearch {
	public static void main(String[] args) {
		ExecutorService service = Executors.newCachedThreadPool();
		try {
			
			IndexSearcher searcher = SearchUtil.getIndexSearcherByParentPath("index",service);
			System.out.println(SearchUtil.getMaxDocId(searcher));
			Term term = new Term("content", "lucene");
			Query query = new TermQuery(term);
			TopDocs docs = SearchUtil.getScoreDocsByPerPage(2, 20, searcher, query);
			ScoreDoc[] scoreDocs = docs.scoreDocs;
			System.out.println("所有的数据总数为："+docs.totalHits);
			System.out.println("本页查询到的总数为："+scoreDocs.length);
			for (ScoreDoc scoreDoc : scoreDocs) {
				Document doc = SearchUtil.getDefaultFullDocument(searcher, scoreDoc.doc);
				//System.out.println(doc);
			}
			System.out.println("\n\n");
			TopDocs docsAll = SearchUtil.getScoreDocs(searcher, query);
			Set<String> fieldSet = new HashSet<String>();
			fieldSet.add("path");
			fieldSet.add("modified");
			for (int i = 0 ; i < 20 ; i ++) {
				Document doc = SearchUtil.getDocumentByListField(searcher, docsAll.scoreDocs[i].doc,fieldSet);
				System.out.println(doc);
			}
			
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}finally{
			service.shutdownNow();
		}
	}
 
}
```

## 代码下载

代码下载请点击[http://download.csdn.net/detail/wuyinggui10000/8703165](http://download.csdn.net/detail/wuyinggui10000/8703067)，运行时请先运行IndexTest类进行索引的创建~！