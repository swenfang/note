---
tags:
	- lucene
categories: lucene
title: lucene的各种Field及其排序
---
# lucene（2）---lucene的各种Field及其排序

## Lucene的Field说明

Lucene存储对象是以document为存储单元，对象中相关的属性值则存放到Field中；

lucene中所有Field都是IndexableField接口的实现

<!--more-->

```java
org.apache.lucene.index.IndexableField
 
Represents a single field for indexing. IndexWriter consumes Iterable<IndexableField> as a document.
```

IndexableField接口提供了一些方法，主要是对field相关属性的获取，包括

```java
  /** 获取field的名称 */
  public String name();
```

```java
  /** 获取field的类型fieldType */
  public IndexableFieldType fieldType();
```

 

```java
  /** 
   *获取当前field的权重（评分值） 只有Field有评分的概念，如果我们想对document进行评分值的设定 必须预先对document中对应的field值进行评分设设定*/  public float boost();
```

```java
  /** 如果此Filed为二进制类型的，返回相应的值*/
  public BytesRef binaryValue();
```

```java
  /**
   * 创建一个用户索引此Field的TokenStream
   */
  public TokenStream tokenStream(Analyzer analyzer, TokenStream reuse) throws IOException;
```

所有的Field均是org.apache.lucene.document.Field的子类；

项目中我们常用的Field类型主要有IntField, LongField, FloatField, DoubleField, BinaryDocValuesField, NumericDocValuesField, SortedDocValuesField, StringField, TextField, StoredField.

## lucene常见Field

IntField 主要对int类型的字段进行存储，需要注意的是如果需要对InfField进行排序使用SortField.Type.INT来比较，如果进范围查询或过滤，需要采用NumericRangeQuery.newIntRange() LongField 主要处理Long类型的字段的存储，排序使用SortField.Type.Long,如果进行范围查询或过滤利用NumericRangeQuery.newLongRange()，LongField常用来进行时间戳的排序，保存System.currentTimeMillions() FloatField 对Float类型的字段进行存储，排序采用SortField.Type.Float,范围查询采用NumericRangeQuery.newFloatRange() BinaryDocVluesField 只存储不共享值，如果需要共享值可以用SortedDocValuesField NumericDocValuesField 用于数值类型的Field的排序(预排序)，需要在要排序的field后添加一个同名的NumericDocValuesField SortedDocValuesField 用于String类型的Field的排序，需要在StringField后添加同名的SortedDocValuesField StringField 用户String类型的字段的存储，StringField是只索引不分词 TextField 对String类型的字段进行存储，TextField和StringField的不同是TextField既索引又分词 StoredField 存储Field的值，可以用IndexSearcher.doc和IndexReader.document来获取此Field和存储的值

## IntField使用

