---
layout: post
tile:  "HadoopHelper使用说明"
date:  2016-04-20 10:55:28
categories: hadoop 
excerpt: HadoopHelper使用说明
---

* content
{:toc}




##1、获取某个HDFS目录的du大小
官方竟然没有提供API。

	public long getDirSize(String dir) throws IOException {
		Long blockSize = 0L;
		FileSystem fs = FileSystem.get(URI.create(dir), _conf);
		FileStatus[] stats =  fs.listStatus(new Path(dir));
		
		if (stats==null || stats.length ==0) {  
	          LOG.error("Cannot access " + dir +   
	              ": No such file or directory.");  
	          throw new FileNotFoundException("Cannot access " + dir +   
	                  ": No such file or directory."); 
	        }  
		
		for(FileStatus stat: stats){
			blockSize += stat.getLen();
		}
		return blockSize;
	}
