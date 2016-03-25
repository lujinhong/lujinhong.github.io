---
layout: post
tile:  "redis及java客户端jedis"
date:  2016-03-25 16:52:23
categories: java, storm, redis 
excerpt: redis及java客户端jedis
---

* content
{:toc}




#一、概述
##（一）快速入门

###1、安装redis

（1）ubuntu/debian

	apt-get install redis
（2）mac

	brew intall redis
然后启动redis：

	redis-server /usr/local/etc/redis.conf &

###2、shell连接

	$ redis-cli
	127.0.0.1:6379> ping
	PONG
	127.0.0.1:6379> set name jason
	OK
	127.0.0.1:6379> set age 25
	OK
	127.0.0.1:6379> get name
	"jason"
	127.0.0.1:6379> get age
	"25"

###3、java连接redis
使用java连接redis有很多种API，但最常用的就是jedis，其余请参考redis的官方文档。

	public static void main(String[] args) {
		JedisPool jedisPool = new JedisPool("localhost", 6379);
		Jedis jedis = null;
		try {
			jedis = jedisPool.getResource();
			jedis.set("k1", "v1");
			jedis.set("k2", "v2");
			System.out.println(jedis.get("k1"));
			System.out.println(jedis.get("k2"));
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			if (jedis != null)
				jedis.close();
			
			jedisPool.close();
		}

# 二、redis
##（一）简介
 Redis从它的许多竞争继承来的三个主要特点：

* Redis数据库完全在内存中，使用磁盘仅用于持久性。
* 相比许多键值数据存储，Redis拥有一套较为丰富的数据类型。
*  Redis可以将数据复制到任意数量的从服务器。

Redis 优势

* 异常快速：Redis的速度非常快，每秒能执行约11万集合，每秒约81000+条记录。
* 支持丰富的数据类型：Redis支持最大多数开发人员已经知道像列表，集合，有序集合，散列数据类型。这使得它非常容易解决各种各样的问题，因为我们知道哪些问题是可以处理通过它的数据类型更好。
* 操作都是原子性：所有Redis操作是原子的，这保证了如果两个客户端同时访问的Redis服务器将获得更新后的值。
* 多功能实用工具：Redis是一个多实用的工具，可以在多个用例如缓存，消息，队列使用(Redis原生支持发布/订阅)，任何短暂的数据，应用程序，如Web应用程序会话，网页命中计数等。

##（二）数据类型&基本操作

Redis支持5种类型的数据类型，它描述如下的：
###1、字符串

Redis字符串是字节序列。Redis字符串是二进制安全的，这意味着他们有一个已知的长度没有任何特殊字符终止，所以你可以存储任何东西，512兆为上限。
	
	redis 127.0.0.1:6379> SET name "jason"
	OK
	redis 127.0.0.1:6379> GET name
	"jason"
	redis 127.0.0.1:6379> DEL name

上面是Redis的set和get命令的例子，Redis名称为yiibai使用的key存储在Redis的字符串值。
###2、哈希

Redis的哈希是键值对的集合。 Redis的哈希值是字符串字段和字符串值之间的映射，因此它们被用来表示对象
	
	redis 127.0.0.1:6379> HMSET user:1 username jason password pw points 200
	OK
	redis 127.0.0.1:6379> HGETALL user:1
	
	1) "username"
	2) "jason"
	3) "password"
	4) "pw"
	5) "points"
	6) "200"

在上面的例子中的哈希数据类型，用于存储其中包含的用户的基本信息用户的对象。这里HMSET，HEGTALL用户命令user:1是键。
###3、列表

Redis的列表是简单的字符串列表，排序插入顺序。您可以添加元素到Redis的列表的头部或尾部。
	
	redis 127.0.0.1:6379> lpush tutoriallist redis
	(integer) 1
	redis 127.0.0.1:6379> lpush tutoriallist mongodb
	(integer) 2
	redis 127.0.0.1:6379> lpush tutoriallist rabitmq
	(integer) 3
	redis 127.0.0.1:6379> lrange tutoriallist 0 10
	
	1) "rabitmq"
	2) "mongodb"
	3) "redis"