```java
package com.lucene.field;
 
import java.io.IOException;
 
 
import org.apache.lucene.document.Document;
import org.apache.lucene.document.Field;
import org.apache.lucene.document.IntField;
import org.apache.lucene.document.NumericDocValuesField;
import org.apache.lucene.index.IndexWriter;
import org.apache.lucene.search.IndexSearcher;
import org.apache.lucene.search.MatchAllDocsQuery;
import org.apache.lucene.search.Query;
import org.apache.lucene.search.ScoreDoc;
import org.apache.lucene.search.Sort;
import org.apache.lucene.search.SortField;
import org.apache.lucene.search.TopFieldDocs;
import org.junit.Test;
 
import com.lucene.index.IndexUtil;
import com.lucene.search.SearchUtil;
 
public class IntFieldTest {
	/**
	 * 保存一个intField
	 */
	@Test
	public void testIndexIntFieldStored() {
		Document document = new Document();
		document.add(new IntField("intValue", 30, Field.Store.YES));
		//要排序必须加同名的field，且类型为NumericDocValuesField
		document.add(new NumericDocValuesField("intValue", 30));
		Document document1 = new Document();
		document1.add(new IntField("intValue", 40, Field.Store.YES));
		document1.add(new NumericDocValuesField("intValue", 40));
		IndexWriter writer = null;
		try {
			writer = IndexUtil.getIndexWriter("intFieldPath", false);
			writer.addDocument(document);
			writer.addDocument(document1);
			
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}finally{
			try {
				writer.commit();
				writer.close();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
	/**
	 * 测试intField排序
	 */
	@Test
	public void testIntFieldSort(){
		try {
			IndexSearcher searcher = SearchUtil.getIndexSearcher("intFieldPath", null);
			//构建排序字段
			SortField[] sortField = new SortField[1];
			sortField[0] = new SortField("intValue",SortField.Type.INT,true);
			Sort sort = new Sort(sortField);
			//查询所有结果
			Query query = new MatchAllDocsQuery();
			TopFieldDocs docs = searcher.search(query, 2, sort);
			ScoreDoc[] scores = docs.scoreDocs;
			//遍历结果
			for (ScoreDoc scoreDoc : scores) {
				System.out.println(searcher.doc(scoreDoc.doc));;
			}
			//searcher.search(query, results);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
	
 
}
```

测试排序结果如下

```
Document<stored<intValue:40>>
Document<stored<intValue:30>>
```

如果修改NumericDocValuesField对应的值，结果会随着其值的大小而改变

## LongField使用

```java
package com.lucene.field;
 
import java.io.IOException;
 
import org.apache.lucene.document.Document;
import org.apache.lucene.document.Field;
import org.apache.lucene.document.LongField;
import org.apache.lucene.document.NumericDocValuesField;
import org.apache.lucene.index.IndexWriter;
import org.apache.lucene.search.IndexSearcher;
import org.apache.lucene.search.MatchAllDocsQuery;
import org.apache.lucene.search.Query;
import org.apache.lucene.search.ScoreDoc;
import org.apache.lucene.search.Sort;
import org.apache.lucene.search.SortField;
import org.apache.lucene.search.TopFieldDocs;
import org.junit.Test;
 
import com.lucene.index.IndexUtil;
import com.lucene.search.SearchUtil;
 
public class LongFieldTest {
 
	/**
	 * 保存一个longField
	 */
	@Test
	public void testIndexLongFieldStored() {
		Document document = new Document();
		document.add(new LongField("longValue", 50L, Field.Store.YES));
		document.add(new NumericDocValuesField("longValue", 50L));
		Document document1 = new Document();
		document1.add(new LongField("longValue", 80L, Field.Store.YES));
		document1.add(new NumericDocValuesField("longValue", 80L));
		IndexWriter writer = null;
		try {
			writer = IndexUtil.getIndexWriter("longFieldPath", false);
			writer.addDocument(document);
			writer.addDocument(document1);
			
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}finally{
			try {
				writer.commit();
				writer.close();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
	/**
	 * 测试longField排序
	 */
	@Test
	public void testLongFieldSort(){
		try {
			IndexSearcher searcher = SearchUtil.getIndexSearcher("longFieldPath", null);
			//构建排序字段
			SortField[] sortField = new SortField[1];
			sortField[0] = new SortField("longValue",SortField.Type.LONG,true);
			Sort sort = new Sort(sortField);
			//查询所有结果
			Query query = new MatchAllDocsQuery();
			TopFieldDocs docs = searcher.search(query, 2, sort);
			ScoreDoc[] scores = docs.scoreDocs;
			//遍历结果
			for (ScoreDoc scoreDoc : scores) {
				//System.out.println(searcher.doc(scoreDoc.doc));;
				Document doc = searcher.doc(scoreDoc.doc);
				System.out.println(doc.getField("longValue").numericValue());
			}
			//searcher.search(query, results);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
}
```

运行结果如下：

```
Document<stored<longValue:80>>
Document<stored<longValue:50>>
```

##  FloatField使用

