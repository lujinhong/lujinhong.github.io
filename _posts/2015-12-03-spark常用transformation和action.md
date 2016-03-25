---
layout: post
tile:  "spark常用transformation和action"
date:  2015-12-03 19:18:01
categories: spark 
excerpt: spark常用transformation和action
---

* content
{:toc}



[TOC]

Spark提供了各种各样的Transformation和Action，用于用户的各种操作。本文介绍了spark的所有transformation及action的用法，并给出示例。
官方的列表请见：
https://spark.apache.org/docs/latest/programming-guide.html


本文分为2部分，第一部分是对所有RDD均有效的Transformation和Action，第二部分是仅对pairRDD有效的Transformation和Action，即对键值对RDD。

#一、基本RDD的Transformation

##（一)map
###1、官方解释
 Return a new distributed dataset formed by passing each element of the source through a function func. 
map的参数是一个用户定义的函数，将RDD的每一个元素传入一个函数，返回的值组成一个新的RDD。
注意，map对于每一个输入元素，只会返回一个输出元素。而flatMap则对于每一个输入元素，可能返回0个或者多个元素。
 
###2、示例
 （1）返回一个RDD中每个元素的平方
	 
	scala> val nums = sc.parallelize(List(1,2,3,4))
	scala> val result = nums.map(x => x*x)
	scala> result.collect
	res10: Array[Int] = Array(1, 4, 9, 16)

##（二）flatMap

###1、官方解释
 Similar to map, but each input item can be mapped to 0 or more output items (so func should return a Seq rather than a single item). 
flatMap的参数也是一个函数，正如上面据说的，flatMap对于每一个输入元素会返回0个或者多个输出元素，因此，作为flatMap参数的函数返回的是一个Seq。

###2、示例
（1）将语句按照空格进行切分，返回一个保存所有单词的RDD
	
	scala> val file = sc.parallelize(List("hello world", "hi spark", "I am lujinhong"))
	file: org.apache.spark.rdd.RDD[String] = ParallelCollectionRDD[16] at parallelize at <console>:15
	
	scala> val word = file.flatMap(words => words.split(" "))
	word: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[17] at flatMap at <console>:17
	
	scala> word.collect
	res11: Array[String] = Array(hello, world, hi, spark, I, am, lujinhong)

（2）另一个例子：wordcoutn
	
	scala> val file = sc.parallelize(List("hello world", "hello spark", "Hi I am lujinhong"))
	file: org.apache.spark.rdd.RDD[String] = ParallelCollectionRDD[19] at parallelize at <console>:21
	
	scala> file.flatMap(words => words.split(" ")).map(word => (word,1)).reduceByKey((x,y) => x+y).collect
	res11: Array[(String, Int)] = Array((am,1), (spark,1), (I,1), (Hi,1), (hello,2), (lujinhong,1), (world,1))
	
##（三）filter
###1、官方解释
 Return a new dataset formed by selecting those elements of the source on which func returns true. 
 参数也是一个函数，将RDD的每一个元素传入这个函数，如果返回是true的话，则保留这个元素，否则将这个元素去掉，然后返回一个新的RDD。
###2、示例
（1）列出奇数
	
	scala> val nums = sc.parallelize(List(1,2,3,4,5))
	nums: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[0] at parallelize at <console>:21
	
	
	scala> val odds = nums.filter(x => !(x%2==0))
	odds: org.apache.spark.rdd.RDD[Int] = MapPartitionsRDD[1] at filter at <console>:23
	
	scala> odds.collect
	res0: Array[Int] = Array(1, 3, 5)

（2）求出文件中包含spark字段的行数
	
	scala> val f = sc.textFile("README.md")
	f: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[3] at textFile at <console>:21
	
	scala> val spark = f.filter(line => line.contains("spark"))
	spark: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[4] at filter at <console>:23
	
	scala> spark.count
	res4: Long = 11

##（四）distinct
###1、解释
 Return a new dataset that contains the distinct elements of the source dataset.
