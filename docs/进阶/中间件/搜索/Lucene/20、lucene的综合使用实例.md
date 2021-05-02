---
tags:
	- lucene
categories: lucene
title: lucene 综合应用实例
---
# lucene（20）---lucene 综合应用实例

最近因项目需求的需要，完成一个"会话检索"功能。该功能是把录音转写成文字，对转写后的文本进行关键字检索。因为该功能对检索条件类型的使用比较完整（例如：时间范围、关键字、语速等不同类型）以及使用的注意点也比较多，所以在这里给大家分享一下。希望可以帮到你。

<!--more-->

## 功能说明

会话检索，支持 多个文件夹同时检索，支持的索引大小为 1300 MB 左右（大约是45万条数据），支持 多个条件进行 and 检索。

## 功能依赖

lucene 使用的版本是 5.5.3 ，相对来说还是比较老的，当前最新的版本是 7.7.0 。

```xml
<dependency>
	<groupId>org.apache.lucene</groupId>
	<artifactId>lucene-core</artifactId>
	<version>5.5.3</version>
</dependency>
<dependency>
	<groupId>org.apache.lucene</groupId>
	<artifactId>lucene-analyzers-common</artifactId>
	<version>5.5.3</version>
</dependency>
<dependency>
	<groupId>org.apache.lucene</groupId>
	<artifactId>lucene-queryparser</artifactId>
	<version>5.5.3</version>
</dependency>
<dependency>
	<groupId>org.apache.lucene</groupId>
	<artifactId>lucene-queries</artifactId>
	<version>5.5.3</version>
</dependency>
<dependency>
	<groupId>org.apache.lucene</groupId>
	<artifactId>lucene-backward-codecs</artifactId>
	<version>5.5.3</version>
</dependency>
<dependency>
	<groupId>org.apache.lucene</groupId>
	<artifactId>lucene-memory</artifactId>
	<version>5.5.3</version>
</dependency>
<dependency>
	<groupId>org.apache.lucene</groupId>
	<artifactId>lucene-highlighter</artifactId>
	<version>5.5.3</version>
</dependency>
<dependency>
	<groupId>org.apache.lucene</groupId>
	<artifactId>lucene-spatial</artifactId>
	<version>5.5.3</version>
</dependency>
<dependency>
	<groupId>org.apache.lucene</groupId>
	<artifactId>lucene-analyzers-smartcn</artifactId>
	<version>5.5.3</version>
</dependency>
```



## 实现过程

### 编码过程

分页处理类 Page

