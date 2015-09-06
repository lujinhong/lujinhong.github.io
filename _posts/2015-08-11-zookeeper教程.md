---
layout: post
tile:  "zookeeper教程"
date:  2015-08-11 17:35:12
categories: zookeeper 大数据 
excerpt: zookeeper教程
---

* content
{:toc}





#（一）安装
见后面附录

#（二）基本操作

1、启动、关闭zookeeper

	bin/zkServer.sh start 
	bin/zkServer.sh stop

2、查看基本状态
		
	bin/zkServer status
	-su: bin/zkServer: No such file or directory
	hadoop@gdc-dn48-formal:~/zookeeper$ bin/zkServer.sh status
	JMX enabled by default
	Using config: /home/hadoop/zookeeper/bin/../conf/zoo.cfg
	Mode: follower 

3、日志清理
zk在运行过程中会产生大量的日志，zk提供了脚本运行清理

	bin/zkCleanup.sh -n 20

只保留最新的20个日志文件

4、常用操作命令
进入命令行 

	bin/zkCli.sh 

（1）显示根目录下、文件： ls / 使用 ls 命令来查看当前 ZooKeeper 中所包含的内容
		
	[zk: localhost:2181(CONNECTED) 0] ls /
	[hbase, hadoop-ha, zookeeper, hive_zookeeper_namespace]
	[zk: localhost:2181(CONNECTED) 1] ls /hbase
	[meta-region-server, acl, backup-masters, table, draining, region-in-transition, table-lock, running, master, balancer, tokenauth, namespace, hbaseid, online-snapshot, replication, splitWAL, recovering-regions, rs]

（2）显示根目录下、文件： ls2 / 查看当前节点数据并能看到更新次数等数据
查看目录：

		[zk: localhost:2181(CONNECTED) 9] ls2 /hbase
	[meta-region-server, acl, backup-masters, table, draining, region-in-transition, table-lock, running, master, balancer, tokenauth, namespace, hbaseid, online-snapshot, replication, splitWAL, recovering-regions, rs]
	cZxid = 0x80001689e
	ctime = Fri Nov 21 13:43:22 HKT 2014
	mZxid = 0x80001689e
	mtime = Fri Nov 21 13:43:22 HKT 2014
	pZxid = 0xc003ce704
	cversion = 138
	dataVersion = 0
	aclVersion = 0
	ephemeralOwner = 0x0
	dataLength = 0
	numChildren = 18
	[zk: localhost:2181(CONNECTED) 18] get /hbase

查看文件：
		
	[zk: localhost:2181(CONNECTED) 4] ls2 /hbase/table/test
	[]
	cZxid = 0x800016b71
	ctime = Fri Nov 21 13:45:20 HKT 2014
	mZxid = 0x800016b72
	mtime = Fri Nov 21 13:45:20 HKT 2014
	pZxid = 0x800016b71
	cversion = 0
	dataVersion = 1
	aclVersion = 0
	ephemeralOwner = 0x0
	dataLength = 31
	numChildren = 0

（3）创建文件，并设置初始内容： create /zk "test" 创建一个新的 znode节点“ zk ”以及与它关联的字符串
（4） 得到一个节点，并获取文件内容： get /zk 确认 znode 是否包含我们所创建的字符串
         与ls2基本一致，但对于目录，ls2会列出目录中包含的内容，get不会；对于文件，get会多出一行类似?master:60000&+:t̖PBUF的内容。
         
							
	[zk: localhost:2181(CONNECTED) 21] get /hbase
	cZxid = 0x80001689e
	ctime = Fri Nov 21 13:43:22 HKT 2014
	mZxid = 0x80001689e
	mtime = Fri Nov 21 13:43:22 HKT 2014
	pZxid = 0xc003ce704
	cversion = 138
	dataVersion = 0
	aclVersion = 0
	ephemeralOwner = 0x0
	dataLength = 0
	numChildren = 18
	[zk: localhost:2181	(CONNECTED) 19] get /hbase/table/test
	?master:60000&+:t̖PBUF
	cZxid = 0x800016b71
	ctime = Fri Nov 21 13:45:20 HKT 2014
	mZxid = 0x800016b72
	mtime = Fri Nov 21 13:45:20 HKT 2014
	pZxid = 0x800016b71
	cversion = 0
	dataVersion = 1
	aclVersion = 0
	ephemeralOwner = 0x0
	dataLength = 31
	numChildren = 0


