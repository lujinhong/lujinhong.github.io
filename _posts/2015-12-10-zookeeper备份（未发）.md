---
layout: post
tile:  "zookeeper备份（未发）"
date:  2015-12-10 11:33:20
categories: others zookeeper storm kafka 
excerpt: zookeeper备份（未发）
---

* content
{:toc}



[TOC]

##小结
zookeeper作为大数据平台的配置中心，其保存的内容对平台十分重要，因此必须做好备份。

1、zk会自动将内存中的内容生成snapshot文件，这就是配置文件，我们只需要配置删除这些文件的策略即可，以免占用过多空间。
2、将snapshot文件放到合适的路径，重启一个新的zk，这个snapshot就会被自动加载。
3、观察snapshot文件的生成时长，如果过短则调整参数，多做点操作再生成，反之亦然。

##详细解释

ZooKeeper Server 不断将 znode 快照文件和事务日志（可选）保存至 Data Directory，使您可以恢复数据。

使用默认配置时，ZooKeeper Server 不会删除快照和日志文件，使它们可以不断累积。您有时需要清理此目录，考虑您的备份计划和流程。为自动化清理，zkCleanup.sh 脚本被提供在 zookeeper base 包的 bin 目录中。根据情况需要修改此脚本。一般而言，您需要根据备份计划，作为 cron 任务运行它。

data directory 通过 dataDir 参数（ZooKeeper 配置文件中）指定数据日志目录通过 dataLogDir 参数指定。
snapCount：每当 snapCount 个事物日志写入时，快照被创建，同时创建新的事务日志文件，默认值100,000。(Java system property: zookeeper.snapCount)

snapCount(系统属性：zookeeper.snapCount) //默认为100000，在新增log（txn log）条数达到snapCount/2 + Random.nextInt(snapCount/2)时，将会对zkDatabase(内存数据库)进行snapshot，将内存中DataTree反序为snapshot文件数据，同时log计数置为0，以此循环。snapshot过程中，同时也伴随txn log的新文件创建（这也是snapCount与preAllocSize参数的互相协调原因）。snapshot时使用随机数的原因：让每个server snapshot的时机具有随即且可控，避免所有的server同时snapshot（snapshot过程中将阻塞请求）。参见SyncRequestProcessor.run()

Using older log and snapshot files, you can look at the previous state of ZooKeeper servers and even restore that state. The LogFormatter class allows an administrator to look at the transactions in a log.
The retention policy of the data and log files is implemented outside of the ZooKeeper server

The PurgeTxnLog utility implements a simple retention policy that administrators can use.

方式一：

In the following example the last count snapshots and their corresponding logs are retained and the others are deleted. The value of should typically be greater than 3 (although not required, this provides 3 backups in the unlikely event a recent log has become corrupted). This can be run as a cron job on the ZooKeeper server machines to clean up the logs daily.

    java -cp zookeeper.jar:lib/slf4j-api-1.6.1.jar:lib/slf4j-log4j12-1.6.1.jar:lib/log4j-1.2.15.jar:conf org.apache.zookeeper.server.PurgeTxnLog -n

方式二：
zkCleanup.sh
	
	if [ "x$ZOODATALOGDIR" = "x" ]
	then
	$JAVA "-Dzookeeper.log.dir=${ZOO_LOG_DIR}" "-Dzookeeper.root.logger=${ZOO_LOG4J_PROP}" \
	     -cp "$CLASSPATH" $JVMFLAGS \
	     org.apache.zookeeper.server.PurgeTxnLog "$ZOODATADIR" $*
	else
	$JAVA "-Dzookeeper.log.dir=${ZOO_LOG_DIR}" "-Dzookeeper.root.logger=${ZOO_LOG4J_PROP}" \
	     -cp "$CLASSPATH" $JVMFLAGS \
	     org.apache.zookeeper.server.PurgeTxnLog "$ZOODATALOGDIR" "$ZOODATADIR" $*
	fi

方式三：
zoo.cfg:
	
	# snapcount
	snapCount=100000
	# snap retain counts when autopurge
	autopurge.snapRetainCount=5
	# snap puger intervals when autopurge
	autopurge.purgeInterval=1

Automatic purging of the snapshots and corresponding transaction logs was introduced in version 3.4.0 and can be enabled via the following configuration parameters autopurge.snapRetainCount and autopurge.purgeInterval.

 snapCount

(Java system property: zookeeper.snapCount)

ZooKeeper logs transactions to a transaction log. After snapCount transactions are written to a log file a snapshot is started and a new transaction log file is created. The default snapCount is 100,000.

 autopurge.snapRetainCount

(No Java system property)

New in 3.4.0: When enabled, ZooKeeper auto purge feature retains the autopurge.snapRetainCount most recent snapshots and the corresponding transaction logs in the dataDir and dataLogDir respectively and deletes the rest. Defaults to 3. Minimum value is 3.

 autopurge.purgeInterval

(No Java system property)

New in 3.4.0: The time interval in hours for which the purge task has to be triggered. Set to a positive integer (1 and above) to enable the auto purging. Defaults to 0.
可考虑做的：

1.修改transaction log dir,让log文件与snapshot文件独立开。
The server can (and should) be configured to store the transaction log files in a separate directory than the data files.
Throughput increases and latency decreases when transaction logs reside on a dedicated log devices.

对应配置：
	
	# the directory where the snapshot is stored.
	dataDir=/home/data/zookeeper
	# the directory where the transaction log is stored
	dataLogDir=/home/data/zookeeper

2.定期再将datadir目录的内容备份到硬盘上。(设置了observer的已有此功能）

3.调整 snapCount 参数。
正式集群zk调小（保证snapshot的备份频率），接入kafka和storm的集群调节大（减少snapshot生成阻塞请求带来的影响），preAllocSize 则相反减少一点。如下：
