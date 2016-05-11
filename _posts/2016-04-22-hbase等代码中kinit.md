---
layout: post
tile:  "hbase等代码中kinit"
date:  2016-04-22 18:13:15
categories: hadoop hbase storm kafka 
excerpt: hbase等代码中kinit
---

* content
{:toc}



##（一）在java代码中kinit的方法
1、Set Kerberos login with the UserGroupInformation API:

import org.apache.hadoop.security.UserGroupInformation;
org.apache.hadoop.conf.Configuration conf = HBaseConfiguration.create();  
conf.set("hadoop.security.authentication", "Kerberos");
UserGroupInformation.setConfiguration(conf);

2、Login with a keytab by calling the UserGroupInformation API:

UserGroupInformation.loginUserFromKeytab("example_user@IBM.COM", "/path/to/example_user.keytab");

完整代码：

	public class HBaseKerberosDemo {
	
		public static void main(String[] args) throws Exception {
			
			Configuration conf = HBaseConfiguration.create();
			conf.set("hadoop.security.authentication", "Kerberos");
			
			UserGroupInformation.setConfiguration(conf);
			UserGroupInformation.loginUserFromKeytab(args[0], args[1]);
			
			Connection connection = ConnectionFactory.createConnection(conf);
			
			Table tbl = connection.getTable(TableName.valueOf(args[2]));
		    Put put = new Put(Bytes.toBytes("r1"));
		    put.addColumn(Bytes.toBytes("f1"), Bytes.toBytes("c1"), Bytes.toBytes("v1"));
		    tbl.put(put);
		    tbl.close();
		    connection.close();
	
		}	
	}

##（二）定时刷新
kinit会有超时时间，如果程序需要长时间运行，则需要定时去刷新一下，可以通过一个线程来实现。

这种应用场景常见于storm，在storm中提供了TickTuple来实现这种定时功能。
