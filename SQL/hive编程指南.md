# 第一章 基础知识
- 简介
  - Hive 是一个架构在 Hadoop 之上的数据仓库基础工具，它可以处理结构化和半结构化数据，它使得查询和分析存储在 Hadoop 上的数据变得非常方便。
  - HiveQL 自动把类 SQL 语句转换成 MapReduce 作业
  - 以将结构化的数据文件映射为一张数据库表
- 限制
  - 不支持记录级别的更新，插入或者删除
  - 查询延时严重，不支持事务
- hadoop和mapreduce的综述
  - mapreduce
    - 将任务分解成很多单个的，可以在服务器集群中并行执行的任务
    - map:将集合中的元素从一种形式转换成另一种形式
    - reduce：将值的集合转换成一个值
  - HDFS 分布式文件系统
    - 每个数据块（block）会被冗余多分，（通常默认3份）
    - 对数据进行sort和shuffle
- hadoop生态系统中的hive
  - 代替使用java开发重复的代码
  - 流程
    - cli,JDBC,ODBC,Thrift服务器进行编程访问
    - 命令和查询进入driver,解析编译，以xml形式编码
    - hive通过于jobtracker通信来初始化任务
    - merastrore元数据存储，是一个独立的关系型数据库
  - pig:常用于ETL(数据抽取，数据转换，数据装载)过程的一部分，语言不是基于SQL的
  - HBASE：分布式，可伸缩的数据存储，支持行级别的数据更新，快速查询和行级数据，不支持多行事务
    - 列存储，列族
    - 使用HDFS缓存数据
    - 使用内存缓存技术，对数据和本地文件进行追加数据更新操作日志
  - cascading cruch和其他
    - 更适合事件流，接近实时响应：spark,storm,kafka
    - 数据处理语言和工具;R,Matlab,Mathematica,SciPy,NumPy
  - java和hive


# 第二章 基础操作
- 安装预先配置好的虚拟机
- 安装步骤
  - 安装java
  - 安装Hadoop
  - 本地模式，伪分布模式，分布式
  - 测试Hadoop
  - 安装hive
- 内部是什么
  - 主要部分是Java代码本身
  - hive服务可执行文件，报错cli
  - thrift服务提供了可远程访问其他进程的功能
  - jdbc和odbc访问hive的功能
  - metastoreservice 元数据服务，存储表模式信息和其他元数据信息
- 启动hive
  - `hive`
  - 关键字大小写无关
- 配置Hadoop环境
  - 本地模式
  - 分布式和伪分布式
  - 使用jdbc连接元数据
- hive命令
- 命令行界面
  - cli选项
  - 变量和属性

# 第三章 架构








