---
tags:
	- lucene
categories: lucene
title: lucene索引优化之多线程创建索引
---

# lucene（6）---lucene索引优化之多线程创建索引

前面了解到lucene在索引创建的时候一个IndexWriter获取到一个读写锁，这样势在lucene创建大数据量的索引的时候，执行效率低下的问题；

<!--more-->

查看前面文档 lucene（5）---lucene的索引构建原理 可以看出，lucene索引的建立，跟以下几点关联很大；

1. 磁盘空间大小，这个直接影响索引的建立，甚至会造成索引写入提示完成，但是没有同步的问题；
2. 索引合并策略的选择，这个类似于sql里边的批量操作，批量操作的数量过多直接影响执行效率，对于lucene来讲，索引合并前是将document放在内存中，因此选择合适的合并策略也可以提升索引的效率；
3. 唯一索引对应的term的选择，lucene索引的创建过程中是先从索引中删除包含相同term的document然后重新添加document到索引中，这里如果term对应的document过多，会占用磁盘IO，同时造成IndexWriter的写锁占用时间延长，相应的执行效率低下；

综上所述，索引优化要保证磁盘空间，同时在term选择上可以以ID等标识来确保唯一性，这样第一条和第三条的风险就规避了；

本文旨在对合并策略和采用多线程创建的方式提高索引的效率；

多线程创建索引，我这边还设计了多目录索引创建，这样避免了同一目录数据量过大索引块合并和索引块重新申请；

废话不多说，这里附上代码，代码示例是读取lucene官网下载并解压的文件夹并给文件信息索引起来

首先定义FileBean来存储文件信息

```java
package com.lucene.bean;
 
public class FileBean {
	//路径
	private String path;
	//修改时间
	private Long modified;
	//内容
	private String content;
	public String getPath() {
		return path;
	}
	public void setPath(String path) {
		this.path = path;
	}
	public Long getModified() {
		return modified;
	}
	public void setModified(Long modified) {
		this.modified = modified;
	}
	public String getContent() {
		return content;
	}
	public void setContent(String content) {
		this.content = content;
	}
}
```

接下来是一个工具类，用以将文件夹的信息遍历读取并转换成FileBean的集合

```JAVA
package com.lucene.index.util;
 
 
import java.io.File;
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.LinkedList;
import java.util.List;
 
import com.lucene.bean.FileBean;
 
public class FileUtil {
 
	/**读取文件信息和下属文件夹
	 * @param folder
	 * @return
	 * @throws IOException
	 */
	public static List<FileBean> getFolderFiles(String folder) throws IOException {
		List<FileBean> fileBeans = new LinkedList<FileBean>();
		File file = new File(folder);
		if(file.isDirectory()){
			File[] files = file.listFiles();
			if(files != null){
				for (File file2 : files) {
					fileBeans.addAll(getFolderFiles(file2.getAbsolutePath()));
				}
			}
		}else{
			FileBean bean = new FileBean();
			bean.setPath(file.getAbsolutePath());
			bean.setModified(file.lastModified());
			bean.setContent(new String(Files.readAllBytes(Paths.get(folder))));
			fileBeans.add(bean);
		}
		return fileBeans;
	}
 
}
```

定义一个公共的用于处理索引的类 

```java
package com.lucene.index;
 
 
 
import java.io.File;
import java.io.IOException;
import java.text.ParseException;
import java.util.List;
import java.util.concurrent.CountDownLatch;
 
import org.apache.lucene.index.IndexWriter;
 
 
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
	public BaseIndex(IndexWriter writer,CountDownLatch countDownLatch1, CountDownLatch countDownLatch2,
			List<T> list){
		super();
		this.writer = writer;
		this.countDownLatch1 = countDownLatch1;
		this.countDownLatch2 = countDownLatch2;
		this.list = list;
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

FileBeanIndex类用于处理FileBean的索引创建 

```java
package com.lucene.index;
 
import java.util.List;
import java.util.concurrent.CountDownLatch;
 
import org.apache.lucene.document.Document;
import org.apache.lucene.document.Field;
import org.apache.lucene.document.LongField;
import org.apache.lucene.document.StringField;
import org.apache.lucene.document.TextField;
import org.apache.lucene.index.IndexWriter;
import org.apache.lucene.index.IndexWriterConfig;
import org.apache.lucene.index.Term;
 
import com.lucene.bean.FileBean;
 
public class FileBeanIndex extends BaseIndex<FileBean>{
 