```java
package com.lucene.field;
 
import java.io.IOException;
 
import org.apache.lucene.document.Document;
import org.apache.lucene.document.Field;
import org.apache.lucene.document.FloatDocValuesField;
import org.apache.lucene.document.FloatField;
import org.apache.lucene.index.IndexWriter;
import org.apache.lucene.search.IndexSearcher;
import org.apache.lucene.search.MatchAllDocsQuery;
import org.apache.lucene.search.Query;
import org.apache.lucene.search.ScoreDoc;
import org.apache.lucene.search.Sort;
import org.apache.lucene.search.SortField;
import org.apache.lucene.search.TopFieldDocs;
import org.junit.Test;
 
import com.lucene.index.IndexUtil;
import com.lucene.search.SearchUtil;
 
public class FloatFieldTest {
 
	/**
	 * 保存一个floatField
	 */
	@Test
	public void testIndexFloatFieldStored() {
		Document document = new Document();
		document.add(new FloatField("floatValue", 9.1f, Field.Store.YES));
		document.add(new FloatDocValuesField("floatValue", 82.0f));
		Document document1 = new Document();
		document1.add(new FloatField("floatValue", 80.1f, Field.Store.YES));
		document1.add(new FloatDocValuesField("floatValue", 80.1f));
		IndexWriter writer = null;
		try {
			writer = IndexUtil.getIndexWriter("floatFieldPath", false);
			writer.addDocument(document);
			writer.addDocument(document1);
			
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}finally{
			try {
				writer.commit();
				writer.close();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
	/**
	 * 测试intField排序
	 */
	@Test
	public void testFloatFieldSort(){
		try {
			IndexSearcher searcher = SearchUtil.getIndexSearcher("floatFieldPath", null);
			//构建排序字段
			SortField[] sortField = new SortField[1];
			sortField[0] = new SortField("floatValue",SortField.Type.FLOAT,true);
			Sort sort = new Sort(sortField);
			//查询所有结果
			Query query = new MatchAllDocsQuery();
			TopFieldDocs docs = searcher.search(query, 2, sort);
			ScoreDoc[] scores = docs.scoreDocs;
			//遍历结果
			for (ScoreDoc scoreDoc : scores) {
				//System.out.println(searcher.doc(scoreDoc.doc));;
				Document doc = searcher.doc(scoreDoc.doc);
				System.out.println(doc.getField("floatValue").numericValue());
			}
			//searcher.search(query, results);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
}
```

结果如下：

```
Document<stored<floatValue:9.1>>
Document<stored<floatValue:80.1>>
```

##  BinaryDocValuesField使用

