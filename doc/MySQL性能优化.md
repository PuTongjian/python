## MySQL性能优化

### 索引的设计原则

1. 创建索引的列并不一定是`select`操作中要查询的列，最适合做索引的列是出现在`where`子句中经常用作筛选条件或连表子句中作为表连接条件的列。
2. 具有唯一性的列，索引效果好；重复值较多的列，索引效果差。
3. 如果为字符串类型创建索引，最好指定一个前缀长度，创建短索引。短索引可以减少磁盘I/O而且在做比较时性能也更好，更重要的是MySQL底层的高速索引缓存能够缓存更多的键值。
4. 创建一个包含N列的复合索引（多列索引）时，相当于是创建了N个索引，此时应该利用最左前缀进行匹配。
5. 不要过度使用索引。索引并不是越多越好，索引需要占用额外的存储空间而且会影响写操作的性能（插入、删除、更新数据时索引也需要更新）。MySQL在生成执行计划时，要考虑各个索引的使用，这个也是需要耗费时间的。
6. 要注意可能使索引失效的场景，例如：模糊查询使用了前置通配符、使用负向条件进行查询等。

### 数据分区
1. RANGE分区：基于连续区间范围，把数据分配到不同的分区。

   ```SQL
   CREATE TABLE tb_emp (
       eno INT NOT NULL,
       ename VARCHAR(20) NOT NULL,
       job VARCHAR(10) NOT NULL,
       hiredate DATE NOT NULL,
       dno INT NOT NULL
   )
   PARTITION BY RANGE( YEAR(hiredate) ) (
       PARTITION p0 VALUES LESS THAN (1960),
       PARTITION p1 VALUES LESS THAN (1970),
       PARTITION p2 VALUES LESS THAN (1980),
       PARTITION p3 VALUES LESS THAN (1990),
       PARTITION p4 VALUES LESS THAN MAXVALUE
   );
   ```

2. LIST分区：基于枚举值的范围，把数据分配到不同的分区。

3. HASH分区 / KEY分区：基于分区个数，把数据分配到不同的分区。

   ```SQL
   CREATE TABLE tb_emp (
       eno INT NOT NULL,
       ename VARCHAR(20) NOT NULL,
       job VARCHAR(10) NOT NULL,
       hiredate DATE NOT NULL,
       dno INT NOT NULL
   )
   PARTITION BY HASH(dno)
   PARTITIONS 4;
   ```

### SQL优化

1. 定位低效率的SQL语句 - 慢查询日志。

   - 查看慢查询日志相关配置

      ```SQL
      mysql> show variables like 'slow_query%';
      +---------------------------+----------------------------------+
      | Variable_name             | Value                            |
      +---------------------------+----------------------------------+
      | slow_query_log            | OFF                              |
      | slow_query_log_file       | /mysql/data/localhost-slow.log   |
      +---------------------------+----------------------------------+

      mysql> show variables like 'long_query_time';
      +-----------------+-----------+
      | Variable_name   | Value     |
      +-----------------+-----------+
      | long_query_time | 10.000000 |
      +-----------------+-----------+
      ```

   - 修改全局慢查询日志配置。

      ```SQL
      mysql> set global slow_query_log='ON'; 
      mysql> set global long_query_time=1;
      ```

      或者直接修改MySQL配置文件启用慢查询日志。

      ```INI
      [mysqld]
      slow_query_log=ON
      slow_query_log_file=/usr/local/mysql/data/slow.log
      long_query_time=1
      ```

2. 通过`explain`了解SQL的执行计划。例如：

   ```SQL
   explain select ename, job, sal from tb_emp where dno=20\G
   *************************** 1. row ***************************
              id: 1
     select_type: SIMPLE
           table: tb_emp
            type: ref
   possible_keys: fk_emp_dno
             key: fk_emp_dno
         key_len: 5
             ref: const
            rows: 7
           Extra: NULL
   1 row in set (0.00 sec)
   ```

   - `select_type`：查询类型（SIMPLE - 简单查询、PRIMARY - 主查询、UNION - 并集、SUBQUERY - 子查询）。
   - `table`：输出结果集的表。
   - `type`：访问类型（ALL - 全表查询性能最差、index、range、ref、eq_ref、const、NULL）。
   - `possible_keys`：查询时可能用到的索引。
   - `key`：实际使用的索引。
   - `key_len`：索引字段的长度。
   - `rows`：扫描的行数，行数越少肯定性能越好。
   - `extra`：额外信息。

3. 通过`show profiles`和`show profile for query`分析SQL。

   MySQL从5.0.37开始支持剖面系统来帮助用户了解SQL执行性能的细节，可以通过下面的方式来查看MySQL是否支持和开启了剖面系统。

   ```SQL
   select @@have_profiling;
   select @@profiling;
   ```

   如果没有开启剖面系统，可以通过下面的SQL来打开它。

   ```SQL
   set profiling=1;
   ```

   接下来就可以通过剖面系统来了解SQL的执行性能，例如：

   ```SQL
   mysql> select count(*) from tb_emp;
   +----------+
   | count(*) |
   +----------+
   |       14 |
   +----------+
   1 row in set (0.00 sec)
   
   mysql> show profiles;
   +----------+------------+-----------------------------+
   | Query_ID | Duration   | Query                       |
   +----------+------------+-----------------------------+
   |        1 | 0.00029600 | select count(*) from tb_emp |
   +----------+------------+-----------------------------+
   1 row in set, 1 warning (0.00 sec)
   
   mysql> show profile for query 1;
   +----------------------+----------+
   | Status               | Duration |
   +----------------------+----------+
   | starting             | 0.000076 |
   | checking permissions | 0.000007 |
   | Opening tables       | 0.000016 |
   | init                 | 0.000013 |
   | System lock          | 0.000007 |
   | optimizing           | 0.000005 |
   | statistics           | 0.000012 |
   | preparing            | 0.000010 |
   | executing            | 0.000003 |
   | Sending data         | 0.000070 |
   | end                  | 0.000012 |
   | query end            | 0.000008 |
   | closing tables       | 0.000012 |
   | freeing items        | 0.000032 |
   | cleaning up          | 0.000013 |
   +----------------------+----------+
   15 rows in set, 1 warning (0.00 sec)
   ```

