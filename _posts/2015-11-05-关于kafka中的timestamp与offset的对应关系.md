---
layout: post
tile:  "关于kafka中的timestamp与offset的对应关系"
date:  2015-11-05 12:58:30
categories: storm kafka 大数据 
excerpt: 关于kafka中的timestamp与offset的对应关系
---

* content
{:toc}




##获取单个分区的情况
kafka通过offset记录每条日志的偏移量，详见《Kafka文件存储机制那些事》。但是当用户想读取之前的信息时，他是不可能知道这些消息对应的offset的，用户只能指定时间，比如说我从昨天的12点开始读取消息。

这就有个问题了，怎么样将用户定义的时间转化为集群内部的offset呢？


先简单重温一下kafka的物理存储机制：每个topic分成多个分区，而一个分区对应磁盘中的一个目录，目录中会有多个文件，比如：

     00000000000000000000.index  00000000000000000000.log  00000000000001145974.index  00000000000001145974.log
     
可以看出来，每个segment file其实有2部分，一个index文件，一个log文件。文件名是这个文件内的第一个消息的offset。log文件记录的是实际的消息内容。而index对log文件作了索引，当需要查看某个消息时，如果指定offset，很容易就定位到log文件中的具体位置。详见上面说的文章。

但正如刚才所说，用户不知道offset，而只知道时间，所以就需要转换了。

kafka用了一个很直观很简单的方法：将文件名中的offset与文件的最后修改时间放入一个map中，然后再查找。详细步骤如下：
（1）将文件名及文件的最后时间放入一个map中，时间使用的是13位的unix时间戳
（2）当用户指定一个时间t0时，在map中找到最后一个时间t1早于t0的时间，然后返回这个文件名，即这个文件的第一个offset。
（3）这里只返回了一个分区的offset，而事实上需要返回所有分区的offset，所以对所有分区采取上述步骤。
（4）使用取到的消息，开始消费消息。

举个例子：
		
	w-r--r-- 1 hadoop hadoop 1073181076  8?? 11 10:20 00000000000066427499.log
	-rw-r--r-- 1 hadoop hadoop      14832  8?? 11 10:20 00000000000066427499.index
	-rw-r--r-- 1 hadoop hadoop 1073187364  8?? 11 10:40 00000000000067642947.log
	-rw-r--r-- 1 hadoop hadoop      14872  8?? 11 10:40 00000000000067642947.index
	-rw-r--r-- 1 hadoop hadoop 1073486959  8?? 11 11:04 00000000000068857698.log
	-rw-r--r-- 1 hadoop hadoop      14928  8?? 11 11:04 00000000000068857698.index
	-rw-r--r-- 1 hadoop hadoop 1073511817  8?? 11 11:25 00000000000070069880.log
	-rw-r--r-- 1 hadoop hadoop      14920  8?? 11 11:25 00000000000070069880.index
	-rw-r--r-- 1 hadoop hadoop   10485760  8?? 11 11:28 00000000000071279203.index
	-rw-r--r-- 1 hadoop hadoop  148277228  8?? 11 11:28 00000000000071279203.log


我们有上述几个文件
（1）当我需要消费从8月11日11：00开始的数据时，它会返回最后修改时间早于8月11日11：00的文件名，此外是修改时间第10：40的文件，offset为67642947.其实由于它的最后修改时间在10：40，我们需要的数据不可能在它里面，它直接返回11：40的文件即可，但可能是出于更保险的考虑，它返回了上一个文件。
（2）其它类似，当我消费11：20的数据，返回的offset为68857698.
（3）而当我消费的数据早于10：20的话，则返回的offset为空，如果是通过数组保存offset的，则提取第一个offset时会出现 java.lang.ArrayIndexOutOfBoundsException 异常。如在kafka编程指南中的SimpleConsumer中的代码：

        long[] offsets = response.offsets(topic, partition);
        return offsets[0];
当然，也可以合理处理，当返回为空时，直接返回最早的offset即可。

（4）当消费的数据晚于最晚时刻，返回最新的消息。


注意：
（1）这里对kafka集群本身没有任何的负担，kafka消息也不需要记录时间点这个字段，只有在需要定位的时候，才临时构建一个map，然后将offset与时间读入这个map中。
（2）冗余很多消息。这种方法粒度非常粗，是以文件作为粒度的，因此冗余的消息数据和文件的大小有关系，默认为1G。如果这个topic的数据非常少，则这1G的数据可以就是几天前的数据了。
（3）有2个特殊的时间点：
  需要查找的 timestamp 是 -1 或者 -2时，特殊处理
		
	  case OffsetRequest.LatestTime =>  // OffsetRequest.LatestTime = -1	
	    startIndex = offsetTimeArray.length -1	
	  case OffsetRequest.EarliestTime => // OffsetRequest.EarliestTime = -2
	    startIndex =0

##同时从所有分区获取消息的情况

1、当同时从多个分区读取消息时，只要有其中一个分区，它的所有文件的修改时间均晚于你指定的时间，就会出错，因为这个分区返回的offset为空，除非你作了合理的处理。

2、storm!!!
storm0.9x版本遇到上述问题时，同样会出错，出现以下异常

	storm.kafka.UpdateOffsetException
而从0.10版本开始，改为了从最早时间开始消费消息。

3、还有个问题，如何将消息均匀的分布但各个分区中。比如在我们一个topic中，其中一个分区已经有60G数据，而另一个分区还不足2G，如果指定时间的话，由于小的那个分区的修改时间肯定是在近期的，所以当指定一个较前的时间点就会出错。而且即使不出错，从不同分区返回的消息也可能时间相差很远。

如何将数据均匀的分布到各个分区，请参考kafka编程指南的partitioner介绍。

