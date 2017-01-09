# Overview

> **Apache Spark™** is a fast and general engine for large-scale data processing.

[Apache Spark](http://spark.apache.org/)（后文简称Spark）是**一个开源的分布式通用集群计算框架**。由UC Berkeley AMPLab的Matei Zaharia在2009年开创，并于次年通过[BSD协议](https://en.wikipedia.org/wiki/BSD_licenses)开源发布。2013年，该项目被捐赠给Apache基金会并切换至[Apache 2.0协议](https://en.wikipedia.org/wiki/Apache_License)，2014年成为Apache的顶级项目。

Spark同时支持海量数据的[ETL](https://en.wikipedia.org/wiki/Extract,_transform,_load)，统计分析，机器学习和图运算等[批处理](https://en.wikipedia.org/wiki/Batch_processing)模式，也支持[流](https://en.wikipedia.org/wiki/Stream_processing)模式。它具有丰富简洁的高层API，支持Scala、Python、R和SQL等多种编程语言。

人们往往称Spark为**集群计算引擎**，或者简单地称为**执行引擎**。

Spark的**目标**是高速、易用、通用和随处运行。

相对于其前辈Hadoop基于磁盘的两阶段MapReduce处理引擎，Spark采用多阶段内存计算的方式，所以其运算速度可以达到Hadoop MapReduce的100倍以上，即便是运行程序于硬盘时，Spark也能快上10倍，因此非常适合迭代运算和交互式数据挖掘。

为了易于构建并行应用，Spark提供了约80个顶层算子。你可以Scala、Python和R等语言的shell与Spark交互。

Spark提供了[SQL（DataFrames）](http://spark.apache.org/sql/)、机器学习（[MLlib](http://spark.apache.org/mllib/)）、图（[GraphX](http://spark.apache.org/graphx/)）和[Spark Streaming](http://spark.apache.org/streaming/)。

![](http://spark.apache.org/images/spark-stack.png)


Spark可运行于[Hadoop YARN](http://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/index.html)、[Apache Mesos](http://mesos.apache.org/)、[Standalone](http://spark.apache.org/docs/latest/spark-standalone.html)或[Amazon EC2](http://spark.apache.org/docs/latest/ec2-scripts.html)，支持从[HDFS](http://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-hdfs/HdfsUserGuide.html)、[Cassandra](http://cassandra.apache.org/)、[HBase](http://hbase.apache.org/)、[Hive](http://hive.apache.org/)、[Tachyon](http://tachyon-project.org/)或任何Hadoop数据源读取数据。


Spark应用从输入创建顶层抽象RDD（Resilient Distributed Dataset），执行一系列的transformation（惰性）将RDD转换成其他形式，并最终通过执行action将数据合并或存储。



## Why Spark











