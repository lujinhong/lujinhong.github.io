---
layout: post
title:  "test2!"
date:   2015-07-17 15:13:20
categories: jekyll update
---

test

[toc]
#一、原理及关键步骤介绍
storm中的storm-kafka组件提供了storm与kafka交互的所需的所有功能，请参考其官方文档：https://github.com/apache/storm/tree/master/external/storm-kafka#brokerhosts
##（一）使用storm-kafka的关键步骤
###1、创建ZkHosts
当storm从kafka中读取某个topic的消息时，需要知道这个topic有多少个分区，以及这些分区放在哪个kafka节点(broker)上，ZkHosts就是用于这个功能。
创建zkHosts有2种形式
```
   public ZkHosts(String brokerZkStr, String brokerZkPath) 
   public ZkHosts(String brokerZkStr)
```
   
（1）默认情况下，zk信息被放到/brokers中，此时可以使用第2种方式：
new ZkHosts("192.168.172.117:2181,192.168.172.98:2181,192.168.172.111:2181,192.168.172.114:2181,192.168.172.116:2181”);

（2）若zk信息被放置在/kafka/brokers中（我们的集群就是这种情形），则可以使用：
 publicZkHosts("192.168.172.117:2181,192.168.172.98:2181,192.168.172.111:2181,192.168.172.114:2181,192.168.172.116:2181",“/kafka") 
 
或者直接：
```
 new ZkHosts("192.168.172.117:2181,192.168.172.98:2181,192.168.172.111:2181,192.168.172.114:2181,192.168.172.116:2181/kafka”)
```
默认情况下，每60秒去读取一次kafka的分区信息，可以通过修改host.refreshFreqSecs来设置。

（3）除了使用ZkHosts来读取分析信息外，storm-kafka还提供了一种静态指定的方法（不推荐此方法），如：
```
    Broker brokerForPartition0 = new Broker("localhost");//localhost:9092
    Broker brokerForPartition1 = new Broker("localhost", 9092);//localhost:9092 but we specified the port explicitly
    Broker brokerForPartition2 = new Broker("localhost:9092");//localhost:9092 specified as one string.
    GlobalPartitionInformation partitionInfo = new GlobalPartitionInformation();
    partitionInfo.addPartition(0, brokerForPartition0);//mapping form partition 0 to brokerForPartition0
    partitionInfo.addPartition(1, brokerForPartition1);//mapping form partition 1 to brokerForPartition1
    partitionInfo.addPartition(2, brokerForPartition2);//mapping form partition 2 to brokerForPartition2
    StaticHosts hosts = new StaticHosts(partitionInfo);
```
 由此可以看出，ZkHosts完成的功能就是指定了从哪个kafka节点读取某个topic的哪个分区。
 
###2、创建KafkaConfig
(1)有2种方式创建KafkaConfig
   public KafkaConfig(BrokerHosts hosts, String topic)
   public KafkaConfig(BrokerHosts hosts, String topic, String clientId)
BrokerHosts就是上面创建的实例，topic就是要订阅的topic名称，clientId用于指定存放当前topic consumer的offset的位置，这个id 应该是唯一的，否则多个拓扑会引起冲突。
事实上，trident的offset并不保存在这个位置，见下面介绍。
真正使用时，有2种扩展，分别用于一般的storm以及trident。
（2）core storm
Spoutconfig is an extension of KafkaConfig that supports additional fields with ZooKeeper connection info and for controlling behavior specific to KafkaSpout. The Zkroot will be used as root to store your consumer's offset. The id should uniquely identify your spout.
public SpoutConfig(BrokerHosts hosts, String topic, String zkRoot, String id);
public SpoutConfig(BrokerHosts hosts, String topic, String id);
In addition to these parameters, SpoutConfig contains the following fields that control how KafkaSpout behaves:
``` 
// setting for how often to save the current kafka offset to ZooKeeper
    public long stateUpdateIntervalMs = 2000;

    // Exponential back-off retry settings.  These are used when retrying messages after a bolt
    // calls OutputCollector.fail().
    // Note: be sure to set backtype.storm.Config.MESSAGE_TIMEOUT_SECS appropriately to prevent
    // resubmitting the message while still retrying.
    public long retryInitialDelayMs = 0;
    public double retryDelayMultiplier = 1.0;
    public long retryDelayMaxMs = 60 * 1000;
```
 KafkaSpout 只接受 SpoutConfig作为参数

（3）TridentKafkaConfig，TridentKafkaEmitter只接受TridentKafkaConfig使用参数
trident消费kafka的offset位置是在建立拓扑中指定，如：
```
topology.newStream(test, kafkaSpout).
```
则offset的位置为：
```
/transactional/test/coordinator/currtx
```
（4）KafkaConfig的一些默认参数 
```
    public int fetchSizeBytes = 1024 * 1024;
    public int socketTimeoutMs = 10000;
    public int fetchMaxWait = 10000;
    public int bufferSizeBytes = 1024 * 1024;
    public MultiScheme scheme = new RawMultiScheme();
    public boolean forceFromStart = false;
    public long startOffsetTime = kafka.api.OffsetRequest.EarliestTime();
    public long maxOffsetBehind = Long.MAX_VALUE;
    public boolean useStartOffsetTimeIfOffsetOutOfRange = true;
    public int metricsTimeBucketSizeInSecs = 60;
```

 可以通过以下方式修改：
```
kafkaConfig.scheme =new SchemeAsMultiScheme(new StringScheme());
```
 
###3、设置MultiScheme
MultiScheme用于指定如何处理从kafka中读取到的字节，同时它用于控制输出字段名称。
```
 public Iterable<List<Object>> deserialize(byte[] ser);
 public Fields getOutputFields();
```
默认情况下，RawMultiScheme读取一个字段并返回一个字节，而发射的字段名称为bytes。
可以通过SchemeAsMultiScheme和 KeyValueSchemeAsMultiScheme改变这种默认行为：
```
 kafkaConfig.scheme =new SchemeAsMultiScheme(new StringScheme());
```
上面的语句指定了将字节转化为字符。
同时建立拓扑时：
```
topology.newStream(“test",kafkaSpout).each(new Fields("str"),new FilterFunction(),new Fields("word”))….
```
会指定发射的字段名称为str。
 
###4、创建Spout
(1)core storm
```
KafkaSpout kafkaSpout = new KafkaSpout(spoutConfig);
```
(2)trident
```
 OpaqueTridentKafkaSpoutkafkaSpout = new OpaqueTridentKafkaSpout(kafkaConfig);
```
###5、建立拓扑：
(1)core-storm
```
builder.setSpout("kafka-reader",new KafkaSpout(spoutConf),12);
```
kafka-reader指定了spout的名称，12指定了并行度。
 
(2)trident
```
topology.newStream(“test", kafkaSpout). each(new Fields("str"), new FilterFunction(),new Fields("word”))….
```
test指定了放置offset的位置，也就是txid的位置。str指定了spout发射字段的名称。
 
完整示例：
Core Spout
```
BrokerHosts hosts = new ZkHosts(zkConnString);
SpoutConfig spoutConfig = new SpoutConfig(hosts, topicName, "/" + topicName, UUID.randomUUID().toString());
spoutConfig.scheme = new SchemeAsMultiScheme(new StringScheme());
KafkaSpout kafkaSpout = new KafkaSpout(spoutConfig);
```
Trident Spout

```
TridentTopology topology = new TridentTopology();
BrokerHosts zk = new ZkHosts("localhost");
TridentKafkaConfig spoutConf = new TridentKafkaConfig(zk, "test-topic");
spoutConf.scheme = new SchemeAsMultiScheme(new StringScheme());
OpaqueTridentKafkaSpout spout = new OpaqueTridentKafkaSpout(spoutConf);
```

##（二）当拓扑出错时，如何从上一次的kafka位置继续处理消息
1、KafkaConfig有一个配置项为KafkaConfig.startOffsetTime，它用于指定拓扑从哪个位置上开始处理消息，可取的值有3个：
（1）kafka.api.OffsetRequest.EarliestTime(): 从最早的消息开始
（2）kafka.api.OffsetRequest.LatestTime(): 从最新的消息开始，即从队列队伍最末端开始。
（3）根据时间点: 
```
kafkaConfig.startOffsetTime =  new SimpleDateFormat("yyyy.MM.dd-HH:mm:ss").parse(startOffsetTime).getTime();
```
可以参阅 How do I accurately get offsets of messages for a certain timestamp using OffsetRequest? 的实现原理。
How do I accurately get offsets of messages for a certain timestamp using OffsetRequest?
 Kafka allows querying offsets of messages by time and it does so at segment granularity.The timestamp parameter is the unix timestamp and querying the offset by timestamp returns the latest possible offset of the message that is appended no later than the given timestamp. There are 2 special values of the timestamp - latest and earliest. For any other value of the unix timestamp, Kafka will get the starting offset of the log segment that is created no later than the given timestamp. Due to this, and since the offset request is served only at segment granularity, the offset fetch request returns less accurate results for larger segment sizes.
 For more accurate results, you may configure the log segment size based on time (log.roll.ms) instead of size (log.segment.bytes). However care should be taken since doing so might increase the number of file handlers due to frequent log segment rolling.
2、由于运行拓扑时，指定了offset在zk中保存的位置，当出现错误时，可以找出offset
当重新部署拓扑时，必须保证offset的保存位置不变，它才能正确的读取到offset。
（1）对于core storm,就是
```
SpoutConfigspoutConf = new SpoutConfig(brokerHosts,topic, zkRoot,id);
```
后2个参数不能变化
（2）对于trident而言，就是
```
topology.newStream(“test", kafkaSpout).
```
第1个参数不能变化。
 3、也就是说只要拓扑运行过一次KafkaConfig.startOffsetTime，之后重新部署时均可从offset中开始。
再看看这2个参数
```
   public booleanforceFromStart =false;
   public long startOffsetTime= kafka.api.OffsetRequest.EarliestTime();
```
如果将forceFromStart(旧版本是ignoreZkOffsets）设置为true，则每次拓扑重新启动时，都会从开头读取消息。
如果为false，则：
第一次启动，从开头读取，之后的重启均是从offset中读取。
一般使用时，将数值设置为以上2个即可。
 
 
##（三）结果写回kafka
 如果想把结果写回kafka，并保证事务性，可以使用 storm.kafka.trident.TridentState, storm.kafka.trident.TridentStateFactory and storm.kafka.trident.TridentKafkaUpdater.
 
以下是官方说明。
 Writing to Kafka as part of your topology
You can create an instance of storm.kafka.bolt.KafkaBolt and attach it as a component to your topology or if you are using trident you can use storm.kafka.trident.TridentState, storm.kafka.trident.TridentStateFactory and storm.kafka.trident.TridentKafkaUpdater.
You need to provide implementation of following 2 interfaces
 
TupleToKafkaMapper and TridentTupleToKafkaMapper
These interfaces have 2 methods defined:
```
    K getKeyFromTuple(Tuple/TridentTuple tuple);
    V getMessageFromTuple(Tuple/TridentTuple tuple);
```
as the name suggests these methods are called to map a tuple to kafka key and kafka message. If you just want one field as key and one field as value then you can use the provided FieldNameBasedTupleToKafkaMapper.java implementation. In the KafkaBolt, the implementation always looks for a field with field name "key" and "message" if you use the default constructor to construct FieldNameBasedTupleToKafkaMapper for backward compatibility reasons. Alternatively you could also specify a different key and message field by using the non default constructor. In the TridentKafkaState you must specify what is the field name for key and message as there is no default constructor. These should be specified while constructing and instance of FieldNameBasedTupleToKafkaMapper.
 
KafkaTopicSelector and trident KafkaTopicSelector
This interface has only one method
```
publicinterface KafkaTopicSelector {
    StringgetTopics(Tuple/TridentTuple tuple);
}
```
The implementation of this interface should return topic to which the tuple's key/message mapping needs to be published You can return a null and the message will be ignored. If you have one static topic name then you can use DefaultTopicSelector.java and set the name of the topic in the constructor.
 
Specifying kafka producer properties
You can provide all the produce properties , see http://kafka.apache.org/documentation.html#producerconfigs section "Important configuration properties for the producer", in your storm topology config by setting the properties map with key kafka.broker.properties.
 
 
附带2个官方的示例
For the bolt :
```        
TopologyBuilder builder = new TopologyBuilder();

        Fields fields = new Fields("key", "message");
        FixedBatchSpout spout = new FixedBatchSpout(fields, 4,
                    new Values("storm", "1"),
                    new Values("trident", "1"),
                    new Values("needs", "1"),
                    new Values("javadoc", "1")
        );
        spout.setCycle(true);
        builder.setSpout("spout", spout, 5);
        KafkaBolt bolt = new KafkaBolt()
                .withTopicSelector(new DefaultTopicSelector("test"))
                .withTupleToKafkaMapper(new FieldNameBasedTupleToKafkaMapper());
        builder.setBolt("forwardToKafka", bolt, 8).shuffleGrouping("spout");

        Config conf = new Config();
        //set producer properties.
        Properties props = new Properties();
        props.put("metadata.broker.list", "localhost:9092");
        props.put("request.required.acks", "1");
        props.put("serializer.class", "kafka.serializer.StringEncoder");
        conf.put(KafkaBolt.KAFKA_BROKER_PROPERTIES, props);

        StormSubmitter.submitTopology("kafkaboltTest", conf, builder.createTopology());
```

 
For Trident:
```      
Fields fields = new Fields("word", "count");
        FixedBatchSpout spout = new FixedBatchSpout(fields, 4,
                new Values("storm", "1"),
                new Values("trident", "1"),
                new Values("needs", "1"),
                new Values("javadoc", "1")
        );
        spout.setCycle(true);

        TridentTopology topology = new TridentTopology();
        Stream stream = topology.newStream("spout1", spout);

        TridentKafkaStateFactory stateFactory = new TridentKafkaStateFactory()
                .withKafkaTopicSelector(new DefaultTopicSelector("test"))
                .withTridentTupleToKafkaMapper(new FieldNameBasedTupleToKafkaMapper("word", "count"));
        stream.partitionPersist(stateFactory, fields, new TridentKafkaUpdater(), new Fields());

        Config conf = new Config();
        //set producer properties.
        Properties props = new Properties();
        props.put("metadata.broker.list", "localhost:9092");
        props.put("request.required.acks", "1");
        props.put("serializer.class", "kafka.serializer.StringEncoder");
        conf.put(TridentKafkaState.KAFKA_BROKER_PROPERTIES, props);
        StormSubmitter.submitTopology("kafkaTridentTest", conf, topology.build());
``` 
 
#二、完整示例
##（一）简介
1、本项目主要完成以下功能：
（1）从kafka中读取一个topic的消息，然后根据空格拆分单词，最后统计数据后写入一个HazelCastState（一个分布式的内存存储框架）。
（2）通过DRPC从上述的HazelCastState中读取结果，并将结果输出。
2、代码可分为3部分：
（1）单词拆分
（2）定义拓扑行为
（3）state定义
以下分为三部分分别介绍。
 
##（二）单词拆分
原理很简单，就是通过空格将单词进行拆分。
```
public class WordSplit extends BaseFunction {
    public void execute(TridentTuple tuple, TridentCollector collector) {
        String sentence = (String) tuple.getValue(0);
        if (sentence != null) {
            sentence = sentence.replaceAll("\r", "");
            sentence = sentence.replaceAll("\n", "");
            for (String word : sentence.split(" ")) {
                collector.emit(new Values(word));
            }
        }
    }
}
```
这里的wordsplit是一个function，它继承自BaseFunction，最后，它将拆分出来的单词逐个emit出去。

  
##（三）定义拓扑行为
 
###1、定义kafka的相关配置
```    
 TridentKafkaConfig kafkaConfig = new TridentKafkaConfig(brokerHosts, "storm-sentence", "storm");
        kafkaConfig.scheme = new SchemeAsMultiScheme(new StringScheme());
        TransactionalTridentKafkaSpout kafkaSpout = new TransactionalTridentKafkaSpout(kafkaConfig); 
```
（1）首先定义一个kafka相关的配置对象，第一个参数是zookeeper的位置，第二个参数是订阅topic的名称，第三个参数是一个clientId
（2）然后对配置进行一些设置，包括一些起始位置之类的，后面再补充具体的配置介绍。
（3）创建一个spout，这里的spout是事务型的，也就是保证每一个仅且只被处理一个

###2、定义拓扑，进行单词统计后，写入一个分布式内存中。
```
TridentTopology topology= new TridentTopology();
TridentState wordCounts = topology.newStream("kafka", kafkaSpout).shuffle().
                each(new Fields("str"), new WordSplit(), new Fields("word")).
                groupBy(new Fields("word")).
                persistentAggregate(new HazelCastStateFactory(), new Count(), new Fields("aggregates_words")).parallelismHint(2);
```
（1）创建一个topo。

（2）首先定义一个输入流，其中第一个参数定义了zk中放置这个topo元信息的信息，一般是/transactional/kafka
（3）对每个输入的消息进行拆分：首先它的输入是字段名称为str的消息，然后经过WordSplit这个Function处理，最后，以字段名称word发送出去
（4）将结果根据word字段的值进行分组，就是说word值相同的放在一起。
（5）将分组的结果分别count一下，然后以字段名称aggregates_words写入HazelCastStateFactory定义的state中，关于state请见下一部分的介绍。
 
###3、从分布式内存中读取结果并进行输出
```
topology.newDRPCStream("words", drpc)
        .each(new Fields("args"), new Split(), new Fields("word"))
        .groupBy(new Fields("word"))
        .stateQuery(wordCounts, new Fields("word"), new MapGet(), new Fields("count"))
        .each(new Fields("count"), new FilterNull())
        .aggregate(new Fields("count"), new Sum(), new Fields("sum"));
```
（1）第三行定义了使用drpc需要处理的内容
（2）查询分布式内存中的内容，查询字段为word，然后以字段名count发送出去。
（3）将不需要统计的过滤掉。
（4）将结果进行聚合。
 
4、主函数
```
String kafkaZk = args[0];
 SentenceAggregationTopology sentenceAggregationTopology = new SentenceAggregationTopology(kafkaZk);
        Config config = new Config();
        config.put(Config.TOPOLOGY_TRIDENT_BATCH_EMIT_INTERVAL_MILLIS, 2000);

        if (args != null && args.length > 1) {
            String name = args[1];
            String dockerIp = args[2];
            config.setNumWorkers(2);
            config.setMaxTaskParallelism(5);
            config.put(Config.NIMBUS_HOST, dockerIp);
            config.put(Config.NIMBUS_THRIFT_PORT, 6627);
            config.put(Config.STORM_ZOOKEEPER_PORT, 2181);
            config.put(Config.STORM_ZOOKEEPER_SERVERS, Arrays.asList(dockerIp));
            StormSubmitter.submitTopology(name, config, sentenceAggregationTopology.buildTopology());
        } else {
            LocalDRPC drpc = new LocalDRPC();
            config.setNumWorkers(2);
            config.setMaxTaskParallelism(2);
            LocalCluster cluster = new LocalCluster();
            cluster.submitTopology("kafka", config, sentenceAggregationTopology.buildTopology(drpc));
            while (true) {
                System.out.println("Word count: " + drpc.execute("words", "the"));
                Utils.sleep(1000);
            }

        }
```
三个参数的含义为： 

  /*args[0]:kafkazk，如：192.168.172.98:2181,192.168.172.111:2181,192.168.172.114:2181,192.168.172.116:2181,192.168.172.117:2181/kafka
     * args[1]:topo名称
     * args[2]:niubus节点，如,192.168.172.98
     */
当参数数据大于1时，将拓扑提交到集群中，否则提交到本地。提交拓扑到集群的比较直观，下面郑重介绍一下drpc的查询。
（1）首先定义一个本地的drpc对象，以及一个本地storm集群。
（2）然后将拓扑群提交到本地集群。
（3）最后，使用drpuc不停的循环查询统计结果并输出。
 
注意上面的拓扑定义了2个流，第一个流用于接收kafka消息，然后拆分统计后写入内存，第二个流则接受drpc的输入，将drpc的输入拆分后，再统计需要查询的每个单词的统计结果。如在本例中，需要显示单词the的数量。
在本例中，drpc和kafka没有本质的区别，它们都是一个用于向storm发送消息的集群，只是输入数据的方式有些不同，kafka通过spout输入，drpc则直接通过execute()进行输入。
 
运行方式：
方式一：直接在eclipse右键运行，参数只填一个，如192.168.172.98:2181,192.168.172.111:2181,192.168.172.114:2181,192.168.172.116:2181,192.168.172.117:2181/kafka。只要保证kafka集群中有对应的topic，则会得到以下输出：
 
Word count: [[2]]
Word count: [[5]]
Word count: [[10]]
Word count: [[17]]
Word count: [[28]]
当然，统计结果根据输入kafka的内容而不同。
 

 
##（四）state定义
在定义拓扑的时候，最终的wordcount结果写在了HazelCastState中：
persistentAggregate(new HazelCastStateFactory(),new Count(),new Fields("aggregates_words"))
下面我们分析一下如何使用state来保存topo的处理结果，或者是中间处理结果。
注意，使用state除了可以保存最终的结果输出，以保证事务型、透明事务型以外，还经常用于保存中间结果。比如blueprint第3章的一个例子中，用于统计疾病的发生数量，如果超过预警值，则向外发信息。如果统计结果成功，但向外发送信息失败，则spout会重发数据，导致统计结果有误，因此，此时可以通过state将结果保存下来。
 
###1、Factory类
```
public class HazelCastStateFactory implements StateFactory {
    @Override
    public State makeState(Map conf, IMetricsContext metrics, int partitionIndex, int numPartitions) {
        return TransactionalMap.build(new HazelCastState(new HazelCastHandler()));
    }
}
```
内容很简单，就是返回一个state，它也是三个state相关的类中唯一对外的接口。
 
2、Handler类
```
public class HazelCastHandler implements Serializable {
    private transient Map<String, Long> state;
    public Map<String, Long> getState() {
        if (state == null) {
            state = Hazelcast.newHazelcastInstance().getMap("state");
        }
        return state;
    }
```

使用单例模式返回一个map。

 
###3、State类
真正处理业务逻辑的类。主要的方法有mutiPut和mutiGet，用于将结果放入state与取出state。
 
```
public class HazelCastState<T> implements IBackingMap<TransactionalValue<Long>> {

    private HazelCastHandler handler;


    public HazelCastState(HazelCastHandler handler) {
        this.handler = handler;
    }

    public void addKeyValue(String key, Long value) {
        Map<String, Long> state = handler.getState();
        state.put(key, value);
    }

    @Override
    public String toString() {
        return handler.getState().toString();
    }


    @Override
    public void multiPut(List<List<Object>> keys, List<TransactionalValue<Long>> vals) {
        for (int i = 0; i < keys.size(); i++) {
            TridentTuple key = (TridentTuple) keys.get(i);
            Long value = vals.get(i).getVal();
            addKeyValue(key.getString(0), value);
            //System.out.println("[" + key.getString(0) + " - " + value + "]");
        }
    }


    public List multiGet(List<List<Object>> keys) {
        List<TransactionalValue<Long>> result = new ArrayList<TransactionalValue<Long>>(keys.size());
        for (int i = 0; i < keys.size(); i++) {
            TridentTuple key = (TridentTuple) keys.get(i);
            result.add(new TransactionalValue<Long>(0L, MapUtils.getLong(handler.getState(), key.getString(0), 0L)));
        }
        return result;
    }
}
```




