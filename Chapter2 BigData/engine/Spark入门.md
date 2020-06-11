# 第1讲：Spark

## 1.Spark是什么

引用[官方文档](http://spark.apache.org/)的一句话

> Apache Spark™ is a unified analytics engine for large-scale data processing.
>
> Apache Spark™是用于大规模数据处理的统一分析引擎。

可以从这句话拆分出几个关键点

- 统一
- 大数据
- 分析引擎/计算引擎



**何为统一**

Spark的主要目标是为编写大数据应用程序提供统一的平台，其中包括**统一计算引擎**和**统一API**。

Spark提供一致的，可组合的API来构建应用程序，使得任务的编写更加高效。

> 例如：使用SQL加载数据，然后使用Spark的ML库评估其上的机器学习模型，引擎能够将这些步骤合并成一次数据扫描。

通用API设计+高性能执行

同时Spark为了实现统一，不仅支持使用自带的标准库，为通用数据分析人物提供统一的API，也支持由开源社区以第三方包形式发布的大量外部库。



**何为计算引擎**

不同于其他大数据软件平台(例如Hadoop)，Spark仅负责加载数据并计算，并不负责数据永久存储(持久化)。因此，Spark可以与多种持久化存储系统结合使用。常用的有以下几种

- 云存储系统（Azure存储、Amazon S3）
- 分布式文件系统（Apache Hadoop）
- 键值存储系统（Apache Cassandra）
- 消息队列系统（Apache Kafka）



## 附录

### Spark安装实录

**0.环境准备**

| 系统版本      | MacOS 10.15.4 |
| ------------- | ------------- |
| MacOS 10.15.4 | 1.8.0_111     |



**1.在[官网](http://spark.apache.org/downloads.html)上下载Spark安装包**

![image-20200611134256668](https://markdown-image-upload.oss-cn-beijing.aliyuncs.com/img/image-20200611134256668.png)

在这里我大胆选择了最新版的Spark+最新版的Hadoop3.2

下载完成后解压即可

ps.我把解压后的目录软链到了~/spark目录下了

---

**2.检查是否可用**

```bash
#进入Spark目录下
~/spark $ bin/spark-shell  
20/06/11 13:58:53 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
Spark context Web UI available at http://192.168.199.36:4040
Spark context available as 'sc' (master = local[*], app id = local-1591855138249).
Spark session available as 'spark'.
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 3.0.0-preview2
      /_/
         
Using Scala version 2.12.10 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_111)
Type in expressions to have them evaluated.
Type :help for more information.

scala> 
```

---

3.配置环境变量**

```bash
#spark
export SAPRK_HOME=/Users/jacobzheng/spark
export PATH=$PATH:$SPARK_HOME/bin
```

配置完成后source一下使得环境变量生效

然后直接spark-shell会有和步骤2一样的效果

---

**4.启动master** 

```bash
~/spark $ ./sbin/start-master.sh
starting org.apache.spark.deploy.master.Master, logging to /Users/jacobzheng/spark/logs/spark-jacobzheng-org.apache.spark.deploy.master.Master-1-MacBook-Pro.local.out
```

根据输出提示的目录地址，查看spark服务日志

```bash
~/spark $ tail -500f /Users/jacobzheng/spark/logs/spark-jacobzheng-org.apache.spark.deploy.master.Master-1-MacBook-Pro.local.out
Spark Command: /Library/Java/JavaVirtualMachines/jdk1.8.0_111.jdk/Contents/Home//bin/java -cp /Users/jacobzheng/spark/conf/:/Users/jacobzheng/spark/jars/* -Xmx1g org.apache.spark.deploy.master.Master --host MacBook-Pro.local --port 7077 --webui-port 8080
========================================
Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
20/06/11 14:10:53 INFO Master: Started daemon with process name: 74179@MacBook-Pro.local
20/06/11 14:10:53 INFO SignalUtils: Registered signal handler for TERM
20/06/11 14:10:53 INFO SignalUtils: Registered signal handler for HUP
20/06/11 14:10:53 INFO SignalUtils: Registered signal handler for INT
20/06/11 14:10:53 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
20/06/11 14:10:53 INFO SecurityManager: Changing view acls to: jacobzheng
20/06/11 14:10:53 INFO SecurityManager: Changing modify acls to: jacobzheng
20/06/11 14:10:53 INFO SecurityManager: Changing view acls groups to: 
20/06/11 14:10:53 INFO SecurityManager: Changing modify acls groups to: 
20/06/11 14:10:53 INFO SecurityManager: SecurityManager: authentication disabled; ui acls disabled; users  with view permissions: Set(jacobzheng); groups with view permissions: Set(); users  with modify permissions: Set(jacobzheng); groups with modify permissions: Set()
20/06/11 14:10:53 INFO Utils: Successfully started service 'sparkMaster' on port 7077.
20/06/11 14:10:53 INFO Master: Starting Spark master at spark://MacBook-Pro.local:7077
20/06/11 14:10:54 INFO Master: Running Spark version 3.0.0-preview2
20/06/11 14:10:54 INFO Utils: Successfully started service 'MasterUI' on port 8080.
20/06/11 14:10:54 INFO MasterWebUI: Bound MasterWebUI to 0.0.0.0, and started at http://192.168.199.36:8080
20/06/11 14:10:54 INFO Master: I have been elected leader! New state: ALIVE
```

可以看到`sparkMaster`服务监听了7077端口，`MasterUI`监听了8080端口

访问 http://localhost:8080/ 来查看到当前master的总体状态。

![image-20200611141551116](https://markdown-image-upload.oss-cn-beijing.aliyuncs.com/img/image-20200611141551116.png)

**5.启动Worker**

```bash
~/spark $ spark-class org.apache.spark.deploy.worker.Worker spark://MacBook-Pro.local:7077

Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
20/06/11 14:17:13 INFO Worker: Started daemon with process name: 74367@MacBook-Pro.local
20/06/11 14:17:13 INFO SignalUtils: Registered signal handler for TERM
20/06/11 14:17:13 INFO SignalUtils: Registered signal handler for HUP
20/06/11 14:17:13 INFO SignalUtils: Registered signal handler for INT
20/06/11 14:17:13 WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
20/06/11 14:17:13 INFO SecurityManager: Changing view acls to: jacobzheng
20/06/11 14:17:13 INFO SecurityManager: Changing modify acls to: jacobzheng
20/06/11 14:17:13 INFO SecurityManager: Changing view acls groups to: 
20/06/11 14:17:13 INFO SecurityManager: Changing modify acls groups to: 
20/06/11 14:17:13 INFO SecurityManager: SecurityManager: authentication disabled; ui acls disabled; users  with view permissions: Set(jacobzheng); groups with view permissions: Set(); users  with modify permissions: Set(jacobzheng); groups with modify permissions: Set()
20/06/11 14:17:13 INFO Utils: Successfully started service 'sparkWorker' on port 55878.
20/06/11 14:17:13 INFO Worker: Starting Spark worker 192.168.199.36:55878 with 16 cores, 15.0 GiB RAM
20/06/11 14:17:14 INFO Worker: Running Spark version 3.0.0-preview2
20/06/11 14:17:14 INFO Worker: Spark home: /Users/jacobzheng/spark
20/06/11 14:17:14 INFO ResourceUtils: ==============================================================
20/06/11 14:17:14 INFO ResourceUtils: Resources for spark.worker:

20/06/11 14:17:14 INFO ResourceUtils: ==============================================================
20/06/11 14:17:14 INFO Utils: Successfully started service 'WorkerUI' on port 8081.
20/06/11 14:17:14 INFO WorkerWebUI: Bound WorkerWebUI to 0.0.0.0, and started at http://192.168.199.36:8081
20/06/11 14:17:14 INFO Worker: Connecting to master MacBook-Pro.local:7077...
20/06/11 14:17:14 INFO TransportClientFactory: Successfully created connection to MacBook-Pro.local/192.168.199.36:7077 after 41 ms (0 ms spent in bootstraps)
20/06/11 14:17:14 INFO Worker: Successfully registered with master spark://MacBook-Pro.local:7077
```

可以看到启动了一个`sparkWorker`监听55878，启动了`WorkerUI`监听8081端口

此时刷新masterUI可以看到多了一个worker的信息

![image-20200611142011867](https://markdown-image-upload.oss-cn-beijing.aliyuncs.com/img/image-20200611142011867.png)

点击进去可以看到worker的信息

![image-20200611142311106](https://markdown-image-upload.oss-cn-beijing.aliyuncs.com/img/image-20200611142311106.png)

