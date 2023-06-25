
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
    很多内存存储系统，是基于细粒度的可变对象  
    spark是基于RDD，不可变的，懒计算  
    只要需要输出一些值到driver或者到非spark系统，如hadoop时，才会计算
    - 懒计算的性能和可用性优势
      - 对于窄依赖，可以将多个算子指令发送到同一分区，避免发送多次，和访问多次结果，理论上可以降低计算复杂度
      - 相比mapreduce,可以用更少的代码
      - 修改函数逻辑时，mapreduce需要对mapper增加filter逻辑，避免多次传输数据
    - 懒执行和容错能力
      - 每个分区都有重新计算的依赖
      - 其他分布式计算方案，一般是记录更新或者跨机器复制数据
      - spark有计算的所有链条信息，可以直接重新计算
    - deug  
    spark 不会直接返回所有信息，会知道执行action算子时，因此在一个能返回所有调试信息的环境测试很重要
- 