列表的最大长度为 232 - 1 元素（4294967295，每个列表中可容纳超过4十亿的元素）。
###4、集合

Redis的集合是字符串的无序集合。在Redis您可以添加，删除和测试文件是否存在，在成员O（1）的时间复杂度。
	
	redis 127.0.0.1:6379> sadd tutoriallist redis
	(integer) 1
	redis 127.0.0.1:6379> sadd tutoriallist mongodb
	(integer) 1
	redis 127.0.0.1:6379> sadd tutoriallist rabitmq
	(integer) 1
	redis 127.0.0.1:6379> sadd tutoriallist rabitmq
	(integer) 0
	redis 127.0.0.1:6379> smembers tutoriallist
	
	1) "rabitmq"
	2) "mongodb"
	3) "redis"

注意：在上面的例子中rabitmq集合添加加两次，但由于集合元素具有唯一属性。

集合中的元素最大数量为 232 - 1 （4294967295，可容纳超过4十亿元素）。
###5、有序集合

Redis的有序集合类似于Redis的集合，字符串不重复的集合。不同的是，一个有序集合的每个成员用分数，以便采取有序set命令，从最小的到最大的成员分数有关。虽然成员具有唯一性，但分数可能会重复。
	
	redis 127.0.0.1:6379> zadd tutoriallist 0 redis
	(integer) 1
	redis 127.0.0.1:6379> zadd tutoriallist 0 mongodb
	(integer) 1
	redis 127.0.0.1:6379> zadd tutoriallist 0 rabitmq
	(integer) 1
	redis 127.0.0.1:6379> zadd tutoriallist 0 rabitmq
	(integer) 0
	redis 127.0.0.1:6379> ZRANGEBYSCORE tutoriallist 0 1000
	
	1) "redis"
	2) "mongodb"
	3) "rabitmq"

###6、其它数据操作

* 删除key：

	    del name  

* key是否存在：

	    exists name  

* key的存活时间：time to live

	    ttl name  

* 查询所有的key：

	    keys *  

* 模糊匹配：

	    keys name*  

* 将key移动到数据库1中：

	    move name 1  

  

* 选择数据库：在Redis中默认有16个数据库(编号从0到15)，默认是对数据库0进行操作。
	
	    select 1  

* 当前数据库中key的数据：

		    dbsize  

* 清空当前数据库：

		    flushdb  

* 清空所有数据库：

		    flushall  

##（三）其它操作
###1、事务
 Redis - 事务

Redis事务让一组命令在单个步骤执行。事务中有两个属性，说明如下：

* 在一个事务中的所有命令按顺序执行作为单个隔离操作。通过另一个客户端发出的请求在Redis的事务的过程中执行，这是不可能的。
* Redis的事务具有原子性。原子意味着要么所有的命令都执行或都不执行。


	    127.0.0.1:6379> MULTI
		OK
		127.0.0.1:6379> SET name jason
		QUEUED
		127.0.0.1:6379> get name
		QUEUED
		127.0.0.1:6379> INCR visitors
		QUEUED
		127.0.0.1:6379> EXEC
		1) OK
		2) "jason"
		3) (integer) 1

###2、脚本

Redis脚本使用Lua解释脚本用于评估计算。它内置的Redis，从2.6.0版本开始使用脚本命令 eval。

	127.0.0.1:6379> EVAL "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 first second
	1) "key1"
	2) "key2"
	3) "first"
	4) "second"

###3、服务器信息
*  获取服务器信息：  

	    info  
* 获取特定信息：

	    info memory  




###4、连接

Redis的连接命令基本上都是用于管理与Redis的服务器客户端连接。

下面的例子说明了一个客户如何通过Redis服务器验证自己，并检查服务器是否正在运行。

	redis 127.0.0.1:6379> AUTH "password"
	OK
	redis 127.0.0.1:6379> PING
	PONG

