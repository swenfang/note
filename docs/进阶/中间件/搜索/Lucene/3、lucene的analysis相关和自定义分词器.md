---
tags:
	- lucene
categories: lucene
title: lucene的analysis相关和自定义分词器
---
# lucene（3）---lucene的analysis相关和自定义分词器

## analysis说明

### lucene ananlysis应用场景

lucene提供了analysis用来将文本转换到索引文件或提供给IndexSearcher查询索引；

对于lucene而言，不管是索引还是检索，都是针对于纯文本输入来讲的；

通过lucene的强大类库我们可以访问各种格式的文档，如HTML、XML、PDF、[Word](https://www.baidu.com/s?wd=Word&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)、TXT等，

我们需要传递给lucene的只是文件中的纯文本内容；

<!--more-->

###  lucene的词语切分

lucene的索引和检索前提是其对文本内容的分析和词组的切分；比如，文档中有一句话叫“Hello World,Welcome to China”

我们想找到包含这段话的文档，而用户输入的查询条件又不尽详细（可能只是hello）

这里我们就需要用到lucene索引该文档的时候预先对文档内容进行切分，将词源和文本对应起来。

有时候对词语进行简单切分还远远不够，我们还需要对字符串进行深度切分，lucene不仅能够对索引内容预处理还可以对请求参数进行切分；

### 使用analyzer

lucene的索引使用如下：

```java
package com.lucene.analysis;
 
import java.io.IOException;
import java.io.StringReader;
 
import org.apache.lucene.analysis.Analyzer;
import org.apache.lucene.analysis.TokenStream;
import org.apache.lucene.analysis.standard.StandardAnalyzer;
import org.apache.lucene.analysis.tokenattributes.OffsetAttribute;
import org.junit.Test;
 
public class AnalysisTest {
	@Test
	public void tokenTest() {
		Analyzer analyzer = new StandardAnalyzer(); // or any other analyzer
		TokenStream ts = null;
		try {
			ts = analyzer.tokenStream("myfield", new StringReader(
					"some text goes here"));
			OffsetAttribute offsetAtt = ts.addAttribute(OffsetAttribute.class);
			ts.reset(); // Resets this stream to the beginning. (Required)
			while (ts.incrementToken()) {
				// Use AttributeSource.reflectAsString(boolean)
				// for token stream debugging.
				System.out.println("token: " + ts.reflectAsString(true));
 
				System.out.println("token start offset: "
						+ offsetAtt.startOffset());
				System.out.println("token end offset: "
						+ offsetAtt.endOffset());
			}
			ts.end(); 
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		} finally {
			try {
				ts.close();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
 
}
```

##  自定义Analyzer和实现自己的analysis模块

1.要实现自己的analyzer，我们需要继承Analyzer并重写其中的分词模块。

2.维护停止词词典

3.重写TokenStreamComponents方法，选择合适的分词方法，对词语进行过滤

示例代码如下

```java
package com.lucene.analysis.self;
 
import java.io.IOException;
 
import org.apache.lucene.analysis.Analyzer;
import org.apache.lucene.analysis.TokenStream;
import org.apache.lucene.analysis.Tokenizer;
import org.apache.lucene.analysis.core.LowerCaseTokenizer;
import org.apache.lucene.analysis.core.StopAnalyzer;
import org.apache.lucene.analysis.core.StopFilter;
import org.apache.lucene.analysis.tokenattributes.CharTermAttribute;
import org.apache.lucene.analysis.util.CharArraySet;
 
public class MyAnalyzer extends Analyzer {
	private CharArraySet stopWordSet;//停止词词典
	
	public CharArraySet getStopWordSet() {
		return stopWordSet;
	}
 
	public void setStopWordSet(CharArraySet stopWordSet) {
		this.stopWordSet = stopWordSet;
	}
 
	
	public MyAnalyzer() {
		super();
		this.stopWordSet = StopAnalyzer.ENGLISH_STOP_WORDS_SET;//可在此基础上拓展停止词
	}
	
	/**扩展停止词
	 * @param stops
	 */
	public MyAnalyzer(String[] stops) {
		this();
		stopWordSet.addAll(StopFilter.makeStopSet(stops));
	}
 
	@Override
	protected TokenStreamComponents createComponents(String fieldName) {
		//正则匹配分词
		Tokenizer source = new LowerCaseTokenizer();
	    return new TokenStreamComponents(source, new StopFilter(source, stopWordSet));
	}
	public static void main(String[] args) {
		Analyzer analyzer = new MyAnalyzer();
		String words = "A AN yuyu";
		TokenStream stream = null;
		
		try {
			stream = analyzer.tokenStream("myfield", words);
			stream.reset(); 
			CharTermAttribute  offsetAtt = stream.addAttribute(CharTermAttribute.class);
			while (stream.incrementToken()) {
				System.out.println(offsetAtt.toString());
			}
			stream.end();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}finally{
			try {
				stream.close();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
}
```

运行结果如下：

```
yuyu
```

说明该分词器对a an 进行了过滤，这些过滤的词在stopWordSet中

###  添加字长过滤器

有时候我们需要对字符串中的短字符进行过滤，比如welcome to BeiJIng中过滤掉长度小于2的字符串，我们期望的结果就变成了Welcome BeiJing,我们仅需要重新实现createComponents方法，相关代码如下:

```java
package com.lucene.analysis.self;
 
import java.io.IOException;
 
import org.apache.lucene.analysis.Analyzer;
import org.apache.lucene.analysis.TokenStream;
import org.apache.lucene.analysis.Tokenizer;
import org.apache.lucene.analysis.core.LowerCaseTokenizer;
import org.apache.lucene.analysis.core.StopAnalyzer;
import org.apache.lucene.analysis.core.StopFilter;
import org.apache.lucene.analysis.core.WhitespaceTokenizer;
import org.apache.lucene.analysis.miscellaneous.LengthFilter;
import org.apache.lucene.analysis.tokenattributes.CharTermAttribute;
import org.apache.lucene.analysis.util.CharArraySet;
 
public class LengFilterAanlyzer extends Analyzer {
	private int len;
	
	public int getLen() {
		return len;
	}
 
 
	public void setLen(int len) {
		this.len = len;
	}
 
 
	public LengFilterAanlyzer() {
		super();
	}
	
 
	public LengFilterAanlyzer(int len) {
		super();
		this.len = len;
	}
 
 
	@Override
	protected TokenStreamComponents createComponents(String fieldName) {
		final Tokenizer source = new WhitespaceTokenizer();
	    TokenStream result = new LengthFilter(source, len, Integer.MAX_VALUE);
	    return new TokenStreamComponents(source,result);
 
	}
	public static void main(String[] args) {
		Analyzer analyzer = new LengFilterAanlyzer(2);
		String words = "I am a java coder";
		TokenStream stream = null;
		
		try {
			stream = analyzer.tokenStream("myfield", words);
			stream.reset(); 
			CharTermAttribute  offsetAtt = stream.addAttribute(CharTermAttribute.class);
			while (stream.incrementToken()) {
				System.out.println(offsetAtt.toString());
			}
			stream.end();
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}finally{
			try {
				stream.close();
			} catch (IOException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}
}
```

程序的执行结果如下：

```
am
java
coder
```

说明小于2个字符的文本被过滤了。

