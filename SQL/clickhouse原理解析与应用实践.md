# 第一章 clickhouse的历史
- 传统BI系统
  - 传统的联机事务处理OLTP
  - BI商业只能系统，数仓，联机分许OLAP
- 现在BI系统
  - SaaS模式，将软件面向大众
  - 传统数据库思路：数据分层，数据集市，数据立方体
- OLAP常见架构分类
  - 联机分析，多维分析
  - ROLAP：Relational 关系性OLAP，星型模型或者雪花模型。对数据的实时处理能力要求高
  - MOLAP：multidimensional 多维性OLAP:多维数组的形式保存数据，预先聚合结果，空间换实践
  - HOLAP:Hybrid 混合架构OLAP
- OLAP实现技术的演进
  - 传统关系型数据库：数据体谅多，维度数目多的情况下存在严重的性能问题
  - 大数据技术阶段：维度爆炸，数据同步实时性不高的问题。主流MOLAP架构可以在亿万级数据下，毫秒级的查询响应时间
- clickhouse诞生
  - BI的SaaS产品：一站式，自服务，实时应答，专业化
  - 使用搜索引擎ElasticSearch
  - 查询响应时间比spark快很多:1亿的数据体量，hive的126倍
  - 社区活跃
- 发展历程
  - mysql时期：顺序存储的数据有更高的查询性能，但它不行
  - metrage:key-value模型，用了LSM树，实时查询改为预处理
    - LSM树：内存中小数，数据局部有序，可以用稀疏索引进一步优化
    - 预处理，立方体建模：维度爆炸
  - OLAPServer：自服务
    - 关系模型，数据文件稀疏索引定位数据段，列存储
  - CLickhouse
    - 选择了实时查询，同时保证查询性能
- 名字含义：click 点击事件 click stream data warehouse
- 场景：使用与商业智能领域
- 不适用：不支持事务，不擅长按照行粒度查询，不擅长按行删除数据

# 第二章 clickhouse 架构概述
- 核心特性
  - 完备的DBMS功能（database management system数据库管理系统）
    - DDL数据定义语言 ,DML数据操作语言 ,权限控制 ,数据备份和恢复 ,分布式管理
  - 列式存储与数据压缩
    - 列式存储。对数据压缩的友好性，数据中重复想越多，压缩率越高，减少数据扫描的数量，只要选定的列
    - 降低IO和存储的压力，支持向量化执行
  - 向量化执行引擎
    - 寄存器硬件层面的特性，为性能带来指数级的提升。
    - 消除程序中循环的优化，多线程执行，在cpu寄存器层面实现数据的并行操作
    - cpu的SIMD指令，单条指令操作多条数据
    - 存储离cpu越近，访问速度越快
    - 对比传统的火山模型（volcano/pipeline model）:一次拉取一条数据，指令解释开销
    - 物化模型：一次处理全部数据，容易oom
    - 向量化/批处理模型 vectorized/batch model
  - 关系模型与SQL查询
    - 星型模型，雪花模型，宽表模型
  - 多样化的表引擎
  - 多线程和分布式
    - 线程级并行，由更高层次的软件层控制
    - 在数据存取方柏霓，支持分区，也支持分片
  - 多主架构：Multi-Master
  - 在线查询：对查询快速响应，无需预处理加工
  - 数据分片和分布式查询：
    - 本地表和分布式表的概念
- 架构设计
  - column与field
    - column：列对象，接口IColumn，实现ColumnString，..
    - Field: 单值对象，聚合的设计模式，Null,Uin64等数据类型和相应的处理逻辑
  - DataType
    - 负责数据的序列化和反序列化，IDataType使用了泛化的设计模式
    - 数据的读取由Column或Field对象获取
  - Block与Block流
    - Block对象：数据对象，数据类型，列名称组成的三元组。Column,DataType,列名称字符串
    - IBlockInputStream,IBlockOutputStream
  - table
    - IStorage接口指代数据表，定义了DDL和read和write方法
    - 根据AST查询语句的只是要求，返回指定列的原始数据
  - Parser与Interpreter
    - parser分析器负责创建AST对象
    - Interpreter解释器负责解释AST，并进一步拆功能键查询的执行管道
  - Functions与Aggregate Functions
    - 采用向量化的方式直接作用与一整列数据
    - 聚合函数有状态，支持序列化与反序列化，能够在分布式节点之间进行传输，实现增量计算
  - Cluster与Replication
    - 集群由分片（shard）与副本（Replica)