###5、基准

Redis基准是公用工具同时运行Ñ命令检查Redis的性能。

redis的基准的基本语法如下所示：

	redis-benchmark [option] [option value]

下面给出的例子检查redis调用100000命令。

	redis-benchmark -n 100000


###6、客户端连接

Redis接受配置监听TCP端口和Unix套接字客户端的连接，如果启用。当一个新的客户端连接被接受以下操作进行：

* 客户端套接字置于非阻塞状态，因为Redis使用复用和非阻塞I/O操作。
* TCP_NODELAY选项设定是为了确保我们没有在连接时延迟。
* 创建一个可读的文件时，这样Redis能够尽快收集客户端的查询作为新的数据可供读取的套接字。

客户端的最大数量

在Redis的配置（redis.conf）属性调用maxclients，它描述客户端可以连接到Redis的最大数量。命令的基本语法是：

	127.0.0.1:6379> config get maxclients
	1) "maxclients"
	2) "10000"

默认情况下，此属性设置为10000（这取决于操作系统的文件描述符限制最大数量），但你可以改变这个属性。

在下面给出的例子中，在启动服务器我们设置客户端的最大数量为10万。

	redis-server --maxclients 100000
###7、管道传输

Redis是一个TCP服务器，并支持请求/响应协议。在redis一个请求完成下面的步骤：

* 客户端发送一个查询到服务器，并从套接字中读取，通常在阻塞的方式，对服务器的响应。
* 服务器处理命令并将响应返回给客户端。

管道传输的含义

管道的基本含义是，客户端可以发送多个请求给服务器，而无需等待答复所有，并最后读取在单个步骤中的答复。

要检查redis的管道，只要启动Redis实例，然后在终端键入以下命令。

	$(echo -en "PING\r\n SET tutorial redis\r\nGET tutorial\r\nINCR visitor\r\nINCR visitor\r\nINCR visitor\r\n"; sleep 10) | nc localhost 6379
	
	+PONG
	+OK
	redis
	:1
	:2
	:3

在上述例子中，我们必须使用PING命令检查Redis的连接，之后，我们已经设定值的Redis字符串命名tutorial ，之后拿到key的值和增量访问量的三倍。在结果中，我们可以检查所有的命令都一次提交给Redis，Redis是在一个步骤给出所有命令的输出。
管道的好处

这种技术的好处是极大地改善协议的性能。通过管道将慢互联网连接速度从5倍的连接速度提高到localhost至少达到百过倍。

###8、订阅

Redis的订阅实现了邮件系统，发送者（在Redis的术语中被称为发布者）发送的邮件，而接收器（用户）接收它们。由该消息传送的链路被称为通道。
在Redis客户端可以订阅任何数目的通道。
（1）在终端1中订阅某个主题

	127.0.0.1:6379> SUBSCRIBE redisChat
	Reading messages... (press Ctrl-C to quit)
	1) "subscribe"
	2) "redisChat"
（2）在终端2中往这个主题写入消息

	127.0.0.1:6379> PUBLISH redisChat "Redis is a great caching technique"
	(integer) 1
	127.0.0.1:6379> PUBLISH redisChat "Learn redis by tutorials point"
	(integer) 1
（3）终端1自动会收到终端2发送的内容

	1) "message"
	2) "redisChat"
	3) "Redis is a great caching technique"
	1) "message"
	2) "redisChat"
	3) "Learn redis by tutorials point"

##（四） 分区

分区是一种将数据分成多个Redis的情况下，让每一个实例将只包含你的键字的子集的过程。

###1、优缺点
分区的好处
* 它允许更大的数据库，使用的多台计算机的存储器的总和。如果不分区，一台计算机的内存可支数量有限。
* 它允许以大规模的计算能力，以多个内核和多个计算机，以及网络带宽向多台计算机和网络适配器。

