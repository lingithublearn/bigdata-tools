

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
  - 使用更小的数据结构
    - 使用原始类型，而不是自定义类: 例如少用scala的tuple
    - 不同类型的转换，一般会导致中间对象的创建
- 映射分区的迭代转换
  - 简介
    - 当一个转换直接使用，并返回一个迭代器，但不经过另一个集合：迭代转换
  - 什么是Iterator-to-Iterator Transformation
    - 迭代器只能遍历一次
    - 迭代器线性执行，一个接着一个。迭代迭代，要求返回的是一个新的迭代器
  - 空间和时间优势
    - 可以让spark有选择的将数据转移到磁盘
    - 一个接着一个，类似微批处理，方便程序判断那些需要溢写到磁盘，而不是整个分区
- 集操作 set
  - 原始的RDD 存在重复数值
- 减少设置开销
  - 简介
    - mapPartitions ,用iterator来共用一个函数，来减少配置开销
    - 在一个action中建立一个连接来存户数据
  - 分享变量
    - 广播变量和累加器：在driver / worker 分别被读或写
  - 广播变量
    - driver上创建一个变量，分发一个只读副本到每一台机器
    - 广播一次和每个任务copy一次会带来很大的不同
    - 不能更新，会存在读差别
    - 不用可以用unpersist（） 处理
  - 累加器
    - 可以对不同类型的数据进行累加，但要重写方法
    - 本质上是耿总任务度量
- 重用RDDs
  - 简介
    - 持久化，缓存，检查点
    - 对小数据量，数据直接重算更好
  - 重用的场景
    - 迭代计算
    - 对一个RDD多级执行
    - checkpoint 可以跨application
    - 如果重算每个分区代价很高
  - 决定是否重算，如果代价不够高
    - 会有内存回收的问题
    - 保存到磁盘和检查点，有mapreduce的缺点
    - 测试增加cash，时候加速
    - GC 和 out-of-memory 时，保存到disk，或者checkpoint
  - 重用的种类
    - 持久化和缓存persist and cache
      - useDisk, useMemory, useOfHeap, deserialized, replication
    - checkpointing
      - 外部存储系统
  - Alluxio (nee Tachyon)
    - 独立与spark的内存存储系统
  - Lru caching
    - Least Recently Used
    - shuffle文件，可以帮助重算，知道RDD离开管道，才会被完全清理
  - 嘈杂集群的注意事项
    - 使用checkPoint
    - 有FIFO等任务安排制度
  - 与累加器相互作用
    - 累加器可能重算时对一个分区累加多次

## 第六章 键值对数据
- 简介
  - 有助于并行处理
  - 可能会导致宽依赖，导致性能下降
  - driver的内存一处，worker的内存溢出，洗牌失败，分散任务
  - 方法：更少洗牌，更好的洗牌
- goldilocks 的例子
  - 背景：有上万熊猫，每个有上万指标，需要找到每个指标中排序第n1,n2,,n3的值
  - 方案
    - 迭代解决 iterative solution
      - 使用循环遍历每一个指标，对指标排序sort,包装上index zipWithIndex,筛选出需要的排序值
    - 通过key-value,实现分割flat,并行计算 
    ```scala
    dataFrame.rdd.flatMap(
    row => Range(0, rowLength).map(i => (i, row.getDouble(i)))
    )
    ```
    - PairRDDFunctions and OrderedRDDFunctions
- key-value的action操作
  - 简介
    - countByKey,会把key带到driver中，容易导致内存溢出
    - 数据的条数，和每条的大小都会影响是否内存溢出
- groupByKey的危险
  - 简介：伸缩性的问题
  - glodilocks的例子
    - 返回的是key,Iterable迭代器
    - 对小数据量，有很多列，但是很少数据，只用shuffle一次，sort是窄转换，可以在一个executor上完成
    - 对大数量级别，会在很多节点上失败
      - 返回的iterator是不可以分割的，会导致读取代价昂贵，节点需要读取几乎所有的shuffled data
      - 如果一个key有太多数据，会导致操作失败
      - 解决
        - 提前map，减少数据量
        - secondary sort，使用返回值不对应一个key的宽转换，可以分区
- 选择一个聚合函数
  - 用聚合操作防止内存溢出
    - 使用combineByKey,有相同key的数据在被shuffle之前已经合并
    - combine操作让数据变小
- 多RDD操作
  - Co-Grouping
    - CoGroupedRDD是join的基本类型
    - key的类型相同，返回一个key对应一个RDD，每个RDD对应一个Iterable
    - join多个，不如使用cogroup
    - 可能内存溢出，如果一个RDD内数据太多到超过一个分区的容量
- 分区和key/value数据
  - 改变分区的方法
    - repartition/coalse:改变分区数目
    - partitionBy:根据一个key分区，用一个已知的分区器
  - 使用spark partitioner 对象
    - 方法： numPartitions:获取分区数目，getPartition:获取key和分区缩索引的映射
    - 实现：HashPartitioner,RangePartitioner
  - Hash Partitioner
    - 定义：key的哈希值决定子分区的索引
    - 默认值 `spark.default.parallelism`
  - Range Partitioning
    - 定义：将在相同范围的key分配给同一个分区，对每个分区sort,则整体有序
    - 通过采样确定分区边界
    - 不平衡的数据，可能导致采样失效，数据倾斜
    - 既是转化，又是action
    - 代价比hash高些
  - 自定义 分区
    - numPartitions
    - getPartition
    - equals
    - hashcode
  - 跨转化，保留分区信息
    - 使用窄转化来保留分区：mapValues
  - 利用RDD的共定位和共分区
    - 共定位:物理上在同一个内存
      - 在cogroup之前call一个action算子，会导致不共定位，产生网络通信消耗
    - 共分区：分区的key相同
  - map和分区函数，对键值对函数的字典
    - mapValues,faltMapValues,keys,values,sampleByKey,partitionBy
