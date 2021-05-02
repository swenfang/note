---
tags:
	- lucene
categories: lucene
title: lucene的中文分词器jcseg和IKAnalyzer分词器及其使用说明
---
# lucene（4）---lucene的中文分词器jcseg和IK Analyzer分词器及其使用说明

## 为什么要使用lucene中文分词器

在lucene的开发过程中，我们常会遇到分词时中文识别的问题，lucene提供了

lucene-analyzers-common-5.0.0.jar包来支持分词，但多的是对英国，法国，意大利等过语言的支持，

因此我们需要引入中文分词的概念。

<!--more-->

## 各种中文分词器及其对比

### jcseg中文分词器

jcseg是使用Java开发的一款开源的中文分词器, 使用mmseg算法. 分词准确率高达
98.4%, 支持中文人名识别, 同义词匹配, 停止词过滤...

jcseg支持三种切分模式：
(1).简易模式：FMM算法，适合速度要求场合。
(2).复杂模式-MMSEG四种过滤算法，具有较高的岐义去除，分词准确率达到了98.41%。
(3).检测模式：只返回词库中已有的词条，很适合某些应用场合。(1.9.4开始)

就分词效率而言，简易模式速度最快

jcseg词库配置丰富，自我感觉功能最强大，详见jcseg开发文档；

jcseg现版本不兼容lucene5，我修改了其analyzer包，相关示例代码如下

```java
package com.lucene.analyzer;
 
import java.io.IOException;
 
import org.apache.lucene.analysis.Analyzer;
import org.apache.lucene.analysis.TokenStream;
import org.apache.lucene.analysis.tokenattributes.CharTermAttribute;
import org.junit.Test;
import org.lionsoul.jcseg.analyzer.JcsegAnalyzer5X;
import org.lionsoul.jcseg.core.JcsegTaskConfig;
 
 
public class JcsegAnalyzerTest {
 
	@Test
	public void tokenTest() {
		Analyzer analyzer = new JcsegAnalyzer5X(JcsegTaskConfig.SIMPLE_MODE);
		//非必须(用于修改默认配置): 获取分词任务配置实例
		JcsegAnalyzer5X jcseg = (JcsegAnalyzer5X) analyzer;
		JcsegTaskConfig config = jcseg.getTaskConfig();
		//追加同义词到分词结果中, 需要在jcseg.properties中配置jcseg.loadsyn=1
		config.setAppendCJKSyn(true);
		//追加拼音到分词结果中, 需要在jcseg.properties中配置jcseg.loadpinyin=1
		config.setAppendCJKPinyin(true);
		//更多配置, 请查看com.webssky.jcseg.core.JcsegTaskConfig类
	String words = "中华人民共和国";
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
			if(stream != null)
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
中华
人民共和国
```

### IKAnalyzer

IK Analyzer是一个开源的，基亍java语言开发的轻量级的中文分词工具包。

采用了特有的“正向迭代最细粒度切分算法“，支持细粒度和智能分词两种切分模式；
在系统环境：Core2 i7 3.4G双核，4G内存，window 7 64位， Sun JDK 1.6_29 64位 普通pc环境测试，IK2012具有160万字/秒（3000KB/S）的高速处理能力。
2012版本的智能分词模式支持简单的分词排歧义处理和数量词合并输出。
采用了多子处理器分析模式，支持：英文字母、数字、中文词汇等分词处理，兼容韩文、日文字符
优化的词典存储，更小的内存占用。支持用户词典扩展定义。特别的，在2012版本，词典支持中文，英文，数字混合词语。

IK Analyzer支持细粒度切分和智能切分两种分词模式;

在细粒度切分下，词语分解到很细的力度，比如“一个苹果”，会被切分成如下

```
一个
一
个
苹果
```

在智能切分模式下，则会分词如下：

```
一个
苹果
```

和jcseg相同，现版本的IK Analyzer只兼容至lucene4版本，我修改了相关源码，使其提供了对lucene5的支持。

IK Analyzer示例代码如下：

```java
package com.lucene.analyzer;
import java.io.IOException;
import org.apache.lucene.analysis.Analyzer;
import org.apache.lucene.analysis.TokenStream;
import org.apache.lucene.analysis.tokenattributes.CharTermAttribute;
import org.junit.Test;
import org.wltea.analyzer.lucene.IKAnalyzer;
 
public class IKAnalyzerTest {
 
	@Test
	public void tokenTest() {
	Analyzer analyzer = new IKAnalyzer();
	String words = "中华人民共和国";
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

其运行结果如下：

```
中华人民共和国
中华人民
中华
华人
人民共和国
人民
共和国
共和
国
```
