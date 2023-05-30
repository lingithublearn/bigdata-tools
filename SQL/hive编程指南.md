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
  - 不能在where 中使用列别名，但是可以用一个嵌套的select 语句
  - 谓词操作符
  - 浮点数比较
    - >0.2 会输出0.2
    - float为0.2000001 double为 0.2000000000001 都为近似值
  - like 和 rlike
    - like 简单正则
    - rlike 正则表达式
- group by
  - 分组
  -  having 语句
-  join 语句
  - inner join xx on xx 等值连接
  - 优化
    - 保证表的大小，从左到右式一次增加的
    - 标记机制 `/*+STREAMTABLE(s)*/`
  - left outer join
  - outer join
  - right outer join
  - full outer join
  - left semi-join 左表满足右表on条件语句
  - 笛卡尔积join
  - map-side join
- order by 和sort by
  - order by是全局排序，可能运行时间过长
  - sort by 局部排序（每个reducer中排序）
- sort by和 distribute by
  - distrubute by 相同key的会分发到同一个reducer中
  - sort by 参上
- cluster by
  - cluster by = distribute by + sort by
  - 会剥夺sort by的并行性
- 类型转换   
  - cast(column as type)
  - 类型转换binary值
- 抽样查询
  - `tablesample(bucker 3 out of 10 on rand())`
  - 数据块抽样`tablesample(0.1 percent)`
  - 分桶表的输入裁剪
- union all 合并表，列类型需要相同

# 第七章 视图
hive 目前暂不支持物化视图

- 使用视图来降低查询难度 `create view table as select xxx`
- 使用视图来限制基于条件过滤的数据
  - 用户需要有访问整个底层原始表的权限，视图才能佛给你做
- 动态分区中的视图和map类型
- 视图零零碎碎的事

# 第八章 索引

索引功能有限
- 创建索引
```
create index in-name 
on table name(column)
as xxx
with deferred rebuild
idexpropreties(xxx=xxx,xxx=xxx)
in table name
partitioned by()
comment ''
```
  - as 指定了索引处理器
  - in table 不一定用，在一张新表里保存索引数据
  - Bitmap索引
- 重建索引 `dererred rebuild alter index` 是原子性的
- 显示索引 `show formatted index on table`
- 删除索引 `frop index if exists index on table name`
  - 不允许直接使用drop table 语句前删除索引表
- 实现一个定制化的索引处理器


# 第九章 模式设计
hive 是反模式的
- 按天划分的表 ： 应该使用分区
- 关于分区
  - 分区过多，会于很多文件，元数据信息可能超过namenode的处理能力
  - 按照时间粒度，分区数量的增涨是均匀的，灭个分区下的文件大小至少是文件系统中块的大小或块大小的数倍
  - 按照两个级别的分区，并且使用不同的维度
- 唯一键和标准化
  - 没有主键和序列密钥生成的自增键，避免对飞标准化数据进行join操作
  - 非标准化：最小化磁盘寻道，优化磁盘驱动器的I/O性能。但是可能导致数据重复或不一致
  - 有别于传统的设计原则，入复杂的数据类型
- 同一份数据多种处理
  - 从一个数据源产生多个数据聚合 `from history insert overwrite sales select * where action = a insert overwrite credits where action = b`
- 对每个表的分区
  - 对临时表分区，可以避免重跑时，对当前的表覆盖，导致出错
  - 隔离数据，优化查询
  - 用户需要管理中间表并删除旧分区
- 分桶表数据存储
  - 将数据集分解成更容易管理的若干部分
  - `clustered by (user_id) into 96 buckets`
  - 桶的数量是固定的，适合抽样
- 为表增加列
  - 允许在原始数据文件之上定义一个模式：为数据文件增加新的字段是，可以容易的适应表定义的模式
  - 提供了SerDe抽象：从输入中提取数据
  - 无法在已有字段的开始或者中间增加字段
- 使用列存储表
  - 通常是行式存储，也提供列式来混合列式存储信息
  - 重复数据
  - 多列
- 总是使用压缩

# 第十章 调优
- 使用EXPLAIN
  - 查询计划和其他一些信息，本身不会执行
  - 打印抽象语法树，解析成token（符号）和literal（字面值）
    - 忽略TOK_前缀
    - 输出写入到一个临时文件`TOK_INSERT (TOK_DESTINATION (TOK_DIR TOK_TMP_FILE))`
    - 包含多个stage（阶段），通常stage越多需要的时间越多
    - stage plan 比骄长和复杂，包含了这个job大部分处理过程
      - map operator tree
      - reduce operator tree: group by operator, File Output Operator
      - Fetch Operator： limit
