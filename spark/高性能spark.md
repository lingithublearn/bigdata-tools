
# 第一章 介绍高性能spark
- 什么是spark,为什么表现重要
  - spark是一个高性能的，分布式计算系统
  - 调优，可以用更少的资源跑更多的数据，用更少的时间。调优方式，适应的场景不一
- 从书里可以知道什么
  - 介绍了 很多技巧，对spark会有更全面的了解
  - 版本大概是2.0.1
- 为什么用Scala
  - 成为一个spark专家，需要学习一些scala
  - 比java的api更好用
  - 表现比python好
  - 为什么不Scala：其他语言的库
  - 学习Scala
- 结论

# 第二章 spark怎么工作
mapreduce的替代品，处理hadoop上的分布式数据。在内存计算中使用惰性计算

## spark在大数据生态系统的什么位置  
spark on yarn 依赖hdfs,s3,cassandra等文件存储管理系统存储数据，还需要集群管理器：yarn，mesos,standalone
- 组件
  - spark core: 主要的数据处理框架
    - RDDs(resilient distributed datasets): 延迟计算的，静态类型的，分布式数据集。
    - 预先定义的粗粒度转换(整个数据集）map,join,reduce操作分布式数据集,也有IO操作
  - spark SQL: 可以和spark core混合使用
    - dataframe: 半结构化数据类型
    - dataset: 半结构化，类型化的RDDs
  - ML，MLlib：机器学习和统计算法
  - spark streaming：微批量数据的流式处理
    - 批的窗口函数
  - GraphX: 图形处理
## rdds：并行计算的模型
     大数据集作为rdd:不可变的，分布式对象集合，存储在executors/slave nodes
     根据配置参数，开启和分配executors执行节点，执行引擎将数据分发到executors上进行计算
     懒式操作rdd，直到rdd需要计算，可以在spark application生命周期内，一直保存rdd
     transform 返回的一个新的rdd，数据是不可变的
     以上特点，让spark易使用，容错，可伸缩的，高效的
- 懒计算
  - 简介
    - 很多内存存储系统，是基于细粒度的可变对象  
    - spark是基于RDD，不可变的，懒计算  
    - 只要需要输出一些值到driver或者到非spark系统，如hadoop时，才会计算
  - 懒计算的性能和可用性优势
    - 对于窄依赖，可以将多个算子指令发送到同一分区，避免发送多次，和访问多次结果，理论上可以降低计算复杂度
    - 相比mapreduce,可以用更少的代码
    - 修改函数逻辑时，mapreduce需要对mapper增加filter逻辑，避免多次传输数据
  - 懒执行和容错能力
    - 每个分区都有重新计算的依赖
    - 其他分布式计算方案，一般是记录更新或者跨机器复制数据
    - spark有计算的所有链条信息，可以直接重新计算
  - deug  
    - spark 不会直接返回所有信息，会知道执行action算子时，因此在一个能返回所有调试信息的环境测试很重要
- 内存持久化和内存管理
  - spark将数据加载到内存中，而不是写入磁盘，方便访问，在重复性计算中，性能最优
  - 三种内存管理
    - 内存中的反序列化数据：时间最快，因为不用序列化，但可能不是内存效率最高的
    - 内存中序列化数据：CPU密集，但内存效率更高，空间效率更高
    - 磁盘：对分区太大的数据，大量计算的唯一科兴方案
  - LRU原则，least recently used 最近最少用原则
- 不变性和RDD接口
  - 简介
    - RDD接口有一些属性：RDD的依赖，数据局部性的信息
    - RDDs有静态属性和不可变性
  - 创建
    - 从一个已有的RDD转换来
    - 从sparkContext来：spark cluster和spark app之间的连接，读取数据存储的数据
    - 从dataframe或dataset转变而来,可以使用spark sql
  - 属性：分区属性，分割，首选位置
    - partitions():返回分布式数据集的分区对象
    - iterator(): 返回父分区给定的迭代器元素
    - dependencies(): 返回依赖项的序列，可以理解为DAG
    - partitioner(): 返回分区对象的scala选项类型
    - preferedLocations(): 返回分区的数据位置信息
- RDD的类型
  - 有属性和共有方法
  - PairRDDFunctions, OrderedRDDFunctions，GroupedRDDFunctions 有自己的特殊方法，隐式类型转换
  - 不同的RDD 实现，有不同的功能
- RDD的功能，转换与操作，transformation and action
  - action: 返回不是RDD
    - 从driver拿数据，输出数据
    - collect, count, collectAsMap, sample, reduce, and take.
    - saveAsTextFile, saveAsSequenceFile, saveAsObjectFile
    -  foreach
  - transfromation: 返回另一个rdd
    - sort, reduce, group, sample, filter, and map distributed data
- 宽窄依赖
  - 窄依赖：父分区只有一个子分区，可以用数据的任意子集计算
    -  map, filter, mapPartitions, flatMap
    -  coalesce
  - 宽依赖：父分区不止一个，需要用到分区信息
    - 数据根据数值分区，需要知道分区内的数据
    -  groupByKey, reduceByKey, sort, and sortByKey
  - shuffle代价很大，数据量越大，越多数据需要移动的时候代价越大


## spark job scheduling 周期计划
- 简介
  - 一个spark集群可以同时运行多个spark程序，这些程序对应一个spark上下文
  - 介绍，spark怎么启动job：计算RDD transform的过程
- 跨应用的资源分配
  - 静态分配：分配有限的最大资源并在生命周期内保留
  - 动态分配：根据一系列估计资源需求的启动，来增加或减少executors,从1.2开始
- spark 应用application
  - 一个driver program 启动一个sparkContext，定义了一系列spark jobs
  - sparkContext 定义了多少资源会被分配给executor
  - 一个node 可以有多个executor，一个executor不能跨节点。一个RDD可以跨executor，一个分区不能跨executor
- 默认spark 调度
  - job 先进先出
  - 循环方式分发task，保证公平调度

## spark job的结构/解剖
- 每个action操作，对应一个job。
- 一个job由一系列stages组成：包含最终的RDD所需的数据转换
- 一个stage 由一系列task: 在executor上执行，并行计算着的
![image](https://github.com/lingithublearn/bigdata-tools/assets/87681054/05a10c54-5b38-43ee-b5e7-11cc8aba3333)




