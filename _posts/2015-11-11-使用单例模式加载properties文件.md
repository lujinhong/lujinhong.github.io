---
layout: post
tile:  "使用单例模式加载properties文件"
date:  2015-11-11 19:00:28
categories: java 
excerpt: 使用单例模式加载properties文件
---

* content
{:toc}



 

 

##先准备测试程序

	package org.jediael.util;
	import static org.junit.Assert.*;
	import org.junit.Test;
	
	public class BasicConfigurationTest {
		@Test
		public void testGetValue(){
			BasicConfiguration configuration = BasicConfiguration.getInstance();
			assertTrue(configuration.getValue("key").equals("value"));
		}
	}


其中properties文件中有一行如下：

	key=value

优先选择方案三

 

##方式一：懒汉方式

到第一次使用实例时，才加载实例
	
	package org.jediael.util;
	
	import java.io.FileInputStream;
	import java.io.FileNotFoundException;
	import java.io.IOException;
	import java.io.InputStream;
	import java.util.Properties;
	
	public class BasicConfiguration {
	
		private static BasicConfiguration configuration = null;
		private Properties pros = null;
		
		public static synchronized BasicConfiguration getInstance(){
			if(configuration == null){
				configuration = new BasicConfiguration();
			}
			return configuration;
		}
		
		public String getValue(String key){
			return pros.getProperty(key);
		}
		
		private BasicConfiguration(){
			readConfig();
		}
	
		private void readConfig() {
			pros = new Properties();
			InputStream in = null;
			try {
				in = new FileInputStream(Thread.currentThread().getContextClassLoader().getResource("")
						.getPath() + "search.properties");
				pros.load(in);
			} catch (FileNotFoundException e) {
				e.printStackTrace();
			}catch (IOException e) {
				e.printStackTrace();
			}finally{
				try {
					in.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
		}
	}

上述程序中，产生了BasicConfiguration的一个单例。

好处是只有到第一次调用getInstance才生成对象，节省了空间。不足之处在于同步锁导致有可能执行过慢。

##2、饿汉方式
	
	package org.jediael.util;
	
	import java.io.FileInputStream;
	import java.io.FileNotFoundException;
	import java.io.IOException;
	import java.io.InputStream;
	import java.util.Properties;
	
	public class BasicConfiguration {
	
		private static BasicConfiguration configuration = new BasicConfiguration();
		private Properties pros = null;
		
		public static BasicConfiguration getInstance(){
			return configuration;
		}
		
		public String getValue(String key){
			return pros.getProperty(key);
		}
		
		private BasicConfiguration(){
			readConfig();
		}
	
		private void readConfig() {
			pros = new Properties();
			InputStream in = null;
			try {
				in = new FileInputStream(Thread.currentThread().getContextClassLoader().getResource("")
						.getPath() + "search.properties");
				pros.load(in);
			} catch (FileNotFoundException e) {
				e.printStackTrace();
			}catch (IOException e) {
				e.printStackTrace();
			}finally{
				try {
					in.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
		}
	}

由于BasicConfiguration的实例是static，因此，当类被加载时就会初始化，但这样即使并不需要使用此实例，也会被初始化，导致内存空间的浪费。

 


##方式三：内部类方式

由于初始化放在内部类中，只有当此内部类被使用时，才会进行初始化。从而既节省了空间，也无需同步代码。 
	
	package org.jediael.util;
	
	import java.io.FileInputStream;
	import java.io.FileNotFoundException;
	import java.io.IOException;
	import java.io.InputStream;
	import java.util.Properties;
	
	public class BasicConfiguration {
	
		
		private Properties pros = null;
		
		private static class ConfigurationHolder{
			private static BasicConfiguration configuration = new BasicConfiguration();
		}
		
		public static BasicConfiguration getInstance(){
			return ConfigurationHolder.configuration;
		}
		
		public String getValue(String key){
			return pros.getProperty(key);
		}
		
		private BasicConfiguration(){
			readConfig();
		}
		
	
	
		private void readConfig() {
			pros = new Properties();
			InputStream in = null;
			try {
				in = new FileInputStream(Thread.currentThread().getContextClassLoader().getResource("")
						.getPath() + "search.properties");
				pros.load(in);
			} catch (FileNotFoundException e) {
				e.printStackTrace();
			}catch (IOException e) {
				e.printStackTrace();
			}finally{
				try {
					in.close();
				} catch (IOException e) {
					e.printStackTrace();
				}
			}
		}
	}
