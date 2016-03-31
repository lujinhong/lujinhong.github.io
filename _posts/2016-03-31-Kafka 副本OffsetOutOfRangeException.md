---
layout: post
tile:  "Kafka 副本OffsetOutOfRangeException"
date:  2016-03-31 12:07:31
categories: kafka 
excerpt: Kafka 副本OffsetOutOfRangeException
---

* content
{:toc}



[TOC]

https://issues.apache.org/jira/browse/KAFKA-2477
影响版本0.8&之前，修复版本0.9.0.0
##1、故障描述
近期，由于kafka集群负载增大，server.log中经常出现下面的错误日志。这是kafka自身的一个bug。

简单说就是由于副本去leader请求同步数据时，发现请求的offset超出了leader的offset范围（原因见下面代码解释），从而认为副本出错了，于是删除副本数据，从leader重新同步一份数据过来。由于每个分区的数据较大（约60G），同步时间较长，在此期间，leader及replication均处于高磁盘、网络IO的状态，导致storm读取数据时超时无响应。

对于opaque拓扑，当发现某个分区不可用时，会读取其它分区。而transactional拓扑必须等这个分区恢复。因此最后的结果是SA的拓扑恢复了，而US/SDC的拓扑挂掉。

	[2016-03-29 18:24:59,403] WARN [ReplicaFetcherThread-3-4], Replica 2 for partition [g17,4] reset its fetch offset from 3501121050 to current leader 4's start offset 3501121050 (kafka.server.ReplicaFetcherThread)
	[2016-03-29 18:24:59,403] ERROR [ReplicaFetcherThread-3-4], Current offset 3781428103 for partition [g17,4] out of range; reset offset to 3501121050 (kafka.server.ReplicaFetcherThread)

##2、故障详细原因：线程同步锁使用了写锁，未使用读锁

（1）某个Wrtier(W1)开始写数据，它对日志只有写锁，未锁定读。**若线程W1将日志已经append到log中，但未更新nextOffset，此时被其它线程取得运行权**。假设此时offset为100，nextOffset为101.
（2）此时副本的一个Reader(R1)过来读数据，它之前已经读到100了，所以请求nextOffset为101, leader发现有offset为101的数据，所以正确返回数据。
（3）然后副本的下一个Reader(R2)来继续读数据，它已经读取到101了，所以请求102，但在leader中，由于nextOffset未更新，它认为102已经超出它当前的100的offset了，所以出现OffsetOutOfRange异常。**kafka认为副本已经损坏，删除副本数据，从leader重传**
（4）W1线程继续执行，更新nextOffset到102，但已经太迟了，异常已经出现。
相关代码：

    val next = nextOffsetMetadata.messageOffset
     if(startOffset == next)
     return FetchDataInfo(nextOffsetMetadata, MessageSet.Empty)
    
     var entry = segments.floorEntry(startOffset)
    
     // attempt to read beyond the log end offset is an error
     // attempt to read beyond the log end offset is an error
     if(startOffset > next || entry == null)
     if(startOffset > next || entry == null)
       throw new OffsetOutOfRangeException("Request for offset %d but we only have log segments in the range %d to %d.".format(startOffset, segments.firstKey, next))

##3、解决建议

###方法一：升级至0.9.0.1
目前版本0.8.2，这个bug在0.9.0.0修复。这是一劳永逸的办法，也是最终的解决的办法。
问题：storm-kafka0.9还处在开发阶段，暂时beta版不建议使用。
结论：最终解决方法，但暂时不可用。

###方法二：修改kafka代码，自己编译一个版本
问题：kafka使用gradle编译的scala代码，重新编译有风险

###方法三：新集群上线，降低问题出现概率
加快新集群上线的速度，当负载降低时，问题出现概率会相应降低。
问题：治标不治本。

###方法四：检查拓扑报警时，出现若是类似异常，直接重启
通过storm REST API获取拓扑异常信息，符合一定条件时，重启拓扑。
暂时可行的方法。

###方法五：拓扑均使用opaque
问题：拓扑实现opaque的MapState会较为复杂。

###建议：
（1）先使用方法四暂时度过。
（2）我们先尝试编译并维护自己的一个版本，同时加快新集群上线。哪个先达成就使用哪个。
（3）最终使用方法一解决问题。