```java
package com.lucene.field;
 
import java.io.IOException;
 
import org.apache.lucene.document.BinaryDocValuesField;
import org.apache.lucene.document.Document;
import org.apache.lucene.document.Field;
import org.apache.lucene.document.FloatDocValuesField;
import org.apache.lucene.document.FloatField;
import org.apache.lucene.document.IntField;
import org.apache.lucene.document.LongField;
import org.apache.lucene.document.NumericDocValuesField;
import org.apache.lucene.index.IndexWriter;
import org.apache.lucene.search.IndexSearcher;
import org.apache.lucene.search.MatchAllDocsQuery;
import org.apache.lucene.search.Query;
import org.apache.lucene.search.ScoreDoc;
import org.apache.lucene.search.Sort;
import org.apache.lucene.search.SortField;
import org.apache.lucene.search.TopFieldDocs;
import org.apache.lucene.util.BytesRef;
import org.junit.Test;
 
import com.lucene.index.IndexUtil;
import com.lucene.search.SearchUtil;
 
public class BinaryDocValuesFieldTest {
 
	/**
	 * 保存一个BinaryDocValuesField
	 */
	@Test
	public void testIndexLongFieldStored() {
		Document document = new Document();
		document.add(new BinaryDocValuesField("binaryValue",new BytesRef("1234".getBytes())));
		Document document1 = new Document();
		document1.add(new BinaryDocValuesField("binaryValue",new BytesRef("2345".getBytes())));
		IndexWriter writer = null;
		try {
			writer = IndexUtil.getIndexWriter("binaryValueFieldPath", false);
			writer.addDocument(document);
			writer.addDocument(document1);
			
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}finally{
			try {
				writer.commit();
				writer.close();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
	/**
	 * 测试BinaryDocValuesField排序
	 */
	@Test
	public void testBinaryDocValuesFieldSort(){
		try {
			IndexSearcher searcher = SearchUtil.getIndexSearcher("binaryValueFieldPath", null);
			//构建排序字段
			SortField[] sortField = new SortField[1];
			sortField[0] = new SortField("binaryValue",SortField.Type.STRING_VAL,true);
			Sort sort = new Sort(sortField);
			//查询所有结果
			Query query = new MatchAllDocsQuery();
			TopFieldDocs docs = searcher.search(query, 2, sort);
			ScoreDoc[] scores = docs.scoreDocs;
			//遍历结果
			for (ScoreDoc scoreDoc : scores) {
				//System.out.println(searcher.doc(scoreDoc.doc));;
				Document doc = searcher.doc(scoreDoc.doc);
				System.out.println(doc);
				//System.out.println(doc.getField("binaryValue").numericValue());
			}
			//searcher.search(query, results);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
}
```

运行结果：

```
Document<>
Document<>
```

为什么这样呢，这是跟BinaryDocValuesField的特性决定的，只索引不存值！

## StringField使用

```java
package com.lucene.field;
 
import java.io.IOException;
 
import org.apache.lucene.document.BinaryDocValuesField;
import org.apache.lucene.document.Document;
import org.apache.lucene.document.Field;
import org.apache.lucene.document.FloatDocValuesField;
import org.apache.lucene.document.FloatField;
import org.apache.lucene.document.IntField;
import org.apache.lucene.document.LongField;
import org.apache.lucene.document.NumericDocValuesField;
import org.apache.lucene.document.SortedDocValuesField;
import org.apache.lucene.document.StringField;
import org.apache.lucene.index.IndexWriter;
import org.apache.lucene.search.IndexSearcher;
import org.apache.lucene.search.MatchAllDocsQuery;
import org.apache.lucene.search.Query;
import org.apache.lucene.search.ScoreDoc;
import org.apache.lucene.search.Sort;
import org.apache.lucene.search.SortField;
import org.apache.lucene.search.TopFieldDocs;
import org.apache.lucene.util.BytesRef;
import org.junit.Test;
 
import com.lucene.index.IndexUtil;
import com.lucene.search.SearchUtil;
 
public class StringFieldTest {
 
	/**
	 * 保存一个StringField
	 */
	@Test
	public void testIndexLongFieldStored() {
		Document document = new Document();
		document.add(new StringField("stringValue","12445", Field.Store.YES));
		document.add(new SortedDocValuesField("stringValue", new BytesRef("12445".getBytes())));
		Document document1 = new Document();
		document1.add(new StringField("stringValue","23456", Field.Store.YES));
		document1.add(new SortedDocValuesField("stringValue", new BytesRef("23456".getBytes())));
		IndexWriter writer = null;
		try {
			writer = IndexUtil.getIndexWriter("stringFieldPath", false);
			writer.addDocument(document);
			writer.addDocument(document1);
			
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}finally{
			try {
				writer.commit();
				writer.close();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
	/**
	 * 测试StringField排序
	 */
	@Test
	public void testStringFieldSort(){
		try {
			IndexSearcher searcher = SearchUtil.getIndexSearcher("stringFieldPath", null);
			//构建排序字段
			SortField[] sortField = new SortField[1];
			sortField[0] = new SortField("stringVal",SortField.Type.STRING,true);
			Sort sort = new Sort(sortField);
			//查询所有结果
			Query query = new MatchAllDocsQuery();
			TopFieldDocs docs = searcher.search(query, 2, sort);
			ScoreDoc[] scores = docs.scoreDocs;
			//遍历结果
			for (ScoreDoc scoreDoc : scores) {
				//System.out.println(searcher.doc(scoreDoc.doc));;
				Document doc = searcher.doc(scoreDoc.doc);
				System.out.println(doc);
				//System.out.println(doc.getField("binaryValue").numericValue());
			}
			//searcher.search(query, results);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
}
```

