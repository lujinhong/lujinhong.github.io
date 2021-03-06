---
layout: post
tile:  "SPARK STREAMING之2：编程指南"
date:  2015-10-09 15:04:33
excerpt: SPARK STREAMING之2：编程指南
---

* content
{:toc}

 
 @(博客文章)[spark|大数据]


##（一）官方入门示例

废话不说，先来个示例，有个感性认识再介绍。

这个示例来自spark自带的example，基本步骤如下：

（1）使用以下命令输入流消息：

	$ nc -lk 9999

（2）在一个新的终端中运行NetworkWordCount，统计上面的词语数量并输出：

	$ bin/run-example streaming.NetworkWordCount localhost 9999

（3）在第一步创建的输入流程中敲入一些内容，在第二步创建的终端中会看到统计结果，如：

第一个终端输入的内容：

	hello world again

第二个端口的输出
		
	-------------------------------------------
	Time: 1436758706000 ms
	-------------------------------------------
	(again,1)
	(hello,1)
	(world,1)

简单解释一下，上面的示例通过手工敲入内容，并传给spark streaming统计单词数量，然后将结果打印出来。

附上代码：
		
	package org.apache.spark.examples.streaming
	
	import org.apache.spark.SparkConf
	import org.apache.spark.streaming.{Seconds, StreamingContext}
	import org.apache.spark.storage.StorageLevel
	
	/**
	 * Counts words in UTF8 encoded, '\n' delimited text received from the network every second.
	 *
	 * Usage: NetworkWordCount
	 *  and  describe the TCP server that Spark Streaming would connect to receive data.
	 *
	 * To run this on your local machine, you need to first run a Netcat server
	 *    `$ nc -lk 9999`
	 * and then run the example
	 *    `$ bin/run-example org.apache.spark.examples.streaming.NetworkWordCount localhost 9999`
	 */
	object NetworkWordCount {
	  def main(args: Array[String]) {
	    if (args.length < 2) {
	      System.err.println("Usage: NetworkWordCount  ")
	      System.exit(1)
	    }
	
	    StreamingExamples.setStreamingLogLevels()
	
	    // Create the context with a 1 second batch size
	    val sparkConf = new SparkConf().setAppName("NetworkWordCount")
	    val ssc = new StreamingContext(sparkConf, Seconds(1))
	
	    // Create a socket stream on target ip:port and count the
	    // words in input stream of \n delimited text (eg. generated by 'nc')
	    // Note that no duplication in storage level only for running locally.
	    // Replication necessary in distributed scenario for fault tolerance.
	    val lines = ssc.socketTextStream(args(0), args(1).toInt, StorageLevel.MEMORY_AND_DISK_SER)
	    val words = lines.flatMap(_.split(" "))
	    val wordCounts = words.map(x => (x, 1)).reduceByKey(_ + _)
	    wordCounts.print()
	    ssc.start()
	    ssc.awaitTermination()
	  }
	}
	 

 

##（二）Spark Streaming kafka示例

本示例使用java+maven来构建一个wordcount

1、创建项目，在pom.xml添加如下的依赖关系
		
	<dependency>
	<groupId>org.slf4j</groupId>
	<artifactId>slf4j-api</artifactId>
	<version>1.7.0</version>
	</dependency>
	<dependency>
	<groupId>org.slf4j</groupId>
	<artifactId>slf4j-log4j12</artifactId>
	<version>1.7.0</version>
	</dependency>
	<dependency>
	<groupId>log4j</groupId>
	<artifactId>log4j</artifactId>
	<version>1.2.17</version>
	</dependency>
	<dependency>
	<groupId>org.apache.spark</groupId>
	<artifactId>spark-core_2.10</artifactId>
	<version>1.4.0</version>
	</dependency>
	<dependency>
	<groupId>org.apache.spark</groupId>
	<artifactId>spark-streaming_2.10</artifactId>
	<version>1.4.0</version>
	</dependency>
	<dependency>
	<groupId>org.apache.spark</groupId>
	<artifactId>spark-streaming-kafka_2.10</artifactId>
	<version>1.4.0</version>
	</dependency>
	 
	<dependency>
	<groupId>org.apache.kafka</groupId>
	<artifactId>kafka_2.10</artifactId>
	<version>0.8.2.1</version>
	</dependency>
 

