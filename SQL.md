
- hive
	- 进入客户端 hive-client
	- db连接控制：dbeaver
	- 表
		- 内部表
			- 数据在hive中有保存
		- 外部表
			- 数据存储在hdfs，hive只是保存文件目录，引用映射
- clickhouse
	- 分布式表（local，view）
	- [function](https://clickhouse.com/docs/en/sql-reference/functions/)
	- [官网](https://clickhouse.com/) 
	- jdbc
		- 官方 8013，8014 协议http
		- 第三方 端口9000，9001 协议 TCP/IP
	- 命令
		- 删除表数据，truncat table xx_local on CLUSTER cluster
		- 删除分区数据，alter table xx_local on CLUSTER cluster delete where datetime = ''
		- 删除表 drop table xxx on CLUSTER cluster
		- 新建表 
	- 命令行客户端
		- clickhouse-client
		```
		clickhouse-client --param_tuple_in_tuple="(10, ('dt', 10))" -q "SELECT * FROM table WHERE val = {tuple_in_tuple:Tuple(UInt8, Tuple(String, UInt8))}"
		```
		```
		--host, -h -– 服务端的host名称, 默认是localhost。您可以选择使用host名称或者IPv4或IPv6地址。
		--port – 连接的端口，默认值：9000。注意HTTP接口以及TCP原生接口使用的是不同端口。
		--user, -u – 用户名。 默认值：default。
		--password – 密码。 默认值：空字符串。
		--query, -q – 使用非交互模式查询。
		--database, -d – 默认当前操作的数据库. 默认值：服务端默认的配置（默认是default）。
		--multiline, -m – 如果指定，允许多行语句查询（Enter仅代表换行，不代表查询语句完结）。
		--multiquery, -n – 如果指定, 允许处理用;号分隔的多个查询，只在非交互模式下生效。
		--format, -f – 使用指定的默认格式输出结果。
		--vertical, -E – 如果指定，默认情况下使用垂直格式输出结果。这与–format=Vertical相同。在这种格式中，每个值都在单独的行上打印，这种方式对显示宽表很有帮助。
		--time, -t – 如果指定，非交互模式下会打印查询执行的时间到stderr中。
		--stacktrace – 如果指定，如果出现异常，会打印堆栈跟踪信息。
		--config-file – 配置文件的名称。
		--secure – 如果指定，将通过安全连接连接到服务器。
		--history_file — 存放命令历史的文件的路径。
		--param_<name> — 查询参数配置查询参数
		```
		- 输入输出数据
		```
		format CSV 为逗号分割，format TSV为制表符分割
		clickhouse-client --port 9800 --query="insert into dw_perf_lte.tb_dw_perf_lte_cell_kpi_h_inc(column) format CSV" < ./tb_dw_perf_lte_cell_kpi_h_inc.csv

		clickhouse-client --port 9000 --query="select * from dw_perf_lte.tb_dw_perf_lte_cell_kpi_h_inc where format CSV" > ./tb_dw_perf_lte_cell_kpi_h_inc.csv
		```
	- zookeeper
		- zkCli.sh
		- deleteall -path
		- 可以删除错误的副本，replica 
- hbase
- sqlserver
	- 数据导入导出
		- 数据库，右键task，导入数据
		- 数据源excel，选择xlxs格式
		- 目标，sqlserver 用户名，密码，目标表 