（5）修改文件内容： set /zk "zkbak" 对 zk 所关联的字符串进行设置
（6） 删除文件： delete /zk 将刚才创建的 znode 删除
（7） 退出客户端： quit
（8）帮助命令： help

附完整命令选项：
			
	[zk: localhost:2181(CONNECTED) 0] help
	ZooKeeper -server host:port cmd args
	 connect host:port
	 get path [watch]
	 ls path [watch]
	 set path data [version]
	 rmr path		
	 delquota [-n|-b] path
	 quit
	 printwatches on|off
	 create [-s] [-e] path data acl
	 stat path [watch]
	 close
	 ls2 path [watch]
	 history
	 listquota path
	 setAcl path acl
	 getAcl path
	 sync path
	 redo cmdno
	 addauth scheme auth
	 delete path [version]
	 setquota -n|-b val path


【附zookeeper的安装过程】：

Zookeeper 是一个分布式开源框架，提供了协调分布式应用的基本服务，它向外部应用暴露一组通用服务——分布式同步（Distributed Synchronization）、命名服务（Naming Service）、集群维护（Group Maintenance）等，简化分布式应用协调及其管理的难度，提供高性能的分布式服务。ZooKeeper本身可以以单机模式安装运行，不过它的长处 在于通过分布式ZooKeeper集群（一个Leader，多个Follower），基于一定的策略来保证ZooKeeper集群的稳定性和可用性，从而 实现分布式应用的可靠性。

本文将向大家主要介绍Zookeeper的安装与配置。

众所周知，Zookeeper有三种不同的运行环境，包括：单机环境、集群环境和集群伪分布式环境。



在此主要向大家介绍集群环境的安装与配置。

1.Zookeeper的下载与解压

通过后面的链接下载Zookeeper：    Zookeeper下载

在此我们下载zookeeper-3.4.5

下载后解压至安装目录下，本文我们解压到目录：/home/haduser/zookeeper

	$:tar -xzvf zookeeper-3.4.5.tar.gz

2.zookeeper的环境变量的配置：
为了今后操作方便，我们需要对Zookeeper的环境变量进行配置，方法如下：
在/etc/profile文件中加入如下的内容：
set zookeeper environment

	export ZOOKEEPER_HOME=/home/haduser/zookeeper/zookeeper-3.4.5
	export PATH=$PATH:$ZOOKEEPER_HOME/bin:$ZOOKEEPER_HOME/conf

3.集群部署：

在Zookeeper集群环境下只要一半以上的机器正常启动了，那么Zookeeper服务将是可用的。因此，集群上部署Zookeeper最好使用奇数台机器，这样如果有5台机器，只要3台正常工作则服务将正常使用。

下面我们将对Zookeeper的配置文件的参数进行设置：

进入zookeeper-3.4.5/conf：

	$:cp zoo_sample.cfg zoo.cfg

	$:vim zoo.cfg

可参考下图配置：

Zookeeper

注意上图的配置中master，slave1分别为主机名，具体的对应的主机可参见之前的Hadoop的安装与配置的博文。

在上面的配置文件中"server.id=host:port:port"中的第一个port是从机器（follower）连接到主机器（leader）的端口号，第二个port是进行leadership选举的端口号。  

接下来在dataDir所指定的目录下创建一个文件名为myid的文件，文件中的内容只有一行，为本主机对应的id值，也就是上图中server.id中的id。例如：在服务器1中的myid的内容应该写入1。

4.远程复制分发安装文件

接下来将上面的安装文件拷贝到集群中的其他机器上对应的目录下：

	haduser@master:~/zookeeper$ scp -r zookeeper-3.4.5/ slave1:/home/haduser/zookeeper/zookeeper-3.4.5

	haduser@master:~/zookeeper$ scp -r zookeeper-3.4.5/ slave2:/home/haduser/zookeeper/zookeeper-3.4.5



拷贝完成后修改对应的机器上的myid。例如修改slave1中的myid如下：

	haduser@slave1:~/zookeeper/zookeeper-3.4.5$ echo "2" > data/myid
	haduser@slave1:~/zookeeper/zookeeper-3.4.5$ cat data/myid 
2



5.启动ZooKeeper集群

在ZooKeeper集群的每个结点上，执行启动ZooKeeper服务的脚本，如下所示：

	$ bin/zkServer.sh start
	$ bin/zkServer.sh start
	$ bin/zkServer.sh start
		
	$ jps
	24466 NodeManager
	24230 SecondaryNameNode
	20357 QuorumPeerMain
	22010 Jstatd
	20396 Jps
	23962 DataNode


