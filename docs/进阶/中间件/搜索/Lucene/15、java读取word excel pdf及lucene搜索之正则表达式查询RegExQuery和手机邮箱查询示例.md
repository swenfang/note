---
tags:
	- lucene
categories: lucene
title: java读取word excel pdf及lucene搜索之正则表达式查询RegExQuery和手机邮箱查询示例
---
# lucene（15）---java读取word excel pdf及lucene搜索之正则表达式查询RegExQuery和手机邮箱查询示例

读取文本文件中的内容，找出文件中的手机号和邮箱，我自己写了一个读取文档的内容的正则查询示例，用于匹配文件中是否含有邮箱或者手机号，这个等于是对之前的文本处理工具的一个梳理，同时结合lucene内部提供的正则匹配查询RegexQuery；

废话不多说了，直接上代码，这里先对文件内容读取分类处理，分为pdf word excel 和普通文本四类，不同的种类读取文本内容不一样

pdf利用pdfbox读取内容，word和excel利用poi进行读取内容，文本文档利用jdk自带的读取

<!--more-->

## 读取pdf、word、excel和普通文本文档内容（支持word excel 2007）

这里代码做了一点调整， 主要是对excel格式的空行和空列的过滤

```java
package com.lucene.index.util;
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.io.InputStream;
import java.nio.charset.Charset;
import java.nio.file.Files;
import java.nio.file.Paths;
import java.util.LinkedList;
import java.util.List;
import org.apache.pdfbox.PDFReader;
import org.apache.pdfbox.pdmodel.PDDocument;
import org.apache.pdfbox.util.PDFTextStripper;
import org.apache.poi.EncryptedDocumentException;
import org.apache.poi.POIXMLDocument;
import org.apache.poi.POIXMLTextExtractor;
import org.apache.poi.hssf.usermodel.HSSFCell;
import org.apache.poi.hssf.usermodel.HSSFRow;
import org.apache.poi.hwpf.extractor.WordExtractor;
import org.apache.poi.openxml4j.exceptions.InvalidFormatException;
import org.apache.poi.openxml4j.exceptions.OpenXML4JException;
import org.apache.poi.openxml4j.opc.OPCPackage;
import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.CellStyle;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.usermodel.Workbook;
import org.apache.poi.ss.usermodel.WorkbookFactory;
import org.apache.poi.xssf.usermodel.XSSFCell;
import org.apache.poi.xssf.usermodel.XSSFRow;
import org.apache.poi.xssf.usermodel.XSSFSheet;
import org.apache.poi.xssf.usermodel.XSSFWorkbook;
import org.apache.poi.xwpf.extractor.XWPFWordExtractor;
import org.apache.xmlbeans.XmlException;
import com.lucene.bean.FileBean;
 
public class FileUtil {
 
	/**读取文件信息和下属文件夹
	 * @param folder
	 * @return
	 * @throws IOException
	 * @throws OpenXML4JException 
	 * @throws XmlException 
	 */
	public static List<FileBean> getFolderFiles(String folder) throws Exception {
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
			String filePath = file.getAbsolutePath();
			bean.setPath(file.getAbsolutePath());
			bean.setModified(file.lastModified());
			String content = "";
			if(filePath.endsWith(".doc") || filePath.endsWith(".docx")){
				content = readDoc(file);
			}else if(filePath.endsWith(".xls") || filePath.endsWith(".xlsx")){
				content = readExcel(file);
			}else if(filePath.endsWith(".pdf")){
				content = readPdf(file);
			}else{
				content = new String(Files.readAllBytes(Paths.get(folder)));
			}
			bean.setContent(content);
			fileBeans.add(bean);
		}
		
		return fileBeans;
	}
	/**讀取excel文件
	 * @param file
	 * @return
	 * @throws IOException 
	 * @throws InvalidFormatException 
	 * @throws EncryptedDocumentException 
	 */
	public static String readExcel(File file) throws Exception {
		String filePath = file.getAbsolutePath();
		StringBuffer content = new StringBuffer("");
		if(filePath.endsWith(".xls")){
			InputStream inp = new FileInputStream(filePath);
		    Workbook wb = WorkbookFactory.create(inp);   
		    Sheet sheet = wb.getSheetAt(0);
		    for(int i = sheet.getFirstRowNum();i<= sheet.getPhysicalNumberOfRows();i++){  
		    	HSSFRow row = (HSSFRow) sheet.getRow(i);  
		    	if (row == null) {  
		    		  continue;  
		    	}
		    	for (int j = row.getFirstCellNum(); j <= row.getLastCellNum(); j++) { 
		    		if(j < 0){
		    			continue;//增加下标判断
		    		}
		    		HSSFCell cell = row.getCell(j);  
		    		if (cell == null) {  
			    		  continue;  
			    	}
		    		content.append(cell.getStringCellValue());
		    		
		    	}
		    }
		    wb.close();
		    inp.close();
		}else{
			XSSFWorkbook xwb = new XSSFWorkbook(file.getAbsolutePath());
			XSSFSheet sheet = xwb.getSheetAt(0);  
			// 定义 row、cell  
			XSSFRow row;  
			String cell;  
			// 循环输出表格中的内容  
			for (int i = sheet.getFirstRowNum(); i < sheet.getPhysicalNumberOfRows(); i++) {  
			    row = sheet.getRow(i);  
			    if(row == null){
			    	continue;
			    }
			    for (int j = row.getFirstCellNum(); j < row.getPhysicalNumberOfCells(); j++) {  
			        // 通过 row.getCell(j).toString() 获取单元格内容，
			    	if(j<0){
			    		continue;
			    	}
			    	XSSFCell xfcell = row.getCell(j);
			    	if(xfcell == null){
			    		continue;
			    	}
			    	xfcell.setCellType(Cell.CELL_TYPE_STRING);//数值型的转成文本型
			        cell = xfcell.getStringCellValue();
			        content.append(cell+" ");
			    }  
			}  
		}
		return content.toString();
	}
	/**讀取word內容
	 * @param file
	 * @return
	 * @throws IOException 
	 * @throws OpenXML4JException 
	 * @throws XmlException 
	 */
	public static String readDoc(File file) throws IOException, XmlException, OpenXML4JException {
		String filePath = file.getAbsolutePath();
		if(filePath.endsWith(".doc")){
			InputStream is = new FileInputStream(file);
			WordExtractor ex = new WordExtractor(is);  
			String text2003 = ex.getText();  
			ex.close();
			is.close();
			return text2003;
		}else{
			OPCPackage opcPackage = POIXMLDocument.openPackage(filePath);  
			POIXMLTextExtractor extractor = new XWPFWordExtractor(opcPackage);  
			String text2007 = extractor.getText();  
			extractor.close();
			return text2007;
		}
	}
	/**讀取pdf內容
	 * @param file
	 * @return
	 * @throws IOException
	 */
	public static String readPdf(File file) throws IOException{
		PDDocument doc = PDDocument.load(file.getAbsolutePath());
		PDFTextStripper stripper = new PDFTextStripper();
		String content = stripper.getText(doc);
		doc.close();
		return content;
	}
}
```