分区的缺点
* 通常不支持涉及多个键的操作。例如，不能两个集合之间执行交叉点，因为它们存储在被映射到不同Redis实例中的键。
* 涉及多个键的Redis事务不能被使用。
* 分区粒度是关键，所以它是不可能分片数据集用一个硕大的键是一个非常大的有序集合。
* 当分区时，数据处理比较复杂，比如要处理多个RDB/AOF文件，使数据备份需要从多个实例和主机聚集持久性文件。
* 添加和删除的能力可能很复杂。比如Redis的集群支持有添加，并在运行时删除节点不支持此功能的能力，但其他系统，如客户端的分区和代理的数据大多是透明的重新平衡。但是有一个叫Presharding技术有助于在这方面。

###2、分区的类型

redis的提供有两种类型的分区。假设我们有四个Redis实例R0，R1，R2，R3和代表用户很多键如：user:1, user:2, ... 等等

**范围分区**
范围分区被映射对象转化为具体的Redis实例的范围内实现。假定在本例中用户ID0〜ID10000将进入实例R0，而用户形成ID10001至20000号将进入实例R1等等。

**散列分区**
在这种类型的分区，一个散列函数（例如，模数函数）被用于转换键成数字，然后数据被存储在不同redis的实例。


##（五）管理

###1、认证
  修改redis.conf文件：

    requirepass pw  

   客户端登录，需要先进行授权操作，提供密码即可。

    auth pw  

可以Redis的数据库更安全，所以相关的任何客户端都需要在执行命令之前进行身份验证。客户端输入密码匹配需要使用Redis设置在配置文件中的密码。

下面给出的例子显示的步骤，以确保您的Redis实例安全。

	127.0.0.1:6379> CONFIG get requirepass
	1) "requirepass"
	2) ""

默认情况下，此属性为空，表示没有设置密码，此实例。您可以通过执行以下命令来更改这个属性

	127.0.0.1:6379> CONFIG set requirepass "yiibai"
	OK
	127.0.0.1:6379> CONFIG get requirepass
	(error) NOAUTH Authentication required.

设置密码，如果客户端运行命令没有验证，会提示（错误）NOAUTH，需要通过验证。错误将返回客户端。因此，客户端需要使用AUTHcommand进行认证。
语法

AUTH命令的基本语法如下所示：

	127.0.0.1:6379> AUTH pw

###2、主从配置
   通过设置Redis的配置文件redis.conf可以进行主从(Master-Slave)设置，可以设置一个Redis节点为Master，同时设置一个或多个Slave节点。
* 在从节点配置redis.conf即可：设置为主节点的IP和端口

	    slaveof 192.168.142.12 6379  

* 如果Master节点设置了密码，Slave节点需要同时设置： 

		    masterauth pw  

* 说明：
   * 通过主从设置，可以进行读写分离：通常使用Master节点负责写数据，Slave节点负责读数据、注意Slave节点不能进行写操作。
   * 数据备份：在Slave节点执行如下命令，然后拷贝dump.rdb即可

    bgsave #该命令在后台执行，进行持久化操作，不会影响客户端的链接  
    save  #如果上述bgsave执行失败，可以使用save进行操作，但是会影响客户端的链接  

     
###3、日志/数据目录
* 创建如下所示的目录：

	    mkdir -p /opt/redis/logs  
	    mkdir -p /opt/redis/data  

* 对日志进行设置：

	    loglevel debug                      #日志级别：默认为notice  
	    logfile /opt/redis/logs/redis.log  #日志输出：默认为stdout  

* 设置数据目录：
	
	    dbfilename redis.rdb        #默认为dump.rdb  
	    dir /opt/redis/data         #默认为./  

    
###4、设置最大内存

    maxmemory 256mb  

[说明]
    @ 设置Redis能够占用的最大内存，防止影响性能甚至造成系统崩溃。
    @ 一定要小于物理内存(512MB)，留有充足的内存供系统及其他应用程序使用。

 

###5、备份策略
（1）使用快照：snapshot

    save 60 1000  

