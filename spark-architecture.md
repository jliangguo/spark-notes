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

Spark Driver或者说是应用的driver进程是独立的JVM进程，它持有SparkContext，是Spark应用的Master节点。在Spark中由SparkContext负责和ClusterManager通信，进行资源的申请、任务的分配和监控等；当Executor部分运行完毕后，Driver负责将SparkContext关闭。通常用SparkContext代表Drive。

Driver的组件如下：

![](/assets/spark-driver.png)

Driver参数设置：

参数|参数默认值|描述|
--|--|--|
spark.driver.blockManager.port|spark.blockManager.port|Port to use for the BlockManager on the driver.More precisely, spark.driver.blockManager.port is used when NettyBlockTransferService is created (while SparkEnv is created for the driver).
spark.driver.host|localHostName|The address of the node where the driver runs on. Set when SparkContext is created
spark.driver.port|0|The port the driver listens to. It is first set to 0 in the driver when SparkContext is initialized. <p>Set to the port of RpcEnv of the driver (in SparkEnv.create) or when client-mode ApplicationMaster connected to the driver (in Spark on YARN).
spark.driver.memory|1g|The driver’s memory size (in MiBs).
spark.driver.cores|1|The number of CPU cores assigned to the driver in cluster deploy mode.<p>NOTE: When Client is created (for Spark on YARN in cluster mode only), it sets the number of cores for ApplicationManager using spark.driver.cores.
spark.driver.extraLibraryPath|||		
spark.driver.extraJavaOptions||Additional JVM options for the driver.
spark.driver.appUIAddress<p>spark.driver.appUIAddress is used exclusively in Spark on YARN. It is set when YarnClientSchedulerBackend starts to run ExecutorLauncher (and register ApplicationMaster for the Spark application).	|spark.driver.libraryPath||	
spark.driver.extraClassPath||the additional classpath entries (e.g. jars and directories) that should be added to the driver’s classpath in **cluster** deploy mode.|

> 在client部署模式下，可以用配置文件或命令行参数设置`spark.driver.extraClassPath`。不要使用`SparkConf`，因为为时已晚了，JVM已启动起来了。

> 在`spark-submit`中，可以使用`--driver-class-path`命令行参数复写配置文件中的`spark.driver.extraClassPath`参数。

### Executor

Executor是负责执行task的分布式组件，如下场景会创建Executor：

- `CoarseGrainedExecutorBackend`收到`RegisteredExecutor`消息时（Standalone和YARN部署模式下）

- `MesosExecutorBackend`注册时（Spark on Mesos部署模式下）

- `LocalEndpoint`创建时（local模式下）

Executor参数设置：

参数|默认值|描述|
--|--|--|
spark.executor.cores||Number of cores for an executor.
spark.executor.extraClassPath|(empty)|	List of URLs representing user-defined class path entries that are added to an executor’s class path.<p>Each entry is separated by system-dependent path separator, i.e. : on Unix/MacOS systems and ; on Microsoft Windows.
spark.executor.extraJavaOptions||Extra Java options for executors.<p>Used to prepare the command to launch CoarseGrainedExecutorBackend in a YARN container.
spark.executor.extraLibraryPath|Extra library paths separated by system-dependent path separator, i.e. : on Unix/MacOS systems and ; on Microsoft Windows.<p>Used to prepare the command to launch CoarseGrainedExecutorBackend in a YARN container.
spark.executor.heartbeat.maxFailures|60|Number of times an executor will try to send heartbeats to the driver before it gives up and exits (with exit code 56).<p>NOTE: It was introduced in SPARK-13522 Executor should kill itself when it’s unable to heartbeat to the driver more than N times.
spark.executor.heartbeatInterval|10s|Interval after which an executor reports heartbeat and metrics for active tasks to the driver.
spark.executor.id|||		
spark.executor.instances|0|Number of executors to use.
spark.executor.logs.rolling.maxSize|||		
spark.executor.logs.rolling.maxRetainedFiles|||		
spark.executor.logs.rolling.strategy|||		
spark.executor.logs.rolling.time.interval|||		
spark.executor.memory|1g|Amount of memory to use per executor process.<p>Equivalent to SPARK_EXECUTOR_MEMORY environment variable.
spark.executor.port|||		
spark.executor.userClassPathFirst|false|Flag to control whether to load classes in user jars before those in Spark jars.
spark.executor.uri||Equivalent to SPARK_EXECUTOR_URI
spark.task.maxDirectResultSize|1048576B||	

## Spark运行逻辑

对于RDD，有两种类型的动作，一种是Transformation，一种是Action。它们本质区别是：

> Transformation返回值还是一个RDD。它使用了链式调用的设计模式，对一个RDD进行计算后，变换成另外一个RDD，然后这个RDD又可以进行另外一次转换。这个过程是分布式的
> Action返回值不是一个RDD。它要么是一个Scala的普通集合，要么是一个值，要么是空，最终或返回到Driver程序，或把RDD写入到文件系统中

![](/assets/transformations_actions.jpg)

上图显示，在Spark应用中，整个执行流程在逻辑上会形成有向无环图（DAG）。Action算子触发之后，将所有累积的算子形成一个有向无环图，然后由调度器调度该图上的任务进行运算。Spark的调度方式与MapReduce有所不同。Spark根据RDD之间不同的依赖关系切分形成不同的阶段（Stage），一个阶段包含一系列函数执行流水线。图中的A、B、C、D、E、F分别代表不同的RDD，RDD内的方框代表分区。数据从HDFS输入Spark，形成RDD A和RDD C，RDD C上执行map操作，转换为RDD D， RDD B和 RDD E执行join操作，转换为F，而在B和E连接转化为F的过程中又会执行Shuffle，最后RDD F 通过函数saveAsSequenceFile输出并保存到HDFS中。

### Spark on Mesos

为了在Mesos框架上运行，安装Mesos的规范和设计，Spark实现两个类，一个是SparkScheduler，在Spark中类名是`MesosScheduler`；一个是SparkExecutor，在Spark中类名是`Executor`。有了这两个类，Spark就可以通过Mesos进行分布式的计算。

Spark会将RDD和MapReduce函数，进行一次转换，变成标准的Job和一系列的Task。提交给SparkScheduler，SparkScheduler会把Task提交给Mesos Master，由Master分配给不同的Slave，最终由Slave中的Spark Executor，将分配到的Task一一执行，并且返回，组成新的RDD，或者直接写入到分布式文件系统。

![](/assets/spark_on_mesos.jpg)

### Spark on YARN

Spark on YARN能让Spark计算模型在云梯YARN集群上运行，直接读取云梯上的数据，并充分享受云梯YARN集群丰富的计算资源。

Spark on YARN架构解析如下：

基于YARN的Spark作业首先由客户端生成作业信息，提交给ResourceManager，ResourceManager在某一NodeManager汇报时把AppMaster分配给NodeManager，NodeManager启动SparkAppMaster，SparkAppMaster启动后初始化作业，然后向ResourceManager申请资源，申请到相应资源后，SparkAppMaster通过RPC让NodeManager启动相应的SparkExecutor，SparkExecutor向SparkAppMaster汇报并完成相应的任务。此外，SparkClient会通过AppMaster获取作业运行状态。

![](/assets/spark_on_yarn.jpg)


## References

1. [【Spark】Spark生态和Spark架构](http://blog.jasonding.top/2015/06/07/Spark/%E3%80%90Spark%E3%80%91Spark%E7%94%9F%E6%80%81%E5%92%8CSpark%E6%9E%B6%E6%9E%84/)

2. [Spark Architecture](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/content/spark-architecture.html)