- 为什么快：自下向上设计
  - 在硬件上设计：在内存中group by，并且使用hashTable装载数据
  - 算法在前：选用性能最好的算法
  - 特定场景，特殊优化：更对不同场景，选用不同算法

# 第三章 安装和部署


# 第四章 数据定义
## 4.1 数据类型
- 基础类型
  - 数值类型：整数Int8/16/32/64，UInt,浮点数Float32/64,正无穷inf，非数字nan，定点数Decimal32/64/128/(P,S)
  - 字符串类型：String,FixedString：固定长度，UUID：32为位，主键类型
  - 时间类型：DateTime,DateTime64,Date
    - 毫秒，微秒只能借助Uint实现
- 复合类型
  - Array
    - `select array(1,2)` or `[1,2]`
    - 同一个数组内可以包含多种类型，但各个类型之间必须兼容
    - 定义表字段时，数据需要指定元素类型
  - Tuple
    - 元素之间可以不同数据类型，不兼容
    - `tuple(1,'a',now())` or `(1,2.0,null,now())`
    - 定义表字段时，要指定类型
    - 写入的时候进行类型检查 `Tuple_TEST values (('abc',123))`
  - Enum
    - `c1 Enum8('ready'=1,'start'=2)`
    - 唯一性
  - Nested：嵌套类型
    - 表字段嵌套层级只支持一级
    - 嵌套类型本质是一种多维数组
    - 同一行内每个数组的字段的长度必须相等
- 特殊类型
  - nullable: 和基础类型搭配使用
  - Domain:域名类型，IPv4,IPv6
    - 便捷性：格式检查；性能更好
    -不支持隐式类型转换：IPv4NumToString

## 4.2 定义数据表
- 数据库
  - `CREATE DATABASE IF NOT EXISTS db_name [ENGINE = engine]`
  - Ordinary默认引擎
  - `DROP DATABASE [IF EXISTS] db_name`
- 数据表
  - 常规定义法
```sql
--常规定义法
CREATE TABLE [IF NOT EXISTS] [db_name.]table_name ( 
    name1 [type] [DEFAULT|MATERIALIZED|ALIAS expr], 
    name2 [type] [DEFAULT|MATERIALIZED|ALIAS expr], 
    省略… 
    ) ENGINE = engine
--复制
CREATE TABLE [IF NOT EXISTS] [db_name1.]table_name AS [db_name2.] table_name2 [ENGINE = engine]
--支持在不同数据库之间复制

--select子句
CREATE TABLE [IF NOT EXISTS] [db_name.]table_name ENGINE = engine AS SELECT …

--DESC查询可以返回表的定义结构

--删除
DROP TABLE [IF EXISTS] [db_name.]table_name
```
- 默认值表达式: default,meterialized,alias
  ```sql
  CREATE TABLE dfv_v1 ( 
      id String, 
      c1 DEFAULT 1000, 
      c2 String DEFAULT c1 
  ) ENGINE = TinyLog
  ```
  - 数据写入：只有default类型的字段可以出现在insert语句中，而MATERIALIZED和ALIAS都不能被显式赋值，它们只能依靠计算取值
  - 数据查询：MATERIALIZED和ALIAS类型的字段不会出现在SELECT *查询的返回结果集中
  - 数据存储：只有DEFAULT和MATERIALIZED类型的字段才支持持久化
  - 修改动作不会印象数据表内先前已经存在的数据

- 临时表 
  - 在普通表的基础上添加temporary 关键词
  - 特性
    - 生命周期是会话session绑定的，所以它只支持Memory表引擎，会话结束，数据表就会被销毁
    - 临时表不属于任何数据库，所以在它的建表语句中，既没有数据库参数也没有表引擎参数
    - 临时表的优先级是大于普通表的
    ```sql
    CREATE TEMPORARY TABLE [IF NOT EXISTS] table_name ( 
    name1 [type] [DEFAULT|MATERIALIZED|ALIAS expr], 
    name2 [type] [DEFAULT|MATERIALIZED|ALIAS expr], )
    ```
- 分区表
  - 提高查询性能，数据的纵向切分
  - 只有合并书MergeTree家族的表引擎才支持数据分区
  ```sql
  CREATE TABLE partition_v1 ( 
  ID String, 
  URL String, 
  EventTime Date 
  ) ENGINE = MergeTree() 
  PARTITION BY toYYYYMM(EventTime) 
  ORDER BY ID
  ```
