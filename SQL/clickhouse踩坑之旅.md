# 1. Clickhouse 实时更新-数据去重（原子性）
  坑点：
  - RepacingMergerTree 表结构
  
  ```
  CREATE TABLE lzm_test.insert_test
(

    `datatime` DateTime COMMENT '数据时间',

    `a` Int64,

    `b` String,

    `c` Int64
)
ENGINE = ReplicatedReplacingMergeTree('/data/clickhouse/data/{shard}/lzm_test/insert_test',
 '{replica}')
PARTITION BY datatime
ORDER BY (datatime,a)
TTL datatime + toIntervalMonth(6)
SETTINGS index_granularity = 8192
  ```
  
    - 数据的去重只会在数据合并期间进行。合并会在后台一个不确定的时间进行，因此你无法预先作出计划。有一些数据可能仍未被处理。尽管你可以调用 OPTIMIZE 语句发起计划外的合并，但请不要依靠它，因为 OPTIMIZE语句会引发对数据的大量读写。
    - 理解：去重不定时，不完全，不可靠
  - final 关键词 `select * from tbName fianl where datatime = ''`
    - 配合Replacing 表结构使用，会针对主键（组合索引）进行去重
    - 理解：分区键没同时写order by里，不算主键，没构建主键索引，会被去重，会去多了
  - group by+ argMax(a,b)
    - 需要增加一个字段，或者有字段能区分新旧数据，类似版本号的结果
    - 理解：程序耦合，增加字段工作量大就G
  - alter table delete where
    - 异步删除的，相当于后台给你新增一个标志位，类似”-1“,等程序删除，查询不可见
    - 理解：新入的数据，如果有相同的，会被视为冗余数据，被删除。结果是不同的入了，相同的被删了

# 2. 重建表，不删除数据的情况下
- 背景
  - 分区键，需要设置进入索引，索引修改只满足新增列
  - 只能删表重建，但补充数据耗时太长，所以想用rename，insert等方法

- 测试方法
  - 新建表，create xxx
  - 转移数据：insert table newtb select * from oldtb where xxx
    - 看数据量，可能时间比较长
    - 可能不能使用on cluster cluster ，需要从视图表中查询全量的，数据的转移量比较大
  - 删掉原表，drop table tbname on cluster cluster
    - 可能存在数据量超过限制
      - 如果仍然需要在不重新启动ClickHouse服务器的情况下删除表，请创建 clickhouse-path/flags/force_drop_table 文件并运行DROP查询
      - touch /clickhouse/flags/force_drop_table
      - chown clickhouse:clickhouse /clickhouse/flags/force_drop_table
      - chmod 666 /clickhouse/flags/force_drop_table
      - 执行drop语句，执行完后，这个文件会被删除
  - 改表名： rename tabel tbName to tbName on cluster cluster
    - 结果是新的表的zookeeper里面的表元数据存储位置是没有改变的
    - 复制表需要在zookeeper上新建一个路径来保存，默认的引擎室atomic，需要480s以后才能重建表
    - 影响的应该是，新建表的时候，不能再使用这个路径了
  - 视图表不用修改，只是查询的作用

# 3. 分布式ddl，节点返回失败
- 报错
``` Watching task /clickhouse/task_queue/ddl/query-0000010620 is executing longer than distributed_ddl_task_timeout (=180) seconds. There are 2 unfinished hosts (0 of them are currently active), they are going to execute the query in background (version 21.3.4.25 (official build))
```
- 测试
  - show processlist FORMAT Vertical
- 可能原因
  - 不能识别自身 `SELECT * FROM system.clusters;`
  - 历史任务有错，卡住
  
  ```
  SELECT * FROM system.zookeeper WHERE path = '/clickhouse/task_queue/ddl/';
  SELECT * FROM system.zookeeper WHERE path = '/clickhouse/task_queue/ddl/query-0000001000/';
  SELECT * FROM system.zookeeper WHERE path = '/clickhouse/task_queue/ddl/' AND name = 'query-0000001000';
  https://kb.altinity.com/altinity-kb-setup-and-maintenance/altinity-kb-ddlworker/there-are-n-unfinished-hosts-0-of-them-are-currently-active/
  
  ```
  
 # 4. Replicated*MergeTree 数据副本
 - 副本是表级别的
 - 复制是多主异步。 INSERT 语句（以及 ALTER ）可以发给任意可用的服务器。数据会先插入到执行该语句的服务器上，然后被复制到其他服务器。由于它是异步的，在其他副本上最近插入的数据会有一些延迟。如果部分副本不可用，则数据在其可用时再写入。
 - 数据块会去重。对于被多次写的相同数据块（大小相同且具有相同顺序的相同行的数据块），该块仅会写入一次。（注意：Replicated*MergeTree 才会去重，不需要 zookeeper 的不带 MergeTree 不会去重）
 - ReplicatedMergeTree('/clickhouse/tables/{shard}/{database}/table_name', '{replica}')
 - 删除
    - 删除元数据目录中的相应 .sql 文件（/var/lib/clickhouse/metadata/）。
    - 删除 ZooKeeper 中的相应路径（/path_to_table/replica_name）。
  
