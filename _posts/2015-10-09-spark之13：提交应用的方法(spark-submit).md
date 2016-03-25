---
layout: post
tile:  "spark之13：提交应用的方法(spark-submit)"
date:  2015-10-09 15:03:24
categories: spark 大数据 
excerpt: spark之13：提交应用的方法(spark-submit)
---

* content
{:toc}




参考自：https://spark.apache.org/docs/latest/submitting-applications.html

常见的语法：

	 ./bin/spark-submit \
	  --class <main-class>
	  --master <master-url> \
	  --deploy-mode <deploy-mode> \
	  --conf <key>=<value> \
	  ... # other options
	  <application-jar> \
	  [application-arguments]

举几个常用的用法例子：
		
	# Run application locally on 8 cores
	./bin/spark-submit \
	  --class org.apache.spark.examples.SparkPi \
	  --master local[8] \
	  /path/to/examples.jar \
	  100
		
	# Run on a Spark Standalone cluster in client deploy mode
	./bin/spark-submit \
	  --class org.apache.spark.examples.SparkPi \
	  --master spark://207.184.161.138:7077 \
	  --executor-memory 20G \
	  --total-executor-cores 100 \
	  /path/to/examples.jar \
	  1000
		
	# Run on a Spark Standalone cluster in cluster deploy mode with supervise
	./bin/spark-submit \
	  --class org.apache.spark.examples.SparkPi \
	  --master spark://207.184.161.138:7077 \
	  --deploy-mode cluster
	  --supervise
	  --executor-memory 20G \
	  --total-executor-cores 100 \
	  /path/to/examples.jar \
	  1000
		
	# Run on a YARN cluster
	export HADOOP_CONF_DIR=XXX
	./bin/spark-submit \
	  --class org.apache.spark.examples.SparkPi \
	  --master yarn-cluster \  # can also be `yarn-client` for client mode
	  --executor-memory 20G \
	  --num-executors 50 \
	  /path/to/examples.jar \
	  1000
	
	# Run a Python application on a Spark Standalone cluster
	./bin/spark-submit \
	  --master spark://207.184.161.138:7077 \
	  examples/src/main/python/pi.py \
	  1000
	 
1、一引起重要的参数说明

（1）—-class： 主类，即main函数所有的类

（2）—- master : master的URL，见下面的详细说明。

（3）—-deploy-mode:client和cluster2种模式

（4）—-conf:指定key=value形式的配置



2、关于jar包

hadoop和spark的配置会被自动加载到SparkContext,因此，提交application时只需要提交用户的代码以及其它依赖包，这有2种做法：

（1）将用户代码打包成jar，然后在提交application时使用—-jar来添加依赖jar包

（2）将用户代码与依赖一起打包成一个大包 assembly jar (or “uber” jar)


关于依赖关系更详细的说明：

When using spark-submit, the application jar along with any jars included with the --jars option will be automatically transferred to the cluster. Spark uses the following URL scheme to allow different strategies for disseminating jars:

file: - Absolute paths and file:/ URIs are served by the driver’s HTTP file server, and every executor pulls the file from the driver HTTP server.

hdfs:, http:, https:, ftp: - these pull down files and JARs from the URI as expected

local: - a URI starting with local:/ is expected to exist as a local file on each worker node. This means that no network IO will be incurred, and works well for large files/JARs that are pushed to each worker, or shared via NFS, GlusterFS, etc.

Note that JARs and files are copied to the working directory for each SparkContext on the executor nodes. This can use up a significant amount of space over time and will need to be cleaned up. With YARN, cleanup is handled automatically, and with Spark standalone, automatic cleanup can be configured with the spark.worker.cleanup.appDataTtl property.

Users may also include any other dependencies by supplying a comma-delimited list of maven coordinates with --packages. All transitive dependencies will be handled when using this command. Additional repositories (or resolvers in SBT) can be added in a comma-delimited fashion with the flag --repositories. These commands can be used with pyspark, spark-shell, and spark-submit to include Spark Packages.

For Python, the equivalent --py-files option can be used to distribute .egg, .zip and .py libraries to executors.

 

3、关于master的值

（1）对于standalone模式，是spark://ip:port/的形式

（2）对于yarn，有yarn-cluster与yarn-cluster2种

（3）对于mesos，目前只有client选项

（4）除此之外，还有local[N]这种用于本地调试的选项

Master URL	Meaning
local	Run Spark locally with one worker thread (i.e. no parallelism at all).
local[K]	Run Spark locally with K worker threads (ideally, set this to the number of cores on your machine).
local[*]	Run Spark locally with as many worker threads as logical cores on your machine.
spark://HOST:PORT	Connect to the given Spark standalone cluster master. The port must be whichever one your master is configured to use, which is 7077 by default.
mesos://HOST:PORT	Connect to the given Mesos cluster. The port must be whichever one your is configured to use, which is 5050 by default. Or, for a Mesos cluster using ZooKeeper, use mesos://zk://....
yarn-client	Connect to a YARN cluster in client mode. The cluster location will be found based on the HADOOP_CONF_DIR or YARN_CONF_DIR variable.
yarn-cluster	Connect to a YARN cluster in cluster mode. The cluster location will be found based on the HADOOP_CONF_DIR or YARN_CONF_DIR variable.
 


