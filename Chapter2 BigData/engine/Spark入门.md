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