其中，QuorumPeerMain是zookeeper进程，启动正常。
如上依次启动了所有机器上的Zookeeper之后可以通过ZooKeeper的脚本来查看启动状态，包括集群中各个结点的角色（或是Leader，或是Follower），如下所示，是在ZooKeeper集群中的每个结点上查询的结果：
		
	hadoop@gdc-dn01-test:~/zookeeper$ bin/zkServer.sh status
	JMX enabled by default
	Using config: /home/hadoop/zookeeper/bin/../conf/zoo.cfg
	Mode: leader



通过上面状态查询结果可见，slave1是集群的Leader，其余的两个结点是Follower。

另外，可以通过客户端脚本，连接到ZooKeeper集群上。对于客户端来说，ZooKeeper是一个整体（ensemble），连接到ZooKeeper集群实际上感觉在独享整个集群的服务，所以，你可以在任何一个结点上建立到服务集群的连接，例如：

$ bin/zkCli.sh
$ ls /

附2：zookeeper的应用场景

　　Zookeeper是hadoop的一个子项目，虽然源自hadoop，但是我发现zookeeper脱离hadoop的范畴开发分布式框架的 运用越来越多。今天我想谈谈zookeeper，本文不谈如何使用zookeeper，而是zookeeper到底有哪些实际的运用，哪些类型的应用能发 挥zookeeper的优势，最后谈谈zookeeper对分布式网站架构能产生怎样的作用。

　　Zookeeper是针对大型分布式系统的高可靠的协调系统。由这个定义我们知道zookeeper是个协调系统，作用的对象是分布式系统。为什么分布式系统需要一个协调系统了？理由如下：

　　开发分布式系统是件很困难的事情，其中的困难主要体现在分布式系统的“部分失败”。“部分失败”是指信息在网络的两个节点之间传送时候，如果网 络出了故障，发送者无法知道接收者是否收到了这个信息，而且这种故障的原因很复杂，接收者可能在出现网络错误之前已经收到了信息，也可能没有收到，又或接 收者的进程死掉了。发送者能够获得真实情况的唯一办法就是重新连接到接收者，询问接收者错误的原因，这就是分布式系统开发里的“部分失败”问题。

　　Zookeeper就是解决分布式系统“部分失败”的框架。Zookeeper不是让分布式系统避免“部分失败”问题，而是让分布式系统当碰到部分失败时候，可以正确的处理此类的问题，让分布式系统能正常的运行。

　　下面我要讲讲zookeeper的实际运用场景：

　　场景一：统一命名服务。有一组服务器向客户端提供某种服务（例如：我前面做的分布式网站的服务端，就是由四台服务器组成的 集群，向前端集群提供服务），我们希望客户端每次请求服务端都可以找到服务端集群中某一台服务器，这样服务端就可以向客户端提供客户端所需的服务。对于这 种场景，我们的程序中一定有一份这组服务器的列表，每次客户端请求时候，都是从这份列表里读取这份服务器列表。那么这分列表显然不能存储在一台单节点的服 务器上，否则这个节点挂掉了，整个集群都会发生故障，我们希望这份列表时高可用的。高可用的解决方案是：这份列表是分布式存储的，它是由存储这份列表的服 务器共同管理的，如果存储列表里的某台服务器坏掉了，其他服务器马上可以替代坏掉的服务器，并且可以把坏掉的服务器从列表里删除掉，让故障服务器退出整个 集群的运行，而这一切的操作又不会由故障的服务器来操作，而是集群里正常的服务器来完成。这是一种主动的分布式数据结构，能够在外部情况发生变化时候主动 修改数据项状态的数据机构。Zookeeper框架提供了这种服务。这种服务名字就是：统一命名服务，它和javaEE里的JNDI服务很像。

　　场景二：分布式锁服务。当分布式系统操作数据，例如：读取数据、分析数据、最后修改数据。在分布式系统里这 些操作可能会分散到集群里不同的节点上，那么这时候就存在数据操作过程中一致性的问题，如果不一致，我们将会得到一个错误的运算结果，在单一进程的程序 里，一致性的问题很好解决，但是到了分布式系统就比较困难，因为分布式系统里不同服务器的运算都是在独立的进程里，运算的中间结果和过程还要通过网络进行 传递，那么想做到数据操作一致性要困难的多。Zookeeper提供了一个锁服务解决了这样的问题，能让我们在做分布式数据运算时候，保证数据操作的一致 性。

