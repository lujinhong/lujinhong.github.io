---
layout: post
tile:  "何时使用hadoop fs、hadoop dfs与hdfs dfs命令"
date:  2015-11-13 15:16:27
categories: hadoop 
excerpt: 何时使用hadoop fs、hadoop dfs与hdfs dfs命令
---

* content
{:toc}



hadoop fs：使用面最广，可以操作任何文件系统。

hadoop dfs与hdfs dfs：只能操作HDFS文件系统相关（包括与Local FS间的操作），前者已经Deprecated，一般使用后者。

 

以下内容参考自stackoverflow

 

Following are the three commands which appears same but have minute differences

    hadoop fs {args}
    hadoop dfs {args}

    hdfs dfs {args}

    hadoop fs <args> 

FS relates to a generic file system which can point to any file systems like local, HDFS etc. So this can be used when you are dealing with different file systems such as Local FS, HFTP FS, S3 FS, and others

 hadoop dfs <args> 

dfs is very specific to HDFS. would work for operation relates to HDFS. This has been deprecated and we should use hdfs dfs instead.

 hdfs dfs <args> 

same as 2nd i.e would work for all the operations related to HDFS and is the recommended command instead of hadoop dfs

below is the list categorized as HDFS commands.

 **#hdfs commands** namenode|secondarynamenode|datanode|dfs|dfsadmin|fsck|balancer|fetchdt|oiv|dfsgroups 

So even if you use Hadoop dfs , it will look locate hdfs and delegate that command to hdfs dfs