只要出现这个问题，都是由于数据不存在，有可能是：

（1）数据真的丢失了

（2）数据倾斜严重

##结论

###如何指定时间
如果需要指定从某个时间点开始处理日志，则：

（1）就指定那个时间即可，不需要提前，因此返回的消息一定是在这个时间点之前的。

（2）如果这个时间点是繁忙时段，它返回的消息时间可能只是这个时间点之前的一小段时间。

（3）如果这个时间点是个空闲时间，它返回的日志时间可能是很长一段时间的日志。

（4）但不管是繁忙时间还是空闲时间，它都是多读一个日志文件，所以冗余的日志数量是相同的。

举个例子：

如果需要处理2015-08-15 15：00：00后的日志，则

（1）直接指定这个时间即可，不需要指定它之前的时间，如1：00， 2：00之类的，因为返回的日志时间决不会是3：00前的。同时，3：00进入kafka的数据有可能是3：00前的数据，决不会是3：00后的数据，所以也不需要考虑指定提前时间。

（2）由于这个时间日志一般较多，它返回的日志可能是2：30左右开始的。相反，如果是凌晨3：00，由于这个时间点日志较少，它返回的日志有可以是2、3小时前的。

###出现UpdateOffsetException时的处理方法
（1）若是小项目，由于数据量不多，建议同头开始处理，并通知SA检查。

（2）若是大项目，一般是出现了数据倾斜，通知SA检查数据情况。





##相关源码略读



###1、入口
Kafka Server 处理 Client 发送来的请求的入口在
文件夹:  core/src/main/scala/kafka/server
类：`kafka.server.KafkaApis`
方法: handle
处理offset请求的函数: 	`handleOffsetRequest`
###2、处理逻辑

处理逻辑主要分为四步
获取partition
从partition中获取offset
high water mark 处理(这一段的资料太少了)
异常处理
由于request中包含查询多个partition的offset的请求。所以最终会返回一个map，保存有每个partition对应的offset
这里主要介绍从某一个partition中获取offset的逻辑，代码位置
`kafka.log.Log#getOffsetsBefore(timestamp, maxNumOffsets)`
从一个partition中获取offset

####（1）建立offset与timestamp的对应关系，并保存到数据中
		
	//每个Partition由多个segment file组成。获取当前partition中的segment列表
	val segsArray = segments.view
	 
	// 创建数组
	var offsetTimeArray: Array[(Long, Long)] =null
	if(segsArray.last.size >0)
	  offsetTimeArray =newArray[(Long, Long)](segsArray.length +1)
	else
	  offsetTimeArray =newArray[(Long, Long)](segsArray.length)
	 
	// 将 offset 与 timestamp 的对应关系添加到数组中
	for(i <-0until segsArray.length)
	  // 数据中的每个元素是一个二元组，(segment file 的起始 offset，segment file的最近修改时间)
	  offsetTimeArray(i) = (segsArray(i).start, segsArray(i).messageSet.file.lastModified)
	if(segsArray.last.size >0)
	  // 如果最近一个 segment file 不为空，将(最近的 offset, 当前之间)也添加到该数组中
	  offsetTimeArray(segsArray.length) = (logEndOffset, time.milliseconds)
	通过这段逻辑，获的一个数据 offsetTimeArray，每个元素是一个二元组，二元组内容是(offset, timestamp)
 
####（2）找到最近的最后一个满足 timestamp < target_timestamp 的 index
		
	var startIndex = -1
	timestamp match {
	  // 需要查找的 timestamp 是 -1 或者 -2时，特殊处理
	  caseOffsetRequest.LatestTime =>  // OffsetRequest.LatestTime = -1
	    startIndex = offsetTimeArray.length -1
	  caseOffsetRequest.EarliestTime => // OffsetRequest.EarliestTime = -2
	    startIndex =0
	  case_ =>
	    var isFound =false
	    debug("Offset time array = "+ offsetTimeArray.foreach(o =>"%d, %d".format(o._1, o._2)))
	    startIndex = offsetTimeArray.length -1  // 从最后一个元素反向找
	    while(startIndex >=0&& !isFound) {    // 找到满足条件或者
	      if(offsetTimeArray(startIndex)._2 <= timestamp)  // offsetTimeArray 的每个元素是二元组，第二个位置是 timestamp
	        isFound =true
	      else
	        startIndex -=1
	    }
	}
通过这段逻辑，实际找到的是 “最近修改时间早于目标timestamp的最近修改的segment file的起始offset”
但是获取offset的逻辑并没有结束，后续仍有处理

####（3）找到满足该条件的offset数组

实际上这个函数的功能是找到一组offset，而不是一个offset。第二个参数 maxNumOffsets 指定最多找几个满足条件的 offset。
		
	获取一组offset的逻辑
	// 返回的数据的长度 = min(maxNumOffsets, startIndex + 1)，startIndex是逻辑2中找到的index
	val retSize = maxNumOffsets.min(startIndex +1)
	val ret = newArray[Long](retSize)
	 
	// 逐个将满足条件的offset添加到返回的数据中
	for(j <-0until retSize) {
	  ret(j) = offsetTimeArray(startIndex)._1
	  startIndex -=1
	}
	 
	// 降序排序返回。offset 越大数据越新。
	// ensure that the returned seq is in descending order of offsets
	ret.toSeq.sortBy(- _)
最终返回这个数组

###3、注意事项

实际找到的offset并不是从目标timestamp开始的第一个offset。需要注意
当 timestamp 小于最老的数据文件的最近修改时间时，返回值是一个空数组。可能会导致使用时的问题。
调整segment file文件拆分策略的配置时，需要注意可能会造成的影响。
