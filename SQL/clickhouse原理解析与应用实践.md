# 第四章 数据定义

## 4.2 如何定义数据表

### 4.2.4 临时表
- 在普通表的基础上添加temporary 关键词
- 特性
  - 生命周期是会话绑定的，所以它只支持Memory表引擎，会话结束，数据表就会被销毁
  - 临时表不属于任何数据库，所以在它的建表语句中，既没有数据库参数也没有表引擎参数
  - 临时表的优先级是大于普通表的
  ```sql
  CREATE TEMPORARY TABLE [IF NOT EXISTS] table_name ( 
  name1 [type] [DEFAULT|MATERIALIZED|ALIAS expr], 
  name2 [type] [DEFAULT|MATERIALIZED|ALIAS expr], )
  ```




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