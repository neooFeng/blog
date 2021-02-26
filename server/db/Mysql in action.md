# Mysql in Action

- What is the difference between Primary Key and Unique Key?  
Answer : Both Primary and Unique Key is implemented for Uniqueness of the column. Primary Key creates a clustered index of column where as an Unique creates unclustered index of column. Moreover, Primary Key doesn’t allow NULL value, however Unique Key does allows one NULL value.

- 一条search语句是如何执行的

- 一条update语句是如何执行的

- redo log & binlog

- MVCC是如何实现的

- InnoDB 索引模型

- 加锁情况分析  
`student
id int pk,
name varchar(50) unique index,
age int index,
nation varchar(10),
...
}`

    - `select * from student where id = 1;`
    没有锁
    
    - `insert into student values(100, "hhah", 18, "汉族");`
    行锁
    
    - `update student set name = 'new name' where name = 'old name';`
    行锁
    
    - `update student set age = 18 where nation = 'not exists';`
    表锁
    
    - `update student set nation = 'test' where age = 18;`
    如果18存在：行锁；如果18不存在：没有锁。where指定唯一索引也是一样
    
    - `update student set name = 'tset' where age = 18;`
    name索引加间隙锁
    
    - `insert into student(id, name, age, nation) select id, name, age, nation from tmp;`
    tmp表锁
    
    - `update student s inner join tmp on s.name = tmp.name set s.nation = 'test';`
    tmp表锁，student行锁和间隙锁
    
    - `update student s inner join tmp on s.nation = tmp.nation set s.age = 18;`
    tmp表锁，student表锁（因为student.nation没有索引）
    
总结：加锁是为了防止数据不一致（比如某些后执行的事务先提交，会导致binlog放到replica上跑之后的结果和master库不一致）  
我总结的加锁规则里面，包含了两个“原则”、两个“优化”和一个“bug”。
* 原则 1：加锁的基本单位是 next-key lock，next-key lock 是前开后闭区间。
* 原则 2：查找过程中访问到的对象才会加锁。
* 优化 1：索引上的等值查询，给唯一索引加锁的时候，next-key lock 退化为行锁。
* 优化 2：索引上的等值查询，向右遍历时且最后一个值不满足等值条件的时候，next-key lock 退化为间隙锁。
* 一个 bug：唯一索引上的范围查询会访问到不满足条件的第一个值为止。

- explain每一列的含义


- 为什么要避免join，什么情况下做join是可以的

- order by是怎么工作的

- group by是怎么工作的

- mysql kill不掉的语句

- mysql服务器值得关注的性能指标


- 
SELECT query, exec_count, total_latency, max_latency FROM sys.statements_with_runtimes_in_95th_percentile;

SELECT LEFT(DIGEST_TEXT, 64) AS DIGEST_TEXT, COUNT_STAR, SUM_TIMER_WAIT, MAX_TIMER_WAIT
FROM performance_schema.events_statements_summary_by_digest
ORDER BY MAX_TIMER_WAIT DESC
LIMIT 10;

SELECT *
FROM sys.metrics
WHERE Variable_name IN ('log_lsn_current', 'log_lsn_last_checkpoint');

- mysql常用优化设置
innodb_buffer_pool_size
innodb_buffer_pool_instances

innodb_log_file_size
innodb_log_files_in_group

innodb_file_per_table

innodb_thread_concurrency
max_connections
table_open_cache
query_cache_type
mysql_reset_connection
innodb_flush_log_at_trx_commit
sync_binlog