运行结果如下：

```
Document<stored,indexed,tokenized,omitNorms,indexOptions=DOCS<stringValue:12445>>
Document<stored,indexed,tokenized,omitNorms,indexOptions=DOCS<stringValue:23456>>
```

##  TextField使用

```java
package com.lucene.field;
 
import java.io.IOException;
 
import org.apache.lucene.document.BinaryDocValuesField;
import org.apache.lucene.document.Document;
import org.apache.lucene.document.Field;
import org.apache.lucene.document.FloatDocValuesField;
import org.apache.lucene.document.FloatField;
import org.apache.lucene.document.IntField;
import org.apache.lucene.document.LongField;
import org.apache.lucene.document.NumericDocValuesField;
import org.apache.lucene.document.SortedDocValuesField;
import org.apache.lucene.document.StringField;
import org.apache.lucene.document.TextField;
import org.apache.lucene.index.IndexWriter;
import org.apache.lucene.search.IndexSearcher;
import org.apache.lucene.search.MatchAllDocsQuery;
import org.apache.lucene.search.Query;
import org.apache.lucene.search.ScoreDoc;
import org.apache.lucene.search.Sort;
import org.apache.lucene.search.SortField;
import org.apache.lucene.search.TopFieldDocs;
import org.apache.lucene.util.BytesRef;
import org.junit.Test;
 
import com.lucene.index.IndexUtil;
import com.lucene.search.SearchUtil;
 
public class TextFieldTest {
 
	/**
	 * 保存一个StringField
	 */
	@Test
	public void testIndexLongFieldStored() {
		Document document = new Document();
		document.add(new TextField("textValue","12345", Field.Store.YES));
		document.add(new SortedDocValuesField("textValue", new BytesRef("12345".getBytes())));
		Document document1 = new Document();
		document1.add(new TextField("textValue","23456", Field.Store.YES));
		document1.add(new SortedDocValuesField("textValue", new BytesRef("23456".getBytes())));
		IndexWriter writer = null;
		try {
			writer = IndexUtil.getIndexWriter("textFieldPath", false);
			writer.addDocument(document);
			writer.addDocument(document1);
			
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}finally{
			try {
				writer.commit();
				writer.close();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
	/**
	 * 测试StringField排序
	 */
	@Test
	public void testStringFieldSort(){
		try {
			IndexSearcher searcher = SearchUtil.getIndexSearcher("textFieldPath", null);
			//构建排序字段
			SortField[] sortField = new SortField[1];
			sortField[0] = new SortField("textValue",SortField.Type.STRING,true);
			Sort sort = new Sort(sortField);
			//查询所有结果
			Query query = new MatchAllDocsQuery();
			TopFieldDocs docs = searcher.search(query, 2, sort);
			ScoreDoc[] scores = docs.scoreDocs;
			//遍历结果
			for (ScoreDoc scoreDoc : scores) {
				//System.out.println(searcher.doc(scoreDoc.doc));;
				Document doc = searcher.doc(scoreDoc.doc);
				System.out.println(doc);
				//System.out.println(doc.getField("binaryValue").numericValue());
			}
			//searcher.search(query, results);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
}
```

运行结果如下：

```
Document<stored,indexed,tokenized<textValue:23456>>
Document<stored,indexed,tokenized<textValue:12345>>
```

## 源码下载地址

[lucene field使用源码](http://download.csdn.net/detail/wuyinggui10000/8669987)