[说明]
    @ 如上的设置，会在60s内、如果有1000个key发生改变就进行持久化。
    @ 可设置多个save选项，默认持久化到dump.rdb。
（2）文件追加(aof)：append-only-file模式。
   @ Redis会将每个接收到的“写命令”通过write函数追加到appendonly.aof文件，重启Redis时通过该文件重建整个数据库。
   @ 由于os内核会缓存write函数所做的“修改”，可以使用fsync函数指定写入到磁盘的方式。
	
	    appendonly yes          #启动aof持久化方式  
	      
	    appendfsync always      #对每条“写命令”立即写至磁盘  
	    appendfsync everysec    #默认：每秒写入一次，在性能和可靠性之间的平衡  
	    appendfsync no          #依赖于os，不指定写入时机  

（3）两种方式的比较：
    @ 快照方式：性能较好，但是快照间隔期间，如果宕机将造成数据丢失。
    @ AOF模式：影响性能，不容易造成数据丢失。
    @ 如果Redis宕机：重启Redis即可，会自动使用redis.rdb、appendonly.aof恢复数据库。
（4）主从备份：从数据安全性角度考虑。
    @ 关闭快照功能。
    @ 同时设置主从服务器都为AOF模式。
    @ 说明：如果仅对Slave进行持久化设置，重启时，Slave自动和Master进行同步，全部数据丢失惊讶。
###6、备份

Redis SAVE命令用来创建当前的 Redis 数据库备份。

对Redis SAVE命令的基本语法如下所示：

	127.0.0.1:6379> SAVE
	1506:M 21 Mar 15:54:22.839 * DB saved on disk
	OK
这个命令将创建dump.rdb文件在Redis目录中。这个目录在哪请看下面。

	/usr/local/var/db/redis$ ls
	dump.rdb
要创建Redis的备份备用命令BGSAVE也可以。这个命令将开始执行备份过程，并在后台运行。

	127.0.0.1:6379> BGSAVE
	Background saving started
###7、还原Redis数据

要恢复Redis的数据只需移动 Redis 的备份文件（dump.rdb）到 Redis 目录，然后启动服务器。为了得到你的 Redis 目录，使用配置命令如下所示：

	127.0.0.1:6379>  CONFIG get dir
	1) "dir"
	2) "/usr/local/var/db/redis"

在上述命令的输出在/usr/local/var/db/redis 目录，在安装redis的服务器安装位置。




#三、jedis

##（一）基本操作

###1、配置文件 redis.properties

	redis.host=127.0.0.1
	redis.port=6379
	redis.timeout=3000
	redis.password=pw
	      
	redis.pool.maxActive=200
	redis.pool.maxIdle=20
	redis.pool.minIdle=5
	redis.pool.maxWait=3000
	      
	redis.pool.testOnBorrow=true
	redis.pool.testOnReturn=true

###2、代码

	public static void main(String[] args) {
		ResourceBundle bundle = ResourceBundle.getBundle("redis");

		JedisPoolConfig config = new JedisPoolConfig();
		String host = bundle.getString("redis.host");
		int port = Integer.parseInt(bundle.getString("redis.port"));
		int timeOut = Integer.parseInt(bundle.getString("redis.timeout"));
		String password = bundle.getString("redis.password");
		// config.setMaxActive(Integer.valueOf(bundle.getString("redis.pool.maxActive")));
		config.setTestOnBorrow(Boolean.valueOf(bundle.getString("redis.pool.testOnBorrow")));
		JedisPool pool = new JedisPool(config, host, port, timeOut, password);

		Jedis jedis = pool.getResource();

		jedis.set("province", "gd");
		String province = jedis.get("province");
		System.out.println(province);
		jedis.del("province");

		jedis.close();
		pool.close();
	}


##（二）常用 操作

###1、使用list：
 可以使用列表模拟队列(queue)、堆栈(stack)，并且支持双向的操作(L或者R)。
 （1）右边入队：

	      jedis.rpush("userList", "James");  

（2）左边出队：右边出栈(rpop)，即为对堆栈的操作。

	    jedis.lpop("userList");  

