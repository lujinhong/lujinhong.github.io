---
layout: post
tile:  "Hadoop使用MultipleOutputs输出多文件或者指定命名"
date:  2016-04-20 10:53:34
categories: hadoop 
excerpt: Hadoop使用MultipleOutputs输出多文件或者指定命名
---

* content
{:toc}





##（一）输出多文件（未测试）
比如将不同国家的信息分别输出到一份对应的文件中。

1、在reduce或map类中创建MultipleOutputs对象，将结果输出

    class reduceStatistics extends Reducer<Text, IntWritable, Text, IntWritable>{  
      
        //将结果输出到多个文件或多个文件夹  
        private MultipleOutputs<Text,IntWritable> mos;  
        //创建对象  
        protected void setup(Context context) throws IOException,InterruptedException {  
            mos = new MultipleOutputs<Text, IntWritable>(context);  
         }  
              
            //关闭对象  
        protected void cleanup(Context context) throws IOException,InterruptedException {  
            mos.close();  
        }  
    }  

 2、在map或reduce方法中使用MultipleOutputs对象输出数据，代替congtext.write()
Java代码  收藏代码

    protected void reduce(Text key, Iterable<IntWritable> values, Context context)  
                throws IOException, InterruptedException {  
            IntWritable V = new IntWritable();  
            int sum = 0;  
            for(IntWritable value : values){  
                sum = sum + value.get();  
            }  
            System.out.println("word:" + key.toString() + "     sum = " + sum);  
            V.set(sum);  
      
            //使用MultipleOutputs对象输出数据  
            if(key.toString().equals("hello")){  
                mos.write("hello", key, V);  
            }else if(key.toString().equals("world")){  
                mos.write("world", key, V);  
            }else if(key.toString().equals("hadoop")){  
                //输出到hadoop/hadoopfile-r-00000文件  
                mos.write("hadoopfile", key, V, "hadoop/");  
            }  
              
        }  

 

 3、在创建job时，定义附加的输出文件，这里的文件名称与第二步设置的文件名相同
Java代码  收藏代码

    //定义附加的输出文件  
                MultipleOutputs.addNamedOutput(job,"hello",TextOutputFormat.class,Text.class,IntWritable.class);  
                MultipleOutputs.addNamedOutput(job,"world",TextOutputFormat.class,Text.class,IntWritable.class);  
                MultipleOutputs.addNamedOutput(job,"hadoopfile",TextOutputFormat.class,Text.class,IntWritable.class);  


##（二）指定输出命名

###1、创建变量

	private static MultipleOutputs<Text, Text> mos;

###2、初始化变量
在map或者reduce的setup()方法中初始化变量

	mos = new MultipleOutputs<Text, Text>(context);

###3、使用变量代替context来write
在map()或者reduce()方法中使用mos作输出：

	mos.write("outputname", key, new Text(""));

###4、关于变量传递
在主类中定义的变量，如定义了一个outputname，需要将其写入conf分发至其它nodemanager：

		Configuration conf = new Configuration();
		//需要将变量分发至所有的nodemanager
		conf.set("outputname", outputName);

然后在map/reduce中从context获取这个变量：

		context.getConfiguration().get("outputname")
