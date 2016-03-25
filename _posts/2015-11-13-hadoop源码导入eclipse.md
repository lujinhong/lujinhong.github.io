---
layout: post
tile:  "hadoop源码导入eclipse"
date:  2015-11-13 14:52:14
categories: hadoop 
excerpt: hadoop源码导入eclipse
---

* content
{:toc}




##一、环境准备
1、安装jdk、maven等

2、下载hadoop源代码，并解压

3、将tools.jar复制到Classes中，具体原因见http://wiki.apache.org/hadoop/HowToSetupYourDevelopmentEnvironment
cd $JAVA_HOME
mkdir Classes  
cp lib/tools.jar Classes/classes.jar
否则会出现以下异常：
 Missing tools.jar at: /Library/Java/JavaVirtualMachines/JDK 1.7.0 Developer

4、安装protobuf
（1）下载地址 ： http://pan.baidu.com/s/1pJlZubT  并解压
由于不能访问google，只能通过其它办法下载了。

2.切换到protobuf文件夹，依次在终端下输入：
. / configure
make
make check
make install

全部执行完后再输入protoc - - version检查是否安装成功。

##二、编译源文件
 mvn eclipse:eclipse -DdownloadSources=true -DdownloadJavadocs=true
时间较长，若有其它异常，则再逐个解决即可。

##三、使用eclipse导入project