```java
public class Page<T> {
	/** 当前第几页(从1开始计算) */
	private int currentPage;
	/** 每页显示几条 */
	private int pageSize;
	/** 总记录数 */
	private int totalRecord;
	/** 总页数 */
	private int totalPage;
	/** 分页数据集合[用泛型T来限定集合元素类型] */
	private Collection<T> items;
	/** 当前显示起始索引(从零开始计算) */
	private int startIndex;
	/** 当前显示结束索引(从零开始计算) */
	private int endIndex;
	/** 一组最多显示几个页码[比如Google一组最多显示10个页码] */
	private int groupSize;
	/** 左边偏移量 */
	private int leftOffset = 5;
	/** 右边偏移量 */
	private int rightOffset = 4;
	/** 当前页码范围 */
	private String[] pageRange;
	/** 分页数据 */
	private List<Document> docList;
	/** 上一页最后一个ScoreDoc对象 */
	private ScoreDoc afterDoc;
	/** 上一页最后一个ScoreDoc对象的Document对象ID */
	private int afterDocId;
    
	public void setRangeIndex() {
		int groupSize = getGroupSize();
		int totalPage = getTotalPage();
		if (totalPage < 2) {
			startIndex = 0;
			endIndex = totalPage - startIndex;
		} else {
			int currentPage = getCurrentPage();
			if (groupSize >= totalPage) {
				startIndex = 0;
				endIndex = totalPage - startIndex - 1;
			} else {
				int leftOffset = getLeftOffset();
				int middleOffset = getMiddleOffset();
				if (-1 == middleOffset) {
					startIndex = 0;
					endIndex = groupSize - 1;
				} else if (currentPage <= leftOffset) {
					startIndex = 0;
					endIndex = groupSize - 1;
				} else {
					startIndex = currentPage - leftOffset - 1;
					if (currentPage + rightOffset > totalPage) {
						endIndex = totalPage - 1;
					} else {
						endIndex = currentPage + rightOffset - 1;
					}
				}
			}
		}
	}

	public int getCurrentPage() {
		if (currentPage <= 0) {
			currentPage = 1;
		} else {
			int totalPage = getTotalPage();
			if (totalPage > 0 && currentPage > getTotalPage()) {
				currentPage = totalPage;
			}
		}
		return currentPage;
	}

	public void setCurrentPage(int currentPage) {
		this.currentPage = currentPage;
	}

	public int getPageSize() {
		if (pageSize <= 0) {
			pageSize = 10;
		}
		return pageSize;
	}

	public void setPageSize(int pageSize) {
		this.pageSize = pageSize;
	}

	public int getTotalRecord() {
		return totalRecord;
	}

	public void setTotalRecord(int totalRecord) {
		this.totalRecord = totalRecord;
	}

	public int getTotalPage() {
		int totalRecord = getTotalRecord();
		if (totalRecord == 0) {
			totalPage = 0;
		} else {
			int pageSize = getPageSize();
			totalPage = totalRecord % pageSize == 0 ? totalRecord / pageSize : (totalRecord / pageSize) + 1;
		}
		return totalPage;
	}

	public void setTotalPage(int totalPage) {
		this.totalPage = totalPage;
	}

	public int getStartIndex() {
		return startIndex;
	}

	public void setStartIndex(int startIndex) {
		this.startIndex = startIndex;
	}

	public int getEndIndex() {
		return endIndex;
	}

	public void setEndIndex(int endIndex) {
		this.endIndex = endIndex;
	}

	public int getGroupSize() {
		if (groupSize <= 0) {
			groupSize = 10;
		}
		return groupSize;
	}

	public void setGroupSize(int groupSize) {
		this.groupSize = groupSize;
	}

	public int getLeftOffset() {
		leftOffset = getGroupSize() / 2;
		return leftOffset;
	}

	public void setLeftOffset(int leftOffset) {
		this.leftOffset = leftOffset;
	}

	public int getRightOffset() {
		int groupSize = getGroupSize();
		if (groupSize % 2 == 0) {
			rightOffset = (groupSize / 2) - 1;
		} else {
			rightOffset = groupSize / 2;
		}
		return rightOffset;
	}

	public void setRightOffset(int rightOffset) {
		this.rightOffset = rightOffset;
	}

	/** 中心位置索引[从1开始计算] */
	public int getMiddleOffset() {
		int groupSize = getGroupSize();
		int totalPage = getTotalPage();
		if (groupSize >= totalPage) {
			return -1;
		}
		return getLeftOffset() + 1;
	}

	public String[] getPageRange() {
		setRangeIndex();
		int size = endIndex - startIndex + 1;
		if (size <= 0) {
			return new String[0];
		}
		if (totalPage == 1) {
			return new String[] { "1" };
		}
		pageRange = new String[size];
		for (int i = 0; i < size; i++) {
			pageRange[i] = (startIndex + i + 1) + "";
		}
		return pageRange;
	}

	public void setPageRange(String[] pageRange) {
		this.pageRange = pageRange;
	}

	public void setItems(Collection<T> items) {
		this.items = items;
	}
    public Collection<T> getItems() {
		return items;
	}
	public void setDocList(List<Document> docList) {
		this.docList = docList;
	}
    public List<Document> getDocList() {
		return docList;
	}

	public void setAfterDoc(ScoreDoc afterDoc) {
		this.afterDoc = afterDoc;
	}
    public ScoreDoc getAfterDoc() {
		setAfterDocId(afterDocId);
		return afterDoc;
	}

	public void setAfterDocId(int afterDocId) {
		this.afterDocId = afterDocId;
		if (null == afterDoc) {
			this.afterDoc = new ScoreDoc(afterDocId, 1.0f);
		}
	}
    public int getAfterDocId() {return afterDocId;}
	
    /*构造方法*/
	public Page() {}
	public Page(int currentPage, int pageSize) {
		this.currentPage = currentPage;
		this.pageSize = pageSize;
	}
	public Page(int currentPage, int pageSize, Collection<T> items) {
		this.currentPage = currentPage;
		this.pageSize = pageSize;
		this.items = items;
	}
	public Page(int currentPage, int pageSize, Collection<T> items, int groupSize) {
		this.currentPage = currentPage;
		this.pageSize = pageSize;
		this.items = items;
		this.groupSize = groupSize;
	}
	public Page(int currentPage, int pageSize, int groupSize, int afterDocId) {
		this.currentPage = currentPage;
		this.pageSize = pageSize;
		this.groupSize = groupSize;
		this.afterDocId = afterDocId;
	}	
}
```