4. 优化CRUD操作。

   - 优化`insert`语句
     - 在`insert`语句后面跟上多组值进行插入在性能上优于分开`insert`。
     - 如果有多个连接向同一个表插入数据，使用`insert delayed`可以获得更好的性能。
     - 如果要从一个文本文件装载数据到表时，使用`load data infile`比`insert`性能好得多。

   - 优化`order by`语句

     - 如果`where`子句的条件和`order by`子句的条件相同，而且排序的顺序与索引的顺序相同，如果还同时满足排序字段都是升序或者降序，那么只靠索引就能完成排序。

   - 优化`group by`语句

     - 在使用`group by`子句分组时，如果希望避免排序带来的开销，可以用`order by null`禁用排序。

   - 优化嵌套查询

     - MySQL从4.1开始支持嵌套查询（子查询），这使得可以将一个查询的结果当做另一个查询的一部分来使用。在某些情况下，子查询可以被更有效率的连接查询取代，因为在连接查询时MySQL不需要在内存中创建临时表来完成这个逻辑上需要多个步骤才能完成的查询。

   - 优化or条件

     - 如果条件之间是`or`关系，则只有在所有条件都用到索引的情况下索引才会生效。

   - 优化分页查询

     - 分页查询时，一个比较头疼的事情是如同`limit 1000, 20`，此时MySQL已经排序出前1020条记录但是仅仅返回第1001到1020条记录，前1000条实际都用不上，查询和排序的代价非常高。一种常见的优化思路是在索引上完成排序和分页的操作，然后根据返回的结果做表连接操作来得到最终的结果，这样可以避免出现全表查询，也避免了外部排序。

       ```SQL
       select * from tb_emp order by ename limit 1000, 20;
       select * from tb_emp t1 inner join (select eno from tb_emp order by ename limit 1000, 20) t2 on t1.eno=t2.eno;
       ```

       上面的代码中，第2行SQL是优于第1行SQL的，当然我们的前提是已经在`ename`字段上创建了索引。

   - 使用SQL提示
     - USE INDEX：建议MySQL使用指定的索引。
     - IGNORE INDEX：建议MySQL忽略掉指定的索引。
     - FORCE INDEX：强制MySQL使用指定的索引。

### 配置优化

可以使用下面的命令来查看MySQL服务器配置参数的默认值。

```SQL
show variables;
show variables like 'key_%';
show variables like '%cache%';
show variables like 'innodb_buffer_pool_size';
```

通过下面的命令可以了解MySQL服务器运行状态值。

```SQL
show status;
show status like 'com_%';
show status like 'innodb_%';
show status like 'connections';
show status like 'slow_queries';
```

1. 调整`max_connections`：MySQL最大连接数量，默认151。在Linux系统上，如果内存足够且不考虑用户等待响应时间这些问题，MySQL理论上可以支持到万级连接，但是通常情况下，这个值建议控制在1000以内。
2. 调整`back_log`：TCP连接的积压请求队列大小，通常是max_connections的五分之一，最大不能超过900。
3. 调整`table_open_cache`：这个值应该设置为max_connections的N倍，其中N代表每个连接在查询时打开的表的最大个数。
4. 调整`innodb_lock_wait_timeout`：该参数可以控制InnoDB事务等待行锁的时间，默认值是50ms，对于反馈响应要求较高的应用，可以将这个值调小避免事务长时间挂起；对于后台任务，可以将这个值调大来避免发生大的回滚操作。
5. 调整`innodb_buffer_pool_size`：InnoDB数据和索引的内存缓冲区大小，以字节为单位，这个值设置得越高，访问表数据需要进行的磁盘I/O操作就越少，如果可能甚至可以将该值设置为物理内存大小的80%。

### 架构优化

1. 分表分库，具体可参考[基于Flask-SQLAlchemy实现分表分库](http://www.pythondoc.com/flask-sqlalchemy/binds.html)。

2. 主从复制和读写分离，具体内容请参考[MySQL主从复制](https://github.com/PuTongjian/python/blob/master/doc/MySQL%E4%B8%BB%E4%BB%8E%E5%A4%8D%E5%88%B6.md)。




> **说明**：本章内容参考了网易出品的《深入浅出MySQL》一书，该书和《高性能MySQL》一样，都对MySQL进行了深入细致的讲解，虽然总体感觉后者更加高屋建瓴，但是前者也算得上是提升MySQL技能的佳作（作者的文字功底稍显粗糙，深度也不及后者），建议有兴趣的读者可以阅读这两本书。