不需要参数，将RDD中的重复元素去除

但注意distince需要各个分区作比较，因此需要网络传输，消耗较大。

###2、示例
	
	scala> val nums = sc.parallelize(List(1,2,3,3,5,5,7))
	nums: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[5] at parallelize at <console>:21
	
	scala> val d = nums.distinct()
	d: org.apache.spark.rdd.RDD[Int] = MapPartitionsRDD[8] at distinct at <console>:23
	
	scala> d.collect
	res5: Array[Int] = Array(1, 5, 2, 3, 7)
##（五）union
###1、解释
 Return a new dataset that contains the union of the elements in the source dataset and the argument. 
 参数为另一个RDD，将2个RDD的元素放在一起，形成一个新的RDD，即使重复元素也会被保存下来。
 
###2、示例

	scala> val rdd1 = sc.parallelize(List(1,2,3,4,5))
	rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[9] at parallelize at <console>:21
	
	scala> val rdd2 = sc.parallelize(List(1,3,5,7,9))
	rdd2: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[10] at parallelize at <console>:21
	
	scala> rdd1.union(rdd2).collect
	res6: Array[Int] = Array(1, 2, 3, 4, 5, 1, 3, 5, 7, 9)
	

##（六）intersection
###1、解释
 Return a new RDD that contains the intersection of elements in the source dataset and the argument. 
 参数是另一个RDD，保留在2个RDD均存在的元素，形成一个新的RDD.
 
###2、示例
	
	scala> val rdd1 = sc.parallelize(List(1,2,3,4,5))
	rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[9] at parallelize at <console>:21
	
	scala> val rdd2 = sc.parallelize(List(1,3,5,7,9))
	rdd2: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[10] at parallelize at <console>:21
	
	scala> rdd1.intersection(rdd2).collect
	res8: Array[Int] = Array(1, 5, 3)


##（七）subtract
###1、解释
官方文档中竟然没有。
参数是另一个RDD，在第一个RDD中删除在第二个RDD中存在的元素。
###2、示例
	
	scala> val rdd1 = sc.parallelize(List(1,2,3,4,5))
	rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[9] at parallelize at <console>:21
	
	scala> val rdd2 = sc.parallelize(List(1,3,5,7,9))
	rdd2: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[10] at parallelize at <console>:21
	
	scala> rdd1.subtract(rdd2).collect
	res9: Array[Int] = Array(4, 2)

##（八）cartisian
###1、解释
 When called on datasets of types T and U, returns a dataset of (T, U) pairs (all pairs of elements). 
 参数是另一个RDD，返回2个RDD中所有元素的笛卡尔积形成的二元组。注意大数据量的笛卡尔计算代价非常大。
笛卡尔积在我们希望考虑所有可能的相似度时比较有用，比如计算各用户对各种产品的预期兴趣程度等。
###2、示例
	
	scala> val rdd1 = sc.parallelize(List(1,2,3,4,5))
	rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[9] at parallelize at <console>:21
	
	scala> val rdd2 = sc.parallelize(List(1,3,5,7,9))
	rdd2: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[10] at parallelize at <console>:21
	
	scala> rdd1.cartesian(rdd2).collect
	res10: Array[(Int, Int)] = Array((1,1), (1,3), (1,5), (1,7), (1,9), (2,1), (2,3), (2,5), (2,7), (2,9), (3,1), (3,3), (3,5), (3,7), (3,9), (4,1), (5,1), (4,3), (5,3), (4,5), (5,5), (4,7), (4,9), (5,7), (5,9))


#二、基本RDD的Action

##（一）count
###1、解释
 Return the number of elements in the dataset. 
 不用解释了吧，就是数一下RDD中元素的个数。
###2、示例
	
	scala> val rdd1 = sc.parallelize(List(1,2,3,4,5))
	rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[9] at parallelize at <console>:21
	
	scala> rdd1.count
	res11: Long = 5


##（二）collect
###1、解释
 Return all the elements of the dataset as an array at the driver program. This is usually useful after a filter or other operation that returns a sufficiently small subset of the data. 