会话记录 IQCConversationInfoBean  实体类 

```java
public class IQCConversationInfoBean extends EntityBean {
	// serialVersionUID
    private static final long serialVersionUID = -4459013070304617092L;
    private String serialNo;// 通话编号
    private Date callTime; // 呼入时间
    private String callDate; // 会话日期
    private Date hangupTime; // 挂机时间
    private String language;  // 语种
    private String agentContent; // 座席通话内容
    private String custContent; // 客户通话内容
    private String allContent; //全部通话内容
    private String agentFirst;  // 座席首句
    private String agentLast; // 座席尾句
    private String custFirst; // 客户首句
    private String custLast; // 客户尾句
    private Float agentMaxSpeed=0f; // 座席最大语速
    private Float agentMinSpeed=0f; // 座席最小语速
    private Float agentAvgSpeed=0f; // 座席平均语速
    private Float custMaxSpeed=0f; // 座席最大语速
    private Float custMinSpeed=0f; // 座席最小语速
    private Float custAvgSpeed=0f; // 座席平均语速
    private Integer silenceSeconds; // 静音时长
    private String sumNo; //小结编号
    private Float silencePercent; // 静音占比
    private String mediaType; // 会话类型
    private Integer maxSilenceSeconds; // 最大静音时长
    private String accountCode; // 座席用户号
    private String empName; // 座席姓名
    private Integer talkSeconds; // 通话时长
    private  Integer minTalkSeconds; // 最小通话时长
    private  Integer maxTalkSeconds; // 最大通话时长
    private Float custMaxEmotion=0f; // 客户最大情绪
    private Float custMinEmotion=0f; // 客户最小情绪
    private Float custAvgEmotion=0f; // 客户平均情绪
    private Float agentMaxEmotion=0f; // 座席最大情绪
    private Float agentMinEmotion=0f; // 座席最小情绪
    private Float agentAvgEmotion=0f; // 座席平均情绪
    private Integer silenceCount; // 静音次数
    private String overLap; // 是否重叠音
    private String callNo; // 来电号码
    private String custNo; // 客户号
    private String custGender; // 客户性别
    private String businessGroupCode; // 座席组别
    private String satisfiedType; // 满意度
    private String custName; // 客户姓名
    private String taskCode; // 任务编号
    private String analysisResult; // 分析结果
    private String channelCode; // 数据渠道
    //扩展字段
    private String filePath; // 文件路径
    private String beginTime; // 查询条件开始时间
    private String endTime; // 查询条件结束时间
    
    .....
}
```

lucene 工具类 LuceneUtils

