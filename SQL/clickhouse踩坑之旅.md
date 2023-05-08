#1. Clickhouse 实时更新-数据去重（原子性）
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
  坑点：
    - 数据的去重只会在数据合并期间进行。合并会在后台一个不确定的时间进行，因此你无法预先作出计划。有一些数据可能仍未被处理。尽管你可以调用 OPTIMIZE 语句发起计划外的合并，但请不要依靠它，因为 OPTIMIZE 语句会引发对数据的大量读写。
    - 理解：去重不定时，不完全，不可靠
  - final 关键词
  坑点：
    - 配合Replacing 表结构使用，会针对主键（组合索引）进行去重
    - 理解：分区键没同时写order by里，不算主键，没构建主键索引，会被去重，会去多了
  - group by+ argMax(a,b)
  坑点：
    - 需要增加一个字段，或者有字段能区分新旧数据，类似版本号的结果
    - 理解：程序耦合，增加字段工作量大就G
  - alter table delete where
  坑点：
    - 异步删除的，相当于后台给你新增一个标志位，类似”-1“,等程序删除，查询不可见
    - 理解：新入的数据，如果有相同的，会被视为冗余数据，被删除。结果是不同的入了，相同的被删了
