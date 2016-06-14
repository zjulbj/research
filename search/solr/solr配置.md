#solr配置
##solrconfig.xml
`solrconfig.xml`里面有很多参数配置,主要有：

- 1 request handlers
- 2 触发某些动作的listeners
- 3 管理HTTP通信的Request Dispatcher
- 4 admin页面接口
- 5 replication和duplication相关参数

Solr配置支持变量替换，如`${propertyname[:option default value]}`。
可以指定默认值和运行时替换配置。
如果没配默认值，运行时必须指定，不然报错。
指定方式：

- 1 JVM 系统属性
```
##conf/solrconfig.xml
<lockType>${solr.lock.type:native}</lockType>

##JVM
java -Dsolr.lock.type=simple -jar start.jar
```

运行时JVM参数`simple`会覆盖掉默认值`native`

- 2 solrcore.properties
```
#conf/solrcore.properties
lock.type=simple
```
- 3 core.properties
```
#core.properties
name=collection2
my.custom.prop=edismax

##conf/solrconfig.xml
<requestHandler name="/select">
<lst name="defaults">
<str name="defType">${my.custom.prop}</str>
</lst>
</requestHandler>
```
##IndexConfig in SolrConfig
`solrconfig.xml`中的`<indexConfig>`部分定义了lucence的底层行为。

- ramBufferSizeMB
最大缓冲内存，如果文档更新达到这个数(MB),更新会被刷入磁盘，会创建新的segments，触发merge

- maxBufferedDocs
最大缓冲文档数，作用和`ramBufferSizeMB`,只是从文档数的维度衡量。
两者仍一条件达到就会触发flush。

- maxIndexingThreads
最大并发线程数，当索引文档的线程达到这个阈值，后续线程会等待。

- mergeFactor