　　场景三：配置管理。在分布式系统里，我们会把一个服务应用分别部署到n台服务器上，这些服务器的配置文件是 相同的（例如：我设计的分布式网站框架里，服务端就有4台服务器，4台服务器上的程序都是一样，配置文件都是一样），如果配置文件的配置选项发生变化，那 么我们就得一个个去改这些配置文件，如果我们需要改的服务器比较少，这些操作还不是太麻烦，如果我们分布式的服务器特别多，比如某些大型互联网公司的 hadoop集群有数千台服务器，那么更改配置选项就是一件麻烦而且危险的事情。这时候zookeeper就可以派上用场了，我们可以把 zookeeper当成一个高可用的配置存储器，把这样的事情交给zookeeper进行管理，我们将集群的配置文件拷贝到zookeeper的文件系统 的某个节点上，然后用zookeeper监控所有分布式系统里配置文件的状态，一旦发现有配置文件发生了变化，每台服务器都会收到zookeeper的通 知，让每台服务器同步zookeeper里的配置文件，zookeeper服务也会保证同步操作原子性，确保每个服务器的配置文件都能被正确的更新。

　　场景四：为分布式系统提供故障修复的功能。集群管理是很困难的，在分布式系统里加入了zookeeper服 务，能让我们很容易的对集群进行管理。集群管理最麻烦的事情就是节点故障管理，zookeeper可以让集群选出一个健康的节点作为 master，master节点会知道当前集群的每台服务器的运行状况，一旦某个节点发生故障，master会把这个情况通知给集群其他服务器，从而重新 分配不同节点的计算任务。Zookeeper不仅可以发现故障，也会对有故障的服务器进行甄别，看故障服务器是什么样的故障，如果该故障可以修 复，zookeeper可以自动修复或者告诉系统管理员错误的原因让管理员迅速定位问题，修复节点的故障。大家也许还会有个疑问，master故障了，那 怎么办了？zookeeper也考虑到了这点，zookeeper内部有一个“选举领导者的算法”，master可以动态选择，当master故障时 候，zookeeper能马上选出新的master对集群进行管理。

　　下面我要讲讲zookeeper的特点：

zookeeper是一个精简的文件系统。这点它和hadoop有点像，但是zookeeper这个文件系统是管理小文件的，而hadoop是管理超大文件的。
zookeeper提供了丰富的“构件”，这些构件可以实现很多协调数据结构和协议的操作。例如：分布式队列、分布式锁以及一组同级节点的“领导者选举”算法。
zookeeper是高可用的，它本身的稳定性是相当之好，分布式集群完全可以依赖zookeeper集群的管理，利用zookeeper避免分布式系统的单点故障的问题。
zookeeper采用了松耦合的交互模式。这点在zookeeper提供分布式锁上表现最为明显，zookeeper可以被用作一个约会机制， 让参入的进程不在了解其他进程的（或网络）的情况下能够彼此发现并进行交互，参入的各方甚至不必同时存在，只要在zookeeper留下一条消息，在该进 程结束后，另外一个进程还可以读取这条信息，从而解耦了各个节点之间的关系。
zookeeper为集群提供了一个共享存储库，集群可以从这里集中读写共享的信息，避免了每个节点的共享操作编程，减轻了分布式系统的开发难度。
zookeeper的设计采用的是观察者的设计模式，zookeeper主要是负责存储和管理大家关心的数据，然后接受观察者的注册，一旦这些数 据的状态发生变化，Zookeeper 就将负责通知已经在 Zookeeper 上注册的那些观察者做出相应的反应，从而实现集群中类似 Master/Slave 管理模式。
　　由此可见zookeeper很利于分布式系统开发，它能让分布式系统更加健壮和高效。

##(三) 一个小工具 zk-web
可以在浏览器中浏览zk中的内容，详细安装步骤如下：

1. 在github上下载源码：https://github.com/qiuxiafei/zk-web、
wget https://github.com/qiuxiafei/zk-web/archive/master.zip
2. 进行解压，使用命令： unzip zk-web-master ，进入目录。
3. 如果没有安装Leiningen,需要安装，直接使用命令：apt-get install leiningen进行程序安装，可能会中断，无法下载，
    之后再使用命令：apt-get update.更新，再apt-get install  leiningen直到完成。
4. 在zk-web-master目录下使用命令：lein deps 继续下载和编译。目录中多出两个目录lib和classes.
5. lein run 运行服务。注意如果8080端口，被占用，请关闭原端口程序。再运行lein run(需要按回车才能运行).
