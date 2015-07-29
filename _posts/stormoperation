---
layout: post
title:  "storm操作指南"
date:   2015-07-29 15:13:20
categories: jekyll update
---

#storm集群操作指南

@(博客文章)[storm|大数据]

 [toc]
#一、storm伪分布式安装
##（一）环境准备
1、OS：debian 7
2、JDK 7.0

##（二）安装zookeeper
1、下载zookeeper并解压
 wget http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.6/zookeeper-3.4.6.tar.gz
 tar -zxvf zookeeper-3.4.6.tar.gz
2、准备配置文件
cd conf
cp zoo_sample.cfg zoo.cfg 
3、启动zookeeper
bin/zkServer.sh start 
4、验证zookeeper的状态
bin/zkServer.sh status 
输出如下：
JMX enabled by default
Using config: /home/jediael/setupfile/zookeeper-3.4.6/bin/../conf/zoo.cfg
Mode: standalone

##（三）安装storm 
1、下载storm并解压
wget http://mirror.bit.edu.cn/apache/storm/apache-storm-0.9.4/apache-storm-0.9.4.tar.gz
tar -zxvf apache-storm-0.9.4.tar.gz
2、启动storm
```
   nohup bin/storm nimbus &
   nohup bin/storm supervisor &
   nohup bin/storm ui & 
```
3、查看进程
```
jediael@jediael:~/setupfile/zookeeper-3.4.6$ jps | grep -v Jps
3235 supervisor
3356 core
3140 QuorumPeerMain
3214 nimbus 
```
4、查看ui界面
http://ip:8080

##(四）运行程序
1、根据《storm分布式实时计算模式》第一章代码及P41的修改，并打包上传到服务器
2、运行job
storm jar word-count-1.0-SNAPSHOT.jar storm.blueprints.chapter1.v1.WordCountTopology wordcount-topology
3、在ui界面上可以看到一个topology正在运行
 
#二、storm集群安装
注意：先安装zookeeper：http://blog.csdn.net/jinhong_lu/article/details/46519899
##（一）下载storm并解压
wget http://mirror.bit.edu.cn/apache/storm/apache-storm-0.9.4/apache-storm-0.9.4.tar.gz
tar -zxvf apache-storm-0.9.4.tar.gz
并在home目录中添加链接
ln -s src/apache-storm-0.9.4 storm 

##（二）配置storm，在storm.yaml中添加以下内容
```
storm.zookeeper.servers:
     - "gdc-nn01-test"
     - "gdc-dn01-test"
     - "gdc-dn02-test"
nimbus.host: "gdc-nn01-test"
supervisor.slots.ports:
    - 6700
    - 6701
    - 6702
    - 6703
storm.local.dir: "/home/hadoop/storm/data”
#jvm setting

nimbus.childopts:"-4096m”
supervisor.childopts:"-Xmx4096m"
nimubs.childopts:"-Xmx3072m”
```
 
说明：
1、关于日志
在初次运行storm程序时，可能会出现各种各样的错误，一般错误均可在日志中发现，在本例中，需要重点关注的日志有：
（1）supervisor上的work日志，位于$STORM_HOME/logs，如果集群正常，但某个topology运行出现错误，一般可以在这些work日志中找到问题。最常见的是CLASSNOTFOUNDEXCEPTION, CLASSNOTDEFINDEXCEPTION,都是缺包导致的，将它们放入$STORM_HOME/lib即可。
（2）nimbus上的日志，位于$STORM_HOME/logs，主要观察整个集群的状态，有以下4个文件
access.log  metrics.log  nimbus.log  ui.log
（3）kafka的日志，位于$KAFKA_HOME/logs，观察kafka是否运行正常。

2.关于emit与transfer(转自http://www.reader8.cn/jiaocheng/20120801/2057699.html）
 storm ui上emit和transferred的区别
最开始对storm ui上展示出来的emit和transferred数量不是很明白, 于是在storm-user上google了一把, 发现有人也有跟我一样的困惑, nathan做了详细的回答:
emitted栏显示的数字表示的是调用OutputCollector的emit方法的次数.
transferred栏显示的数字表示的是实际tuple发送到下一个task的计数.
如果一个bolt A使用all group的方式(每一个bolt都要接收到)向bolt B发射tuple, 此时bolt B启动了5个task, 那么trasferred显示的数量将是emitted的5倍.
如果一个bolt A内部执行了emit操作, 但是没有指定tuple的接受者, 那么transferred将为0.

这里还有关于spout, bolt之间的emitted数量的关系讨论, 也解释了我的一些疑惑:
有 的bolt的execture方法中并没有emit tuple, 但是storm ui中依然有显示emitted, 主要是因为它调用了ack方法, 而该方法将emit ack tuple到系统默认的acker bolt. 因此如果anchor方式emit一个tuple, emitted一般会包含向acker bolt发射tuple的数量.

另外collector.emit(new Values(xxx))和collector.emit(tuple, new Values(xxx)) 这两种不同的emit方法也会影响后面bolt的emitted和transferred, 如果是前者, 则后续bolt的这两个值都是0, 因为前一个emit方法是非安全的, 不再使用acker来进行校验.

##（三）关于包依赖的关系
注意、重点：storm运行topology时会有一大堆的包依赖问题，建议保存好现有的包，在新集群中直接导入即可，而且都放到集群中的每一个机器上。

##（四）文件同步
将storm整个目录scp到dn01,dn02,dn03

##（五）启动storm
（1）在nn01上启动nimbus,ui
nohup bin/storm nimbus &
nohup bin/storm ui &
（2）在dn0[123]上启动
nohup bin/storm superivsor &

##（六）验证
(1)打开页面看状态
http://192.168.169.91:8080/index.html 
(2)在example目录下执行一个示例topology
$ /home/hadoop/storm/bin/storm jar storm-starter-topologies-0.9.4.jar storm.stater.WordCountTopology word-count

然后再到ui上看看是否已经提交成功

#三、storm集群的启停
storm有2种启动方式：命令模式以及supervisor模式
##（一）命令模式
1、在nimbus上启动nimbus及ui
```
nohup bin/storm nimbus &
nohup bin/storm ui &
```
（2）在各个supervisor上启动superviosr与logviewer
```
nohup bin/storm superivsor &
nohup bin/storm superivsor &
```
（3）如果有需要的话，启动drpc
```
nohup bin/storm drpc &
```
##（二）supervisor模式
待补充