以array的形式 返回RDD的全部元素。注意这会将所有数据保存在driver所有机器的内存中，除非是小数据量的返回，否则慎用。

它的返回不再是一个RDD，而一个array。

###2、示例
	
	scala> val rdd1 = sc.parallelize(List(1,2,3,4,5))
	rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[9] at parallelize at <console>:21
	
	scala> rdd1.collect
	res12: Array[Int] = Array(1, 2, 3, 4, 5)

##（三）reduce
###1、解释
 Aggregate the elements of the dataset using a function func (which takes two arguments and returns one). The function should be commutative and associative so that it can be computed correctly in parallel. 
参数是一个函数，这个函数接收2个参数，并返回一个相同类型结果。RDD中的所有元素经过这个函数的计算后最终形成一个结果。

###2、示例

	scala> val rdd1 = sc.parallelize(List(1,2,3,4,5))
	rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[9] at parallelize at <console>:21

	scala> rdd1.reduce(_+_)
	res14: Int = 15
也可以这样写：

	scala> rdd1.reduce((a,b)=>a+b)
	res15: Int = 15

##（四）first
###1、解释
 Return the first element of the dataset (similar to take(1)). 
 返回第一个元素，也就是take(1)。其实还是有区别的，first返回的是类型与元素自身的类型相同，面take(1)返回的是一个array。

###2、示例

	scala> val rdd1 = sc.parallelize(List(1,2,3,4,5))
	rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[9] at parallelize at <console>:21

	scala> rdd1.first
	res16: Int = 1
	

##（五）take
###1、解释
 Return an array with the first n elements of the dataset. 
 返回前面n个元素。注意，它会访问尽量少的分区，因此该操作得到的是一个不均衡的集合。
 
###2、示例
	
	scala> val rdd1 = sc.parallelize(List(1,2,3,4,5))
	rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[9] at parallelize at <console>:21
	
	scala> rdd1.take(1)
	res17: Array[Int] = Array(1)

##（六）fold
###1、解释
官方文档无解释
与reduce一样，但需要提初始值。

###2、示例
	
	scala> val rdd1 = sc.parallelize(List(1,2,3,4,5))
	rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[9] at parallelize at <console>:21

	scala> rdd1.fold(1)(_+_)
	res18: Int = 20


##（七）top
###1、解释
官方文档无解释
返回前面n个元素。但是从后面开始读取的。

###2、示例

	scala> val rdd1 = sc.parallelize(List(1,2,3,4,5))
	rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[9] at parallelize at <console>:21
		
	scala> rdd1.top(2)
	res19: Array[Int] = Array(5, 4)


##（八）foreach
###1、解释
 Run a function func on each element of the dataset. This is usually done for side effects such as updating an Accumulator or interacting with external storage systems.
Note: modifying variables other than Accumulators outside of the foreach() may result in undefined behavior. See Understanding closures for more details.

参数是一个函数，将函数作用在RDD中的每一个元素。一般是用作一些副作用，如打印，保存到外部存储等。

###2、示例

	scala> val rdd1 = sc.parallelize(List(1,2,3,4,5))
	rdd1: org.apache.spark.rdd.RDD[Int] = ParallelCollectionRDD[9] at parallelize at <console>:21
	scala> rdd1.foreach(println)
	3
	4
	1
	2
	5

##（九）aggregate
###1、解释
和reduce类似，但是通常返回不同类型的函数。它也需要提供初始值。然后通过一个函数把RDD中的元素合并起来放入累加器，考虑到每个节点是在本地进行累加的，最终还需要提供第二个函数来将累加器两两合并。


###2、示例
以下示例计算一个RDD的平均值，先计算各个分区中的value总和以及数字的个数，再将各个分区的value总和与数字个数分别相加。



##（十）saveAsTextFile
###1、解释
 Write the elements of the dataset as a text file (or set of text files) in a given directory in the local filesystem, HDFS or any other Hadoop-supported file system. Spark will call toString on each element to convert it to a line of text in the file. 
 将RDD的内容写入文件中，参数为路径。