2、写代码，此部分代码使用了官方的代码：
		
	package com.ljh.demo.kafkaStreaming;
	
	import java.util.Map;
	import java.util.HashMap;
	import java.util.regex.Pattern;
	
	import scala.Tuple2;
	import com.google.common.collect.Lists;
	import org.apache.spark.SparkConf;
	import org.apache.spark.api.java.function.FlatMapFunction;
	import org.apache.spark.api.java.function.Function;
	import org.apache.spark.api.java.function.Function2;
	import org.apache.spark.api.java.function.PairFunction;
	import org.apache.spark.streaming.Duration;
	import org.apache.spark.streaming.api.java.JavaDStream;
	import org.apache.spark.streaming.api.java.JavaPairDStream;
	import org.apache.spark.streaming.api.java.JavaPairReceiverInputDStream;
	import org.apache.spark.streaming.api.java.JavaStreamingContext;
	import org.apache.spark.streaming.kafka.KafkaUtils;
	
	/**
	 * Consumes messages from one or more topics in Kafka and does wordcount.
	 *
	 * Usage: JavaKafkaWordCount
	 * is a list of one or more zookeeper servers that make quorum
	 * is the name of kafka consumer group
	 * is a list of one or more kafka topics to consume from
	 *is the number of threads the kafka consumer should use
	 *
	 * To run this example:
	 *   `$ bin/run-example org.apache.spark.examples.streaming.JavaKafkaWordCount zoo01,zoo02, \
	 *    zoo03 my-consumer-group topic1,topic2 1`
	 */
	
	public final class JavaKafkaWordCount {
	  private static final Pattern SPACE = Pattern.compile(" ");
	
	  private JavaKafkaWordCount() {
	  }
	
	  public static void main(String[] args) {
	    if (args.length < 4) {
	      System.err.println("Usage: JavaKafkaWordCount
	");
	      System.exit(1);
	    }
	
	    SparkConf sparkConf = new SparkConf().setAppName("JavaKafkaWordCount");
	    // Create the context with a 1 second batch size
	    JavaStreamingContext jssc = new JavaStreamingContext(sparkConf, new Duration(2000));
	
	    int numThreads = Integer.parseInt(args[3]);
	    Map topicMap = new HashMap();
	    String[] topics = args[2].split(",");
	    for (String topic: topics) {
	      topicMap.put(topic, numThreads);
	    }
	
	    JavaPairReceiverInputDStream messages =
	            KafkaUtils.createStream(jssc, args[0], args[1], topicMap);
	
	    JavaDStream lines = messages.map(new Function() {
	      @Override
	      public String call(Tuple2 tuple2) {
	        return tuple2._2();
	      }
	    });
	
	    JavaDStream words = lines.flatMap(new FlatMapFunction() {
	      @Override
	      public Iterable call(String x) {
	        return Lists.newArrayList(SPACE.split(x));
	      }
	    });
	
	    JavaPairDStream wordCounts = words.mapToPair(
	      new PairFunction() {
	        @Override
	        public Tuple2 call(String s) {
	          return new Tuple2(s, 1);
	        }
	      }).reduceByKey(new Function2() {
	        @Override
	        public Integer call(Integer i1, Integer i2) {
	          return i1 + i2;
	        }
	      });
	
	    wordCounts.print();
	    jssc.start();
	    jssc.awaitTermination();
	  }
	}
	 

3、上传到服务器中然后编译

	mvn clean package
4、提交job到spark中
		
	/home/hadoop/spark/bin/spark-submit --jars ../mylib/metrics-core-2.2.0.jar,../mylib/zkclient-0.3.jar,../mylib/spark-streaming-kafka_2.10-1.4.0.jar,../mylib/kafka-clients-0.8.2.1.jar,../mylib/kafka_2.10-0.8.2.1.jar  --class com.lujinhong.demo.kafkaStreaming.JavaKafkaWordCount --master spark://192.168.16.102:7077  target/kafkaStreaming-0.0.1-SNAPSHOT.jar 192.168.172.111:2181/kafka my-consumer-group test 3
 

当然，前提是kafka集群已经正常运行，且存在test这个topic

 

5、验证

打开一个console producer，输入内容，然后观察wordcount的结果。

结果形式如下：

(hi,1)

　　

##（三）基本步骤

本部分介绍创建一个spark streaming应用的基本步骤

1、构建依赖关系，以maven为例，需要在pom.xml中添加以下内容
		
	<dependency>
	<groupId>org.apache.spark</groupId>
	<artifactId>spark-streaming_2.10</artifactId>
	<version>1.4.0</version>
	</dependency>

 如果需要使用其它数据源，则还需要将相应的依赖关系放入pom.xml。

如使用kafka作为数据源：
		
	<dependency>
	<groupId>org.apache.spark</groupId>
	<artifactId>spark-streaming_2.10</artifactId>
	<version>1.4.0</version>
	</dependency>
	<dependency>
	<groupId>org.apache.spark</groupId>
	<artifactId>spark-streaming-kafka_2.10</artifactId>
	<version>1.4.0</version>
	</dependency>
 
当然，spark的核心包也要包含：
			
	<dependency>
	<groupId>org.apache.spark</groupId>
	<artifactId>spark-core_2.10</artifactId>
	<version>1.4.0</version>
	</dependency>	