```java
public class LuceneUtils {
    // 打开索引目录
	public static FSDirectory openFSDirectory(String luceneDir) {
		FSDirectory directory = null;
		try {
			directory = FSDirectory.open(Paths.get(luceneDir));
			/**
			 * 注意：isLocked方法内部会试图去获取Lock,如果获取到Lock， 会关闭它，否则return
			 * false表示索引目录没有被锁， 这也就是为什么unlock方法被从IndexWriter类中移除的原因
			 */
			IndexWriter.isLocked(directory);
		} catch (IOException e) {
			e.printStackTrace();
		}
		return directory;
	}
    
    //  创建索引阅读器（多目录）
	public static MultiReader getMultiReader(List<String> dirPathList) throws IOException {
	    List<IndexReader> indexReaders = new ArrayList<IndexReader>();
	    for(String dirPath : dirPathList){
	        Directory directory = openFSDirectory(dirPath);
	        if(DirectoryReader.indexExists(directory)){
	            IndexReader reader = DirectoryReader.open(directory);
	            indexReaders.add(reader);
	        }
	    }
	    if(indexReaders.size()>0){
	        return new MultiReader(indexReaders.toArray(new IndexReader[indexReaders.size()]));
	    }else{
	        return null;
	    }
	}
    
    // 创建索引查询器（多目录）
	public static IndexSearcher getMultiIndexSearcher
        (MultiReader multiReader, ExecutorService executor) {
        if(null != executor) {
            return new IndexSearcher(multiReader, executor);
        }else{
            return new IndexSearcher(multiReader);
        }
    }
    
    // 获取符合条件的总记录数
    public static ScoreDoc[] searchTotalRecord(IndexSearcher search, Query query) {
        ScoreDoc[] docs = null;
        try {
            TopDocs topDocs = search.search(query, Integer.MAX_VALUE);
            if (topDocs==null || topDocs.scoreDocs==null || topDocs.scoreDocs.length==0) {
                return docs;
            }
            docs = topDocs.scoreDocs;
        }catch (IOException e) {
            e.printStackTrace();
        }
        return docs;
    }
    
    // Lucene多目录分页查询
    public static void pageQuery(IndexSearcher searcher, Query query, Page<Document> page)
        throws IOException {
        ScoreDoc[] scoreDocs = searchTotalRecord(searcher, query);
        if (null != scoreDocs) {
            // 设置总记录数
            page.setTotalRecord(scoreDocs.length);
            ScoreDoc afterDoc = null;
            if (page.getCurrentPage() > 1) {
                afterDoc = scoreDocs[(page.getCurrentPage() - 1) * page.getPageSize() - 1];
            }
            TopDocs topDocs = searcher.searchAfter(afterDoc, query, page.getPageSize());
            List<Document> docList = new ArrayList<Document>();
            ScoreDoc[] docs = topDocs.scoreDocs;
            int index = 0;
            for (ScoreDoc scoreDoc : docs) {
                int docID = scoreDoc.doc;
                Document document = searcher.doc(docID);
                if (index == docs.length - 1) {
                    page.setAfterDoc(scoreDoc);
                    page.setAfterDocId(docID);
                }
                docList.add(document);
                index++;
            }
            page.setItems(docList);
        }
    }
    
}
```

service

