# mysql运维常用命令

## 查看死锁

执行 show engine innodb status 命令得到的部分输出。这个命令会输出很多信息，有一节 LATESTDETECTED DEADLOCK，就是记录的最后一次死锁信息。  

## 看锁等待

同样show engine innodb status 命令，锁信息是在这个命令输出结果的 TRANSACTIONS 这一节。

## 查出是谁占着这个写锁

如果你用的是 MySQL 5.7 版本，可以通过 sys.innodb_lock_waits 表查到。通过查询 sys.schema_table_lock_waits 这张表，我们就可以直接找出造成阻塞的 process id。

`mysql> select * from information_schema.processlist where id=1;`

## mysql 脏页比例

`mysql> 
select VARIABLE_VALUE into @a from global_status where VARIABLE_NAME = 'Innodb_buffer_pool_pages_dirty';
select VARIABLE_VALUE into @b from global_status where VARIABLE_NAME = 'Innodb_buffer_pool_pages_total';
select @a/@b;`  

InnoDB Buffer Pool 的大小是由参数 innodb_buffer_pool_size 确定的，一般建议设置成可用物理内存的 60%~80%。

可以在 show engine innodb status 结果中，查看一个系统当前的 BP 命中率。一般情况下，一个稳定服务的线上系统，要保证响应时间符合要求的话，内存命中率要在 99% 以上。

## join

所以你在判断要不要使用 join 语句时，就是看 explain 结果里面，Extra 字段里面有没有出现“Block Nested Loop”字样。

第二个问题是：如果要使用 join，应该选择大表做驱动表还是选择小表做驱动表？如果是 Index Nested-Loop Join 算法，应该选择小表做驱动表；如果是 Block Nested-Loop Join 算法：在 join_buffer_size 足够大的时候，是一样的；在 join_buffer_size 不够大的时候（这种情况更常见），应该选择小表做驱动表。所以，这个问题的结论就是，总是应该使用小表做驱动表。

如果被驱动表是一个大表，并且是一个冷数据表，除了查询过程中可能会导致 IO 压力大以外，你觉得对这个 MySQL 服务还有什么更严重的影响吗？

因为 join_buffer 不够大，需要对被驱动表做多次全表扫描，也就造成了“长事务”。除了老师上节课提到的导致undo log 不能被回收，导致回滚段空间膨胀问题，还会出现：1. 长期占用DML锁，引发DDL拿不到锁堵慢连接池； 2. SQL执行socket_timeout超时后业务接口重复发起，导致实例IO负载上升出现雪崩；3. 实例异常后，DBA kill SQL因繁杂的回滚执行时间过长，不能快速恢复可用；4. 如果业务采用select *作为结果集返回，极大可能出现网络拥堵，整体拖慢服务端的处理；5. 冷数据污染buffer pool，block nested-loop多次扫描，其中间隔很有可能超过1s，从而污染到lru 头部，影响整体的查询体验。

## MySQL 什么时候会使用内部临时表

1. 如果语句执行过程可以一边读数据，一边直接得到结果，是不需要额外内存的，否则就需要额外的内存，来保存中间结果；
2. join_buffer 是无序数组，sort_buffer 是有序数组，临时表是二维表结构；
3. 如果执行逻辑需要用到二维表特性，就会优先考虑使用临时表。比如我们的例子中，union 需要用到唯一索引约束， group by 还需要用到另外一个字段来存累积计数。

## group by 语法的执行过程和优化办法

1. 如果对 group by 语句的结果没有排序要求，要在语句后面加 order by null；
2. 尽量让 group by 过程用上表的索引，确认方法是 explain 结果里没有 Using temporary 和 Using filesort；
3. 如果 group by 需要统计的数据量不大，尽量只使用内存临时表；也可以通过适当调大 tmp_table_size 参数，来避免用到磁盘临时表；
4. 如果数据量实在太大，使用 SQL_BIG_RESULT 这个提示，来告诉优化器直接使用排序算法得到 group by 的结果。