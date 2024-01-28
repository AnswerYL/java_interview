## 优化

#### 如何定位慢查询？

1. 调试接口发现返回结果较慢

2. 开启mysql慢日志查询（/ect/my.cnf）
   
   ```java
   // 开启mysql慢日志查询(默认0未开启)
   show_query_log=1
   // 设置慢日志的时间为2秒
   long_query_time=2
   ```

3. 配置完成后，重启mysql，查看慢日志文件中记录的信息（/var/lib/mysql/localhost-slow.log）

#### 如何分析慢查询sql语句

通过EXPLAN或DESC命令获取mysql执行信息

- possible_keys：当前sql可能会使用到的索引

- key：当前sql实际命中的索引

- key_len：索引占用的大小

- Extra：额外的优化建议
  
  - Using where;Using index：当前sql使用了索引，需要的数据都在索引列中能够找到，不需要回表查询数据
  
  - Using index condition：当前sql使用了索引，但需要回表查询数据

- type：当前sql的连接类型，性能由好到差为NULL、sysytem、const、eq_ref、ref、range、index、all；
  
  index或all情况，当前sql最好进行优化
  
  - NULL 未查询到表
  
  - system 查询的是mysql系统的表
  
  - const 根据主键查询
  
  - eq_ref 根据主键索引查询或唯一索引查询（只能返回一条数据）
  
  - ref 根据索引查询
  
  - range 走索引，但范围查询
  
  - index 索引树扫描（全索引扫描）
  
  - all 不走索引，全盘扫描

#### 什么是索引？

1. 高效获取数据的数据结构（有序）

2. 提高数据检索效率，减少数据库IO成本（不需要全表扫描）

3. 通过索引列对数据进行排序，降低数据排序成本，降低cpu消耗

#### 索引底层数据结构？

InnoDB引擎索引默认采用的是B+树

- 阶数更多，路径更短

- 磁盘读写代价B+树更低，非叶子节点只存储节点，叶子节点才存储数据

- B+树叶子节点通过双向链表连接，便于进行扫库和区间查询

#### 什么是聚簇索引，非聚簇索引？回表查询？

聚簇索引：数据和索引放在一起存储，索引叶子节点保存数据，必须有且只有一个（如主键索引）

非聚簇索引：数据和索引分开，索引叶子节点关联的是对应的主键，可以存在多个

回表查询：通过非聚簇索引查询到的数据，由于查询到的是主键，则需要重新通过主键聚簇索引查询，这个过程就是回表查询

#### 什么是覆盖索引？

覆盖索引是指查询使用了索引，并且需要返回的列，在该索引保存的数据中都能够找到。

#### mysql超大分页怎么处理？

当数据量较大时，使用limit进行分页，需要对数据进行排序，效率低

```sql
select * from t_table limit 900000,10;
# 需要对900010条数据进行排序，然后取最后10条，其余数据舍弃，造成效率浪费
```

可以通过覆盖查询+子查询解决（id是覆盖索引，操作会比较快）

```sql
select * from t_table t,
(select id from t_table order by id limit 900000,10) a
where t.id=a.id;
```

#### 索引创建的原则？（加粗重要）

1. **针对数据量较大，且查询比较频繁的表建立索引。**（单表10w+）

2. **针对常作为查询条件（where）、排序（order by）、分组（group by）操作的字段建立索引**

3. 尽量选区分度高的列作为索引，尽量建立唯一索引，区分度越高，使用索引的效率越高。

4. 如果是字符串类型的字段，字段的长度较长，可以针对字段的特点，建立前缀索引

5. **尽量使用联合索引，减少单列索引，查询时联合索引很多时候可以覆盖索引，节省存储空间，避免回表，提高查询效率。**

6. **要控制索引的数量，索引并不是越多越好，索引越多，维护索引结构的代价也就越大，会影响增删改的效率**

7. 如果索引列不能存储NULL值，请在创建时使用NOT NULL来约束

#### 什么情况下索引会失效？

1. 违反最左前缀法则；使用联合索引，查询条件要满足最左前缀法则，否则索引失效
   
   ```sql
   # 创建联合索引name,mobile,status
   select * from t_table where name='aa' 
   and mobile='xxx' and status=1;# 命中索引，符合最左前缀
   
   select * from t_table where 
   mobile='xxx' and status=1; # 索引失效
   
   select * from t_table where name='aa' 
   and status=1; # 仅命中name索引，status失效
   ```

2. 范围查询，右边的列使用索引失效 
   
   ```sql
   # 创建联合索引name,mobile,status
   select * from t_table where name='aa' 
   and mobile>'xxx' and status=1;
   #仅命中name和mobile索引，status索引失效
   ```

3. 对索引字段进行运算操作，索引失效
   
   ```sql
   # 创建联合索引name,mobile,status
   select * from t_table where substring(name,0,1)='a';
   # 索引失效 
   ```