# 5. 窗口函数
  - v21.9版本正式支持，早期的版本需要借助array函数
  
# 6. UDF
  ```CREATE FUNCTION {fn_name} as ({parameters}) -> {code} 
    create function testFunc as (a,b) -> (a+b)
     select testFunc(1,2)
  在data/user_defined 目录下保存了UDF的源数据
  也可以用drop function 删除UDF
  select * from system.functions where name = 'testFunc'
  ```
  家里是20.8，山东现场时21.3
  不支持create function + lamda，应该要21.10
  可执行的UDF，最低要v21.11
  
 - 用户自定义外部函数
    -  在config.xml中增加`<user_defined_executable_functions_config>*_function.xml</user_defined_executable_functions_config>`
    - 在对应的xml中自定义声明文件
    - 声名方法
    - 编写python 脚本
      - 处理多行输入：`for line in sys.stdin:` + `line`
      - 注意数据输入和输出格式：一般为TabSeparaterd或者JSONEachRow
      - python方法;sys.stdin 测试通过先
      - 注意tab和空格的区别，不能混用
    - 方法测试 `system reload functions   select * from system.functions where name = ''`
    - 查看报错`set send_logs_level='error';` 可以是trace
  
  
# 7 位图
  - 简介
    - 优点：运算效率高，占用内存小
    - 缺点：数据稀疏时，浪费空间
    - 需要一种高效的压缩算法，来解决空间浪费问题
  - 位图压缩算法-roaringBitmap
    - 思路：根据32位无符号整数，根据高16位对数据进行分桶，将数据放入container
    - 底层数据结构
      - ArrayContainer:当容量超过4096，会将container转换为bitmap container
      - BitmapContainer
      - runContainer
      - 算法会根据容量自动选择数据结构
  - 于clickhouse的实践
    - 官方的驱动
    - bitmap字段使用AggregateFunction(groupBitmap,UInt32)类型，可以存储位图数据
    - 大批量写入CK的官方推荐写法
    - 位图类型转换，ck包自带的类型
    - 性能测试
  - 使用场景
    - 基站用户快照：亿级别的数据
    - 用户数据过滤
      - 位运算
    - 数据集合的去重，交并集计算
  - 常见的SQL
    - bitmapToArray() : 转换
    - groupBitmapMergeState() 合并去重
    - bitmapCardinality() 函数用于返回位图中的不同数字的数量
  - bug
    - 超过32位无法显示——修改序列化方式
    - long类型的整数需要用到有符号的Int64类型
  - 问题
    - 社区
    - 怎么查 

# 8 物化视图
- 


# 9 性能
- 单个大查询的吞吐量
  - 一个不太复杂的查询在单个服务器上大约能够以2-10GB／s（未压缩）的速度进行处理（对于简单的查询，速度可以达到30GB／s）
  - 如果数据没有在page cache中的话，那么速度将取决于你的磁盘系统和数据的压缩率。例如，如果一个磁盘允许以400MB／s的速度读取数据，并且数据压缩率是3，则数据的处理速度为1.2GB/s
  - 对于分布式处理，处理速度几乎是线性扩展的，但这受限于聚合或排序的结果不是那么大的情况下
- 处理短查询的延迟时间
  - 如果一个查询使用主键并且没有太多行(几十万)进行处理，并且没有查询太多的列，那么在数据被page cache缓存的情况下，它的延迟应该小于50毫秒(在最佳的情况下应该小于10毫秒)
  - 在数据没有加载的情况下，查询所需要的延迟可以通过以下公式计算得知： 查找时间（10 ms） * 查询的列的数量 * 查询的数据块的数量
- 处理大量短查询的吞吐量
  - 在相同的情况下，ClickHouse可以在单个服务器上每秒处理数百个查询（在最佳的情况下最多可以处理数千个）。但是由于这不适用于分析型场景。因此我们建议每秒最多查询100次。
- 写入性能
  - 建议每次写入不少于1000行的批量写入，或每秒不超过一个写入请求。当使用tab-separated格式将一份数据写入到MergeTree表中时，写入速度大约为50到200MB/s
- 为什么快
  - 列式存储：相同列的数据，数据的类型相同，压缩和解压缩的

  
  
  
  
  
