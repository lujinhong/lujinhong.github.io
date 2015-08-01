---
layout: post
title:  "使用java向rsyslog发送日志"
date:   2015-08-03 14:06:05
categories: java linux
excerpt: 使用java向rsyslog发送日志
---

* content
{:toc}

#使用java向rsyslog发送日志

#补充rsyslog的配置方法
#（一）基本使用方法
1、开通rsyslog远程UDP访问

 vi /etc/rsyslog.conf  
 
将下面两段前面的#号去掉

	#$ModLoad imudp  
	#$UDPServerRun 514  

如果原来没有话则加上这2行

2，建立存放log的文件地址
 vi /etc/rsyslog.conf  
加入以下两段

	local2.info       /var/log/login_info.log  
	local2.debug       /var/log/login_debug.log  


3，开通防火墙
将UDP端口514对外开放出来
 
4，重启rsyslog
	 service rsyslog restart  


5，log4j的配置
将日志同时输出到syslog和console

	log4j.rootLogger =ALL,systemOut,SYSLOG
	log4j.appender.systemOut = org.apache.log4j.ConsoleAppender 
	log4j.appender.systemOut.layout = org.apache.log4j.PatternLayout 
	log4j.appender.systemOut.layout.ConversionPattern = [%-5p][%-22d{yyyy/MM/dd HH:mm:ssS}][%l]%n%m%n 
	log4j.appender.systemOut.Threshold = DEBUG 
	log4j.appender.systemOut.ImmediateFlush = TRUE 
	log4j.appender.systemOut.Target = System.out 
	log4j.appender.SYSLOG=org.apache.log4j.net.SyslogAppender  
	log4j.appender.SYSLOG.syslogHost=192.168.172.114
	log4j.appender.syslog.Threshold=DEBUG  
	log4j.appender.SYSLOG.layout=org.apache.log4j.PatternLayout  
	log4j.appender.SYSLOG.layout.ConversionPattern=%-4r [%t] %-5p %c %x - %m%n  
	log4j.appender.SYSLOG.Header=true
	log4j.appender.SYSLOG.Facility=local2   


6、写java代码

	package com.lujinhong.demo.log4j;
	
	import org.apache.log4j.Logger;
	
	public class Log4jDemo {
	    private static Logger LOG  =  Logger.getLogger(Log4jDemo. class );
	
	  public static void main(String[] args) {
	    //System.setProperty("LOGDIR","/Users/liaoliuqing/Downloads/");
	    LOG.info("lujinhong !!INFO MESSAGE!!");
	    LOG.error("ERROR MESSAGE!!!!");
	    LOG.debug("DEBUG MESSAGE!");
	    LOG.warn("WARN MESSAGE!!");
	    
	    new MyJavaClass().adder(2, 3);
	  }
	}


7，运行应用程序，可以在/var/log下看到不同级别的日志信息，如login_info.log和login_debug.log

#（二）不使用配置文件
有时候，log4j的配置文件会互相覆盖，真的很烦，因此可以将配置写到代码中去
1、定义一个单例类，将相关配置写入：

	package com.lujinhong.demo.log4j;
	
	import org.apache.log4j.Level;
	import org.apache.log4j.Logger;
	import org.apache.log4j.PatternLayout;
	import org.apache.log4j.net.SyslogAppender;
	
	
	public class MySysLogger {
		//日志级别
		private static Level level = Level.INFO;
	
	
		private static Logger LOG = null;
	
		public static synchronized Logger getInstance() {
			if (LOG == null) {
				new MySysLogger();
			}
			return LOG;
		}
	
		private MySysLogger() {
	
			LOG = Logger.getRootLogger();
			
			LOG.setLevel(level);
			SyslogAppender appender = new SyslogAppender();
			appender.setName("ma30");
	
			appender.setSyslogHost("123.58.172.98");
			appender.setLayout(new PatternLayout(
					"%r [%t] %-5p %c%x: - %m%n"));
			appender.setHeader(true);
			appender.setFacility("local7");
			//appender.setWriter(new PrintWriter(System.out));
			//如果是文件是RollingFileAppender:setWriter(new PrintWriter(new File("F:/test/_debug.log")));
			LOG.addAppender(appender);
		}
	}

2、在需要日志输出的类中添加以下语句

	private static Logger LOG = MySysLogger.getInstance();

然后就可以使用`LOG`变量进行日志输出了

吐槽几句，log4j的坑啊....
（1）CLASSPATH中不能有多个log4j的版本本，否则有有奇形怪状的NoSuchMethod, NoSuchFiled， NoClassDefineFound等异常。明明是太多了，还告诉你没有
（2）与slf4j的搭建，必须版本一致，如slf4j-1.7.2对应log4j-1.2.17
（3）配置文件啊，如果你引用的第三方包有log4j.properties，而又没有提供给你编辑，那恭喜你，慢慢调吧。把log4j的配置写入代码吧，不要用配置文件了