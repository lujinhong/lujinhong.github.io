---
layout: post
tile:  "使用ResourceBundle加载properties文件"
date:  2016-03-22 09:13:42
categories: java 
excerpt: 使用ResourceBundle加载properties文件
---

* content
{:toc}




##1、ResourceBundle介绍

说的简单点，这个类的作用就是读取资源属性文件（properties），然后根据.properties文件的名称信息（本地化信息），匹配当前系统的国别语言信息（也可以程序指定），然后获取相应的properties文件的内容。
 
使用这个类，要注意的一点是，这个properties文件的名字是有规范的，一般的命名规范是： **自定义名_语言代码_国别代码.properties*，如果是默认的，直接写为：自定义名.properties
	
	myres_en_US.properties
	myres_zh_CN.properties
	myres.properties
 
当在中文操作系统下，如果myres_zh_CN.properties、myres.properties两个文件都存在，则优先会使用myres_zh_CN.properties，当myres_zh_CN.properties不存在时候，会使用默认的myres.properties。
 
没有提供语言和地区的资源文件是系统默认的资源文件。
**资源文件都必须是ISO-8859-1编码**，因此，对于所有非西方语系的处理，都必须先将之转换为Java Unicode Escape格式。转换方法是通过JDK自带的工具native2ascii.

##2、示例一：英文环境

###1、准备properties文件：person.properties

	name=lujinhong
	age=30
	gender=male
###2、java代码

	ResourceBundle rb = ResourceBundle.getBundle("person");
	System.out.println(rb.getString("name") + "\t" + rb.getString("age") + "\t" +rb.getString("gender") );
输出如下：

lujinhong	30	male


##3、示例二：中文环境
###1、准备properties文件：person_zh_CN.properties
注意资源文件都必须是ISO-8859-1编码。

	name=\u9646\u9526\u6D2A
	age=30
	gender=\u7537
###2、java代码

	Locale locale = new Locale("zh","CN");
	ResourceBundle rb2 = ResourceBundle.getBundle("person2",locale);
	System.out.println(rb2.getString("name") + "\t" + rb2.getString("age") + "\t" +rb2.getString("gender") );
输出如下：

陆锦洪	30	男
