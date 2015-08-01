---
layout: post
title:  "storm编程指南"
date:   2015-08-01 10:06:05
categories: Front-end java
excerpt: storm编程指南
---

* content
{:toc}

本文介绍了storm的基本编程，关于trident的编程，请见？？？

本示例使用storm运行经典的wordcount程序，拓扑如下：
sentence-spout—>split-bolt—>count-bolt—>report-bolt
分别完成句子的产生、拆分出单词、单词数量统计、统计结果输出
完整代码请见 https://github.com/jinhong-lu/stormdemo
以下是关键代码的分析。

##（一）创建spout

```
public class SentenceSpout extends BaseRichSpout {
    private SpoutOutputCollector collector;
    private int index = 0;
    private String[] sentences = { "when i was young i'd listen to the radio",
            "waiting for my favorite songs", "when they played i'd sing along",
            "it make me smile",
            "those were such happy times and not so long ago",
            "how i wondered where they'd gone",
            "but they're back again just like a long lost friend",
            "all the songs i love so well", "every shalala every wo'wo",
            "still shines.", "every shing-a-ling-a-ling",
            "that they're starting", "to sing so fine"};

    public void open(Map conf, TopologyContext context,
            SpoutOutputCollector collector) {
        this.collector = collector;
    }
    public void declareOutputFields(OutputFieldsDeclarer declarer) {
        declarer.declare(new Fields("sentence"));
    }
    public void nextTuple() {
        this.collector.emit(new Values(sentences[index]));
        index++;
        if (index >= sentences.length) {
            index = 0;
        }
        try {
            Thread.sleep(1);
        } catch (InterruptedException e) {
            //e.printStackTrace();
        }
    }
}
```

上述类中，将string数组中内容逐行发送出去，主要的方法有：
（1）open()方法完成spout的初始化工作，与bolt的prepare()方法类似
（2）declareOutputFileds()定义了发送内容的字段名称与字段数量，bolt中的方法名称一样。
（3）nextTuple()方法是对每一个需要处理的数据均会执行的操作，也bolt的executor()方法类似。它是整个逻辑处理的核心，通过emit()方法将数据发送到拓扑中的下一个节点。

##（二）创建split-bol

```
public class SplitSentenceBolt extends BaseRichBolt{
    private OutputCollector collector;

    public void prepare(Map stormConf, TopologyContext context,
            OutputCollector collector) {
        this.collector = collector;
    }
    
    public void declareOutputFields(OutputFieldsDeclarer declarer) {
        declarer.declare(new Fields("word"));
    }

    public void execute(Tuple input) {
        String sentence = input.getStringByField("sentence");
        String[] words = sentence.split(" ");
        for(String word : words){
            this.collector.emit(new Values(word));
            //System.out.println(word);
        }
    }
}
```