```java
@Service("iqcConversationIndexService")
public class IQCConversationIndexService{
    private SimpleDateFormat formatter = new SimpleDateFormat("yyyyMM");
    // 获取索引目录
    private static final String INDEX_ROOT_PATH = SysConstant.config.getProperty("indexRootPath");
    
    public ResultBean<IQCConversationInfoBean> getIQCConversationInfoPageByBean
        (IQCConversationInfoBean bean, int currentPage, int pageSize) throws Exception {
        ResultBean<IQCConversationInfoBean> rb = new ResultBean<IQCConversationInfoBean>() ;
        ExecutorService _executorService = Executors.newFixedThreadPool(10);
        MultiReader multiReader = null;
        IndexSearcher searcher = null;
        try {
            Page<Document> page = new Page<Document>(currentPage, pageSize);
             // 根据时间获取检索文件夹
            Date beginDate = formatter.parse(bean.getBeginTime().substring(0,6));
            Date endDate = formatter.parse(bean.getEndTime().substring(0,6));
            String dirName = "";
            //计算时间区间内每个日期文件夹并解析其中的文件
            Calendar tempStart = Calendar.getInstance();
            tempStart.setTime(endDate);
            List<String> indexDirecorys = new ArrayList<String>();
            while(beginDate.getTime() <= endDate.getTime()){
                dirName = JCalendar.getDateStr(endDate, "yyyyMM");
                indexDirecorys.add(INDEX_ROOT_PATH + File.separator + dirName);
                tempStart.add(Calendar.MONTH, -1);
                endDate = tempStart.getTime();
            }
            multiReader = LuceneUtils.getMultiReader(indexDirecorys);
            if(null!=multiReader){
                searcher = LuceneUtils.getMultiIndexSearcher(multiReader,_executorService);
                BooleanQuery booleanQuery = dealBooleanQueryTerms(bean);
                LuceneUtils.pageQuery(searcher, booleanQuery, page);
                if (page==null || page.getItems()==null || page.getItems().size()==0) {
                    log.debug("未检索到记录");
                    rb.setTotal(0l);
                }else{
                    for(Document doc : page.getItems()){
                        IQCConversationInfoExtBean temp = new IQCConversationInfoExtBean();
                        for(IndexableField field : doc.getFields()){
                            setConversationValue(temp, field);
                        }
                        rb.getRows().add(temp);
                    }
                    rb.setTotal((long) page.getTotalRecord());
                }
            }else{
                log.info("不存在索引会话记录【"+bean.getBeginTime()+"】至【"+bean.getEndTime()+"】");
                rb.setTotal(0l);
            }
            rb.setReturnCode(SysConstant.SYS_RETURN_SUCCESS_CODE);
            rb.setReturnMessage(SysConstant.SYS_RETURN_SUCCESS_MESSAGE);
        }catch (Exception e) {
            log.error("会话检索异常", e);
            rb.setReturnCode(SysConstant.SYS_RETURN_EXCEPTION_CODE);
            rb.setReturnMessage("检索异常："+e.getMessage());
        }finally{
            if(!_executorService.isShutdown()){
                _executorService.shutdown();
            }
            if(null!=searcher){
                searcher.getIndexReader().close();
            }
            if(null!=multiReader){
                 multiReader.close();
            }
        }
        return rb;
    }
    
	// 多种条件组合检索
    private BooleanQuery dealBooleanQueryTerms
        (IQCConversationInfoBean bean) throws Exception{
        BooleanQuery.Builder booleanQueryBuilder = new BooleanQuery.Builder();
        //关键词
        if(StringUtil.isNotEmpty(bean.getAllContent())){
            Term t = new Term("allContent", ".*"+bean.getAllContent()+".*");
            Query query = new RegexQuery(t);
            booleanQueryBuilder.add(query, BooleanClause.Occur.MUST);
        }
        //客户语速
        if(null!=bean.getCustMinSpeed() && null!=bean.getCustMaxSpeed()){
            Query query = NumericRangeQuery.newFloatRange(
                "custMaxSpeed", bean.getCustMinSpeed(), bean.getCustMaxSpeed(), true, true);
            booleanQueryBuilder.add(query, BooleanClause.Occur.MUST);
        }
        //来电号码
        if(StringUtil.isNotEmpty(bean.getCallNo())){
            Query query = new TermQuery(new Term("callNo", bean.getCallNo()));
            booleanQueryBuilder.add(query, BooleanClause.Occur.MUST);
        }
        //呼入时间
        if(null!=bean.getBeginTime() && null!=bean.getEndTime()){
            long beginTime = JCalendar.getDate(bean.getBeginTime(), "yyyyMMddHHmmss").getTime();
            long endTime = JCalendar.getDate(bean.getEndTime(), "yyyyMMddHHmmss").getTime();
            Query query = NumericRangeQuery.newLongRange(
                "callTime", beginTime,endTime, true, true);
            booleanQueryBuilder.add(query, BooleanClause.Occur.MUST);
        }
        //通话时长
        if(null!=bean.getMinTalkSeconds() && null!=bean.getMaxTalkSeconds()){
            Query query = NumericRangeQuery.newIntRange(
                "maxTalkSeconds", bean.getMinTalkSeconds(), bean.getMaxTalkSeconds(), true, true);
            booleanQueryBuilder.add(query, BooleanClause.Occur.MUST);
        }
        
    }
    
  	// 通过反射设置对象的值-单层不考虑继承
    public static void setConversationValue(IQCConversationInfoBean iqcConversationInfoBean,
                                            IndexableField indexableField){
        String fieldName = indexableField.name();
        if (null != iqcConversationInfoBean && !fieldName.equals("serialVersionUID")) {
            try {
                Class<?> clazz1 = IQCConversationInfoBean.class;
                Field[] fields = clazz1.getDeclaredFields();
                for(Field field : fields){
                    if(field.getName().equals(fieldName)){
                        String name=firstLetterUpperCase(field.getName());
                        String setmethodName="set"+name;
                        Method m = clazz1.getDeclaredMethod(setmethodName, field.getType());
                        switch(field.getType().getSimpleName()){
                           case "String": 
                                m.invoke(iqcConversationInfoBean, indexableField.stringValue());
                                break;
                           case "Float": 
                                m.invoke(iqcConversationInfoBean, indexableField.numericValue());
                                break;
                           case "Long": 
                                m.invoke(iqcConversationInfoBean, indexableField.numericValue());
                                break;
                           case "Date": 
                                m.invoke(iqcConversationInfoBean,
                                         new Date((long) indexableField.numericValue()));
                                break;
                        }
                        break;
                    }
                }
            }catch (Exception e) {
                e.printStackTrace();
            }
        }
    }
    
    //
    public static String firstLetterUpperCase(String str){
        if(str==null||str.length()<2){
            return str;
        }else{
            String first=str.substring(0, 1).toUpperCase();
            return first+str.substring(1,str.length());
        }
    }
    
         
}
```

