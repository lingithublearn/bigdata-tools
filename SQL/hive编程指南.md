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
  - 一次使用 命令
  - 从文件中执行hive查询
  - hiverc文件
  - 自动补全功能
  - 查看操作命令历史
  - 执行shell命令： ！+ ；
  - 使用hadoop 命令 dfs 
  - 注释 --
  - 显示字段名称， set hive.cli.print.header=true;
  - 

# 第三章 数据类型和文件格式
- 基本数据类型
  -  数据类型都是对java 中的接口的实现
  -  timestamp 可以是整数，也可以是浮点数，也可以是字符串
  -  比较：会隐式的将类型转换为两个整数类型中值比较打的那个
  -  也可以显示的转换类型 `cast（a as int）`
-  集合数据类型
  - struct:对象，通过.first访问
  - map: 键值对
  - array：数组
- 文本文件数据编码
  - 分隔符文本文件，CSV，TSV
  - ROW FORMAT DELIMITED
  - ROW FORMAT DELIMITED FIELDSTERMINATED BY `\001` hive将使用^A作为列分割符
  - ROW FORMAT DELIMITEDCOLLECTION ITEMS TERMINATED BY `\002`将使用^B作为集合元素之间的分隔符
  - ROW FORMAT DELIMITEDMAP KEYS TERMINATED BY `\003`map
  - LINES TERMINATED BY `\n`
- 读时模式
  - 传统数据库：写时模式：数据在写入数据库时对模式进行检查
  - hive：读时模式：在查询时进行验证


# 第四章 HIveSQL数据定义
不支持行级插入操作，更新操作和删除操作，不支持事务
- hive中的数据库
  - 表的一个目录或者命名空间`create database if not exists financials`
  - schema 可以替代 table
  - default 没有自己的目录
  - 数据库所在的目录位于： hive.metastore.warshouse.dir所指定的顶层目录之后
  - `describe database financials`可以看到local
  - extended 查看额外信息
  - 删除 drop, 需要先删除掉数据库的表 caseade,会自行先删除表
- 修改数据库
  - `alter databse abc set dbproperties()`
- 创建表
  - 简介
    - if not exists
    - 两个表属性，last_modified_by last_modified_time
    - 拷贝表 like
    - describe extended/formatted tablename
  - 管理表：内部表，hive控制数据的生命周期
  - 外部表
    - 关键词 external
    - location 路径
- 分区表，管理表
  - 优势
    - 将数据从物理上转义到和使用最频繁的用户更近的地方
    - 将数据以一种符合逻辑的方式组织，如分层存储
  - 目的： 更快的查询
  - 外部分区表
    - 逻辑数据管理
    - 高性能查询
  - 自定义表的存储格式
- 删除表`drop table if exists tablename`
- 修改表
  - `alter table ` 修改元数据，不改数据
  - 重命名 ` alter table tablename rename to xxx`
  - 增加，修改，删除表分区 `alter table tablename add/drop partition`
  - 修改列信息,增加，删除或者替换`alter table name change/add/replace column name xxx xxxx xxxx after`
  - 修改表属性
  - 修改存储属性 `alter table name set fileformat sequencefile`

# 第五章 数据操作
- 向管理表中装载数据
  - overwrite 数据会鲜先被删除
  - inpath 路径下不能有任何文件夹
```
load data local inpath `xxx`
overwrite into table name 
partition ()
```
- 通过查询语句向表中插入数据
  - insert overwrite table xx select * from xxx
  - 动态分区插入
- 单个查询语句中创建表并加载数据
  - `create table name as select xx,xx from xx`
- 导出数据
  - hadoop fs-cp xxx xxx
  - `insert overwrite local derectoty 'xxx' select xxx`

# 第六章 查询
- select from
  - 表别名 e
  - 集合数据类型，用json语法输出
  - 引用集合元素，map元素，struct元素
  - 正则表达式 \`price.\*\`
  - 使用列值计算
  - 算术运算符，隐式的类型转换
  - 使用函数
    - 数学函数：floor,round ,ceil向上取整，返回bigint类型
    - 聚合函数
      - count()
      - avg()
      - min,max
      - collect_set()
    - 表生成函数：单列扩展成多列或者多行
      - explode()
    - 其他内置函数
      - 字符串函数：concat_ws()
      - 日期函数
  - limit语句
  - 列别名
  - 嵌套 select语句 from 可以再select前
  - case when then else end
  - 什么情况下避免mapreduce
    - 简单的select * from xx
- where 子句 






