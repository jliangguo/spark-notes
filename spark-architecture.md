# Spark Architecture

Spark是一个通用的大规模数据快速处理引擎。可以简单理解为Spark就是一个**大数据分布式处理框架**。

Spark是基于Map-Reduce算法实现的分布式计算框架，但不同的是Spark的中间结果输出和结果输出可以保存在内存中，从而不需要读写HDFS，因此Spark能更好地用于数据挖掘与机器学习等需要迭代的Map-Reduce算法中。

## Spark架构

Spark架构采用了分布式计算中的Master-Slave模型。Master是对应集群中的含有Master进程的节点，Slave是集群中含有Worker进程的节点。Master作为整个集群的控制器，负责整个集群的正常运行；Worker相当于计算节点，接收主节点命令与进行状态汇报；Executor负责任务的执行；Client作为用户的客户端负责提交应用，Driver负责控制一个应用的执行。

![](/assets/spark_architecture.png)

Spark集群部署后，需要在主节点和从节点分别启动Master进程和Worker进程，对整个集群进行控制。在一个Spark应用的执行过程中，Driver和Worker是两个重要角色。**Driver 程序是应用逻辑执行的起点，负责作业的调度，即Task任务的分发，而多个Worker用来管理计算节点和创建Executor并行处理任务。在执行阶段，Driver会将Task和Task所依赖的file(_哪些？？？_)和jar序列化后传递给对应的Worker机器，同时Executor对相应数据分区的任务进行处理**。

Spark的整体流程为：Client 提交应用，Master找到一个Worker启动Driver，Driver向Master或者资源管理器申请资源，之后将应用转化为RDD Graph，再由DAGScheduler将RDD Graph转化为Stage的有向无环图提交给TaskScheduler，由TaskScheduler提交任务给Executor执行。在任务执行的过程中，其他组件协同工作，确保整个应用顺利执行。

Spark中的基本组件：

- **ClusterManager**：在Standalone模式中即为Master（主节点），控制整个集群，监控Worker。在YARN模式中为资源管理器。
- **Worker**：从节点，负责控制计算节点，启动Executor或Driver。在YARN模式中为NodeManager，负责计算节点的控制。
- **Driver**：运行Application的main()函数并创建SparkContext。
- **Executor**：执行器，在worker node上执行任务的组件、用于启动线程池运行任务。每个Application拥有独立的一组Executors。
- **SparkContext**：整个应用的上下文，控制应用的生命周期。
- **RDD：Spark**的基本计算单元，一组RDD可形成执行的有向无环图RDD Graph。
- **DAGScheduler**：根据作业（Job）构建基于Stage的DAG，并提交Stage给TaskScheduler。
- **TaskScheduler**：将任务（Task）分发给Executor执行。
- **SparkEnv**：线程级别的上下文，存储运行时的重要组件的引用。

 SparkEnv内创建并包含如下一些重要组件的引用。
 - MapOutPutTracker：负责Shuffle元信息的存储。
 - BroadcastManager：负责广播变量的控制与元信息的存储。
 - BlockManager：负责存储管理、创建和查找块。
 - MetricsSystem：监控运行时性能指标信息。
 - SparkConf：负责存储配置信息。

### Driver


### Executor


## Spark运行逻辑


## References

1. [【Spark】Spark生态和Spark架构](http://blog.jasonding.top/2015/06/07/Spark/%E3%80%90Spark%E3%80%91Spark%E7%94%9F%E6%80%81%E5%92%8CSpark%E6%9E%B6%E6%9E%84/)

2. [Spark Architecture](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/content/spark-architecture.html)