### 检索流程

根据关键词解析（queryParser）出查询条件query（Termquery）,利用检索工具（indexSearcher）去索引库获取文档的id,然后再根据文档 id去文档信息库获取文档信息。

分词器不同，建立的索引数据就不同；比较通用的一个中文分词器IKAnalyzer的用法。

## 结果展示

![](http://blogimg.nos-eastchina1.126.net/shenwf20190316111029-773415.jpg)

![](http://blogimg.nos-eastchina1.126.net/shenwf20190316111103-683410.jpg)

## 注意事项

使用多线程。在使用多线程时，只需要创建线程池即可。事实上，Lucene 在  IndexSearcher 中 判断是否有 executor ,如果 IndexSearcher 有 executor ，则会由每个线程控制一部分索引的读取，而且查询的过程采用的是 future 机制，这种方式是边读边往结果集里边追加数据，这样异步处理机制提升了效率。具体源码可看 IndexSearcher  的 search。

控制检索文件夹。如果同时检索的文件夹太多的时，会增加 GC 负担

![](http://blogimg.nos-eastchina1.126.net/shenwf20190316111130-898673.jpg)

在你能承受的范围内设置更多的内存。以免造成内存溢出

![](http://blogimg.nos-eastchina1.126.net/shenwf20190316111155-172723.jpg)

## 总结

本文是对 Lucene 多条件检索的记录。实现多目录多线程的检索方式；实现分页功能；实现多种类型的条件查询以及数据量较大时检索的注意点进行记录。为了更好的使用 Lucene 后面将总结如何提高 Lucene 的检索效率。

全文检索，lucene 在 匹配效果、速度和效率是极大的优于数据库的。