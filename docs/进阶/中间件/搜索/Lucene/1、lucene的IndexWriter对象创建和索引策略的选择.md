---
tags:
	- lucene
categories: lucene
title: lucene的IndexWriter对象创建和索引策略的选择
---
# lucene（1）---lucene的IndexWriter对象创建和索引策略的选择

因工作的需要（数据量大造成原有系统查询效率低），最近做了搜索引擎相关的内容，选择了lucene5版本（15年发布的）。

<!--more-->

lucene是一个开放源代码的全文搜索引擎开发工具包，提供了简单强大的搜索引擎接口，其优点如下：

- 数据以索引文件的形式存储，索引文件可以跨平台，只要保证索引完整，复制到任何机器或者磁盘空间均可以查询索引内容；
- 在传统全文检索引擎的倒排索引的基础上，实现了分块索引，能够针对新的文件建立小文件索引，提升索引速度。然后通过与原有索引的合并，达到优化的目的；
- 索引的构建和查询都十分简洁，有强大的类库实现相关功能；
- 开发源代码，论坛和资源十分丰富。

索引的构建过程描述如下：

 1）判断JRE版本是否为64位和是否支持堆外内存，并创建

​               1.1  如果满足条件，创建MMapDirectory，此种Directory可以有效的利用虚拟机内存地址空间 ；

​               1.2  如果不满足以上条件，判断系统是否是windows,如果满足条件，创建SimpleFSDirectory，此种directory提供了性能不太高的多线程支持，lucene推荐使用[NIOFSDirectory](https://blog.csdn.net/wuyinggui10000/article/details/45502445)`或者MMapDirectory来替代之；`

​               1.3 如果以上均不满足，创建NIOFSDirectory对象，此种directory的英文说明为

```java
An FSDirectory implementation that uses java.nio's FileChannel's positional read, which allows multiple threads to read from the same file without synchronizing
```

大意是一个利用了java nio中FileChannel的FSDirectory实现，允许无syschronized的对同一文件进行多线程读

 2）词库分析器Analyzer创建（需要注意的是使用哪种Analyzer进行索引查询，创建的时候也要使用对应的索引器，否则查询结果有问题）

 3）IndexWriterConfig对象创建,并获取IndexWriter对象

​            3.1 判断是覆盖索引还是追加索引，如果是覆盖索引indexWriterConfig.setOpenMode(IndexWriterConfig.OpenMode.CREATE);

​            3.2 如果追加indexWriterConfig.setOpenMode(IndexWriterConfig.OpenMode.CREATE_OR_APPEND);

​    4) 遍历根据要索引的对象列表，对单个对象的field进行lucene相关field构建，添加到Document对象中

​    5）IndexWriter对索引进行写入；

​    6）IndexWriter执行commit()和close()结束索引创建过程

以lucene5为例，索引器的创建如下：

```java
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
	    if (create){
	        iwc.setOpenMode(IndexWriterConfig.OpenMode.CREATE);
	    }else {
	        iwc.setOpenMode(IndexWriterConfig.OpenMode.CREATE_OR_APPEND);
	    }
	    IndexWriter writer = new IndexWriter(dir, iwc);
	    return writer;
	}
```

下面给出当时工作需要的创建索引测试例子

