# Spark RDD

## 简述

**RDD**(Resilient Distributed Dataset，弹性分布式数据集)是Spark的最基本抽象，是对分布式内存的抽象使用，实现了以操作本地集合的方式来操作分布式数据集的抽象实现。RDD是Spark最核心的东西，它表示已被分区，不可变的并能够被并行操作的数据集合，不同的数据集格式对应不同的RDD实现。RDD必须是可序列化的。RDD可以cache到内存中，每次对RDD数据集的操作之后的结果，都可以存放到内存中，下一个操作可以直接从内存中输入，省去了MapReduce大量的磁盘IO操作。这对于迭代运算比较常见的机器学习算法, 交互式数据挖掘来说，效率提升比较大。

RDD是MapReduce模型一种简单的扩展和延伸，它为了实现迭代、交互性和流查询等功能，需要保证RDD具备**在并行计算阶段之间能够高效地数据共享**的功能特性。RDD运用高效的数据共享概念和类似于MapReduce的操作方式，使得所有的计算工作可以有效地执行，并可以在当前特定的系统中获得关键性的优化。

> 你将RDD理解为一个大的集合，将所有数据都加载到内存中，方便进行多次重用。第一，它是分布式的，可以分布在多台机器上，进行计算。第二，它是弹性的，在计算处理过程中，机器的内存不够时，它会和硬盘进行数据交换，某种程度上会减低性能，但是可以确保计算得以继续进行。

## RDD特性

**RDD是分布式只读且已分区集合对象**。这些集合是弹性的，如果数据集一部分丢失，则可以对它们进行重建。具有自动容错、位置感知调度和可伸缩性，而容错性是最难实现的，大多数分布式数据集的容错性有两种方式：数据检查点和记录数据的更新。对于大规模数据分析系统，数据检查点操作成本很高，主要原因是大规模数据在服务器之间的传输带来的各方面的问题，相比记录数据的更新，RDD 也只支持粗粒度的转换，也就是记录如何从其它 RDD 转换而来（即 Lineage，中文称血统），以便恢复丢失的分区。

其特性为：

- 数据存储结构不可变
- 支持跨集群的分布式数据操作
- 可对数据记录按key进行分区
- 提供了粗粒度的转换操作
- 数据存储在内存中，保证了低延迟性

## RDD的好处