- 视图
  - 普通：只是一层简单的查询代理:`CREATE VIEW [IF NOT EXISTS] [db_name.]view_name AS SELECT ...`
  - 物化视图: 有独立的村塾，数据保存形式有表引擎决定
  ```sql
  CREATE [MATERIALIZED] VIEW [IF NOT EXISTS] [db.]table_name [TO[db.]name] [ENGINE = engine] [POPULATE] AS SELECT
  ```
    - 源表写入新数据，物化视图也会同步更新
    - populate决定了初始化测录入，已存在的数据一并导入
    - 不支持同步删除
    - 删表 `drop table view_name`'

## 4.3 数据表的基本操作
目前只有MergeTree、Merge和Distributed这三类表引擎支持ALTER查询
- 追加新字段
```sql
ALTER TABLE tb_name ADD COLUMN [IF NOT EXISTS] name [type] [default_expr] [AFTER name_after]

ALTER TABLE testcol_v1 ADD COLUMN OS String DEFAULT 'mac'

ALTER TABLE testcol_v1 ADD COLUMN IP String AFTER ID
```
- 修改数据类型
```sql
ALTER TABLE tb_name MODIFY COLUMN [IF EXISTS] name [type] [default_expr]
--类型需要兼容
ALTER TABLE testcol_v1 MODIFY COLUMN IP IPv4
```
- 修改备注 comment
- 删除已有字段 drop
- 移动数据表 `RENAME TABLE [db_name11.]tb_name11 TO [db_name12.]tb_name12` 只能单节点
- 清空数据表 `TRUNCATE TABLE [IF EXISTS] [db_name.]tb_name`

## 4.4 数据分区的基本操作
- 查询分区信息 `SELECT partition_id,name,table,database FROM system.parts WHERE table = 'partition_v2'`
- 删除指定分区 `ALTER TABLE tb_name DROP PARTITION partition_expr`
- 复制分区数据 `ALTER TABLE B REPLACE PARTITION partition_expr FROM A`
  - 有相同的分区键，表结构完全相同
- 重置分区数据: 初始值 `ALTER TABLE tb_name CLEAR COLUMN column_name IN PARTITION partition_expr`
- 卸载和装载分区:分区数据迁移和备份 `ALTER TABLE tb_name DETACH PARTITION partition_expr`, `ALTER TABLE tb_name ATTACH PARTITION partition_expr`
- 备份和还原分区，freeze和fetch

## 4.5 分布式DDL执行
- DDL语句支持分布式执行，`on cluster cluster`

## 4.6 数据的写入
```sql
INSERT INTO [db.]table [(c1, c2, c3…)] VALUES (v11, v12, v13…), (v21, v22, v23…), ...
-- ，c1、c2、c3是列字段声明，可省略
-- 在使用VALUES格式的语法写入数据时，支持加入表达式或函数

INSERT INTO [db.]table [(c1, c2, c3…)] FORMAT format_name data_set
INSERT INTO partition_v2 FORMAT CSV \ 
    'A0017','www.nauu.com', '2019-10-01' \ 
    'A0018','www.nauu.com', '2019-10-01'

INSERT INTO [db.]table [(c1, c2, c3…)] SELECT ...
```
- 表达式和函数会带来额外的性能开销
- 数据操作式面向Block数据块的，每个数据块最多可以写入1048576行数据（由max_insert_block_size参数控制）具有原子性

## 4.7 数据的删除与修改
- delete和update能力，是mutation查询
- 更适合批量数据的修改和删除
- 不支持事务，一旦提交，就会对现有数据产生影响，不能回滚
- 异步的后台过程，具体执行进度，查询过system.mutations系统表
- is_done =1 表示执行完毕
- 等到mergertree引擎下一次合并动作出发时候，非激活目录才会物理上被删除
- update不能修改分区键和主键

# 第五章 数据字典

用键值和属性映射的形式定义数据，字段中的数据会主动或被动加载到内存，并支持动态更新。适合常量，避免给不必要的join(如根据ID转名称)

- 内置字典
  - 只有Yandex.Metrica字典，但不开放。只是提供了字典的定义机制和取数函数
  - 配置说明
    - 默认禁止，开启将config.xml文件中path_to_regions_hierarchy_file和path_to_regions_names_files
    - path_to_regions_hierarchy_file等同于区域数据的主表：由1个regions_hierarchy.txt和多个regions_hierarchy_[name].txt区域层次的数据文件共同组成
    - path_to_regions_names_files等同于区域数据的维度表，记录了与区域ID对应的区域名称
  - 使用内置字典
    - `mkdir /opt/geo`
    - 复制数据到目录下
    - 打开配置
    - `SELECT regionToName(toUInt32(20009))`
