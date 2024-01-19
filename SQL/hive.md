# hive入库
- Inserting data into Hive Tables from queries
  - INSERT OVERWRITE TABLE tablename1 [PARTITION (partcol1=val1, partcol2=val2 ...) [IF NOT EXISTS]] select_statement1 FROM from_statement;
    - if not exists 当分区存在的时候，会跳过，避免覆盖
  - INSERT INTO TABLE tablename1 [PARTITION (partcol1=val1, partcol2=val2 ...)] select_statement1 FROM from_statement;
  - 性能上应该是into 更好些，不用检查和删除数据
  - 举例-设置动态分区hive.exec.dynamic.partition.mode=nonstrict
    - insert into table ottvtest.o_se_lte_s1mme_h partition(statis_hour)
      select start_time,imsi,src_eci,dcnr_ue,n1_mode_ue,statis_hour from lcgxc.o_se_lte_s1mme_h
- 写入内部表和外部表
  - 内部表通常提供更多的管理和一致性，但可能会牺牲一些性能，特别是在写入大量数据时
  - 内部表的写入操作通常涉及将数据从临时位置复制到表的存储位置，这可能会导致额外的I/O操作
  - 外部表通常更适合需要更高写入性能和数据自主管理的情况
- 建表
  - 查询 describe formatted xxx ;

# 窗口函数
- lag 返回当前行的前第几行，对null值做处理
- lead 返回当前行的后第几行，对null值做处理
- first
- last