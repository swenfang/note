---
tags:
	- lucene
categories: lucene
title: lucene搜索之facet查询原理和facet查询实例
---
# lucene（14）---lucene搜索之facet查询原理和facet查询实例

## Facet说明

我们在浏览网站的时候，经常会遇到按某一类条件查询的情况，这种情况尤以电商网站最多，以天猫商城为例，我们选择某一个品牌，系统会将该品牌对应的商品展示出来，效果图如下：

<!--more-->

![](http://blogimg.nos-eastchina1.126.net/shenwf20190316105905-628459.jpg)

如上图，我们关注的是品牌，选购热点等方面，对于类似的功能我们用lucene的term查询当然可以，但是在数据量特别大的情况下还用普通查询来实现显然会因为FSDirectory.open等耗时的操作造成查询效率的低下，同时普通查询是全部document都扫描一遍，这样显然造成了查询效率低；

lucene提供了facet查询用于对同一类的document进行聚类化，这样在查询的时候先关注某一个方面，这种显然缩小了查询范围，进而提升了查询效率；

facet模块提供了多个用于处理facet的统计和值处理的方法；

要实现facet的功能，我们需要了解facetField,FacetField定义了dim和此field对应的path,需要特别注意的是我们在做facetField索引的时候，需要事先调用FacetsConfig.build(Document);

FacetField的indexOptions设置为了DOCS_AND_FREQS_AND_POSITIONS的,即既索引又统计出现的频次和出现的位置，这样做主要是为了方便查询和统计；

相应的在存储的时候我们需要利用FacetsConfig和DirectoryTaxonomyWriter；

DirectoryTaxonomyWriter用来利用Directory来存储Taxono信息到硬盘；

DirectoryTaxonomyWriter的构造器如下:

```java
public DirectoryTaxonomyWriter(Directory directory, OpenMode openMode,
      TaxonomyWriterCache cache) throws IOException {
 
    dir = directory;
    IndexWriterConfig config = createIndexWriterConfig(openMode);
    indexWriter = openIndexWriter(dir, config);
 
    // verify (to some extent) that merge policy in effect would preserve category docids 
    assert !(indexWriter.getConfig().getMergePolicy() instanceof TieredMergePolicy) : 
      "for preserving category docids, merging none-adjacent segments is not allowed";
    
    // after we opened the writer, and the index is locked, it's safe to check
    // the commit data and read the index epoch
    openMode = config.getOpenMode();
    if (!DirectoryReader.indexExists(directory)) {
      indexEpoch = 1;
    } else {
      String epochStr = null;
      Map<String, String> commitData = readCommitData(directory);
      if (commitData != null) {
        epochStr = commitData.get(INDEX_EPOCH);
      }
      // no commit data, or no epoch in it means an old taxonomy, so set its epoch to 1, for lack
      // of a better value.
      indexEpoch = epochStr == null ? 1 : Long.parseLong(epochStr, 16);
    }
    
    if (openMode == OpenMode.CREATE) {
      ++indexEpoch;
    }
    
    FieldType ft = new FieldType(TextField.TYPE_NOT_STORED);
    ft.setOmitNorms(true);
    parentStreamField = new Field(Consts.FIELD_PAYLOADS, parentStream, ft);
    fullPathField = new StringField(Consts.FULL, "", Field.Store.YES);
 
    nextID = indexWriter.maxDoc();
 
    if (cache == null) {
      cache = defaultTaxonomyWriterCache();
    }
    this.cache = cache;
 
    if (nextID == 0) {
      cacheIsComplete = true;
      // Make sure that the taxonomy always contain the root category
      // with category id 0.
      addCategory(new FacetLabel());
    } else {
      // There are some categories on the disk, which we have not yet
      // read into the cache, and therefore the cache is incomplete.
      // We choose not to read all the categories into the cache now,
      // to avoid terrible performance when a taxonomy index is opened
      // to add just a single category. We will do it later, after we
      // notice a few cache misses.
      cacheIsComplete = false;
    }
  }
```

由上述代码可知，DirectoryTaxonomyWriter先打开一个IndexWriter,在确保indexWriter打开和locked的前提下，读取directory对应的segments中需要提交的内容，如果读取到的内容为空，说明是上次的内容，设置indexEpoch为1，接着对cache进行设置；判断directory中是否还包含有document，如果有设置cacheIsComplete为false,反之为true;

时候不早了，今天先写到这里，明天会在此基础上补充，大家见谅

## 编程实践

我对之前的读取文件夹内容的做了个facet索引的例子

对BaseIndex修改了facet的设置，相关代码如下 

```java
package com.lucene.index;
 
 
 
import java.io.File;
import java.io.IOException;
import java.nio.file.Paths;
import java.text.ParseException;
import java.util.List;
import java.util.concurrent.CountDownLatch;
 
import org.apache.lucene.document.Document;
import org.apache.lucene.document.Field;
import org.apache.lucene.facet.FacetResult;
import org.apache.lucene.facet.Facets;
import org.apache.lucene.facet.FacetsCollector;
import org.apache.lucene.facet.FacetsConfig;
import org.apache.lucene.facet.taxonomy.FastTaxonomyFacetCounts;
import org.apache.lucene.facet.taxonomy.directory.DirectoryTaxonomyReader;
import org.apache.lucene.facet.taxonomy.directory.DirectoryTaxonomyWriter;
import org.apache.lucene.index.IndexOptions;
import org.apache.lucene.index.IndexWriter;
import org.apache.lucene.index.Term;
import org.apache.lucene.search.IndexSearcher;
import org.apache.lucene.search.MatchAllDocsQuery;
import org.apache.lucene.search.Query;
import org.apache.lucene.store.Directory;
import org.apache.lucene.store.FSDirectory;
import org.apache.lucene.store.RAMDirectory;
 
import com.lucene.search.SearchUtil;
 
 
public abstract class BaseIndex<T> implements Runnable{
	/**
	 * 父级索引路径
	 */
	private String parentIndexPath;
	/**
	 * 索引编写器
	 */
	private IndexWriter writer;
	private int subIndex;
	/**
	 * 主线程
	 */
	private final CountDownLatch countDownLatch1;  
	/**
	 *工作线程 
	 */
	private final CountDownLatch countDownLatch2; 
	/**
	 * 对象列表
	 */
	private List<T> list;
	/**
	 * facet查询
	 */
	private String facet;
	protected final FacetsConfig config = new FacetsConfig();  
	protected final static String indexPath = "index1";
	protected final static DirectoryTaxonomyWriter taxoWriter;
	static{
		try {
			Directory directory = FSDirectory.open(Paths.get(indexPath, new String[0]));
			taxoWriter = new DirectoryTaxonomyWriter(directory);
		} catch (IOException e) {
			throw new ExceptionInInitializerError("BaseIndex initializing error");
		}
	}
	public BaseIndex(IndexWriter writer,CountDownLatch countDownLatch1, CountDownLatch countDownLatch2,
			List<T> list, String facet){
		super();
		this.writer = writer;
		this.countDownLatch1 = countDownLatch1;
		this.countDownLatch2 = countDownLatch2;
		this.list = list;
		this.facet = facet;
	}
	public BaseIndex(String parentIndexPath, int subIndex,
			CountDownLatch countDownLatch1, CountDownLatch countDownLatch2,
			List<T> list) {
		super();
		this.parentIndexPath = parentIndexPath;
		this.subIndex = subIndex;
		try {
			//多目录索引创建
			File file = new File(parentIndexPath+"/index"+subIndex);
			if(!file.exists()){
				file.mkdir();
			}
			this.writer = IndexUtil.getIndexWriter(parentIndexPath+"/index"+subIndex, true);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		};
		this.subIndex = subIndex;
		this.countDownLatch1 = countDownLatch1;
		this.countDownLatch2 = countDownLatch2;
		this.list = list;
	}
	public BaseIndex(String path,CountDownLatch countDownLatch1, CountDownLatch countDownLatch2,
			List<T> list) {
		super();
		try {
			//单目录索引创建
			File file = new File(path);
			if(!file.exists()){
				file.mkdir();
			}
			this.writer = IndexUtil.getIndexWriter(path,true);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		};
		this.countDownLatch1 = countDownLatch1;
		this.countDownLatch2 = countDownLatch2;
		this.list = list;
	}
	
	/**创建索引
	 * @param writer
	 * @param carSource
	 * @param create
	 * @throws IOException 
	 * @throws ParseException 
	 */
	public abstract void indexDoc(IndexWriter writer,T t) throws Exception;
	/**批量索引创建
	 * @param writer
	 * @param t
	 * @throws Exception
	 */
	public void indexDocs(IndexWriter writer,List<T> t) throws Exception{
		for (T t2 : t) {
			indexDoc(writer,t2);
		}
	}
	/**带group的索引创建
	 * @param writer
	 * @param docs
	 * @throws IOException 
	 */
	public void indexDocsWithGroup(IndexWriter writer,String groupFieldName,String groupFieldValue,List<Document> docs) throws IOException{
		 Field groupEndField = new Field(groupFieldName, groupFieldValue, Field.Store.NO, Field.Index.NOT_ANALYZED);
		 docs.get(docs.size()-1).add(groupEndField);
		 //
		 writer.updateDocuments(new Term(groupFieldName, groupFieldValue),docs);
		 writer.commit();
		 writer.close();
	}
	@Override
	public void run() {
		try {
			countDownLatch1.await();
			System.out.println(writer);
			indexDocs(writer,list);
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}finally{
			countDownLatch2.countDown();
			try {
				writer.commit();
				writer.close();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
}
```

相应得，document的索引需要利用DirectoryTaxonomyWriter来进行原有document的处理 

```java
package com.lucene.index;
 
import java.util.List;
import java.util.concurrent.CountDownLatch;
 
import org.apache.lucene.document.Document;
import org.apache.lucene.document.Field;
import org.apache.lucene.document.LongField;
import org.apache.lucene.document.StringField;
import org.apache.lucene.document.TextField;
import org.apache.lucene.facet.FacetField;
import org.apache.lucene.index.IndexWriter;
import org.apache.lucene.index.IndexWriterConfig;
import org.apache.lucene.index.Term;
 
import com.lucene.bean.FileBean;
 
public class FileBeanIndex extends BaseIndex<FileBean>{
	private static String facet;
 
	public FileBeanIndex(IndexWriter writer, CountDownLatch countDownLatch12, CountDownLatch countDownLatch1,
			List<FileBean> fileBeans, String facet1) {
		super(writer, countDownLatch12, countDownLatch1, fileBeans, facet);
		facet = facet1;
	}
	@Override
	public void indexDoc(IndexWriter writer, FileBean t) throws Exception {
		Document doc = new Document();
		String path = t.getPath();
		System.out.println(t.getPath());
		doc.add(new StringField("path", path, Field.Store.YES));
		doc.add(new LongField("modified", t.getModified(), Field.Store.YES));
		doc.add(new TextField("content", t.getContent(), Field.Store.YES));
		doc.add(new FacetField("filePath", new String[]{facet}));
		//doc = config.build(taxoWriter,doc);
		if (writer.getConfig().getOpenMode() == IndexWriterConfig.OpenMode.CREATE){
	        //writer.addDocument(doc);
			writer.addDocument(this.config.build(taxoWriter, doc));
	    }else{
	    	writer.updateDocument(new Term("path", t.getPath()), this.config.build(taxoWriter, doc));
	    }
		taxoWriter.commit();
	}
 
 
}
```

测试facet功能的测试类： 

```java
package com.lucene.search;
 
import java.io.IOException;
import java.nio.file.Paths;
import java.util.ArrayList;
import java.util.List;
 
import org.apache.lucene.facet.FacetResult;
import org.apache.lucene.facet.Facets;
import org.apache.lucene.facet.FacetsCollector;
import org.apache.lucene.facet.FacetsConfig;
import org.apache.lucene.facet.LabelAndValue;
import org.apache.lucene.facet.taxonomy.FastTaxonomyFacetCounts;
import org.apache.lucene.facet.taxonomy.TaxonomyReader;
import org.apache.lucene.facet.taxonomy.directory.DirectoryTaxonomyReader;
import org.apache.lucene.index.DirectoryReader;
import org.apache.lucene.search.IndexSearcher;
import org.apache.lucene.search.MatchAllDocsQuery;
import org.apache.lucene.search.Query;
import org.apache.lucene.store.Directory;
import org.apache.lucene.store.FSDirectory;
import org.junit.Test;
 
public class TestSearchFacet {
	public static Directory directory;
	public static Directory taxoDirectory;
	public static TaxonomyReader taxoReader;
	protected final static FacetsConfig config = new FacetsConfig();
	static {
		try {
			directory = FSDirectory.open(Paths.get("index", new String[0]));
			taxoDirectory = FSDirectory.open(Paths.get("index1", new String[0]));
			taxoReader = new DirectoryTaxonomyReader(taxoDirectory);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
 
	public static void testSearchFacet() {
		try {
			DirectoryReader indexReader = DirectoryReader.open(directory);
			IndexSearcher searcher = new IndexSearcher(indexReader);
			FacetsCollector fc = new FacetsCollector();
			FacetsCollector.search(searcher, new MatchAllDocsQuery(), indexReader.maxDoc(), fc);  
			Facets facets = new FastTaxonomyFacetCounts(taxoReader, config, fc);
			List<FacetResult> results =facets.getAllDims(100);
			for (FacetResult facetResult : results) {
				System.out.println(facetResult.dim);
				LabelAndValue[] values = facetResult.labelValues;
				for (LabelAndValue labelAndValue : values) {
					System.out.println("\t"+labelAndValue.label +"       "+labelAndValue.value);
				}
				
			}
			
			indexReader.close();
			taxoReader.close();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
 
	public static void main(String[] args) {
		testSearchFacet();
	}
 
}
```

相关代码下载

<http://download.csdn.net/detail/wuyinggui10000/8738651>