## 正则查询query构建

在原有 lucene 查询的工具类的基础上加入正则查询的构建

```java
    /**获取regexQuery对象
	 * @param field
	 * @param regex
	 * @return
	 */
	public static Query getRegexExpQuery(String field,String regex){
		Query query = null;
		Term term = new Term(field, regex);
		query = new RegexpQuery(term);
		return query;
	}
```

最终的searchUtil的内容为

```java
package com.lucene.search;
import java.io.File;
import java.io.IOException;
import java.nio.file.Paths;
import java.util.concurrent.ExecutorService;
import org.apache.lucene.analysis.Analyzer;
import org.apache.lucene.analysis.standard.StandardAnalyzer;
import org.apache.lucene.document.Document;
import org.apache.lucene.index.DirectoryReader;
import org.apache.lucene.index.IndexReader;
import org.apache.lucene.index.MultiReader;
import org.apache.lucene.index.Term;
import org.apache.lucene.search.BooleanClause.Occur;
import org.apache.lucene.search.BooleanQuery;
import org.apache.lucene.search.IndexSearcher;
import org.apache.lucene.search.MatchAllDocsQuery;
import org.apache.lucene.search.NumericRangeQuery;
import org.apache.lucene.search.Query;
import org.apache.lucene.search.RegexpQuery;
import org.apache.lucene.search.ScoreDoc;
import org.apache.lucene.search.Sort;
import org.apache.lucene.search.SortField;
import org.apache.lucene.search.SortField.Type;
import org.apache.lucene.search.TermQuery;
import org.apache.lucene.search.TopDocs;
import org.apache.lucene.search.TopFieldCollector;
import org.apache.lucene.store.FSDirectory;
import org.wltea.analyzer.lucene.IKAnalyzer;
import com.lucene.index.IndexUtil;
 
public class SearchUtil {
	public static final Analyzer analyzer = new IKAnalyzer();
	/**获取IndexSearcher对象（适合单索引目录查询使用）
	 * @param indexPath 索引目录
	 * @return
	 * @throws IOException
	 * @throws InterruptedException 
	 */
	public static IndexSearcher getIndexSearcher(String indexPath,ExecutorService service,boolean realtime) throws IOException, InterruptedException{
		DirectoryReader reader = DirectoryReader.open(IndexUtil.getIndexWriter(indexPath, true), realtime);
		IndexSearcher searcher = new IndexSearcher(reader,service);
		if(service != null){
			service.shutdown();
		}
		return searcher;
	}
	
	/**多目录多线程查询
	 * @param parentPath 父级索引目录
	 * @param service 多线程查询
	 * @return
	 * @throws IOException
	 * @throws InterruptedException 
	 */
	public static IndexSearcher getMultiSearcher(String parentPath,ExecutorService service,boolean realtime) throws IOException, InterruptedException{
		MultiReader multiReader;
		File file = new File(parentPath);
		File[] files = file.listFiles();
		IndexReader[] readers = new IndexReader[files.length];
		if(!realtime){
			for (int i = 0 ; i < files.length ; i ++) {
				readers[i] = DirectoryReader.open(FSDirectory.open(Paths.get(files[i].getPath(), new String[0])));
			}
		}else{
			for (int i = 0 ; i < files.length ; i ++) {
				readers[i] = DirectoryReader.open(IndexUtil.getIndexWriter(files[i].getPath(), true), true);
			}
		}
	
		multiReader = new MultiReader(readers);
		IndexSearcher searcher = new IndexSearcher(multiReader,service);
		if(service != null){
			service.shutdown();
		}
		return searcher;
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
					Term term = new Term(field, queryStr);
					q = new TermQuery(term);
				}
			}
		}else{
			q= new MatchAllDocsQuery();
		}
		
		System.out.println(q);
		return q;
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
	
	/**获取regexQuery对象
	 * @param field
	 * @param regex
	 * @return
	 */
	public static Query getRegexExpQuery(String field,String regex){
		Query query = null;
		Term term = new Term(field, regex);
		query = new RegexpQuery(term);
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
	/**对多个条件进行排序构建排序条件
	 * @param fields
	 * @param type
	 * @param reverses
	 * @return
	 */
	public static Sort getSortInfo(String[] fields,Type[] types,boolean[] reverses){
		SortField[] sortFields = null;
		int fieldLength = fields.length;
		int typeLength = types.length;
		int reverLength = reverses.length;
		if(!(fieldLength == typeLength) || !(fieldLength == reverLength)){
			return null;
		}else{
			sortFields = new SortField[fields.length];
			for (int i = 0; i < fields.length; i++) {
				sortFields[i] = new SortField(fields[i], types[i], reverses[i]);
			}
		}
		return new Sort(sortFields);
	}
	/**根据查询器、查询条件、每页数、排序条件进行查询
	 * @param query 查询条件
	 * @param first 起始值
	 * @param max 最大值
	 * @param sort 排序条件
	 * @return
	 */
	public static TopDocs getScoreDocsByPerPageAndSortField(IndexSearcher searcher,Query query, int first,int max, Sort sort){
		try {
			if(query == null){
				System.out.println(" Query is null return null ");
				return null;
			}
			TopFieldCollector collector = null;
			if(sort != null){
				collector = TopFieldCollector.create(sort, first+max, false, false, false);
			}else{
				sort = new Sort(new SortField[]{new SortField("modified", SortField.Type.LONG)});
				collector = TopFieldCollector.create(sort, first+max, false, false, false);
			}
			searcher.search(query, collector);
			return collector.topDocs(first, max);
		} catch (IOException e) {
			// TODO Auto-generated catch block
		}
		return null;
	}
	/**获取上次索引的id,增量更新使用
	 * @return
	 */
	public static Integer getLastIndexBeanID(IndexReader multiReader){
		Query query = new MatchAllDocsQuery();
		IndexSearcher searcher = null;
		searcher = new IndexSearcher(multiReader);
		SortField sortField = new SortField("id", SortField.Type.INT,true);
		Sort sort = new Sort(new SortField[]{sortField});
		TopDocs docs = getScoreDocsByPerPageAndSortField(searcher,query, 0, 1, sort);
		ScoreDoc[] scoreDocs = docs.scoreDocs;
		int total = scoreDocs.length;
		if(total > 0){
			ScoreDoc scoreDoc = scoreDocs[0];
			Document doc = null;
			try {
				doc = searcher.doc(scoreDoc.doc);
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
			return new Integer(doc.get("id"));
		}
		return 0;
	}
}
```