- OrderedRDDOperations的字典
  - sortBykey,repartitionAndSortWithPartiton,filterByRange
  - 用两个key sortByKey
- 辅助排序和repartitionAndSortWithinPartitions
  - 简介
    - 用rangePartitions+mapPartitions比 sortByKey慢
    - secondary sort,将一些排序工作，在shuffle阶段完成
  - 用key组和value排序函数，使用repartitionAndSortWithinPartitions
    - 一种场景：通过key汇聚，每个分区内通过value排序
    - 类似 groupByKeyAndSortValue，但是spark中可能没有
  - 如何不按两个排序
    - 其他方式不能保证正确的结果，如
    - `indexValuePairs.sortByKey().groupByKey()`
    - `indexValuePairs.sortByKey.map(_.swap()).sortByKey`
  - Goldilocks Version 2: Secondary Sort
    - 用repartitionAndSortWithinPartitions 代替groupByKey+sort
    - 自定义分区器，key为 (column index, value)
    - 每个分区过滤，map加上一个虚拟值后的((1,2.0),1),遍历每个分区，找到目标排序值，保留下来
    - groupSorted:按照Key将所有value汇聚，使用了列表的模式匹配::
    - 性能
      - 对任意数据量，比groupByKey好，在shuffle后sort,可以实现迭代转化，和避免一个分区的所有数据存在内存中
      - 列的数量很多的时候，依然会导致失败
  - 不同的方法
    - 回顾优化手段
      - 窄转换，可以并行计算
      - shuffle后可以用窄转换，保留分区本地化
      - 宽转化，最好用唯一key，避免执行器上数据太多
      - sortByKey，及那个数据排序推到shuffle阶段的本地机器
      - 迭代转换，可以避免整个分区加载到内存
      - 有时shuffle文件可以避免宽转换的重算
    - 对单元格排序并知道每个分区上每列的元素数量，就可以定位第n个元素
      - map处理为键值对(cell value, index)
      - 用sortByKey排序，
      - 用mapPartitionsWithIndex，计数计算每个分区，每一列的元素数目，返回[(0, [2, 1, 1]), (1, [0, 1, 1])]，收集到driver，
      - 计算目标排序的定位: (分区index，[(列index,排序值),..])
      - 用mapPartitions过滤出目标排序,返回 (columnIndex, rank statistic)键值对
    - Goldilocks Version 3: Sort on Cell Values
      - 可读性糟糕，但是不会有内存报错，速度比其他方法快一个数量级
      - 最后两个mapPartitions涉及减少数据，可以用迭代转化实现
    ```scala
     def findRankStatistics(dataFrame: DataFrame, targetRanks: List[Long]):
     Map[Int, Iterable[Double]] = {
     val valueColumnPairs: RDD[(Double, Int)] = getValueColumnPairs(dataFrame)
     val sortedValueColumnPairs = valueColumnPairs.sortByKey()
     sortedValueColumnPairs.persist(StorageLevel.MEMORY_AND_DISK)
    
     val numOfColumns = dataFrame.schema.length
     val partitionColumnsFreq = getColumnsFreqPerPartition(sortedValueColumnPairs, numOfColumns)
    
     val ranksLocations = getRanksLocationsWithinEachPart(targetRanks, partitionColumnsFreq, numOfColumns)
    
     val targetRanksValues = findTargetRanksIteratively(sortedValueColumnPairs, ranksLocations)
     targetRanksValues.groupByKey().collectAsMap()
     }
    ```
- 滞后检测和倾斜数据
  - 简介
    - 滞后原因：没正确分配资源，数据没平均分区
    - 检查：web UI, 有些分区用时太长，重试多次
    - 解决：用其他作为key,增加随机噪声给key，用map减少/联合/去重在shuffle之前
    - sortBykey也可能造成数据倾斜
  - 例子
    - 0值的存在，可能导致数据倾斜
  - Goldilocks Version 4： 每个分区去重
    - 在每个分区聚合成 ((cell value, column index), count)
      - 使用hashMap,最后转化成Iterator,内存效率更高
    - 排序并找到排序位置
- Goldilocks 后期分析
  - 0： 迭代循环访问，分布式排序，只有一个stage,代价昂贵
  - 1：groupByKey: 将同样的key shuffle到同一个分区，在每个分组中使用mapPartitions进行sort
  - 2: Secondary Sort: 用repartitionAndSortWithinPartitions，将sort放在shuffle阶段
  - 3： 对所有数据cell value sort: 用值排序，用一系列窄转换来收集结果，新的排序键比每个分组的重复值更少
  - 4： 每个分区去重
  - 数据决定性能的因素
    - 原始数据的条数
    - 计算度量的分组数
    - 每个组中重复数目的比例
  - 少量数据意味着数据要小于所有executor的计算内存
  - 对单一组，0更好
  - 对很多组，2表现好
  - 组不多的时候，数据量大时候，比较少重复时候，3表现好
  - 4，如果很少重复，容易hash map内存溢出
  - 1因为问题，表现都不好
- 结论
  - 内存溢出和groupByKey有关，没有减少空间的使用
  - 减少shullfe:用聪明的分区器，用窄转换保存分区信息，用join的共位置
  - 数据倾斜，大量重复数据；会减慢shuffle,内存问题
  - 高性能的方法，往往需要大改

## 第七章 跨越scala
- 简介
  - 如何用其他语言高效的完成spark工作
  - 不同的语言性能不同，什么影响了
  - 有方式用JVM以外
- JVM中跨越Scala