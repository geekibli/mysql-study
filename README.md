# mysql-study

### [01.基础架构：一条SQL查询语句是如何执行的？](https://github.com/geekibli/mysql-study/blob/main/doc/01.%E5%9F%BA%E7%A1%80%E6%9E%B6%E6%9E%84%EF%BC%9A%E4%B8%80%E6%9D%A1SQL%E6%9F%A5%E8%AF%A2%E8%AF%AD%E5%8F%A5%E6%98%AF%E5%A6%82%E4%BD%95%E6%89%A7%E8%A1%8C%E7%9A%84%EF%BC%9F.pdf)
一条SQL查询语句是如何执行的 等同于 请你讲一下mysql的基本架构。
连接器(管理链接&校验权限)->查询缓存->分析器（语法，词法）->优化器（执行计划&索引选择）->执行器（操作引擎&返回结果）

### [02.日志系统：一条SQL更新语句是如何执行的？](https://github.com/geekibli/mysql-study/blob/main/doc/02.%E6%97%A5%E5%BF%97%E7%B3%BB%E7%BB%9F%EF%BC%9A%E4%B8%80%E6%9D%A1SQL%E6%9B%B4%E6%96%B0%E8%AF%AD%E5%8F%A5%E6%98%AF%E5%A6%82%E4%BD%95%E6%89%A7%E8%A1%8C%E7%9A%84%EF%BC%9F.pdf)
WAL(Write ahead logging) 关键点是先写日志，再写磁盘。
redo log: **循环写的文件，不是追加的**  innodb特有的
两阶段提交：保证redo log 和 bin log 两个日志文件的一致性

### [03.事务隔离：为什么你改了我还看不见？](https://github.com/geekibli/mysql-study/blob/main/doc/03.%E4%BA%8B%E5%8A%A1%E9%9A%94%E7%A6%BB%EF%BC%9A%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BD%A0%E6%94%B9%E4%BA%86%E6%88%91%E8%BF%98%E7%9C%8B%E4%B8%8D%E8%A7%81%EF%BC%9F.pdf)
1、介绍事务隔离级别（RU RC RR SE）
2、第一次提到视图的概念，这里的视图是read view （区别mysql查询数据的那个视图）
> 可重复读是在每次事务开始的时候，创建视图，整个事务都用这个视图。 读未提交是在每一次sql开始执行时创建的。 读提交总是能读到最新的记录，所以没有视图可言。
3、MVCC: 多版本并发控制。 每一个记录，不同的事务可能有多个版本。 如果通过新版本找旧版本（也就是快照读）需要依靠redo log，这一点后面会提到。

### [04.深入浅出索引（上）](https://github.com/geekibli/mysql-study/blob/main/doc/04.%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BA%E7%B4%A2%E5%BC%95%EF%BC%88%E4%B8%8A%EF%BC%89.pdf)
1、介绍什么是索引，类似于书的目录，提高查询效率  
2、三种数据结果的分析： 哈希（适合等值查询）、有序数组（适合静态数据）、搜索树（N叉树，一般最多3层，并且第2层大概率在内存中，查询快）   
3、聚簇索引（索引存的是整条数据）， 非聚簇索引（也叫二级索引，索引上存只有主键）  
4、为什么innodb要有主键自增，为了维护索引有序性，降低页的合并和分裂，如果产生空洞，可以重建索引。一般不重建主键，而是 alter tabel t engine=InnoDB  
5、int/bigint做主键自增，在一些场景下可以节省二级索引节点上的存储空间   

### [05.深入浅出索引（下）](https://github.com/geekibli/mysql-study/blob/main/doc/05.%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BA%E7%B4%A2%E5%BC%95%EF%BC%88%E4%B8%8B%EF%BC%89.pdf)
1、【回表】：索引 -> 主键 -> 主键索引 -> 要查的数据  
2、【覆盖索引】：在二级索引上，添加要查询的字段，可以避免回表（这个看具体业务，不可盲目添加）  
3、【最左匹配原则】：这个可以思考一下二级索引是怎样的，索引结果明白了，这个自然比较好理解  
4、【索引下推】：ICP,在遍历索引的时候，对查询条件进行过滤，减少了回表的次数  
> 这一节的概念比较重要 ‼️ ‼️  


