---
layout: post
tile:  "REST、SOAP、protocolbuf、thrift、avro"
date:  2016-04-02 21:57:30
categories: kafka 
excerpt: REST、SOAP、protocolbuf、thrift、avro
---

* content
{:toc}



为提供异构环境或者跨语言的通信，需要一个中间协议进行转换，常用的有2大类：

一是传统的明文协议，如REST/SOAP等，这类协议最常用的方式是通过XML/JSON方式进行数据交互，然后通过解释这些文件得到实际所需的数据。其优点是内容可读，缺点是需要在大量的冗余信息，如XML的头，标签等内容。这对于有大量交互的情况时，会浪费大量的带宽。

第二种是protocolbuf、thrift、avro等二进制RPC层。最早是google实现的protocol buffer，但由于google并未将之开源，因此后来facebook写了一套thrift。最后，hadoop项目的创建者Doug Cutting写了一套avro。它们的功能都类似，细节肯定有不同之处。

举个例子，用户如何通过python来访问hbase的基本原理：

1. hbase启动一个thrift服务
2. 客户端使用python的API对表进行访问
3. 这个API会被转化为某种格式（如REST的JSON，或者thrift的二进制），并发送至thift服务器
4. thrift服务器解释这个JSON文件（或者其它格式），然后转化为内部的API（如HTable）
5. 通过hbase的内部实现（java）来访问真正的数据。