## 正则查询测试

正则查询测试类，主要是测试是否包含手机号或邮箱号，这里的手机号验证有点粗糙，希望不要介意

```java
package com.lucene.index.test;
import java.io.IOException;
import java.util.concurrent.Executors;
import org.apache.lucene.document.Document;
import org.apache.lucene.search.IndexSearcher;
import org.apache.lucene.search.Query;
import org.apache.lucene.search.ScoreDoc;
import org.apache.lucene.search.TopDocs;
import com.lucene.search.SearchUtil;

public class TestSearch {
	public static void main(String[] args) {
		try {
			IndexSearcher searcher = SearchUtil.getMultiSearcher("index", Executors.newCachedThreadPool(), false);
			Query phoneQuery = SearchUtil.getRegexExpQuery("content", "1[0-9]{10}");
			Query mailQuery = SearchUtil.getRegexExpQuery("content", "([a-z0-9A-Z]+[-_|\\.]?)+[a-z0-9A-Z]*@([a-z0-9A-Z]+(-[a-z0-9A-Z]+)?\\.)+[a-zA-Z]{2,}");
			Query finaQuery = SearchUtil.getMultiQueryLikeSqlIn(new Query[]{phoneQuery,mailQuery}); 
			TopDocs topDocs = SearchUtil.getScoreDocsByPerPageAndSortField(searcher, finaQuery, 0, 20, null);
			System.out.println("符合条件的数据总数："+topDocs.totalHits);
			System.out.println("本次查询到的数目为："+topDocs.scoreDocs.length);
			ScoreDoc[] scoreDocs = topDocs.scoreDocs;
			for (ScoreDoc scoreDoc : scoreDocs) {
				Document doc = searcher.doc(scoreDoc.doc);
				System.out.println(doc.get("path")+"    "+doc.get("content"));
			}
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} catch (InterruptedException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
	}
 
}
```

最终测试结果如下：

```
content:/1[0-9]{10}/
content:/([a-z0-9A-Z]+[-_|\.]?)+[a-z0-9A-Z]*@([a-z0-9A-Z]+(-[a-z0-9A-Z]+)?\.)+[a-zA-Z]{2,}/
符合条件的数据总数：6
本次查询到的数目为：6
D:\hadoop\lucene_regexSearch\testDir\2.txt.txt    ﻿电话号码：18519237811
D:\hadoop\lucene_regexSearch\testDir\3.txt.txt    电子邮箱yinggui_Wu@163.com
D:\hadoop\lucene_regexSearch\testDir\1.docx    邮箱内容yinggui_Wu@163.com
 
D:\hadoop\lucene_regexSearch\testDir\1.pdf    邮箱内容 yinggui_Wu@163.com 
 
D:\hadoop\lucene_regexSearch\testDir\1.xlsx    1 2 3 18510539956 
D:\hadoop\lucene_regexSearch\testDir\1.txt.txt    ﻿<a target=_blank href="mailto:fanyi@qq.com">fanyi@qq.com</a>
```

代码下载地址

<http://download.csdn.net/detail/wuyinggui10000/8746407>