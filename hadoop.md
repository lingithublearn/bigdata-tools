
- linux 
	- sh xxx.sh > /dev/null 2>&1 &
	- tail -f 
	- 定时任务
		- 启动 crontab -e 网页版本crontab验证即可
		- 删除 `00 02 * * * find /home/datadir -name "*" -mtime +7 -exec rm -rf {} \;`
	- less
		- 搜索 / 向上N,向下n
		- 翻页 向下空格，向上b
		- 置顶g，到尾巴G 
	- shell 脚本
		- `#!/bin/bash` 第一行
		- 多行编辑 ` \`
		- 时间命令 `date +"%Y%m%d-%H%M%S"` 脚本中需要%转义，命令需要`
		- crontab -e 中配置date的时候，需要对%,需要使用\进行转义
	- vim
		- (替换)[http://xstarcd.github.io/wiki/vim/vim_replace_encodeing.html]
		- :%s/vivian/sky/(等同于 :g/vivian/s//sky/) 替换每一行的第一个 vivian 为 sky
	- find
		- find / -name tnsnames.ora
	
	- 验证连接
		- telnet ip port
		- wget ip:port 用http连接测试，相当于一个get请求
		- ping ip 测试网络是否通，有没有丢包
	- 服务
		- 在目录  /etc/systemd/system 中用root创建 xx.service
		- 赋予权限
		- system status xx.service
		- systemctl start xx.service
		- systemctl daemon-reload
		- journalctl -u hdfsToCK_lte.service 检查日志
	```[Unit]
	Description= hdfsToCK_lte service
	After=network.target

	[Service]
	User=hdfs
	WorkingDirectory=/usr/local/wangyou/0804Spark/lte
	ExecStart=/usr/local/wangyou/0804Spark/lte/hdfsToCK_lte.sh
	Restart=always

	[Install]
	WantedBy=multi-user.target
	```
- hdfs
	- hadoop fs -ls 
	- hadoop fs -cat (对特殊的压缩包 如 gzip的 ：|zat)
	- hadoop fs -du -h 
	- hadoop fs -put/-get
	- NameNode管理文件系统的命名空间。它维护着文件系统树及整棵树内所有的文件和目录
	- NameNode用来存储一些元数据信息的，而DataNode却是用来存放真实数据的
	- hadoop多节点部署最少可以是2个节点，即一个NameNode和一个DataNode。
	- ZooKeeper是一个独立的组件，它可以和HDFS配合使用，但没有非得部署在一起的要求，只要网络通就可以。另外，ZooKeeper建议最少安装在3个节点上，且数目为奇数。
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