4、关于client与cluster模式

A common deployment strategy is to submit your application from a gateway machine that is physically co-located with your worker machines (e.g. Master node in a standalone EC2 cluster). In this setup, client mode is appropriate. In client mode, the driver is launched directly within the spark-submit process which acts as a client to the cluster. The input and output of the application is attached to the console. Thus, this mode is especially suitable for applications that involve the REPL (e.g. Spark shell).

Alternatively, if your application is submitted from a machine far from the worker machines (e.g. locally on your laptop), it is common to usecluster mode to minimize network latency between the drivers and the executors. Note that cluster mode is currently not supported for Mesos clusters. Currently only YARN supports cluster mode for Python applications.

 

5、加载本地的配置文件

The spark-submit script can load default Spark configuration values from a properties file and pass them on to your application. By default it will read options from conf/spark-defaults.conf in the Spark directory. For more detail, see the section on loading default configurations.
Loading default Spark configurations this way can obviate the need for certain flags to spark-submit. For instance, if the spark.master property is set, you can safely omit the --master flag from spark-submit. In general, configuration values explicitly set on a SparkConf take the highest precedence, then flags passed to spark-submit, then values in the defaults file.

If you are ever unclear where configuration options are coming from, you can print out fine-grained debugging information by running spark-submit with the --verbose option.

 
 

附spark-submit的完整命令：
		
	hadoop@gdc-nn01-logtest:~/spark$ bin/spark-submit
	Usage: spark-submit [options]  [app arguments]
	Usage: spark-submit --kill [submission ID] --master [spark://...]
	Usage: spark-submit --status [submission ID] --master [spark://...]
	
	Options:
	  --master MASTER_URL         spark://host:port, mesos://host:port, yarn, or local.
	  --deploy-mode DEPLOY_MODE   Whether to launch the driver program locally ("client") or
	                              on one of the worker machines inside the cluster ("cluster")
	                              (Default: client).
	  --class CLASS_NAME          Your application's main class (for Java / Scala apps).
	  --name NAME                 A name of your application.
	  --jars JARS                 Comma-separated list of local jars to include on the driver
	                              and executor classpaths.
	  --packages                  Comma-separated list of maven coordinates of jars to include
	                              on the driver and executor classpaths. Will search the local
	                              maven repo, then maven central and any additional remote
	                              repositories given by --repositories. The format for the
	                              coordinates should be groupId:artifactId:version.
	  --repositories              Comma-separated list of additional remote repositories to
	                              search for the maven coordinates given with --packages.
	  --py-files PY_FILES         Comma-separated list of .zip, .egg, or .py files to place
	                              on the PYTHONPATH for Python apps.
	  --files FILES               Comma-separated list of files to be placed in the working
	                              directory of each executor.
	
	  --conf PROP=VALUE           Arbitrary Spark configuration property.
	  --properties-file FILE      Path to a file from which to load extra properties. If not
	                              specified, this will look for conf/spark-defaults.conf.
	
	  --driver-memory MEM         Memory for driver (e.g. 1000M, 2G) (Default: 512M).
	  --driver-java-options       Extra Java options to pass to the driver.
	  --driver-library-path       Extra library path entries to pass to the driver.
	  --driver-class-path         Extra class path entries to pass to the driver. Note that
	                              jars added with --jars are automatically included in the
	                              classpath.
	
	  --executor-memory MEM       Memory per executor (e.g. 1000M, 2G) (Default: 1G).
	
	  --proxy-user NAME           User to impersonate when submitting the application.
	
	  --help, -h                  Show this help message and exit
	  --verbose, -v               Print additional debug output
	  --version,                  Print the version of current Spark
	
	 Spark standalone with cluster deploy mode only:
	  --driver-cores NUM          Cores for driver (Default: 1).
	
	 Spark standalone or Mesos with cluster deploy mode only:
	  --supervise                 If given, restarts the driver on failure.
	  --kill SUBMISSION_ID        If given, kills the driver specified.
	  --status SUBMISSION_ID      If given, requests the status of the driver specified.
	
	 Spark standalone and Mesos only:
	  --total-executor-cores NUM  Total cores for all executors.
	
	 Spark standalone and YARN only:
	  --executor-cores NUM        Number of cores per executor. (Default: 1 in YARN mode,
	                              or all available cores on the worker in standalone mode)
	
	 YARN-only:
	  --driver-cores NUM          Number of cores used by the driver, only in cluster mode
	                              (Default: 1).
	  --queue QUEUE_NAME          The YARN queue to submit to (Default: "default").
	  --num-executors NUM         Number of executors to launch (Default: 2).
	  --archives ARCHIVES         Comma separated list of archives to be extracted into the
	                              working directory of each executor.
	  --principal PRINCIPAL       Principal to be used to login to KDC, while running on
	                              secure HDFS.
	  --keytab KEYTAB             The full path to the file that contains the keytab for the
	                              principal specified above. This keytab will be copied to
	                              the node running the Application Master via the Secure
	                              Distributed Cache, for renewing the login tickets and the
	                              delegation tokens periodically.
	
	15/07/22 11:03:25 INFO util.Utils: Shutdown hook called
