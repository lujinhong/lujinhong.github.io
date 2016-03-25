---
layout: post
tile:  "storm hook的使用"
date:  2015-11-05 12:49:52
categories: storm 
excerpt: storm hook的使用
---

* content
{:toc}



##（一）原理

1、先看一下storm的hook是什么东西： http://storm.apache.org/documentation/Hooks.html

Storm provides hooks with which you can insert custom code to run on any number of events within Storm. You create a hook by extending the BaseTaskHook class and overriding the appropriate method for the event you want to catch. There are two ways to register your hook:
In the open method of your spout or prepare method of your bolt using the TopologyContext method.
Through the Storm configuration using the "topology.auto.task.hooks" config. These hooks are automatically registered in every spout or bolt, and are useful for doing things like integrating with a custom monitoring system.

2、storm的hook也是一个典型的钩子，当某些事情发生时(比如说执行execute方法，执行ack方法时)，相应的代码会自动被调用。

3、通过继承BaseTaskHook，并覆盖其方法来创建一个hook。

4、如何把hook注册到一个拓扑中，有2种方法：
（1）在spout的open方法或者bolt的prepare方法中调用：
TopologyContext.addTaskHook(new **Hook())
（2）在storm的配置文件中修改topology.auto.task.hooks项，这会自己注册到每一个spout和bolt。这种情况对于一些集成应用或者监控之类的有用。

##（二）入门例子

1、先创建hook
这个hook很简单，就是当execute或者ack方法被调用时，将相应的信息打印出来：
	
	package com.lujinhong.demo.storm.hook;
	
	import backtype.storm.hooks.BaseTaskHook;
	import backtype.storm.hooks.info.BoltAckInfo;
	import backtype.storm.hooks.info.BoltExecuteInfo;
	
	public class TraceTaskHook extends BaseTaskHook {
		
		@Override
		public void boltExecute(BoltExecuteInfo info) {
			super.boltExecute(info);
			System.out.println("executingTaskId:" + info.executingTaskId);
			System.out.println("executedLatencyMs:" + info.executeLatencyMs);
			System.out.println("execute msg:" + info.tuple.getString(0));
		}
		
		@Override
		public void boltAck(BoltAckInfo info) {
			super.boltAck(info);
			System.out.println("ackingTaskId:" + info.ackingTaskId);
			System.out.println("processLatencyMs:" + info.processLatencyMs);
			System.out.println("ack msg:" + info.tuple.getString(0));
		}
	
	}

2、创建topo，并且在topo中注册钩子

（1）拓扑很简单，就是storm-starter中的ExlamationTopoloty，它的spout会随机发送名字，然后经过2个bolt，每个bolt均在后面加!!!，最后10S后将拓扑kill掉。
（2）然后在bolt中定义hook：

    @Override
    public void prepare(Map conf, TopologyContext context, OutputCollector collector) {
      _collector = collector;
      context.addTaskHook(new TraceTaskHook());
    }

完整代码如下：

	
	package com.lujinhong.demo.storm.hook;
	
	import backtype.storm.Config;
	import backtype.storm.LocalCluster;
	import backtype.storm.StormSubmitter;
	import backtype.storm.task.OutputCollector;
	import backtype.storm.task.TopologyContext;
	import backtype.storm.testing.TestWordSpout;
	import backtype.storm.topology.OutputFieldsDeclarer;
	import backtype.storm.topology.TopologyBuilder;
	import backtype.storm.topology.base.BaseRichBolt;
	import backtype.storm.tuple.Fields;
	import backtype.storm.tuple.Tuple;
	import backtype.storm.tuple.Values;
	import backtype.storm.utils.Utils;
	
	import java.util.Map;
	
	public class ExclamationTopology {
	
	  public static class ExclamationBolt extends BaseRichBolt {
	    OutputCollector _collector;
	
	    @Override
	    public void prepare(Map conf, TopologyContext context, OutputCollector collector) {
	      _collector = collector;
	      context.addTaskHook(new TraceTaskHook());
	    }
	
	    @Override
	    public void execute(Tuple tuple) {
	      _collector.emit(tuple, new Values(tuple.getString(0) + "!!!"));
	      _collector.ack(tuple);
	    }
	
	    @Override
	    public void declareOutputFields(OutputFieldsDeclarer declarer) {
	      declarer.declare(new Fields("word"));
	    }
	
	
	  }
	
	  public static void main(String[] args) throws Exception {
	    TopologyBuilder builder = new TopologyBuilder();
	
	    builder.setSpout("word", new TestWordSpout(), 10);
	    builder.setBolt("exclaim1", new ExclamationBolt(), 3).shuffleGrouping("word");
	    builder.setBolt("exclaim2", new ExclamationBolt(), 2).shuffleGrouping("exclaim1");
	
	    Config conf = new Config();
	    conf.setDebug(true);
	
	    if (args != null && args.length > 0) {
	      conf.setNumWorkers(3);
	
	      StormSubmitter.submitTopologyWithProgressBar(args[0], conf, builder.createTopology());
	    }
	    else {
	
	      LocalCluster cluster = new LocalCluster();
	      cluster.submitTopology("test", conf, builder.createTopology());
	      Utils.sleep(10000);
	      cluster.killTopology("test");
	      cluster.shutdown();
	    }
	  }
	}

3、运行结果
类似于以下的内容
	
	ackingTaskId:5
	processLatencyMs:null
	ack msg:nathan!!!
	executingTaskId:5
	executedLatencyMs:null
	execute msg:nathan!!!

#（三）hook的类型
	
storm在BaseTaskHook中支持的类型可见下面的类定义。一般而言，就是在topo发生某项事件（如发射，确认，cleanup等）执行一些操作

	public class BaseTaskHook implements ITaskHook {
	    @Override
	    public void prepare(Map conf, TopologyContext context) {
	    }
	
	    @Override
	    public void cleanup() {
	    }    
	
	    @Override
	    public void emit(EmitInfo info) {
	    }
	
	    @Override
	    public void spoutAck(SpoutAckInfo info) {
	    }
	
	    @Override
	    public void spoutFail(SpoutFailInfo info) {
	    }
	
	    @Override
	    public void boltAck(BoltAckInfo info) {
	    }
	
	    @Override
	    public void boltFail(BoltFailInfo info) {
	    }
	
	    @Override
	    public void boltExecute(BoltExecuteInfo info) {
	    }
	}

##（四）应用场景

1、可以用于调试代码，比如确认某个方法（execute, prepare等）是否被调用

2、用于将topo中的一些信息发送出去，可以作为监控，日志等。