###2、示例


##（十一）saveAsSequenceFile
###1、解释
 Write the elements of the dataset as a Hadoop SequenceFile in a given path in the local filesystem, HDFS or any other Hadoop-supported file system. This is available on RDDs of key-value pairs that implement Hadoop's Writable interface. In Scala, it is also available on types that are implicitly convertible to Writable (Spark includes conversions for basic types like Int, Double, String, etc). 
仅适用于java/scala。

###2、示例




#三、pairRDD专用Transformation和Action

##（一）reduceByKey
###1、解释
 When called on a dataset of (K, V) pairs, returns a dataset of (K, V) pairs where the values for each key are aggregated using the given reduce function func, which must be of type (V,V) => V. Like in groupByKey, the number of reduce tasks is configurable through an optional second argument. 
 参数是一个函数，具有相同key的元素会经过这个函数处理，返回一个结果
 
###2、示例

	scala> val rdd = sc.parallelize(List(("lujinhong",2),("jason",3),("ljh",5),("jason",6),("ljh",8)))
	rdd: org.apache.spark.rdd.RDD[(String, Int)] = ParallelCollectionRDD[33] at parallelize at <console>:21
	
	scala> rdd.reduceByKey(_+_).collect
	res27: Array[(String, Int)] = Array((lujinhong,2), (ljh,13), (jason,9))

另一个例子请见mapvalues

##（二）groupByKey
###1、解释
 When called on a dataset of (K, V) pairs, returns a dataset of (K, Iterable<V>) pairs.
Note: If you are grouping in order to perform an aggregation (such as a sum or average) over each key, using reduceByKey or aggregateByKey will yield much better performance.
Note: By default, the level of parallelism in the output depends on the number of partitions of the parent RDD. You can pass an optional numTasks argument to set a different number of tasks. 
无参数，只是单纯将具有相同key的元素放在一起，不作任何处理。注意与reduceByKey的区别。

###2、示例

	scala> val rdd = sc.parallelize(List(("lujinhong",2),("jason",3),("ljh",5),("jason",6),("ljh",8)))
	rdd: org.apache.spark.rdd.RDD[(String, Int)] = ParallelCollectionRDD[33] at parallelize at <console>:21
	
	scala> rdd.groupByKey.collect
	res26: Array[(String, Iterable[Int])] = Array((lujinhong,CompactBuffer(2)), (ljh,CompactBuffer(5, 8)), (jason,CompactBuffer(3, 6)))


##（三）agrregateByKey

###1、官方解释

aggregateByKey(zeroValue)(seqOp, combOp, [numTasks]) 

  When called on a dataset of (K, V) pairs, returns a dataset of (K, U) pairs where the values for each key are aggregated using the given combine functions and a neutral "zero" value. Allows an aggregated value type that is different than the input value type, while avoiding unnecessary allocations. Like in groupByKey, the number of reduce tasks is configurable through an optional second argument. 

###2、简单例子
	
	scala> def seq(a:Int, b:Int) : Int ={
	     |  math.max(a,b)
	     | }
	seq: (a: Int, b: Int)Int
	
	scala> def comb(a:Int, b:Int) : Int ={
	     |  a + b
	     | }
	comb: (a: Int, b: Int)Int
	
	scala> val data = sc.parallelize(List((1,3),(1,2),(1, 4),(2,3)))
	data: org.apache.spark.rdd.RDD[(Int, Int)] = ParallelCollectionRDD[0] at parallelize at <console>:21
	
	scala> data.aggregateByKey(3,4)(seq, comb).collect
	res0: Array[(Int, Int)] = Array((1,10), (2,3))

###3、原理介绍

aggerateByKey用在k-v形式的RDD，根据Key的值进行聚合，与groupByKey类似。

参数"3"代表做比较的初始值，参数"4"代表并行化分区的数量。

