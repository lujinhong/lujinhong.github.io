---
layout: post
tile:  "scala调用java代码"
date:  2015-11-13 15:49:11
categories: scala 
excerpt: scala调用java代码
---

* content
{:toc}


 

详细代码请见https://github.com/lujinhong/scalademo

在scala中调用java代替非常非常简单，直接调用即可

（一）一个简单示例

1、创建一个java类
	
	package com.lujinhong.demo.scala;
	
	public class MyJavaClass {
		
		public int adder(int a, int b){
			return a+b;
		}
	
	}

2、创建scala代码并调用上述类
	
	package com.lujinhong.demo.scala
	
	object InvokeJavaClass {
	  
	  def main(args :Array[String])={
	    val javaClass2 = new MyJavaClass()
	    val addResult = javaClass2.adder(3,4)
	    println(addResult);
	  }
	  
	}

 

（二）调用java的类库

也是一样，先import，然后直接调用即可
	
	package com.lujinhong.demo.scala
	
	import scala.io.Source
	import java.io.PrintWriter
	import com.lujinhong.demo.scala.MyJavaClass
	
	object IODemo {
	
	  def main(args: Array[String]) = {
	    val outFile = "/Users/liaoliuqing/Downloads/1.txt"
	
	
	    //将第15行数据输出到一个文件中
	    writeToFile(outFile, “hello scala")
	    
	
	  }
	
	
	
	  //将内容写入某个文件中,由于scala没有提供写文件的支持，可以使用java.io中的类代替
	  def writeToFile(outFile: String, content: String) {
	    val out = new PrintWriter(outFile)
	    out.write(content)
	    out.close()
	  }
	}