- 外部扩展字典
  - 以插件形式注册
  - 准备字典数据：csv格式，用于flat,hashed,cache,complex_key_hashed,complex_key_cache,range_hashed,ip_trie
  - 扩展字典配置文件的元素组成
    - 由config.xml文件中的dictionaries_config 指定，默认识别/etc/clickhouse-server目录下所有以_dictionary.xml结尾的配置
    ```xml
    <?xml version="1.0"?> 
    <dictionaries> 
      <dictionary>
        <name>dict_name</name>
        <structure><!—字典的数据结构 --></structure> 
        <layout> <!—在内存中的数据格式类型 --> </layout> 
        <source> <!—数据源配置 --> </source>
        <lifetime> <!—字典的自动更新频率 --> </lifetime>
       </dictionary> 
    </dictionaries>
    ```
  - 数据结构：structure
    - 由key 和属性attribute 组成 `<structure> <!— <id> 或 <key> --> <id><!—Key属性--> </id> <attribute> <!—字段属性--> </attribute> ... </structure>`
    - key：至少一个
      - 数值型：支持flat,hashed、range_hashed和cache类型的字典 `<id><!—名称自定义--> <name>Id</name> </id>`
      - 复合型：使用tuple 定义 仅支持complex_key_hashed、complex_key_cache和ip_trie类型的字典`<key><attribute> <name>field1</name> <type>String</type> </attribute> <attribute> <name>field2</name> <type>UInt64</type> </attribute> 省略… </key>`
    - attribute
      ```
      <attribute> 
      <name>Name</name> 
      <type>DataType</type> 
      <!—空字符串--> 
      <null_value></null_value> 
      <expression>generateUUIDv4()</expression> 
      <hierarchical>true</hierarchical> 
      <injective>true</injective> 
      <is_object_id>true</is_object_id> 
      </attribute>
      ```
  - 扩展字典的类型
    - flat: 性能最好，只是用Uint数值型key，内存中用数组保存，最多500000行数据
      - 检查 ` SELECT name, type, key, attribute.names, attribute.types FROM system.dictionaries`
    - hashed ；Uint64数值型key,内存中使用散列结构保存，没有存储上限制
    - range_hashed: 增加了指定时间区间的特性，以散列结构存储并按照时间排序，用range_min和range_max元素指定
    - cache: Uintkey,内存中用固定长度的向量数组保存/cells,由 size_in_cells指定，必须是2的整数被
      - 不会一次将所有数据载入内存，性能不稳定，取决与缓存的命中率
      - 使用本地数据做数据源，必须用executable形式
    - complex_key_hashed: 复合型key
    - complex_key_cache:
    - ip_trie: 复合型key,指定单个String类型字段，用于指代IP前缀，内存中用trie树结构保存
  - 数据源
    - 文件类型
      - 本地文件：path用绝对路径，用file定义
      - 可执行文件：command 用cat 访问绝对路径，使用exectable定义
      - 远程文件：用http元素，url,相当于post请求
    - 数据库类型
      - mysql
      - clickhouse:TinyLog
      - MongoDB
    - 其他类型
  - 数据更新策略：lifetime,min,max
    - 在数据更新的过程中，旧版本的字典将持续提供服务,不支持增量更新
    - 当min和max都是0的时候，将禁用字典更新
  - 基本操作
    - 元数据查询：`SELECT name, type, key, attribute.names, attribute.types, source FROM system.dictionaries`
    - 数据查询 `SELECT dictGet('test_flat_dict','name',toUInt64(1))`, `SELECT dictGet('test_ip_trie_dict', 'asn', tuple(IPv4StringToNum('82.118.230.0')))`
    - 字典表
    ```sql
    CREATE TABLE tb_test_flat_dict ( 
    id UInt64, 
    code String, 
    name String 
    ) ENGINE = Dictionary(test_flat_dict);
    ```
    - 使用DDL查询创建
    ```sql
    CREATE DICTIONARY test_dict( 
    id UInt64, 
    value String 
    )PRIMARY KEY id 
    LAYOUT(FLAT()) 
    SOURCE(FILE(PATH '/usr/bin/cat' FORMAT TabSeparated)) 
    LIFETIME(1)
    ```