- RDD只能从持久存储或通过Transformations操作产生，相比于[分布式共享内存(DSM)](https://en.wikipedia.org/wiki/Distributed_shared_memory)可以更高效实现容错，对于丢失部分数据分区只需根据它的lineage就可重新计算出来，而不需要做特定的[Checkpoint](https://en.wikipedia.org/wiki/Application_checkpointing)。
- RDD的不变性，可以实现类Hadoop MapReduce的推测式执行。
- RDD的数据分区特性，可以通过数据的本地性来提高性能，这与Hadoop MapReduce是一样的。
- RDD都是可序列化的，在内存不足时可自动降级为磁盘存储，把RDD存储于磁盘上，这时性能会有大的下降但不会差于现在的MapReduce。

## RDD编程接口

对于RDD，有两种类型的动作，一种是Transformation，一种是Action。它们本质区别是：

- Transformation返回值还是一个RDD。它使用了链式调用的设计模式，对一个RDD进行计算后，变换成另外一个RDD，然后这个RDD又可以进行另外一次转换。这个过程是分布式的。
- Action返回值不是一个RDD。它要么是一个Scala的普通集合，要么是一个值，要么是空，最终或返回到Driver程序，或把RDD写入到文件系统中。

Transformations转换操作，返回值还是一个 RDD，如 map、 filter、 union；
Actions行动操作，返回结果或把RDD持久化起来，如 count、 collect、 save。

![](/assets/rdd-transformation-actions.jpg)

[Transformation Functions](http://spark.apache.org/docs/latest/programming-guide.html#transformations):

Transformation|Meaning|
--|--|
map(func)|Return a new distributed dataset formed by passing each element of the source through a function func.
filter(func)|Return a new dataset formed by selecting those elements of the source on which func returns true.
flatMap(func)|Similar to map, but each input item can be mapped to 0 or more output items (so func should return a Seq rather than a single item).
mapPartitions(func)|Similar to map, but runs separately on each partition (block) of the RDD, so func must be of type Iterator<T> => Iterator<U> when running on an RDD of type T.
mapPartitionsWithIndex(func)|Similar to mapPartitions, but also provides func with an integer value representing the index of the partition, so func must be of type (Int, Iterator<T>) => Iterator<U> when running on an RDD of type T.
sample(withReplacement, fraction, seed)|Sample a fraction fraction of the data, with or without replacement, using a given random number generator seed.
union(otherDataset)|Return a new dataset that contains the union of the elements in the source dataset and the argument.
intersection(otherDataset)|Return a new RDD that contains the intersection of elements in the source dataset and the argument.
distinct([numTasks]))|Return a new dataset that contains the distinct elements of the source dataset.
groupByKey([numTasks])|When called on a dataset of (K, V) pairs, returns a dataset of (K, Iterable<V>) pairs. <p>Note: If you are grouping in order to perform an aggregation (such as a sum or average) over each key, using reduceByKey or aggregateByKey will yield much better performance. <p>Note: By default, the level of parallelism in the output depends on the number of partitions of the parent RDD. You can pass an optional numTasks argument to set a different number of tasks.
reduceByKey(func, [numTasks])|When called on a dataset of (K, V) pairs, returns a dataset of (K, V) pairs where the values for each key are aggregated using the given reduce function func, which must be of type (V,V) => V. Like in groupByKey, the number of reduce tasks is configurable through an optional second argument.
aggregateByKey(zeroValue)(seqOp, combOp, [numTasks])|When called on a dataset of (K, V) pairs, returns a dataset of (K, U) pairs where the values for each key are aggregated using the given combine functions and a neutral "zero" value. Allows an aggregated value type that is different than the input value type, while avoiding unnecessary allocations. Like in groupByKey, the number of reduce tasks is configurable through an optional second argument.
sortByKey([ascending], [numTasks])|When called on a dataset of (K, V) pairs where K implements Ordered, returns a dataset of (K, V) pairs sorted by keys in ascending or descending order, as specified in the boolean ascending argument.
join(otherDataset, [numTasks])|When called on datasets of type (K, V) and (K, W), returns a dataset of (K, (V, W)) pairs with all pairs of elements for each key. Outer joins are supported through leftOuterJoin, rightOuterJoin, and fullOuterJoin.
cogroup(otherDataset, [numTasks])|When called on datasets of type (K, V) and (K, W), returns a dataset of (K, (Iterable<V>, Iterable<W>)) tuples. This operation is also called groupWith.
cartesian(otherDataset)|	When called on datasets of types T and U, returns a dataset of (T, U) pairs (all pairs of elements).
pipe(command, [envVars])|Pipe each partition of the RDD through a shell command, e.g. a Perl or bash script. RDD elements are written to the process's stdin and lines output to its stdout are returned as an RDD of strings.
coalesce(numPartitions)	|Decrease the number of partitions in the RDD to numPartitions. Useful for running operations more efficiently after filtering down a large dataset.
repartition(numPartitions)|Reshuffle the data in the RDD randomly to create either more or fewer partitions and balance it across them. This always shuffles all data over the network.
repartitionAndSortWithinPartitions(partitioner)|Repartition the RDD according to the given partitioner and, within each resulting partition, sort records by their keys. This is more efficient than calling repartition and then sorting within each partition because it can push the sorting down into the shuffle machinery.

[Action Functions](http://spark.apache.org/docs/latest/programming-guide.html#actions):

Action|Meaning|
--|--|
reduce(func)|Aggregate the elements of the dataset using a function func (which takes two arguments and returns one). The function should be commutative and associative so that it can be computed correctly in parallel.
collect()|Return all the elements of the dataset as an array at the driver program. This is usually useful after a filter or other operation that returns a sufficiently small subset of the data.
count()|Return the number of elements in the dataset.
first()|Return the first element of the dataset (similar to take(1)).
take(n)|Return an array with the first n elements of the dataset.
takeSample(withReplacement, num, [seed])|Return an array with a random sample of num elements of the dataset, with or without replacement, optionally pre-specifying a random number generator seed.
takeOrdered(n, [ordering])|Return the first n elements of the RDD using either their natural order or a custom comparator.
saveAsTextFile(path)|Write the elements of the dataset as a text file (or set of text files) in a given directory in the local filesystem, HDFS or any other Hadoop-supported file system. Spark will call toString on each element to convert it to a line of text in the file.
saveAsSequenceFile(path)<p>(Java and Scala)|Write the elements of the dataset as a Hadoop SequenceFile in a given path in the local filesystem, HDFS or any other Hadoop-supported file system. This is available on RDDs of key-value pairs that implement Hadoop's Writable interface. In Scala, it is also available on types that are implicitly convertible to Writable (Spark includes conversions for basic types like Int, Double, String, etc).
saveAsObjectFile(path)<p>(Java and Scala)	|Write the elements of the dataset in a simple format using Java serialization, which can then be loaded using SparkContext.objectFile().
countByKey()|Only available on RDDs of type (K, V). Returns a hashmap of (K, Int) pairs with the count of each key.
foreach(func)|Run a function func on each element of the dataset. This is usually done for side effects such as updating an Accumulator or interacting with external storage systems. <p>Note: modifying variables other than Accumulators outside of the foreach() may result in undefined behavior. See Understanding closures for more details.

## RDD依赖关系

不同的操作依据其特性，可能会产生不同的依赖，RDD之间的依赖关系有以下两种：

- **窄依赖(Narrow Dependencies)**
 
 一个父RDD分区最多被一个子RDD分区引用，表现为一个父RDD的分区；
对应于一个子RDD的分区或多个父RDD的分区对应于一个子RDD的分区，也就是说一个父RDD的一个分区不可能对应一个子RDD的多个分区，如map、filter、union等操作则产生窄依赖；
- **宽依赖(Wide Dependencies)**
 
 一个子RDD的分区依赖于父RDD的多个分区或所有分区，也就是说存在一个父RDD的一个分区对应一个子RDD的多个分区，如groupByKey等操作则产生宽依赖操作；

下图中，蓝色实心方框代表一个partition，蓝边矩形框代表一个RDD：

![](/assets/rdd-dependencies.jpg)

## Stage DAG

Spark提交Job之后会把Job生成多个Stage，多个Stage之间是有依赖的，Stage之间的依赖关系就构成了**DAG**（有向无环图）。

对于窄依赖，Spark会尽量多地将RDD转换放在同一个Stage中；而对于宽依赖，但大多数时候是shuffle操作，因此Spark会将此Stage定义为`ShuffleMapStage`，以便于向`MapOutputTracker`注册shuffle操作。Spark通常将shuffle操作定义为stage的边界。

![](/assets/rdd-stage-example.jpg)

## RDD数据存储管理

RDD可以被抽象地理解为一个大的数组（Array），但是这个数组是分布在集群上的。逻辑上RDD的每个分区叫一个**Partition**。

在Spark的执行过程中，RDD经历一个个的Transfomation算子之后，最后通过Action算子进行触发操作。 逻辑上每经历一次变换，就会将RDD转换为一个新的RDD，RDD之间通过Lineage产生依赖关系，这个关系在容错中有很重要的作用。变换的输入和输出都是RDD。 RDD会被划分成很多的分区分布到集群的多个节点中。分区是个逻辑概念，**变换前后的新旧分区在物理上可能是同一块内存存储**。 这是很重要的优化，以防止函数式数据不变性（immutable）导致的内存需求无限扩张。有些RDD是计算的中间结果，其分区并不一定有相应的内存或磁盘数据与之对应，如果要迭代使用数据，可以调cache()函数缓存数据。

![](/assets/rdd-storage.jpg)

上图中，RDD1含有5个分区（p1、 p2、 p3、 p4、 p5），分别存储在4个节点（Node1、 node2、 Node3、 Node4）中。RDD2含有3个分区（p1、 p2、 p3），分布在3个节点（Node1、 Node2、 Node3）中。

在物理上，RDD对象实质上是一个元数据结构，存储着Block、 Node等的映射关系，以及其他的元数据信息。一个RDD就是一组分区，在物理数据存储上，RDD的每个分区对应的就是一个Block，Block可以存储在内存，当内存不够时可以存储到磁盘上。

每个Block中存储着RDD所有数据项的一个子集，暴露给用户的可以是一个Block的迭代器（例如，用户可以通过mapPartitions获得分区迭代器进行操作），也可以就是一个数据项（例如，通过map函数对每个数据项并行计算）。

如果是从HDFS等外部存储作为输入数据源，数据按照HDFS中的数据分布策略进行数据分区，HDFS中的一个Block对应Spark的一个分区。同时Spark支持重分区，数据通过Spark默认的或者用户自定义的分区器决定数据块分布在哪些节点。例如，支持**Hash分区**（按照数据项的Key值取Hash值，Hash值相同的元素放入同一个分区之内）和**Range分区**（将属于同一数据范围的数据放入同一分区）等分区策略。

## References

1. [【Spark】弹性分布式数据集RDD概述](http://blog.jasonding.top/2015/07/08/Spark/%E3%80%90Spark%E3%80%91%E5%BC%B9%E6%80%A7%E5%88%86%E5%B8%83%E5%BC%8F%E6%95%B0%E6%8D%AE%E9%9B%86RDD/)

2. [RDD — Resilient Distributed Dataset](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/content/spark-rdd.html), Mastering Apache Spark 2



