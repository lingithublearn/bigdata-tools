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
  - 改表名： rename tabel tbName to tbName on cluster cluster
    - 结果是新的表的zookeeper里面的表元数据存储位置是没有改变的
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
