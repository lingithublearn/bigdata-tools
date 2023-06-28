
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
  - LRU原则，least recently used 最少最近用原则
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
- 简介
  - 每个action操作，对应一个job。
  - 一个job由一系列stages组成：包含最终的RDD所需的数据转换
  - 一个stage 由一系列task: 在executor上执行，并行计算着的
![image](https://github.com/lingithublearn/bigdata-tools/assets/87681054/05a10c54-5b38-43ee-b5e7-11cc8aba3333)

- DAG 有向无环图
  - 为每一个job的stages 构建一个DAG
  - 确定每个task的位置，并传递信息给任务调度
- jobs
  - 是spark执行层级最大的元素
  - DAG的边缘由宽窄依赖决定
- stages
  - 宽依赖定义了jog里stages的边界
  - 每个stage 对应了宽依赖导致的清洗依赖，即一个stage包含多个联系的窄依赖，即一个新的stage意味着需要节点之间的网络通信
- tasks
  - 执行层级中的最小粒度，代表一个本地计算，一个stage中的所有task是对数据的不同分片执行相同的代码
  - task的数目由输出的RDD的分区数目决定
  - execcutor核心数 = executor数目* 一个executor的core数目
  - 每个分区的数据完成计算的最小单位

# 第三章 DataFrames Datasets and spark SQL
- 简介
  - 有更高效的存储选项，先进的优化器，对序列化数据的直接操作
  - dataframe/dataset，对比RDD有额外的结构信息
  - dataframe 比 dataset多了row object，没有编译时类型检查
- 从sparksession开始
  - import
  - 使用builder 模式，getOrCreate()
  - hiveContext，SQLcontext是1.0版本的
- spark SQL 依赖
```maven
    <dependency> <!-- Spark dependency -->
   <groupId>org.apache.spark</groupId>
   <artifactId>spark-sql_2.11</artifactId>
   <version>2.0.0</version>
  </dependency>
  <dependency> <!-- Spark dependency -->
   <groupId>org.apache.spark</groupId>
   <artifactId>spark-hive_2.11</artifactId>
   <version>2.0.0</version>
  </dependency>
```
  - 管理 spark依赖
    - sbt-spark-package插件
    - hive Metastore
  - 避免hive jars
- 基础的表结构
  - dataFrame有schema
  - 显示表结构，printSchema()
  - structField 可以嵌套，structTypes
- dataFrame api
  - transformations
    - filter:pandaInfo.filter(pandaInfo("happy") !== true)
    - 实现了很多运算符，数学运算符
    - explode,when-otherwise
    - 特殊函数，应对缺失数据和脏数据
    - 跨越逐行转换：去重
    - 聚合 aggregates and groupBy: 打印常用的统计值
    - 窗口：order by/partitionBy/rowsBetween
    - sorting 排序，可以配置limit获得topK,取样可以用sampling
    - 多个dataFrame join
    - 累集操作：unionAll，intersect,except,distinct
  - 旧的SQL查询和与HIVE data交互
    - 注册为临时表/save
    - 也可以直接从文件路径查询
- dataFrames /datasets 的数据含义
  - Tungsten:是spark SQL的组件之一
  - 压缩率更好，时间更短，比java和Kryo
- 数据下载和保存函数 load / save
  - dataFrameWriter and DataFrame Reader
  - formats 格式
    - JSON：采样比可调
    - JDBC：jar包，类型包含在url中，read（），write()
    - parquet: 列式存储格式
    - hive tables:直接 read,write
    - RDDS: createDataFrame(rowRdd,schema),返回，使用 .rdd
    - 本地集合
    - 额外的格式：CSV
  - 保存模式
    - overwrite，append,ignore,errorIfExists
  - 分区：用于写出，和下流使用，partitionBy
- DataSets
  - 简介：支持编译时类型检查
  - 与RDD,DataFrame 本地数据集合的转换
    - dataset是dataframe的别名（Scala）
    - 使用as转化dataframe to dataset
    - createDataSet()可以转化成本地数据集合为dataset
      
  ```scala
   def fromDF(df: DataFrame): Dataset[RawPanda] = {
     df.as[RawPanda]
   }
    def toRDD(ds: Dataset[RawPanda]): RDD[RawPanda] = {
     ds.rdd
   }
  def toDF(ds: Dataset[RawPanda]): DataFrame = {
     ds.toDF()
   }
  ```
  - 编译时强类型：清楚输入和输出时的类型要求
  - 更容易使用功能性转换（类似RDD）`ds.map{rp => rp.attributes.filter(_ > 0).sum}`
  - 关系转换（relation transfromation）`ds.select($"id".as[Long], ($"attributes"(0) > 0.5).as[Boolean])`
  - 多个dataset 关系转换 `unionAll intersect except distinct`
  - 汇聚操作: 需要类型申明
    ```scala
     def maxPandaSizePerZip(ds: Dataset[RawPanda]): Dataset[(String, Double)] = {
     ds.map(rp => MiniPandaInfo(rp.zip, rp.attributes(2)))
     .groupByKey(mp => mp.zip).agg(max("size").as[Double])
     }
     def maxPandaSizePerZipScala(ds: Dataset[RawPanda]): Dataset[(String, Double)] = {
       ds.groupByKey(rp => rp.zip).mapGroups{ case (g, iter) =>
         (g, iter.map(_.attributes(2)).reduceLeft(Math.max(_, _)))
       }
     }
    ```
- 用户自定义函数，聚合函数进行扩展
  ```scala
   def setupUDFs(sqlCtx: SQLContext) = {
   sqlCtx.udf.register("strLen", (s: String) => s.length())
   }
   def setupUDAFs(sqlCtx: SQLContext) = {
   class Avg extends UserDefinedAggregateFunction {
   // Input type
   def inputSchema: org.apache.spark.sql.types.StructType =
   StructType(StructField("value", DoubleType) :: Nil)
   def bufferSchema: StructType = StructType(
   StructField("count", LongType) ::
   StructField("sum", DoubleType) :: Nil
   )
   // Return type
   def dataType: DataType = DoubleType
   def deterministic: Boolean = true
   def initialize(buffer: MutableAggregationBuffer): Unit = {
   buffer(0) = 0L
   buffer(1) = 0.0
   }
   def update(buffer: MutableAggregationBuffer,input: Row): Unit = {
   buffer(0) = buffer.getAs[Long](0) + 1
   buffer(1) = buffer.getAs[Double](1) + input.getAs[Double](0)
   }
   def merge(buffer1: MutableAggregationBuffer, buffer2: Row): Unit = {
   buffer1(0) = buffer1.getAs[Long](0) + buffer2.getAs[Long](0)
   buffer1(1) = buffer1.getAs[Double](1) + buffer2.getAs[Double](1)
   }
   def evaluate(buffer: Row): Any = {
   buffer.getDouble(1) / buffer.getLong(0)
   }
   }
   // Optionally register
   val avg = new Avg
   sqlCtx.udf.register("ourAvg", avg)
   }
  ``` 
- 查询优化
  - 逻辑和物理计划：优化，模式匹配，规则导向，代价导向，谓词下推到数据源级别
  - 代码生成：java
  - 大查询，迭代算法：通常是迭代算法导致的大查询，可以用缓存成RDD解决
  ```scala
   val rdd = df.rdd
   rdd.cache()
   sqlCtx.createDataFrame(rdd, df.schema)
  ```
- debug SQL 查询
  - 注意过滤下推
- JDBC/ODBC SERVER
- 结论
  - dataframes/dataset,有更多的优化器，处理序列化数据更快捷，RDD，更具有功能性，能实现更多的功能，更底层
  - dataset有强类型检查

# 第四章 joins 连接操作（SQL and Core）
- 简介
  - 可能需要特殊的性能条件，网络通信，也可能创建超出性能的大数据集和
- spark Core Joins
  - 简介
    - 父RDD有分区
    - 部分RDD有分区
    - 都有分区，只用窄依赖，RDD的相同分区将汇聚成一个，或打乱
  - 选择join类型
    - 有重复键容易导致数据激增
    - 内连接，容易导致丢数据，建议左外或右外
    - 有容易定义的键的子集，且不需要，建议尽早过滤，减少shuffle
    ```scala
     def outerJoinScoresWithAddress(scoreRDD : RDD[(Long, Double)],
       addressRDD: RDD[(Long, String)]) : RDD[(Long, (Double, Option[String]))]= {
         val joinedRDD = scoreRDD.leftOuterJoin(addressRDD)
         joinedRDD.reduceByKey( (x, y) => if(x._1 > y._1) x else y )
     }
    ```
  - 选择执行计划
    - 原理：将不同的RDD按照相同的哈希分区计算，保证 同一个键值的数据在同一个分区
    - 避免:如果RDD是共定位的，shuffle可以避免
      - RDD都有已知的分区
        ```scala
       def joinScoresWithAddress3(scoreRDD: RDD[(Long, Double)],
       addressRDD: RDD[(Long, String)]) : RDD[(Long, (Double, String))]= {
       // If addressRDD has a known partitioner we should use that,
       // otherwise it has a default hash parttioner, which we can reconstruct by
       // getting the number of partitions.
       val addressDataPartitioner = addressRDD.partitioner match {
       case (Some(p)) => p
       case (None) => new HashPartitioner(addressRDD.partitions.length)
       }
       val bestScoreData = scoreRDD.reduceByKey(addressDataPartitioner,
       (x, y) => if(x > y) x else y)
       bestScoreData.join(addressRDD)
       }
      ```
      - 其中一个足够小到内存中可以存下，用广播哈希join
      ```scala
      def manualBroadCastHashJoin[K : Ordering : ClassTag, V1 : ClassTag,
      V2 : ClassTag](bigRDD : RDD[(K, V1)],
       smallRDD : RDD[(K, V2)])= {
       val smallRDDLocal: Map[K, V2] = smallRDD.collectAsMap()
       bigRDD.sparkContext.broadcast(smallRDDLocal)
       bigRDD.mapPartitions(iter => {
       iter.flatMap{
       case (k,v1 ) =>
       smallRDDLocal.get(k) match {
       case None => Seq.empty[(K, (V1, V2))]
       case Some(v2) => Seq((k, (v1, v2)))
       }
       }
       }, preservesPartitioning = true)
       }
      //end:coreBroadCast[]
      }
      ```
      - 部分手动广播
        - 计数键，来看哪个键收益最大
        - 过滤出量小的键值，手动join
- SPARK SQL joins
  - 简介
    - 降低，或者重排序操作，来使join更高效
    - 但是也放弃了操作分区的功能
  - dataframe join
    - `df.join(otherDf, sqlCondition, joinType)`
    - 特殊，left-semi: 用右表进行过滤，保留存在的。left-anti:用右表进行过滤，保留不存在的
    - 广播hash join `df1.join(broadcast(df2), "key")`
    - 自己 join `val joined = df.as("a").join(df.as("b")).where($"a.name" === $"b.name")`

# 第五章 高效的转化

- 宽窄转化
  - 简介
    - 从输出开始看，往回看输入
    - 从父分区出发看是否要分发到多个子区
  - 对性能的影响
    - 窄依赖，不用数据迁移，可以在一个stage中完成
    - 宽依赖，需要数据迁移和电位磁盘I/O，下一步计算需要等待上一步shuffle完成
  - 对容灾
    - 宽依赖需要更多的重算
    - 通过检查点，保存中间结果
  - coalesce 合并的特殊情况
    - 减少是窄依赖
    - 增加分区需要用到shuffle
- 转换返回什么类型的RDD
  - 很多转换，需要特定的类型才能使用，dataframe 转化为RDD时，会丢弃掉类型信息，容易报错
- 最小化对像创建
  - 简介
    - 垃圾回收会占用比较多的序列化时间
    - 可以使用重用已存在对象减少对象的数量，使用占用内存更少的数据结构
  - 重用已存在的对象
    - 使用 Scala的this.type,累加计算用的是第一个累加器
    - 但是可变数据结构，可能导致数据结果不准确
    - 