- Explain extended
  - 可以产生更多的输出信息
- 限制调整
  - 配置属性 `hive.limit.optimize.enable` 使用limit语句时，对源数据进行抽样
  - 可能输入中有用的数据永远不会被处理到
- join 优化
  - 最大的表放在最右边，map_side
- 本地模式
  - 对小数据集，可以使用本地模式在单台机器上处理所有任务
  - `hive.exec.mode.local.auto`
- 并行执行
  - `hive.exec.parallel`开启并发执行
- 严格模式
  - 禁止三类查询
  - 对于分区表，必须有对分区字段过滤
  - 使用了order by 语句，必须使用limit
  - 限制笛卡尔积的查询
- 调整mapper和reducer的个数
  - cli打印reducer个数
  - dfs -count 来计算输入量大小，类似 du -s
  - mapred.reduce.tasks 确定给你使用较多还是较少的reducer来缩短执行时间
- JVM的重用
  - 会一直占用使用到的task插槽，以便进行重用
- 索引
- 动态分区调整
  - hive限制分区数在1000个左右
  - 严格模式，保证至少有一个分区是静态的
- 推测执行
  - 可以触发执行一些重复的任务，加快获取的那个task的结果以及进行侦测，将执行满的tasktracker加入到黑名单
- 单个mapreduce中多个group by
  - 将多个group 组装到单个mapreduce
- 虚拟列
  - 输入的文件名
  - 文件中的块偏移量

# 第十一章 其他文件格式和压缩方法
不会强制要求将数据转换成特定的格式才能使用
- 确定安装编解码器
  -  `hive -e "set io.compression.codes"`
-  选择一种压缩编/解码器
  - 减少磁盘和网络I/O操作，但是压缩和解压缩过程会增加CPU的开销
  - 压缩率，压缩/解压缩速度
  - 压缩格式的文件是否可以分割：知道文件中记录的边界
  - inputFormat定义了如何读取划分，如何将划分分割成记录
  - OutputFormat定义了如何将这些划分写回到文件或控制台输出中
- 开启中间压缩
  - 对中间数据进行压缩，可以减少map和reduce task间额数据传数量
  - 选择一个低CPU开销的编码/解码器
  - `hive.exec.compress.intermediate`
  - `mapred.map.output.compression.codec`修改编码/解码器
- 最终输出结果压缩
  - `hive.exec.compress.output`
- sequence file 压缩格式
  - 支持将一个文件分割成多个块
  - `create table a stored as sequencefile`
  - 压缩方式，none, record,block
- 使用压缩实践
  - hive 可以正常查看压缩数据，cat 不推荐
  - zcat
  - dfs -text 查看sequence数据
- 存档分区
  - 归档文件
  - 减轻namenode压力
  - 查询效率不高
  - `alter table xxx archive partition`将表转化成一个归档表
  - `alter table unarchive partition` 重新提取出来
- 压缩，包扎


# 第十二章 开发

- 修改log4j 属性
  - `hive-log4j.properties`文件来控制CLI和其他本地执行组件的日志
  - `hive-exec-log4j.properties` 控制marreeduce task的日志
- 连接java 调试器到hive
  - `hive --help --debug`
- 从源码编译hive
  - 从svn中下载一份发行版代码，对源码进行编译
  - 执行hive测试用例
  - 执行hook
- 配置hive 和Eclipse
- maven 工程中使用hive,导入hive包
- hive 中用hive_test进行单元测试
  - 使用thrift 通过hiveService 来访问hive,然后写一些应用
- 新增的插件开发工具箱（PDK）


# 第十三章 函数
用户自定义函数：更多的是对java定义的

- 发现和描述函数
  - show functions :列举但钱hive会话中加载的所有函数名称
  - describe functions [extended ]
- 调用函数
- 标准函数：用一行数据中的一列或者多列数据作为参数，然后返回结果是一个值的函数。同样可以返回一个复杂对象
- 聚合函数
  - 聚合函数，用户自定义函数，内置函数=用户自定义聚合函数（UDAF）
  - 接受从零行到多行的零个到多个列，饭后返回单一制