三个方法的含义与spout类似，这个类根据空格把收到的句子进行拆分，拆成一个一个的单词，然后把单词逐个发送出去。
input.getStringByField("sentence”)可以根据上一节点发送的关键字获取到相应的内容。

##（三）创建wordcount-bolt

```
public class WordCountBolt extends BaseRichBolt{
    private OutputCollector collector;
    private Map<String,Long> counts = null;

    public void prepare(Map stormConf, TopologyContext context,
            OutputCollector collector) {
        this.collector = collector;
        this.counts = new HashMap<String, Long>();
    }
    public void declareOutputFields(OutputFieldsDeclarer declarer) {
        declarer.declare(new Fields("word","count"));
    }

    public void execute(Tuple input) {
        String word = input.getStringByField("word");
        Long count = this.counts.get(word);
        if(count == null){
            count = 0L;
        }
        count++;
        this.counts.put(word, count);
        this.collector.emit(new Values(word,count));
        //System.out.println(count);
    }
}
```

本类将接收到的word进行数量统计，并把结果发送出去。
这个bolt发送了2个filed：

```
        declarer.declare(new Fields("word","count"));
        this.collector.emit(new Values(word,count));
```

##（四）创建report-bolt

```
public class ReportBolt extends BaseRichBolt{
    private Map<String, Long> counts;
    public void prepare(Map stormConf, TopologyContext context,
            OutputCollector collector) {
        this.counts = new HashMap<String,Long>();
    }
    public void declareOutputFields(OutputFieldsDeclarer declarer) {
    }
    public void execute(Tuple input) {
        String word = input.getStringByField("word");
        Long count = input.getLongByField("count");
        counts.put(word, count);
    }
    public void cleanup() {
        System.out.println("Final output");
        Iterator<Entry<String, Long>> iter = counts.entrySet().iterator();
        while (iter.hasNext()) {
            Entry<String, Long> entry = iter.next();
            String word = (String) entry.getKey();
            Long count = (Long) entry.getValue();
            System.out.println(word + " : " + count);
        }
        
        super.cleanup();
    }    
    
}
```


本类将从wordcount-bolt接收到的数据进行输出。
先将结果放到一个map中，当topo被关闭时，会调用cleanup()方法，此时将map中的内容输出。

##（五）创建topo

```
public class WordCountTopology {
    private static final String SENTENCE_SPOUT_ID = "sentence-spout";
    private static final String SPLIT_BOLT_ID = "split-bolt";
    private static final String COUNT_BOLT_ID = "count-bolt";
    private static final String REPORT_BOLT_ID = "report-bolt";
    private static final String TOPOLOGY_NAME = "word-count-topology";

    public static void main(String[] args) {
        SentenceSpout spout = new SentenceSpout();
        SplitSentenceBolt splitBolt = new SplitSentenceBolt();
        WordCountBolt countBolt = new WordCountBolt();
        ReportBolt reportBolt = new ReportBolt();
        TopologyBuilder builder = new TopologyBuilder();
        builder.setSpout(SENTENCE_SPOUT_ID, spout);
        builder.setBolt(SPLIT_BOLT_ID, splitBolt).shuffleGrouping(
                SENTENCE_SPOUT_ID);
        builder.setBolt(COUNT_BOLT_ID, countBolt).fieldsGrouping(SPLIT_BOLT_ID,
                new Fields("word"));
        builder.setBolt(REPORT_BOLT_ID, reportBolt).globalGrouping(
                COUNT_BOLT_ID);
        Config conf = new Config();
        if (args.length == 0) {
            LocalCluster cluster = new LocalCluster();
            cluster.submitTopology(TOPOLOGY_NAME, conf,
                    builder.createTopology());
            try {
                Thread.sleep(10000);
            } catch (InterruptedException e) {
            }
            cluster.killTopology(TOPOLOGY_NAME);
            cluster.shutdown();
        } else {
            try {
                StormSubmitter.submitTopology(args[0], conf,builder.createTopology());
            } catch (AlreadyAliveException e) {
                e.printStackTrace();
            } catch (InvalidTopologyException e) {
                e.printStackTrace();
            }
        }
    }
}
```

关键步骤为：
（1）创建TopologyBuilder，并为这个builder指定spout与bolt

```
        builder.setSpout(SENTENCE_SPOUT_ID, spout);
        builder.setBolt(SPLIT_BOLT_ID, splitBolt).shuffleGrouping(
                SENTENCE_SPOUT_ID);
        builder.setBolt(COUNT_BOLT_ID, countBolt).fieldsGrouping(SPLIT_BOLT_ID,
                new Fields("word"));
        builder.setBolt(REPORT_BOLT_ID, reportBolt).globalGrouping(
                COUNT_BOLT_ID);
```

(2)创建conf对象

```
    Config conf = new Config();
```

这个对象用于指定一些与拓扑相关的属性，如并行度、nimbus地址等。
(3)创建并运行拓扑，这里使用了2种方式
一是当没有参数时，建立一个localcluster，在本地上直接运行，运行10秒后，关闭集群：

```
LocalCluster cluster = new LocalCluster();
cluster.submitTopology(TOPOLOGY_NAME, conf,builder.createTopology());
Thread.sleep(10000);
cluster.killTopology(TOPOLOGY_NAME);
cluster.shutdown();
```

二是有参数是，将拓扑提交到集群中：

```
StormSubmitter.submitTopology(args[0], conf,builder.createTopology());
```

第一个参数为拓扑的名称。

6、本地运行
直接在eclipse中运行即可，输出结果在console中看到

7、集群运行
（1）编译并打包

```
mvn clean compile
```

（2）把编译好的jar包上传到nimbus机器上，然后

```
storm jar com.ljh.storm.5_stormdemo  com.ljh.storm.wordcount.WordCountTopology  topology_name
```

将拓扑提交到集群中。

#（六）一些说明
1、关于分布式编程的一点说明


在分布式系统中，由于有多个机器、多个进程在同时运行程序，而它们之间由于运行在不同的JVM中，因此它们之间的变量是无法共享的。

以storm为例：
如果在主程序中设置了某个变量，如

```
 topoName = args[0];
```

在bolt中想要取得这个变量是不可能的，因为这个变量只保存在了当前的JVM中。
因此，如果在bolt中也要使用这个变量，则必须将其放入一个由分布式系统提供的共享参数中，如：

```
config.put(Config.TOPOLOGY_NAME, topologyName);
```

然后，在bolt中的prepare()方法中取得这个参数：

```
	@Override
	public void prepare(Map conf, TridentOperationContext context) {
        String topoName = (String) conf.get(Config.TOPOLOGY_NAME);
   }
```

其它的分布式系统也类似，切记，不要以为在main函数定义了一个参数，就可以在任何地方使用，它只能在本JVM内使用！！！
