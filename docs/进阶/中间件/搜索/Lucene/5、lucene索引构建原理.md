---
tags:
	- lucene
categories: lucene
title: lucene的索引构建原理
---

# lucene（5）---lucene的索引构建原理

## lucene创建索引的原理

### IndexWriter的addDocument方法详解

今天看了IndexWriter类的addDocument方法，IndexWriter对此方法的说明如下：

<!--more-->

```
Adds a document to this index. 
Note that if an Exception is hit (for example disk full) then the index will be consistent, but this document may not have been added. Furthermore, it's possible the index will have one segment in non-compound format even when using compound files (when a merge has partially succeeded).
This method periodically flushes pending documents to the Directory (see above), and also periodically triggers segment merges in the index according to the MergePolicy in use.
Merges temporarily consume space in the directory. The amount of space required is up to 1X the size of all segments being merged, when no readers/searchers are open against the index, and up to 2X the size of all segments being merged when readers/searchers are open against the index (see forceMerge(int) for details). The sequence of primitive merge operations performed is governed by the merge policy. 
Note that each term in the document can be no longer than MAX_TERM_LENGTH in bytes, otherwise an IllegalArgumentException will be thrown.
```

大意如下：

此方法向索引中添加一个document；

需要注意的是如果执行过程中发生异常（比如磁盘空间不足）的时候索引会保持一致性，但是这个document也许并没有被添加，此外，即使使用符合文件也有可能索引包含一个非复合格式的segment（当合并索引有部分成功的时候）

此方法会定期的flush索引文件目录，并且会根据合并策略定期去触发索引文件中segment的合并操作；

刚方法会对合并临时的索引空间，当没有reader或者searcher读取或写入索引文件的时候所需要占用的磁盘空间至少要超过需要合并的segments文件的一倍，反之将会占用两倍以上的空间；序列的合并操作的优化取决于合并策略‘

要确保document中的每一个term占用的字节长度都不能超过MAX_TERM_LENGTH，否则会抛出IllegalArgumentException异常；

其实际的执行方法为：

![](http://blogimg.nos-eastchina1.126.net/shenwf20190316100523-767792.jpg)

继续跟进updateDocument方法，其实现如下

![](http://blogimg.nos-eastchina1.126.net/shenwf20190316100618-658422.jpg)

可以看见updateDocument是先从索引中删除包含相同term的document然后重新添加document到索引中；

此操作需要确保IndexWriter没有被关闭，其实现是先有[DocumentsWriter](https://blog.csdn.net/wuyinggui10000/article/details/45625351)类的updateDocument方法判断，这里先判断将根据term找到对应的document，并先放到待删除的document队列中，然后从队列中读取document，再将要flush的documents写入磁盘，同时更新flush队列中的索引状态；

相关源码如下

![](http://blogimg.nos-eastchina1.126.net/shenwf20190316101101-308029.jpg)

在此期间有一个ThreadState类型的[读写锁](https://www.baidu.com/s?wd=%E8%AF%BB%E5%86%99%E9%94%81&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)，lucene判断ThreadState的状态，如果此锁被激活，从内存中获取document并更新到索引文件且重置内存中索引的数量和状态，最后释放相关的资源。

此即为IndexWriter的索引构建过程，看代码[晕头转向](https://www.baidu.com/s?wd=%E6%99%95%E5%A4%B4%E8%BD%AC%E5%90%91&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)的，以后为大家带来一点干货，明天带来lucene索引优化之多线程创建索引。