（3）返回列表范围：从0开始，到最后一个(-1) [包含] 

		    List<String> userList = jedis.lrange("userList", 0, -1);  

   Redis的TopN操作，即使用list完成：lrange
（4）删除：使用key

		    jedis.del("userList");  

（5）设置：位置1处为新值

    jedis.lset("userList", 1, "Nick Xu");  

（6）返回长度：


    Long size = jedis.llen("userList");  

（7）进行裁剪：包含

    jedis.ltrim("userList", 1, 2);  

 
###2、 使用set：和列表不同，集合中的元素是无序的，因此元素也不能重复。
（1）添加到set：可一次添加多个


    jedis.sadd("fruit", "apple");  
    jedis.sadd("fruit", "pear", "watermelon");  
    jedis.sadd("fruit", "apple");  

（2）遍历集合：

    Set<String> fruit = jedis.smembers("fruit");  

 （3）移除元素：remove


    jedis.srem("fruit", "pear");  

  （4）返回长度：


    Long size = jedis.scard("fruit");  

  （5）是否包含：

    Boolean isMember = jedis.sismember("fruit", "pear");  

  （6）集合的操作：包括集合的交运算(sinter)、差集(sdiff)、并集(sunion)


    jedis.sadd("food", "bread", "milk");   
    Set<String> fruitFood = jedis.sunion("fruit", "food");  

   
###3、使用sorted set
有序集合在集合的基础上，增加了一个用于排序的参数。
（1）有序集合：根据“第二个参数”进行排序。

    jedis.zadd("user", 22, "James");  
（2） 再次添加：元素相同时，更新为当前的权重。


    jedis.zadd("user", 24, "James");  

（3）zset的范围：找到从0到-1的所有元素。

    Set<String> user = jedis.zrange("user", 0, -1);  

 说明：我们可能还有一个疑虑，集合是怎么做到有序的呢？
   实际上，上述user的数据类型为java.util.LinkedHashSet
   
###4、使用hash
（1）存放数据：使用HashMap

    Map<String, String>  capital = new HashMap<String, String>();  
    capital.put("shannxi", "xi'an");  
    ...  
    jedis.hmset("capital", capital);  

  （2）获取数据：

    List<String> cities = jedis.hmget("capital", "shannxi", "shanghai");  

  
###5、其他操作：
（1） 对key的操作：
  @ 对key的模糊查询：


    Set<String> keys = jedis.keys("*");  
    Set<String> keys = jedis.keys("user.userid.*");  

  @ 删除key：


    jedis.del("city");  

    @ 是否存在：


    Boolean isExists = jedis.exists("user.userid.14101");  

  （2）失效时间：
  @ expire：时间为5s


    jedis.setex("user.userid.14101", 5, "James");  

  @ 存活时间(ttl)：time to live


    Long seconds = jedis.ttl("user.userid.14101");  

  @ 去掉key的expire设置：不再有失效时间


    jedis.persist("user.userid.14101");  

 （3）自增的整型：
  @ int类型采用string类型的方式存储：


    jedis.set("amount", 100 + "");  

  @ 递增或递减：incr()/decr()

    jedis.incr("amount");  

  @ 增加或减少：incrBy()/decrBy()


    jedis.incrBy("amount", 20);  

 （4）数据清空：
  @ 清空当前db：


    jedis.flushDB();  

    @ 清空所有db：


    jedis.flushAll();  

  （5）事务支持：
  @ 获取事务：


    Transaction tx = jedis.multi();  

    @ 批量操作：tx采用和jedis一致的API接口


    for(int i = 0;i < 10;i ++) {  
         tx.set("key" + i, "value" + i);   
         System.out.println("--------key" + i);  
         Thread.sleep(1000);    
    }  

  @ 执行事务：针对每一个操作，返回其执行的结果，成功即为Ok


    List<Object> results = tx.exec();  

##（三）分区

http://hello-nick-xu.iteye.com/blog/2078153