```java
public class MultiThreadIndexTest {
    
    private static Pattern p_html = Pattern.compile("<[^>]+>", Pattern.CASE_INSENSITIVE);
    
	public static void main(String[] args) throws IllegalArgumentException, IllegalAccessException, ParseException {
		String indexPath = "D:\\LuceneIndex\\IQC\\ag\\voice\\20190201";
		System.out.println("开始创建索引");
		IQCConversationInfoBean iqcConversationInfoBean = new IQCConversationInfoBean();
		iqcConversationInfoBean.setMediaType("voice");
		iqcConversationInfoBean.setCallDate("20190201");
		iqcConversationInfoBean.setCallTime(JCalendar.getDate("20190201145454", "yyyyMMddHHmmss"));
		iqcConversationInfoBean.setHangupTime(JCalendar.getDate("20190201145959", "yyyyMMddHHmmss")); // 挂机时间
		iqcConversationInfoBean.setChannelCode("ag");
		iqcConversationInfoBean.setAccountCode("Admin");
		iqcConversationInfoBean.setEmpName("超级管理员");
		iqcConversationInfoBean.setLanguage("1");
		iqcConversationInfoBean.setCustNo("123456");
		iqcConversationInfoBean.setCustName("洪尼玛");
		iqcConversationInfoBean.setCallNo("18565279427");
		iqcConversationInfoBean.setSatisfiedType("1"); // 满意度
        // 座席通话内容
		iqcConversationInfoBean.setAgentContent("您好，请问有什么可以帮到您请问是陈女士吗啊，对从您实名号，您是通过什么渠道转的？手机银行的您，是同行转账，还是跨行转账？那真没有到账的话，要以系统处理为准呢？他这些要以系统处理为准的，今天是年30系统的。我这边帮您看一下，嗯撤销不了哇，因为您当时选择的普通转账那这个你这什么时候到账，就看那边的系统处理为准的那您可以用此到家吗，你干嘛用普通到账呢啊啊我这边看到您用的是普通转账来的是系统转帐，");
		// agentFirst
		iqcConversationInfoBean.setAgentFirst("您好，请问有什么可以帮到您请问是陈女士吗啊，对从您实名号，您是通过什么渠道转的？手机银行的您，是同行转账，还是跨行转账？那真没有到账的话，要以系统处理为准呢？他这些要以系统处理为准的，今天是年30系统的。我这边帮您看一下，嗯撤销不了哇，因为您当时选择的普通转账那这个你这什么时候到账，就看那边的系统处理为准的那您可以用此到家吗，你干嘛用普通到账呢啊啊我这边看到您用的是普通转账来的是系统转帐，");
		// agentLast
		iqcConversationInfoBean.setAgentLast("已经登记好了");
		// 客户通话内容
		iqcConversationInfoBean.setCustContent("你好，我想问一下，我这将是广州农农商银行，现在我回来到这边那个密码，搞忘记了，可不可以在我换地方改密改密码呢，还还找密码呢");
		// 客户首句
		iqcConversationInfoBean.setCustFirst("你好，我想问一下，我这将是广州农农商银行，现在我回来到这边那个密码，搞忘记了，可不可以在我换地方改密改密码呢，还还找密码呢");
		// 客户尾句
		iqcConversationInfoBean.setCustLast("结果登记");
		// 全部通话内容
		    for(int i=0;i<5000;i++){
                iqcConversationInfoBean.setAllContent("<li start=13070 end=14870 emotion=5.0 speed=4.0 >坐席：您好，请问有什么可以帮您</li>" +
                        "<li start=14880 end=21470 emotion=6.0 speed=3.0 >客户：然后你们这个银行，这里呀，那柜员机老是故障啊，嗯，也没用过的维修的</li>" +
                        "<li start=0 silence=6  class=\"silences\"  > 6S. </li>" +
                        "<li start=21480 end=23270 emotion=6.0 speed=2.68 >坐席：嗯，哪个柜员机呀</li>" +
                        "<li start=23280 end=27780 emotion=6.0 speed=1.46 >客户：我们广州市白云区江高镇</li>" +
                        "<li start=29000 end=30200 emotion=6.0 speed=3.0 >客户：然后，新楼村</li>" +
                        "<li start=0 silence=6  class=\"silences\"  > 6S. </li>" +
                        "<li start=30210 end=32250 emotion=6.0 speed=4.11 >坐席：嗯桂圆是有没有那个订单号啊？</li>" +
                        "<li start=32890 end=36190 emotion=6.0 speed=3.81 >坐席：在屏幕上方有一个本机终端号的，有没有看到？</li>" +
                        "<li start=36200 end=38590 emotion=6.0 speed=3.51 >客户：没看，但是我我报你帮我查一下</li>" +
                        "<li start=38600 end=40690 emotion=6.0 speed=4.01 >坐席：那个地址在哪里啊，刚刚在哪里</li>" +
                        "<li start=40700 end=45490 emotion=6.0 speed=2.12 >客户：呃，江高镇，然后新楼村新楼路31号</li>" +
                        "<li start=0 silence=4  class=\"silences\"  > 4S. </li>" +
                        "<li start=45500 end=46630 emotion=5.0 speed=1.06 >坐席：嗯好</li>" +
                        "<li start=0 silence=11  class=\"silences\"  > 11S. </li>" +
                        "<li start=58500 end=60550 emotion=6.0 speed=3.51 >坐席：他旁边没有网点的他，旁边</li>" +
                        "<li start=61420 end=64720 emotion=6.0 speed=2.36 >客户：没有就这个柜员机，我们学校</li>" +
                        "<li start=0 silence=4  class=\"silences\"  > 4S. </li>" +
                        "<li start=64730 end=66520 emotion=6.0 speed=2.34 >坐席：嗯，什么学校啊</li>" +
                        "<li start=66530 end=68230 emotion=6.0 speed=2.47 >客户：广东、江南理工</li>" +
                        "<li start=68870 end=73970 emotion=6.0 speed=3.17 >坐席：学校里面的嘛，对江南理工就是那个江西的，在南方的南吗？</li>" +
                        "<li start=73980 end=79170 emotion=6.0 speed=1.38 >客户：然后是江南江南是南方的南</li>" +
                        "<li start=0 silence=6  class=\"silences\"  > 6S. </li>" +
                        "<li start=80140 end=84040 emotion=6.0 speed=3.69 >坐席：江南理工没有看到他这个有柜员，机的地方啊，他是在</li>" +
                        "<li start=84050 end=86900 emotion=6.0 speed=2.1 >客户：哦，江南理工技工学校</li>" +
                        "<li start=0 silence=4  class=\"silences\"  > 4S. </li>" +
                        "<li start=88210 end=89410 emotion=6.0 speed=3.0 >坐席：在哪个区的？</li>" +
                        "<li start=89420 end=91140 emotion=6.0 speed=2.09 >客户：白云区江高镇</li>" +
                        "<li start=98540 end=99740 emotion=6.0 speed=3.0 >客户：然后那个金融</li>" +
                        "<li start=0 silence=10  class=\"silences\"  > 10S. </li>" +
                        "<li start=99750 end=105570 emotion=6.0 speed=2.47 >坐席：他是在那个往港、煤炭地质局对面，那个技工学校吗？</li>" +
                        "<li start=106250 end=109250 emotion=6.0 speed=3.0 >客户：不是我们是白云区江高镇新楼村的</li>" +
                        "<li start=0 silence=3  class=\"silences\"  > 3S. </li>" +
                        "<li start=109260 end=111950 emotion=6.0 speed=2.23 >坐席：新农村新是哪个新啊？</li>" +
                        "<li start=111960 end=113150 emotion=6.0 speed=2.52 >客户：新中国的新</li>" +
                        "<li start=113160 end=114350 emotion=4.0 speed=0.5 >坐席：嗯</li>" +
                        "<li start=114360 end=116880 emotion=6.0 speed=3.33 >客户：楼市大龙的龙村，是村庄的村。</li>" +
                        "<li start=0 silence=3  class=\"silences\"  > 3S. </li>" +
                        "<li start=118240 end=120340 emotion=6.0 speed=3.14 >坐席：没有看到他这个地点有啊</li>" +
                        "<li start=120350 end=123340 emotion=6.0 speed=3.01 >客户：那我，们在银行的奇怪呢，是因为</li>" +
                        "<li start=0 silence=3  class=\"silences\"  > 3S. </li>" +
                        "<li start=123350 end=126640 emotion=6.0 speed=3.1 >坐席：您要看一下，他去柜员机的位置才行啊</li>" +
                        "<li start=126650 end=129340 emotion=6.0 speed=4.01 >客户：我们会员就直接就是在我们学校"+i+"里面吗？</li>" +
                        "<li start=129350 end=135630 emotion=7.0 speed=3.53 >坐席：是什么学校啊，刚才跟你说了广东将的，其实没有跟他有这个学校，有没有全称呢？</li>" +
                        "<li start=136300 end=139210 emotion=7.0 speed=2.26 >客户：广东、江南理工技工学校</li>" +
                        "<li start=0 silence=6  class=\"silences\"  > 6S. </li>" +
                        "<li start=142190 end=148190 emotion=6.0 speed=2.8 >坐席：但是，我们收他技工，学校就只有出来刚刚那个旺港煤炭地质局</li>" +
                        "<li start=148200 end=158090 emotion=6.0 speed=2.73 >客户：您的商品一双鞋呢，不，是旺旺，我们是在广州市白云区江高镇新楼村新楼路31号，这个详细的地址</li>" +
                        "<li start=0 silence=9  class=\"silences\"  > 9S. </li>" +
                        "<li start=158100 end=162340 emotion=6.0 speed=4.1 >坐席：嗯，您学校地址，但是，他柜员机不，他不一定是这么登记的吗？</li>" +
                        "<li start=162980 end=172900 emotion=6.0 speed=3.81 >客户：那你这个是你们的问题了，你像我这个我是我现在就是就是现在要要告诉你们呢，这里有我们这个，这个地址啊，你们所学校有因为台风的呀啊</li>" +
                        "<li start=0 silence=11  class=\"silences\"  > 11S. </li>" +
                        "<li start=173540 end=174440 emotion=6.0 speed=3.33 >坐席：好评返现。</li>" +
                        "<li start=174450 end=178640 emotion=7.0 speed=3.57 >客户：我整天啊，有问题一直，搞错了，你们要反映上去，上面</li>" +
                        "<li start=0 silence=4  class=\"silences\"  > 4S. </li>" +
                        "<li start=178650 end=184940 emotion=7.0 speed=3.52 >坐席：但是，您这样去把您那个柜员机上面，它有一个模板套，您的会员区发布宝贝订单号</li>" +
                        "<li start=185550 end=186360 emotion=7.0 speed=2.22 >坐席：好不好</li>" +
                        "<li start=184950 end=185540 emotion=6.0 speed=1.01 >客户：嗯</li>" +
                        "<li start=187000 end=190300 emotion=6.0 speed=3.81 >坐席：但是，我们确实现在没有看到您这个地方，有啊</li>" +
                        "<li start=190310 end=192100 emotion=7.0 speed=3.68 >客户：那您说一下怎么办，因为</li>" +
                        "<li start=192110 end=193600 emotion=6.0 speed=4.83 >坐席：我们没有查到我们就没有办</li>" +
                        "<li start=193610 end=196850 emotion=7.0 speed=4.44 >客户：反馈到我想问一下，你，说你，你说让我现在怎么办？</li>" +
                        "<li start=0 silence=3  class=\"silences\"  > 3S. </li>" +
                        "<li start=197550 end=201450 emotion=6.0 speed=4.76 >坐席：所以，您看一下您那个柜员机，他旁边是只有一台吗，还是什么情况啊</li>" +
                        "<li start=201460 end=205500 emotion=6.0 speed=2.97 >客户：有一台机黑屏，什么都看不到，整天出故障啊</li>" +
                        "<li start=0 silence=5  class=\"silences\"  > 5S. </li>" +
                        "<li start=206990 end=208360 emotion=6.0 speed=3.06 >坐席：嗯，稍等一下啊</li>" +
                        "<li start=209060 end=211320 emotion=6.0 speed=3.71 >客户：你们现在真的老子真的有问题啊</li>" +
                        "<li start=213040 end=220180 emotion=6.0 speed=3.78 >客户：你报一个详细地址了，还问我是在哪个，你说的那个不是在那里，这里你们就往那low爆了，知道吧</li>" +
                        "<li start=220820 end=223260 emotion=6.0 speed=2.7 >客户：你要找人过来核实修改呀</li>" +
                        "<li start=0 silence=33  class=\"silences\"  > 33S. </li>" +
                        "<li start=256910 end=257430 emotion=4.0 speed=1.15 >客户：嗯</li>" +
                        "<li start=258590 end=259330 emotion=5.0 speed=1.62 >客户：是吧</li>");
                iqcConversationInfoBean.setSerialNo(UUID.randomUUID().toString());
		        iqcConversationInfoBean.setAgentMaxSpeed((float)Math.round(Math.random()*100));
		        iqcConversationInfoBean.setAgentMinSpeed((float)Math.round(Math.random()*100));
		        iqcConversationInfoBean.setAgentAvgSpeed((float)Math.round(Math.random()*100));
		        iqcConversationInfoBean.setAgentMaxEmotion((float)Math.round(Math.random()*100));
		        iqcConversationInfoBean.setAgentMinEmotion((float)Math.round(Math.random()*100));
		        iqcConversationInfoBean.setAgentAvgEmotion((float)Math.round(Math.random()*100));
		        iqcConversationInfoBean.setCustMaxSpeed((float)Math.round(Math.random()*100));
		        iqcConversationInfoBean.setCustMinSpeed((float)Math.round(Math.random()*100));
		        iqcConversationInfoBean.setCustAvgSpeed((float)Math.round(Math.random()*100));
		        iqcConversationInfoBean.setCustMaxEmotion((float)Math.round(Math.random()*100));
		        iqcConversationInfoBean.setCustMinEmotion((float)Math.round(Math.random()*100));
		        iqcConversationInfoBean.setCustAvgEmotion((float)Math.round(Math.random()*100));
		        Map<String, Object> textFiled = ReflectUtil.reflectObjectToMap(iqcConversationInfoBean, true);
		        
		        Document doc = createDoc(textFiled);
		        if(null!=doc){
		            addDoc(indexPath, doc, null);
		        }
		        //System.out.println(i+"索引创建完毕");
		    }
		System.out.println("创建索引完毕");
	}
	
    private static Document createDoc(Map<String, Object> textFiled) {
        Document doc = null;
        try {
            doc = null;
            if (textFiled != null && textFiled.size() > 0) {
                doc = new Document();
                // 遍历需要增加到索引的属性
                List<String> NOT_ANALYZED = new ArrayList<String>();// 非分析域，但是需要保存
                for (String Not : "id;status;update;agentFirst;agentLast;agentContent;custFirst;custLast;custContent;allContent;".split(";")) {
//                    for (String Not : "id;status;update;mediaType;callDate;channelCode;accountCode;empName;language;custNo;custName;callNo;satisfiedType;agentFirst;agentLast;agentContent;custFirst;custLast;custContent;allContent;".split(";")) {
                    NOT_ANALYZED.add(Not);
                }
                for (String keyName : textFiled.keySet()) {
                    if (keyName != null && !keyName.isEmpty()) {
                        // if
                        // (keyName.equals(SysConstant.config.getProperty("kbmsID"))||keyName.indexOf(SysConstant.config.getProperty("DIM"))>=0)
                        // {
                        if (keyName.equals("serialNo")) {
                            doc.add(LuceneHelper.getnotanalyzedField(keyName, textFiled.get(keyName)));
                        } else if (NOT_ANALYZED.contains(keyName)) {
                            Matcher m_html = p_html.matcher((String) textFiled.get(keyName));
                            doc.add(new Field(keyName, m_html.replaceAll(""), Field.Store.YES,
                                    Field.Index.NOT_ANALYZED));
                        } else {
                            doc.add(LuceneHelper.getField(keyName, textFiled.get(keyName)));
                        }
                    }
                }
            } else {
                System.out.println("无法初始化doc");
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return doc;
    }   
    public static Field getField(String fieldName, Object value) {
        FieldType fieldType = new FieldType();
        fieldType.setOmitNorms(true);
        fieldType.setIndexOptions(IndexOptions.DOCS_AND_FREQS_AND_POSITIONS_AND_OFFSETS);
        fieldType.setStored(true);
        fieldType.setTokenized(false);
        if (value instanceof Integer) {
            return new IntField(fieldName, Integer.parseInt(value.toString()), fieldType);
            // return new
            // NumericDocValuesField(fieldName,Long.parseLong(value.toString()));
        } else if (value instanceof Long) {
            return new LongField(fieldName, (Long) value, fieldType);
        } else if (value instanceof Float) {
            return new FloatField(fieldName, (Float) value, fieldType);
        } else if (value instanceof Date) {
            return new LongField(fieldName, ((Date) value).getTime(), fieldType);
        } else {
            return new TextField(fieldName, value.toString(), Field.Store.YES);
        }
    }
    private static boolean addDoc(String indexPath, Document doc, String message) {
        boolean res = false;
        // 索引配置器
        IndexWriterConfig iwc = null;
        IndexWriter indexWriter = null;
        try {
            if (indexPath != null && !indexPath.isEmpty() && doc != null) {
                File indexDir = new File(indexPath);
                if (!indexDir.exists())
                    indexDir.mkdirs();
                if (indexDir.exists()) {
                    /* 创建索引文件 */
                    iwc = new IndexWriterConfig(ChineseAnalyzerUtil.getAnalyzer());
                    // 创建索引文件对象
                    Directory dir = FSDirectory.open(indexDir.toPath());
                    boolean isNeedCreate = indexDir.listFiles().length > 0 ? false : true;
                    if (isNeedCreate) {// 是否需要创建,默认 CREATE_OR_APPEND
                        iwc.setOpenMode(OpenMode.CREATE);
                        System.out.println("创建新索引,OpenMode==>>CREATE");
                    } else {
                    }
                    // 写入索引文件对象
                    indexWriter = new IndexWriter(dir, iwc);
                    // 加载到索引中
                    // 遍历需要增加到索引的属性
                    // 加载到索引文档
                    // indexWriter.addDocument(doc);
                    IndexableField kbmsID = doc.getField("serialNo");
                    if (kbmsID != null) {
                        String termId = kbmsID.name();
                        String termValue = kbmsID.stringValue();
                        indexWriter.updateDocument(new Term(termId, termValue), doc);
                    } else {
                        indexWriter.addDocument(doc);
                    }
                    // 提交到索引
                    indexWriter.commit();
                    indexWriter.close();
                    res = true;
                } else {
                    message = "contentPath 参数为空，正文无法读取，无法创建索引";
                }

            } else {
                message = "indexPath:" + indexPath + " 目录不存在，无法创建索引";
            }

        } catch (Exception e) {
            e.printStackTrace();
            System.out.println("创建Lucene索引异常");
        }
        if (message != null && !message.isEmpty())
            System.out.println("message-" + message);
        return res;
    }
}
```

`注意`：创建索引的内用是对一通通话的内容进行