参数seq代表与初始化值比较的函数。  参数comb是进行合并的方法。

将这个测试程序拿文字做一下描述就是：在data数据集中，按key将value进行分组合并，合并时在seq函数与指定的初始值3进行比较，保留大的值；然后在comb中来处理合并的方式。

详细一点解释：
（1）将List中的各个K-V值中的V值与初始值传递给Seq函数，得出新的V值。如上面的例子中sep函数是求出大的那个值，而初始值为3，初始List为List((1,3),(1,2),(1, 4),(2,3))，经过seq处理后为：List((1,3),(1,3),(1, 4),(2,3))。即第2个元素的V值由2修改为3。
（2）将List的元素按照key进行合并，形成List((1,(3,3,4)),(2,3))的形成
（3）将已经聚合的List的各个元素的值使用com函数进行处理，得出结果List((1,10),(2,3))


##（四）combineByKey
###1、解释

def combineByKey[C](createCombiner: (V) ⇒ C, mergeValue: (C, V) ⇒ C, mergeCombiners: (C, C) ⇒ C): RDD[(K, C)]

Generic function to combine the elements for each key using a custom set of aggregation functions.

Simplified version of combineByKey that hash-partitions the resulting RDD using the existing partitioner/parallelism level.
combineByKey有三个参数，均为一个函数，分别为初始化函数，各分区运行的一个函数，分区之间合并的一个函数。

详见P46


###2、示例
	
	scala> val rdd = sc.parallelize(List(("lujinhong",90),("ljh",80),("lujinhong",89),("jason",46),("jason",64),("lujinhong",84),("ljh",99)))
	rdd: org.apache.spark.rdd.RDD[(String, Int)] = ParallelCollectionRDD[0] at parallelize at <console>:21
	
	scala> val result = rdd.combineByKey(
	      (v) => (v,1),
	      (acc:(Int,Int),v)=>(acc._1+v,acc._2+1),
	      (acc1: (Int,Int),acc2:(Int,Int)) => (acc1._1+acc2._1,acc1._2,acc2._2)).collect

我们先将value值转化为一个(v,1)形式的map，然后在各个分区计算某个key出现的次数，以及其值的总和，最后计算所有分区key出现的次数以及总和。


##（五）sortByKey
 When called on a dataset of (K, V) pairs where K implements Ordered, returns a dataset of (K, V) pairs sorted by keys in ascending or descending order, as specified in the boolean ascending argument.

按照key的值进行排序
	
	scala> val rdd = sc.parallelize(List((1,2),(2,3),(3,4),(2,1)))
	rdd: org.apache.spark.rdd.RDD[(Int, Int)] = ParallelCollectionRDD[26] at parallelize at <console>:21
	
	scala> rdd.sortByKey().collect
	res14: Array[(Int, Int)] = Array((1,2), (2,3), (2,1), (3,4))

##（五）join
2个集合的连接，只有某个key在2个RDD均存在才会输出。
同理还有leftOuterJoin和rightOuterJoin

##（七）mapValues

如果我们只想访问pari RDD的值部分，这时操作二元组很麻烦。由于这是一种常见的使用模式，因此spark提供了mapValues(func)函数，功能类似于map{case(x,y):(x,func(y))}。

下面这个例子返回的是(key,(sum,count))的内容。

scala> val pRdd = sc.parallelize(List(("panda",0),("pink",3),("pirate",3),("panda",1),("pink",4)))
pRdd: org.apache.spark.rdd.RDD[(String, Int)] = ParallelCollectionRDD[2] at parallelize at <console>:21

scala> pRdd.mapValues(x=>(x,1)).reduceByKey((x,y)=>(x._1+y._1,x._2+y._2)).collect
res5: Array[(String, (Int, Int))] = Array((pirate,(3,1)), (panda,(1,2)), (pink,(7,2)))


#四、pairRDD专用Action


##（一） countByKey
###1、解释
 Only available on RDDs of type (K, V). Returns a hashmap of (K, Int) pairs with the count of each key. 
###2、示例