## 第六章 MergeTree 原理解析





#  第七章 mergeTree系列表引擎

## 7.3 SummingMergeTree
- 场景
  - 终端用户只需要查询数据的汇总结果， 不关心明细数据，并且数据的汇总条件是预先明确的（GROUP BY条件明确，且不会随意改变）
  - 直接方案：MergeTree 存储，使用group by 查询，用sum聚合函数汇总结果
    - 存在额外的存储开销
    - 额外的查询开销
- 定义: 能够在合并分区的时候按照预先定义的条件聚合汇总数据，将同一分组下的多行数据汇总合并成一行，这样既减少了数据行，又降低了后续汇总查询的开销
  - 主键索引，可以和order by 不一样
  - `ENGINE = SummingMergeTree((col1,col2,…))`如若不填写此参数，则会将所有非主键的数值类型字段进行SUM汇总
  - 触发汇总: `optimize TABLE summing_table FINAL`
```sql
CREATE TABLE summing_table( 
    id String, 
    city String, 
    v1 UInt32, 
    v2 Float64, 
    create_time DateTime 
)ENGINE = SummingMergeTree()
PARTITION BY toYYYYMM(create_time) 
ORDER BY (id, city) 
PRIMARY KEY id
```
- 处理逻辑
  - 用ORBER BY排序键作为聚合数据的条件Key
  - 只有在合并分区的时候才会触发汇总的逻辑
  - 数据分区为单位来聚合数据。当分区合并时，同一数据分区内聚合Key相同的数据会被合并汇总，而不同分区之间的数据则不会被汇总
  - 如果在定义引擎时指定了columns汇总列（非主键的数值类型字段），则SUM汇总这些列字段；如果未指定，则聚合所有非主键的数值类型字段
  - 在进行数据汇总时，因为分区内的数据已经基于ORBER BY排序，所以能够找到相邻且拥有相同聚合Key的数据
  - 对于那些非汇总字段，则会使用第一行数据的取值
  - 支持嵌套结构，但列字段名称必须以Map后缀结尾。嵌套类型中，默认以第一个字段作为聚合Key。除第一个字段以外，任何名称以Key、Id或Type为后缀结尾的字段，都将和第一个字段一起组成复合Key

## 7.4 AggregatingMergeTree
- 引子：数据立方体，空间换时间的方式提升查询性能
- 定义：合并分区的时候，按照预先定义的条件聚合数据以二进制的形式存储中间状态结果
- 配置
  - 使用何种聚合函数，以及针对哪些列字段计算，定义AggregateFunction数据类型实现的
  - 写入需要调用*State函数；查询时，则需要调用相应的*Merge函数；*表示定义时使用的聚合函数
  - `uniqState('code1')`,`SELECT id,city,uniqMerge(code),sumMerge(value) FROM agg_table`
- 规则
  - 用ORBER BY排序键作为聚合数据的条件Key
  - 使用AggregateFunction字段类型定义聚合函数的类型以及聚合的字段
  - 只有在合并分区的时候才会触发聚合计算的逻辑
  - 以数据分区为单位来聚合数据
  - 在进行数据计算时，因为分区内的数据已经基于ORBER BY排序，所以能够找到那些相邻且拥有相同聚合Key的数据
  - 非主键、非AggregateFunction类型字段，则会使用第一行数据的取值
  - AggregateFunction类型的字段使用二进制存储，在写入数据时，需要调用*State函数；而在查询数据时，则需要调用相应的*Merge函数。其中，*表示定义时使用的聚合函数
  - AggregatingMergeTree通常作为物化视图的表引擎，与普通MergeTree搭配使用
- 后面分析 SimpleAggregateFunction和AggregateFunction的区别
```sql
CREATE TABLE agg_table( 
    id String, city String,
     code AggregateFunction(uniq,String), 
     value AggregateFunction(sum,UInt32), 
     create_time DateTime 
 )ENGINE = AggregatingMergeTree() 
PARTITION BY toYYYYMM(create_time) 
ORDER BY (id,city) 
PRIMARY KEY id

CREATE MATERIALIZED VIEW agg_view 
ENGINE = AggregatingMergeTree() 
PARTITION BY city ORDER BY (id,city) 
AS SELECT 
          id, 
          city, 
          uniqState(code) AS code, 
          sumState(value) AS value 
FROM agg_table_basic 
GROUP BY id, city
```