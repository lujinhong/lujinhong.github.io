---
layout: post
tile:  "hadoop API之：文件操作"
date:  2015-11-13 15:10:55
categories: hadoop 
excerpt: hadoop API之：文件操作
---

* content
{:toc}



[TOC]

Hadoop提供了大量的API对文件系统中的文件进行操作，主要包括：

（1)读取文件

（2)写文件

（3)读取文件属性

（4)列出文件

（5)删除文件


##1､读取文件

以下示例中，将hdfs中的一个文件读取出来，并输出到标准输出流中。

	
	package org.jediael.hadoopdemo.fsdemo;
	
	import java.io.IOException;
	import java.net.URI;
	
	import org.apache.hadoop.conf.Configuration;
	import org.apache.hadoop.fs.FSDataInputStream;
	import org.apache.hadoop.fs.FileSystem;
	import org.apache.hadoop.fs.Path;
	import org.apache.hadoop.io.IOUtils;
	
	public class FileSystemDoubleCat {
	
		public static void main(String[] args) throws IOException {
	
			String fileName = args[0];
			Configuration conf = new Configuration();
	
			FileSystem fs = FileSystem.get(URI.create(fileName), conf);
			FSDataInputStream in = null;
			try {
				in = fs.open(new Path(fileName));
				IOUtils.copyBytes(in, System.out, 4096, false);
				in.seek(0);
				IOUtils.copyBytes(in, System.out, 4096, false);
			} finally {
				in.close();
			}
	
		}
	
	}

（1)其中FSDataInputStream实现了Seekable接口，可以对文件进行随机定位，但注意，seek()的代价较高，如无必要，尽量少使用。

##2､文件复制


	package org.jediael.hadoopdemo.fsdemo;
	
	import java.io.BufferedInputStream;
	import java.io.FileInputStream;
	import java.io.IOException;
	import java.io.InputStream;
	import java.io.OutputStream;
	import java.net.URI;
	
	import org.apache.hadoop.conf.Configuration;
	import org.apache.hadoop.fs.FileSystem;
	import org.apache.hadoop.fs.Path;
	import org.apache.hadoop.io.IOUtils;
	
	public class FileCopy {
	
		public static void main(String[] args) throws IOException {
			String sourceFile = args[0];
			String destFile = args[1];
	
			InputStream in = null;
			OutputStream out = null;
			try {
				//1､准备输入流
				in = new BufferedInputStream(new FileInputStream(sourceFile));
				//2､准备输出流
				Configuration conf = new Configuration();
				FileSystem fs = FileSystem.get(URI.create(destFile), conf);
				out = fs.create(new Path(destFile));
				//3､复制
				IOUtils.copyBytes(in, out, 4096, false);
			} finally {
				in.close();
				out.close();
			}
	
		}
	
	}



##3、获取文件属性

文件属性以FileStatus对象进行封装，使用FileSystem对象的getFileStatus()方法，可以获取到文件的FileStatus对象。

	
	package org.jediael.hadoopdemo.fsdemo;
	
	import java.io.IOException;
	import java.net.URI;
	
	import org.apache.hadoop.conf.Configuration;
	import org.apache.hadoop.fs.FileStatus;
	import org.apache.hadoop.fs.FileSystem;
	import org.apache.hadoop.fs.Path;
	
	public class FileStatusDemo {
	
		public static void main(String[] args) throws IOException {
			
			String fileName = args[0];
			
			Configuration conf = new Configuration();
			FileSystem fs = FileSystem.get(URI.create(fileName), conf);
			//获取FileSystem对象。
			FileStatus status = fs.getFileStatus(new Path(fileName));
			System.out.println(status.getOwner()+" "+status.getModificationTime());
			
		
		}
	
	}



##4、列出某个目录下的文件

使用FileSystem的ListStatus方法，可以获取到某个目录下所有文件的FileStatus对象。
	
	package org.jediael.hadoopdemo.fsdemo;
	
	import java.io.IOException;
	import java.net.URI;
	
	import org.apache.hadoop.conf.Configuration;
	import org.apache.hadoop.fs.FileStatus;
	import org.apache.hadoop.fs.FileSystem;
	import org.apache.hadoop.fs.FileUtil;
	import org.apache.hadoop.fs.Path;
	
	public class ListStatusDemo {
	
		public static void main(String[] args) throws IOException {
			
			String dir = args[0];
			
			Configuration conf = new Configuration();
			FileSystem fs = FileSystem.get(URI.create(dir), conf);
			FileStatus[] stats =  fs.listStatus(new Path(dir));
			
			Path[] paths = FileUtil.stat2Paths(stats);
			for(Path path : paths){
				System.out.println(path);
			}
		}
	
	}