- 表生成函数（UDTF）
  - 接受零个或多个输入，然后产生多列或多行输出
  - arraya(). explode()
  - 无法从表中产生其他的列
  - 可以通过lateral view 来实现查询
  - `select name ,sub from employees lateral view explode(columnName) subView as sub`
- 一个通过日期计算其星座的UDF
  - 继承UDF类，并实现evaluate()函数
  - 编译为jar包
  - 加入类路径，create funvtion语句定义函数
- UDF和GenericUDF
  - 可以支持更好的null值处理，同时可以处理一些标准UDF无法支持的编程操作`case .. when ...`
  - nvl()
  - 继承 GenericUDF类
  - initialize():会被输入的每个参数调用，并最终传入到一个ObjectInspector对象中
  - evaluate 的输入是一个deferredobject 对象数据
  - getDisplayString()
- 不变函数  
  - 将自己的函数永久的加入到hive中，就需要对HIVE的java文件进行简单的修改，重新编译hive
- 用户自定义聚合函数
  - 会分多个阶段进行处理，基于UDAF执行的转换不同，在不同阶段的返回值类型也可能是不同的
  - 创建一个collect udaf来模拟group_concat
- 用户自定义表生成函数
  - 可以返回多列或者多行的程序接口
  - 可以产生多行数据的UDTF
  - 可以产生具有多个字段的单行数据的UDTF
  - 可以模拟复杂数据类型的UDTF
- 在UDF中访问分布式缓存
  - 可以访问分布式缓存，本地文件系统，但会显著地降低执行效率
- 以函数的方式使用注解
  - 定数性标注
  - 状态性标注
  - 唯一性
- 宏命令 : 注意版本是否实现
  - `create temporary macro sigmoid(x double) 1.0/(1.0+ exp(-x))   select sigmoid(2) from src limit 1`

# 第十四章 streaming
管道计算模型：为外部进程开启一个I/O管道，数据传给进程，从标准输入中读取数据，然后通过标准输出来写结果数据，最后返回到streaming api job
对Linux/unix系统的支持
语法：map(),reduce(),transform()
- 恒等变换 `select transform(a,b) using '/bin/cat' as newA,newb from default.a`
- 改变类型 `using '/bin/cat' as (newA int,newB double) from a`
- 投影变换 
  - 使用cut命令提取或者映射出特定的字段
  - `using '/vin/cut -f1' as newA,newB from b`
- 操作变换
  - /bin/sed 接受输入数据流，然后按照用户的指定进行编辑，最后将编辑结果输出到数据流中
  - `/bin/sed s/4/10` 将字符串4替换成10
- 使用分布式内存
  - add file 将脚本文件加入到分布式缓存中，可以使transform task可以直接使用脚本
- 由一行产生多行
- 使用streaming 进行聚合计算
- cluster by, distribute by, sort by
  - cluster by：类似的数据可以分发到同一个reduce task中，保证数据有序
    - 使用了两个py脚本
  - cluster by = distribute by+sort by
- GrnericMR tools for streaming to JAVA
  - 结合java代码
- 计算cogroup
  - 使用union all和cluster by 可以实现group by操作的常见效果


# 第十五章 自定义hive文件和记录格式
- 文件于记录格式
- 阐明create table 句式
- 文件格式
- 记录格式：SerDE
- CSV 和 TSV
- ObjectInspector
- Thing Big Hive Reflection ObjectInspector
- XML UDF
- Xpath 相关的函数
- json SerDe
- Avro Hive SerDe
- 二进制输出


# 第十六章 Hive的Thrift服务

允许通过指定端口访问hive。软件框架，用于跨语言的服务开发
- 启动thrift server
- 配置 groovy使用hiveServer
- 连接到HiveServer
- 获取集群状态信息
- 结果集模式
- 获取结果
- 获取执行计划
- 元数据存储方法
- 管理hiveServer
- Hive THriftMetastore


# 第十七章 存储处理程序和NoSQL

存储处理程序时一个结合InputFormat,OutputFormat,SerDe和HIVE需要使用的特定的代码，来将外部实体作为标准的Hive表进行处理的整体
- Storage Handler Background
  - inputFormat 对不同源的数据格式化
  - outputFormat 输出输入到一个实体中
  - 可用于从其他数据源中读取和存放数据（关系型数据库，NoSQL)
- HiveStorageHandler