4. 索引进行类型转换，索引失效
   
   ```sql
   # 创建联合索引name,mobile,status
   select * from t_table where name='aa' and mobile=153;
   # 索引进行类型转换，索引失效
   ```

5. 以%开头的like查询索引失效，放在尾部则不会。
   
   ```sql
   # 创建联合索引name,mobile,status
   select * from t_table where name like '%a'; #索引失效
   select * from t_table where name like 'a%'; #命中索引
   ```

#### 谈谈对sql优化的经验

- 表的设计优化
  
  - 根据需要存储的内容选择合适的字段类型

- 索引优化（查看上述笔记）

- SQL语句优化
  
  - **避免使用（select *），务必指明字段名称，可以使用到覆盖索引**
  
  - **避免造成索引失效**
  
  - 尽量使用union all 替代union，union 会过滤重复数据，效率低
  
  - **避免在where子句中对字段进行表达式操作**
  
  - join优化，能用inner join就不用left join 或 right join，如必须使用一定要以小表为驱动，内连接会对两个表进行优化，优先把小表放在外边，把大表放在里面。而左右连接则不会调整顺序

- 主从复制、读写分离
  
  - 主库写入，从库读取，提高效率

- 分库分表（后续笔记）

## 其他面试题

#### 事务的特性？

ACID：原子性，一致性，隔离性，持久性

#### 并发事务有哪些问题？

脏读：一个事务读到另一事务还未提交的数据。

不可重复读：一个事务先后读取同一条记录，但两次读取的数据不一致

幻读：一个事务按照条件查询数据时，没有对应数据，但在插入数据时，又发现这行数据已经存在

#### 事务隔离级别？

RU：未提交读，不能解决并发事务产生的问题

RC：读已提交，解决脏读问题

RR（默认）：可重复读，解决脏读、不可重复读问题

串行化：放弃并发，可以解决所有问题

#### undo log 和 redo log 的区别

redo log：记录的是数据页的物理变化，服务宕机时可用来同步数据

undo log：记录的是逻辑日志，当事务回滚时，通过逆操作恢复原来的数据

redo log 保证事务的持久性，undo log保证事务的原子性和一致性

#### MVCC原理（多版本并发控制）

使读写没有冲突

- 记录中的隐藏字段
  
  - **DB_TRX_ID：最近修改事务id，记录插入这条记录或最后一次修改该记录的事务id。（自增数值）**
  
  - **DB_ROLL_PTR：回滚指针，指向这条记录的上一个版本，用于配合undo log使用。**
  
  - DB_ROW_ID：隐藏主键，如果表结构没有指定主键，将会生成该隐藏字段 

- undo log 回滚日志
  
  - 在insert，update，delete时产生的便于数据回滚的日志。
    
    当insert时，产生的undo log日志只在回滚时需要，在事务提交后，可被立即删除。
    
    当update、delete时，产生的日志不仅在回滚时需要，mvcc版本访问也需要，不会立即删除
  
  - undo log版本链
    
    不同事务或同一事务对同一条记录进行修改，会导致该记录的undolog生成一条记录版本链条，通过roll_pointer指针，链表的头部是最新的旧记录，尾部则是最早的旧记录。

- readview（读视图）
  
  readview是快照读sql执行时MVCC提取数据的依据，记录并维护系统当前活跃的事务（未提交）的id。
  
  RC：在事务中每一次执行快照读时生成readview。
  
  RR：仅在事务第一次执行快照读时生成readview，同一事务复用该readview。
  
  - m_ids：当前活跃的事务id集合
  
  - min_trx_id：最小活跃事务id
  
  - max_trx_id：预分配事务id，当前最大事务id+1（事务di是自增）
  
  - creator_trx_id：readview创建者的失误id

#### 主从同步原理

核心就是二进制日志（binlog日志），包含DDL（数据定义语言）和DML（数据操作语言），不包括select、show等语句。

同步分成三步：

1. Master主库在事务提交时，会把数据变更记录在binlog日志文件中。

2. Slave从库（IOthread线程）读取主库的binlog日志，写入到从库的中继日志Relay log。

3. Slave从库（SQLthread线程）重做中继日志中的事件，将改变自己的数据。

#### 分库分表

1. 业务介绍（根据简历找到一个数据量较大的业务，请求数多或业务累计大）

2. 达到什么样的量级（单表1000w或超过20G）
- 具体拆分策略（水平拆分都需要使用中间件mycat等）
  
  - 水平分库，将一个库的数据拆分到多个库中，解决海量数据存储和高并发问题。
  
  - 水平分表，解决单表存储和性能问题。
  
  - 垂直分库，根据业务进行拆分，高并发下提高磁盘io和网络连接数
  
  - 垂直分表，冷热数据分离，多表互不影响
