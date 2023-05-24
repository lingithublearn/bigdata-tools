
#  常用函数操作
- 行转列：多行转为一行:concat_ws 字符串拼接
    -  `concat_ws(',',collect_set(c))` 数据去重
    -  `concat_ws(',',collect_list(c))` 数据不去重
- 列转行  
    - `explode（expr）` 用于处理array 和map结构的数据，分成多行
- lateral view 子句
    - 其实explode是一个UDTF函数（一行输入多行输出），这个时候如果要select除了explode得到的字段以外的多个字段，需要创建虚拟表
    - `select uid, game from user_game LATERAL VIEW explode(split(game_list,",")) tmpTable as game`
    - 对于string,使用split进行切割，获得数组
    - game 是给 explode(split(game_list,",")) 列起的别名
- 排序
    - order by ：保证最终数据的顺序，还可以按照分区来排序
    - distribute by 对表进行重新分区
    - sort by   对分区内进行排序，分区不止一个时，回返回部分排序结果 `SELECT  name, age, zip_code FROM person SORT BY name ASC, age DESC;`
    - cluster by 对数据重分区，每个分区内数据排序 = 先distribute by 后 sort by,不保证数据的总顺序
- 对连续性数目的校验，转化成对第一个不连续的排序值进行校验
    - 日期，减去排序的序号，对结果分组
        - 对满足条件的及结果进行排序的，相当于删去不满足条件的记录，不连续的会分布在其他日期
    - 按照日期排序，
        - rank() 不连续的排序 over（partition by xx order by xx）
        - dense_rank() 连续的排序
        - row_number() 行号
    - 条件筛选后，用min()或者max() 对排序值进行筛选即可
    - 错行开窗
        - lag lead
- 直接定位星期二
    - sqlserver datename(dw,getDate(n))
- 窗口函数
    - 语法：`窗口函数` `over` `窗口定义`
    - 窗口定义
        - partition分区：按照哪个key分组
        - order 排序信息，组内数据如何排序
        - frame窗框定义：分组排序好的数据上，如何划定窗框，
    - frame 窗框
        - RowType根据行偏移的范围来划定窗口, 对应语句中是`rows between xx and xx`这类的  
        - RangeType根据列的值的范围来划定窗口, 对应语句中是`range between xx and xx`这类的
            - 举例来说, 如果有个股票价格表记录各支股票每天的价格, 如果要计算MA30(30日均线值)的话, 就可以`select *, avg(price) over (partition by stock_id order by dt range between 30 PRECEDING and current row) --假定dt是数值表示的`
        - 5种边界值
            - UNBOUNDED PRECEDING: 无限往前, 或者说从无限小/从第一行开始, 无上界. 对于RowType和RangeType是一个效果的.
            - UNBOUNDED FOLLOWING: 无限往后, 或者说是直到最后一行, 无下界. 对于RowType和RangeType是一个效果的
            - offset PRECEDING: offset是一个数值 如30, 当是RowType时, 表示从当前行往前30行开始. 当RangeType时, 表示从当前行OrderBy那列的值减30开始(所以必须是能做减法的数值类型), 比如上面的MA30的例子, 是按dt日期排序, 取30天前到现在的范围定义的窗框.
            - offset FOLLOWING: 同上
            - CURRENT ROW: 当前行
```
SELECT name, dept, RANK() OVER (PARTITION BY dept ORDER BY salary) AS rank FROM employees;

SELECT name, dept, DENSE_RANK() OVER (PARTITION BY dept ORDER BY salary ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS dense_rank FROM employees;

SELECT name, salary, LAG(salary) OVER (PARTITION BY dept ORDER BY salary) AS lag,
    LEAD(salary, 1, 0) OVER (PARTITION BY dept ORDER BY salary) AS lead FROM employees;
```

