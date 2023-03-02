#  大数据工具记录
# 1. hadoop
- linux 
	- sh xxx.sh > /dev/null 2>&1 &
	- 定时启动 crontab -e 网页版本crontab验证即可
	- 时间命令 `date +"%Y%m%d-%H%M%S"` 脚本中需要%转义，命令需要`
	- 定时删除 `00 02 * * * find /home/datadir -name "*" -mtime +7 -exec rm -rf {} \;`
	- shell 脚本
		- `#!/bin/bash` 第一行
		- 多行编辑 ` \`
- hdfs
	- hadoop fs -ls 
	- hadoop fs -cat (对特殊的压缩包 如 gzip的 ：|zat)
	- hadoop fs -du -h 
	- hadoop fs -put/-get
- yarn
	- yarn命令行
		- yarn top | grep xxx
		- yarn application -list | grep queue
		- yarn applicaiton -kill applicationId
	- yarn ui 
		- hadoop ui IP:8088
			- RM Home
			- Scheduler/ running
		- 监控 ui ip:50070
			- Utilities
				- file system
# 2. spark
- spark on yarn 
- spark 1.*
- spark 2.*
	- [spark 2.4.5 官网](https://spark.apache.org/docs/2.4.5/index.html)
	- spark session
	- spark dataset
- spark SQL
	- 官网的[sql function](https://spark.apache.org/docs/2.4.5/api/sql/index.html)，注意since from *.*版本限制
	- 环境的版本可以使用 spark-shell查看
- 启动参数
- 调优

# 3. Flink
# 4. SQL
- hive
- clickhouse
- hbase