### [06.全局锁和表锁 ：给表加个字段怎么有这么多阻碍？](https://github.com/geekibli/mysql-study/blob/main/doc/06.%E5%85%A8%E5%B1%80%E9%94%81%E5%92%8C%E8%A1%A8%E9%94%81%20%EF%BC%9A%E7%BB%99%E8%A1%A8%E5%8A%A0%E4%B8%AA%E5%AD%97%E6%AE%B5%E6%80%8E%E4%B9%88%E6%9C%89%E8%BF%99%E4%B9%88%E5%A4%9A%E9%98%BB%E7%A2%8D%EF%BC%9F.pdf)
1、全局锁：对整个数据库实例加锁， `Flush tabel with read lock`（FTWRL）, 数据库只读，DML DDL 事务更新都阻塞。 用在全库逻辑备份上。
2、表级锁： 
    - 表锁： `lock tables t1, tx read/write`, innodb一般不用，因为它有行锁
    - matedata lock (MDL) 系统默认添加，当对一个表做增删改查的时候添加MDL读锁， 做DDL的时候，加MDL写锁。 




### [18、为什么这些sql业务逻辑相同，性能却查这么多？](https://github.com/geekibli/mysql-study/blob/main/doc/18.%E4%B8%BA%E4%BB%80%E4%B9%88%E8%BF%99%E4%BA%9BSQL%E8%AF%AD%E5%8F%A5%E9%80%BB%E8%BE%91%E7%9B%B8%E5%90%8C%E6%80%A7%E8%83%BD%E5%8D%B4%E5%B7%AE%E5%BC%82%E5%B7%A8%E5%A4%A7%EF%BC%9F.pdf)
本文讲解了三种情况下，不会走索引的情况，基本都是因为显示或隐式的使用的函数。  
- 使用month函数导致索引失效
- 类型转换导致（隐式）索引失效 
- 对于字符串类型，字符编码不一致会导致索引失效，保证查询的字段和传入的参数编码格式一致，如果无法保证，就转换一下请求参数的编码格式，设置成要查询的字段的编码格式

### [19.为什么我只查一行的语句也执行这么慢？](https://github.com/geekibli/mysql-study/blob/main/doc/19.%E4%B8%BA%E4%BB%80%E4%B9%88%E6%88%91%E5%8F%AA%E6%9F%A5%E4%B8%80%E8%A1%8C%E7%9A%84%E8%AF%AD%E5%8F%A5%E4%B9%9F%E6%89%A7%E8%A1%8C%E8%BF%99%E4%B9%88%E6%85%A2%EF%BC%9F.pdf)

本文主要介绍了一个简单查询，但是很慢的情况，分成两种情况（前提不考虑mysql服务器，io等其他因素）
- 查询时间长不返回 （有可能在等别的线程释放MDL写锁,因为别的MDL写锁和接下来的读操作是阻塞的）
- 等待flush （flush table会锁表 ， 正常情况下flush动作很快，如果查询慢是因为等待flush造成的，那么可能flush也被阻塞了）
- 等待行锁 （如果你是当前读，可能别的事务在update，你就必须wait）
- 扫描行数多 （limit 50000 1 虽然只查询一行，但是如果没有索引，需要扫描50000行）
- 快照读（快照落后版本太多，也就是版本太低，需要根据当前版本，结合redo log会退 ，这属于mvcc的知识，如果中间版本间隔太多的话，可能会比较慢）

### [20.幻读是什么幻读有什么问题？](https://github.com/geekibli/mysql-study/blob/main/doc/20.%E5%B9%BB%E8%AF%BB%E6%98%AF%E4%BB%80%E4%B9%88%E5%B9%BB%E8%AF%BB%E6%9C%89%E4%BB%80%E4%B9%88%E9%97%AE%E9%A2%98%EF%BC%9F.pdf)

主要介绍了什么是幻读：在可重复读下，一般的查询事快照查，是不会查到别的事务提交的记录的，但是，当前读是读取的最新版本，因此幻读是在 【当前读】下发生的。 **幻读专只的是“新插入的行”**
幻读有什么问题呢：  
- 造成语义上的不一致，本来要锁一行的，结果新插入几条数据（符合筛选条件）
- 造成数据的不一致，可能导致binlog在做数据同步的时候，主从数据不一致
如何解决幻读：  
间隙锁（只有在可重复读下存在），加锁的时候，不但锁住所有扫描的记录，记录之间的间隙也锁住，这样就不能insert了。**间隙锁+行锁= Next-Key Lock(前开后闭) **

### [21.为什么我只改一行的语句锁这么多？](https://github.com/geekibli/mysql-study/blob/main/doc/21.%E4%B8%BA%E4%BB%80%E4%B9%88%E6%88%91%E5%8F%AA%E6%94%B9%E4%B8%80%E8%A1%8C%E7%9A%84%E8%AF%AD%E5%8F%A5%E9%94%81%E8%BF%99%E4%B9%88%E5%A4%9A%EF%BC%9F.pdf)
本文主要介绍了innodb的加锁规则。介绍了具体的规则和优化，还有8个案例辅助来学习。  
其实这一篇文章不是很好理解，简易多读几遍，另外，评论区也有很多值得学习的知识。  