问题
* [9.2、Spark SQL]()
    - [1）、Spark SQL和Hive区别？Spark SQL一定比Hive快么？](#1spark-sqlhivespark-sqlhive)
    - [2）、Spark SQL有使用过么？在哪些项目中使用过？](#2spark-sql)
    - [3）、Spark SQL中UDF使用？](#3spark-sqludf)
    - [4）、SparkSession、SparkContext和SQLContext区别？](#4sparksessionsparkcontextsqlcontext)
    - [5）、Spark SQL用过哪些算子？遇到哪些问题？如何解决的？](#5spark-sql)
    - [6）、Spark SQL程序调优？](#6spark-sql)
    - [7）、Spark SQL运行原理？](#7spark-sql)
    - [8）、Spark SQL适用的场景,Spark Core不适合的?]()
    - [9）、Spark SQL2.0和3.0区别?]()
    - [10）、Spark SQL的DataFrame和RDD有啥区别?]()

###### 1、Spark SQL和Hive区别？Spark SQL一定比Hive快么？
    Spark SQL 比 Hadoop Hive 快，是有一定条件的，而且不是 Spark SQL 的引擎比 Hive 的引擎快，相反，Hive 的 HQL 引擎还比 Spark SQL 的引擎更快。其实，关键还是在于 Spark 本身快。
    1.Shuffle时候spark sql处理数据不一定落盘,而hive是强制写磁盘
    2.spark提供丰富的算子,而hive每次都是一个完整的mr操作
    3.JVM优化,hive每次mr是启动一个进程处理,而spark的Mr是基于线程的

###### 3、Spark SQL中UDF使用？
    spark.sql.function中的已经包含了大多数常用的函数，但是总有一些场景是内置函数无法满足要求的，此时就需要使用自定义函数了(UDF)。
    1、在dataframe中使用：
    // 注册自定义函数
    ss.udf.register("add_one", add_one _)
    // 使用自定义函数
    import org.apache.spark.sql.functions
    val frame = df.withColumn("age2", functions.callUDF("add_one", functions.col("age")))
    // 定义自定义函数
    def add_one(col: Double) = {
        col + 1
    }
    
    2、在sparkSQL中使用
    # 定义自定义函数
    def strLen(col: String) = {
        str.length()
    }
    # 注册自定义函数
    spark.udf.register("strLen", strLen _)
    # 使用自定义函数
    spark.sql("select name,strLen(name) from table ")

###### 4、SparkSession、SparkContext和SQLContext区别？
    SparkContext:驱动程序使用与集群进行连接和通信，它可以帮助执行Spark任务，并与资源管理器(如YARN 或Mesos)进行协调。
    SQLContext:是通往SparkSQL的入口。有了SQLContext，就可以开始处理DataFrame、DataSet等
    HiveContext:是通往hive入口。HiveContext具有SQLContext的所有功能。
    SparkSession:是在Spark 2.0中引入的，替换了旧的SQLContext和HiveContext。

###### 5、Spark SQL用过哪些算子？遇到哪些问题？如何解决的？
    toDF、join、foreach、head等等
    遇到问题：join等聚合操作时候发生数据倾斜
        要区分开数据倾斜与数据量过量这两种情况，
            数据倾斜是指少数task被分配了绝大多数的数据，因此少数task运行缓慢；
            数据过量是指所有task被分配的数据量都很大，相差不多，所有task都运行缓慢。
    解决方案：
        1、聚合原数据
        2、过滤导致倾斜的key：在sql中用where条件
        3、提高shuffle并行度：groupByKey(1000)，spark.sql.shuffle.partitions（默认是200）
        4、reduce join转换为map join：spark.sql.autoBroadcastJoinThreshold
            - 将小表读取后，对大表进行filter操作in小表，小表被传输到大表所在的位置，减少shuffle的开销
        5、随机key与热点数据再分区

###### 6、Spark SQL程序调优？
    1、参数调优：
    spark.sql.shuffle.partitions：并行度
    spark.sql.autoBroadcastJoinThreshold：Join操作时，要被广播的表的最大字节数
    spark.sql.tungsten.enabled：开启tungsten优化
    spark.sql.planner.externalSort：根据需要执行Sort溢出到磁盘上，否则在每个分区内存中
    2、代码调优
    数据缓存、聚合算子

###### 7、Spark SQL运行原理？
    SparkSQL的运行架构：
    sparksql先会将SQL语句进行解析（parse）形成一个Tree,然后使用Rule对Tree进行绑定，优化等处理过程，通过模式匹配对不同类型的节点采用不同操作。
    而sparksql的查询优化器是，它负责处理查询语句的解析，绑定，优化和生成物理执行计划等过程，catalyst是sparksql最核心部分。
    
    Spark SQL由core，catalyst，hive和hive-thriftserver4个部分组成。
    core: 负责处理数据的输入/输出，从不同的数据源获取数据（如RDD,Parquet文件和JSON文件等），然后将结果查询结果输出成Data Frame。
    catalyst: 负责处理查询语句的整个处理过程，包括解析，绑定，优化，生成物理计划等。 
    hive: 负责对hive数据的处理。
    hive-thriftserver：提供client和JDBC/ODBC等接口。
    
    sql实际执行过程：
    1、语法和词法解析：对写入的sql语句进行词法和语法解析（parse），分辨出sql语句在哪些是关键词（如select ,from 和where）,
        哪些是表达式，哪些是projection ，哪些是datasource等，判断SQL语法是否规范，并形成逻辑计划。
    2、绑定：将SQL语句和数据库的数据字典（列，表，视图等）进行绑定（bind）,如果相关的projection和datasource等都在的话，
        则表示这个SQL语句是可以执行的。
    3、优化（optimize）：一般的数据库会提供几个执行计划，这些计划一般都有运行统计数据，数据库会在这些计划中选择一个最优计划。
    4、执行（execute）：执行前面的步骤获取最有执行计划，返回查询的数据集。
    
    运行原理原理分析：
    1.使用SesstionCatalog保存元数据
        在解析sql语句前需要初始化sqlcontext,它定义sparksql上下文，在输入sql语句前会加载SesstionCatalog，
        初始化sqlcontext时会把元数据保存在SesstionCatalog中，包括库名，表名，字段，字段类型等。这些数据将在解析未绑定的逻辑计划上使用。
    2.使用Antlr生成未绑定的逻辑计划
        Spark2.0版本起使用Antlr进行词法和语法解析，Antlr会构建一个按照关键字生成的语法树，也就是生成的未绑定的逻辑计划。
    3.使用Analyzer绑定逻辑计划
        在这个阶段Analyzer 使用Analysis Rules,结合SessionCatalog元数据，对未绑定的逻辑计划进行解析，生成已绑定的逻辑计划。
    4.使用Optimizer优化逻辑计划
        Opetimize（优化器）的实现和处理方式同Analyzer类似，在该类中定义一系列Rule,利用这些Rule对逻辑计划和Expression进行迭代处理，
        达到树的节点的合并和优化。
    5.使用SparkPlanner生成可执行计划的物理计划
        SparkPlanner使用Planning Strategies对优化的逻辑计划进行转化，生成可执行的物理计划。
    6.使用QueryExecution执行物理计划