	public FileBeanIndex(IndexWriter writer, CountDownLatch countDownLatch1,
			CountDownLatch countDownLatch2, List<FileBean> list) {
		super(writer, countDownLatch1, countDownLatch2, list);
	}
	public FileBeanIndex(String parentIndexPath, int subIndex, CountDownLatch countDownLatch1,
			CountDownLatch countDownLatch2, List<FileBean> list) {
		super(parentIndexPath, subIndex, countDownLatch1, countDownLatch2, list);
	}
	@Override
	public void indexDoc(IndexWriter writer, FileBean t) throws Exception {
		Document doc = new Document();
		System.out.println(t.getPath());
		doc.add(new StringField("path", t.getPath(), Field.Store.YES));
		doc.add(new LongField("modified", t.getModified(), Field.Store.YES));
		doc.add(new TextField("content", t.getContent(), Field.Store.YES));
		if (writer.getConfig().getOpenMode() == IndexWriterConfig.OpenMode.CREATE){
	        writer.addDocument(doc);
	    }else{
	    	writer.updateDocument(new Term("path", t.getPath()), doc);
	    }
	}
 
 
}
```

IndexUtil工具类里边设置索引合并的策略 

```java
package com.lucene.index;
 
import java.io.IOException;
import java.nio.file.Paths;
 
import org.apache.lucene.analysis.Analyzer;
import org.apache.lucene.analysis.standard.StandardAnalyzer;
import org.apache.lucene.index.IndexWriter;
import org.apache.lucene.index.IndexWriterConfig;
import org.apache.lucene.index.LogByteSizeMergePolicy;
import org.apache.lucene.index.LogMergePolicy;
import org.apache.lucene.store.Directory;
import org.apache.lucene.store.FSDirectory;
 
 
 
public class IndexUtil {
	/**创建索引写入器
	 * @param indexPath
	 * @param create
	 * @return
	 * @throws IOException
	 */
	public static IndexWriter getIndexWriter(String indexPath,boolean create) throws IOException{
		Directory dir = FSDirectory.open(Paths.get(indexPath, new String[0]));
	    Analyzer analyzer = new StandardAnalyzer();
	    IndexWriterConfig iwc = new IndexWriterConfig(analyzer);
	    LogMergePolicy mergePolicy = new LogByteSizeMergePolicy();
	    //设置segment添加文档(Document)时的合并频率          //值较小,建立索引的速度就较慢          //值较大,建立索引的速度就较快,>10适合批量建立索引        
	    mergePolicy.setMergeFactor(50);                     
	    //设置segment最大合并文档(Document)数         
	    //值较小有利于追加索引的速度         
	    //值较大,适合批量建立索引和更快的搜索         
	    mergePolicy.setMaxMergeDocs(5000);                     
	    if (create){
	        iwc.setOpenMode(IndexWriterConfig.OpenMode.CREATE);
	    }else {
	        iwc.setOpenMode(IndexWriterConfig.OpenMode.CREATE_OR_APPEND);
	    }
	    IndexWriter writer = new IndexWriter(dir, iwc);
	    return writer;
	}
}
```

TestIndex类执行测试程序 

```java
package com.lucene.index.test;
 
import java.util.List;
import java.util.concurrent.CountDownLatch;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
 
import org.apache.lucene.index.IndexWriter;
 
import com.lucene.bean.FileBean;
import com.lucene.index.FileBeanIndex;
import com.lucene.index.util.FileUtil;
 
public class TestIndex {
	public static void main(String[] args) {
		try {
			List<FileBean> fileBeans = FileUtil.getFolderFiles("C:\\Users\\lenovo\\Desktop\\lucene\\lucene-5.1.0");
			int totalCount = fileBeans.size();
			int perThreadCount = 3000;
			System.out.println("查询到的数据总数是"+fileBeans.size());
			int threadCount = totalCount/perThreadCount + (totalCount%perThreadCount == 0 ? 0 : 1);  
			ExecutorService pool = Executors.newFixedThreadPool(threadCount);  
			CountDownLatch countDownLatch1 = new CountDownLatch(1);  
			CountDownLatch countDownLatch2 = new CountDownLatch(threadCount);  
			System.out.println(fileBeans.size());
			
			for(int i = 0; i < threadCount; i++) { 
				int start = i*perThreadCount;
				int end = (i+1) * perThreadCount < totalCount ? (i+1) * perThreadCount : totalCount;
				List<FileBean> subList = fileBeans.subList(start, end);
				Runnable runnable = new FileBeanIndex("index",i, countDownLatch1, countDownLatch2, subList);
				//子线程交给线程池管理  
				pool.execute(runnable);  
			}  
			countDownLatch1.countDown();  
			System.out.println("开始创建索引");  
			//等待所有线程都完成  
			countDownLatch2.await();  
			 //线程全部完成工作  
			System.out.println("所有线程都创建索引完毕");  
			//释放线程池资源  
			pool.shutdown();  
		} catch (Exception e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
	}
 
}
```

以上即是多线程多目录索引，大家有什么疑问的欢迎交流； 