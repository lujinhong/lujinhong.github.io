---
layout: post
tile:  "hadoop中如何控制map的数量"
date:  2015-11-13 14:54:37
categories: hadoop 
excerpt: hadoop中如何控制map的数量
---

* content
{:toc}



hadooop提供了一个设置map个数的参数mapred.map.tasks，我们可以通过这个参数来控制map的个数。但是通过这种方式设置map的个数，并不是每次都有效的。原因是mapred.map.tasks只是一个hadoop的参考数值，最终map的个数，还取决于其他的因素。

  为了方便介绍，先来看几个名词：
block_size : hdfs的文件块大小，默认为64M，可以通过参数dfs.block.size设置
total_size : 输入文件整体的大小
input_file_num : 输入文件的个数

（1）默认map个数
     如果不进行任何设置，默认的map个数是和blcok_size相关的。
     
     default_num = total_size / block_size;

（2）期望大小
     可以通过参数mapred.map.tasks来设置程序员期望的map个数，但是这个个数只有在大于default_num的时候，才会生效。
     goal_num = mapred.map.tasks;

（3）设置处理的文件大小
     可以通过mapred.min.split.size 设置每个task处理的文件大小，但是这个大小只有在大于block_size的时候才会生效。
     
     split_size = max(mapred.min.split.size, block_size);
     split_num = total_size / split_size;

（4）计算的map个数

	compute_map_num = min(split_num,  max(default_num, goal_num))

  除了这些配置以外，mapreduce还要遵循一些原则。 mapreduce的每一个map处理的数据是不能跨越文件的，也就是说min_map_num >= input_file_num。 所以，最终的map个数应该为：
  
     final_map_num = max(compute_map_num, input_file_num)

   经过以上的分析，在设置map个数的时候，可以简单的总结为以下几点：
（1）如果想增加map个数，则设置mapred.map.tasks 为一个较大的值。
（2）如果想减小map个数，则设置mapred.min.split.size 为一个较大的值。
（3）如果输入中有很多小文件，依然想减少map个数，则需要将小文件merger为大文件，然后使用准则2。
