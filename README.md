# mysql-study

### [01.基础架构：一条SQL查询语句是如何执行的？](https://github.com/geekibli/mysql-study/blob/main/doc/01.%E5%9F%BA%E7%A1%80%E6%9E%B6%E6%9E%84%EF%BC%9A%E4%B8%80%E6%9D%A1SQL%E6%9F%A5%E8%AF%A2%E8%AF%AD%E5%8F%A5%E6%98%AF%E5%A6%82%E4%BD%95%E6%89%A7%E8%A1%8C%E7%9A%84%EF%BC%9F.pdf)
1、一条SQL查询语句是如何执行的 等同于 请你讲一下mysql的基本架构。  
2、连接器(管理链接&校验权限) -》查询缓存 -》分析器（语法，词法）-》 优化器（执行计划&索引选择）-》 执行器（操作引擎&返回结果）    

### [02.日志系统：一条SQL更新语句是如何执行的？](https://github.com/geekibli/mysql-study/blob/main/doc/02.%E6%97%A5%E5%BF%97%E7%B3%BB%E7%BB%9F%EF%BC%9A%E4%B8%80%E6%9D%A1SQL%E6%9B%B4%E6%96%B0%E8%AF%AD%E5%8F%A5%E6%98%AF%E5%A6%82%E4%BD%95%E6%89%A7%E8%A1%8C%E7%9A%84%EF%BC%9F.pdf)
1、WAL(Write ahead logging) 关键点是先写日志，再写磁盘。  
2、redo log: **循环写的文件，不是追加的**  innodb特有的，刷到磁盘  
3、两阶段提交：保证redo log 和 binlog 两个日志文件的一致性      

### [03.事务隔离：为什么你改了我还看不见？](https://github.com/geekibli/mysql-study/blob/main/doc/03.%E4%BA%8B%E5%8A%A1%E9%9A%94%E7%A6%BB%EF%BC%9A%E4%B8%BA%E4%BB%80%E4%B9%88%E4%BD%A0%E6%94%B9%E4%BA%86%E6%88%91%E8%BF%98%E7%9C%8B%E4%B8%8D%E8%A7%81%EF%BC%9F.pdf)
1、介绍事务隔离级别（RU RC RR SE）  
2、第一次提到视图的概念，这里的视图是read view （区别mysql查询数据的那个视图）  

> 可重复读是在每次事务开始的时候，创建视图，整个事务都用这个视图。 读未提交是在每一次sql开始执行时创建的。 读提交总是能读到最新的记录，所以没有视图可言。  

3、MVCC: 多版本并发控制。 每一个记录，不同的事务可能有多个版本。 如果通过新版本找旧版本（也就是快照读）需要依靠undo log，这一点后面会提到。  

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
2、表级锁:    

- 表锁： `lock tables t1, tx read/write`, innodb一般不用，因为它有行锁
- matedata lock (MDL) 系统默认添加，当对一个表做增删改查的时候添加MDL读锁， 做DDL的时候，加MDL写锁。 

### [07.行锁功过：怎么减少行锁对性能的影响？](https://github.com/geekibli/mysql-study/blob/main/doc/07.%E8%A1%8C%E9%94%81%E5%8A%9F%E8%BF%87%EF%BC%9A%E6%80%8E%E4%B9%88%E5%87%8F%E5%B0%91%E8%A1%8C%E9%94%81%E5%AF%B9%E6%80%A7%E8%83%BD%E7%9A%84%E5%BD%B1%E5%93%8D%EF%BC%9F.pdf)

1、两阶段锁协议：行锁在需要的时候才添加，但是在事务提交的时候才释放。**如果你的事务中需要多行锁，要把最可能造成冲突和最可能造成并发度的锁尽量往后放。 **   
2、死锁和死锁检测（等待锁超时，默认50s ； 死锁检测：默认开启）  

### [08.事务到底是隔离的还是不隔离的？](https://github.com/geekibli/mysql-study/blob/main/doc/08.%E4%BA%8B%E5%8A%A1%E5%88%B0%E5%BA%95%E6%98%AF%E9%9A%94%E7%A6%BB%E7%9A%84%E8%BF%98%E6%98%AF%E4%B8%8D%E9%9A%94%E7%A6%BB%E7%9A%84%EF%BC%9F.pdf)

1、一致性视图（consistent read view）用于支持RC&RR隔离级别的实现。  
2、当前读：事务中的update是先查后改，这个查，必须是查当前数据的最新版本  

>这里的update先查后改是在事务中，准确说是innodb范畴的，这个要和我们执行一个update写redo log哪些流程区分开，这一点可能理解少会有交叉，一定要区分两者。  可以看一下「第二讲」。

3、快照读（一致性读）：普通的查询，添加 `lock in share mode`读锁 或者 `for update`写锁也可以读当前最新版本  

### [09.普通索引和唯一索引，应该怎么选择](https://github.com/geekibli/mysql-study/blob/main/doc/09.%E6%99%AE%E9%80%9A%E7%B4%A2%E5%BC%95%E5%92%8C%E5%94%AF%E4%B8%80%E7%B4%A2%E5%BC%95%EF%BC%8C%E5%BA%94%E8%AF%A5%E6%80%8E%E4%B9%88%E9%80%89%E6%8B%A9%EF%BC%9F.pdf)

1、**change buffer**: 修改数据页，如果在内存，直接改，如果没有，写change buffer,下一次加载数据页到内存，merge change buffer得到正确的数据。change buffer存内存，也持久化到磁盘。**目的是减少磁盘io从而提升性能。**    
2、唯一索引不能使用change buffer，因为唯一索引添加数据是，必须要判断存不存在数据，而change buffer就没用了。  
3、适用于写多读少的场景，不适合写完就读的场景，因为要merge操作，消耗性能。（**merge仅仅得到新的数据页没有刷盘**）  
4、这里重点看一下插入后查询时，redo log 和 change buffer是如何的流程

### [10.MySQL为什么有时候会选错索引？](https://github.com/geekibli/mysql-study/blob/main/doc/10.MySQL%E4%B8%BA%E4%BB%80%E4%B9%88%E6%9C%89%E6%97%B6%E5%80%99%E4%BC%9A%E9%80%89%E9%94%99%E7%B4%A2%E5%BC%95%EF%BC%9F.pdf)

1、其实就是讲优化器如何选择索引的，影响索引选择的因素有，扫描的行数，回表次数，临时表，排序性能（索引自带排序）  
2、mysql是基于统计来计算索引基数，`show indexs`可以查看索引的基数，也会影响优化器选择索引  
3、如果发现explain扫描的行数和实际不一致的话，可以使用`analyze tabel t`来重新统计索引信息

### [11.怎么给字符串字段加索引？](https://github.com/geekibli/mysql-study/blob/main/doc/11.%E6%80%8E%E4%B9%88%E7%BB%99%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%AD%97%E6%AE%B5%E5%8A%A0%E7%B4%A2%E5%BC%95%EF%BC%9F.pdf)

1、前缀索引：定义字符串的前一部分做为索引，创建索引的时候，设置长度，默认整个字段的字符串做索引    
2、前缀索引优点是节省空间，但是可能会增加读数据的次数或者回表的次数  
3、使用前缀索引会造成覆盖索引失效，因为系统不确定前缀索引是否截取的全部的信息，所以每次都要回表校验。

> Varchar(length) length最多存放多少字符，为什么建议length一般不要超过255？ 超过255只少需要2个字节存length

### [12.为什么我的MySQL会“抖”一下？](https://github.com/geekibli/mysql-study/blob/main/doc/12.%E4%B8%BA%E4%BB%80%E4%B9%88%E6%88%91%E7%9A%84MySQL%E4%BC%9A%E2%80%9C%E6%8A%96%E2%80%9D%E4%B8%80%E4%B8%8B%EF%BC%9F.pdf)

1、flush刷盘，把脏页数据刷新到磁盘  
2、redo log（更新阻塞，性能差）不足时，内存不足时，mysql空闲时，mysql正常关闭时会触发flush  
3、合理设置innodb_io_capacity的值，太低的话，现象是机器io正常，mysql tps很低。  
4、在flush是，相邻的脏页可能会一起被刷盘，`innodb_flush_neignbors = 1`开启连坐，mysql8.0中默认0关闭。

### [13.为什么表数据删掉一半，表文件大小不变？](https://github.com/geekibli/mysql-study/blob/main/doc/13.%E4%B8%BA%E4%BB%80%E4%B9%88%E8%A1%A8%E6%95%B0%E6%8D%AE%E5%88%A0%E6%8E%89%E4%B8%80%E5%8D%8A%EF%BC%8C%E8%A1%A8%E6%96%87%E4%BB%B6%E5%A4%A7%E5%B0%8F%E4%B8%8D%E5%8F%98%EF%BC%9F.pdf)

1、innodb在删除一行数据的时候，只是标记删除，下一次在**这个范围内**插入数据时，优先覆盖已经标记删除的空间。那么如果整个数据页被删除也是一样的，整个数据页标记删除，等到被覆盖（**没有位置和范围限制**）。  
2、频繁增删改的表容易产生“空洞”，可以用重建表来进行优化 `alter tebel t engine=InnoDB`   

> 上述重建表是innodb+tmp file+row log完成的，重建表执行的时候，MDL写锁退化成MDL读锁，允许增删改操作。另外，tmp file占用磁盘资源。    

3、`optimize table t` = recreate t + `analyser table t`

### [14.count(×)这么慢，我该怎么办？](https://github.com/geekibli/mysql-study/blob/main/doc/14.count(%C3%97)%E8%BF%99%E4%B9%88%E6%85%A2%EF%BC%8C%E6%88%91%E8%AF%A5%E6%80%8E%E4%B9%88%E5%8A%9E%EF%BC%9F.pdf)

1、为什么InnoDB不像MyISAM一样存一个全局变量，因为MVCC的设计，不同事务在同一时刻可能返回不一样的count  
2、这里关联一下「第10讲」里面索引统计影响索引选择，`show table status`返回的行数是统计得出的，不准确  
3、按照排序效率：count(field) <  count(id)  < count(i) 约等于 count(*) 推荐

- count(field) 扫描所有非空的字段；
-  count(id) 扫描所有的非空id并计数累加； 
-  count(1) innodb扫描所有行 server每行标记1并计数
- count(*) 直接计数扫描的行数

### [15.答疑文章（一）：日志和索引相关问题](https://github.com/geekibli/mysql-study/blob/main/doc/15.%E7%AD%94%E7%96%91%E6%96%87%E7%AB%A0%EF%BC%88%E4%B8%80%EF%BC%89%EF%BC%9A%E6%97%A5%E5%BF%97%E5%92%8C%E7%B4%A2%E5%BC%95%E7%9B%B8%E5%85%B3%E9%97%AE%E9%A2%98.pdf)

这块是关于update + redo log + binlog的灵魂拷问，你可接住喽！  
<font color=#FF9999>**为什么update的时候，写redo log和binlog要用两阶段提交？** **反证法，不用两阶段，直接提交**</font>      
```
1、先写redo log，crash，后写binlog，当前mysql数据更新了，但是从库没有更新；  
2、先写binlog ， crash, 后写redo log, 从库数据更新了，但是当前mysql没有更新
```

<font color=#FF9999>**以上是在主从同步的角度上分析的，如果仅仅一个mysql实例情况下，没有两阶段提交会有什么问题呢？**</font>  
```text
前提是Innodb中，redo log已经commit（非parpare状态）不能回滚。如果写redo log成功了，现在要回滚，是无法回滚的。
```
两阶段提交能保证mysql集群中所有实例的数据一致性。❗️❗️

<font color=#FF9999>**只有binlog能不能保证崩溃恢复呢？**</font>  
```
不能，**binlog无法保证恢复数据页**，update先改数据页，写binlog，但是磁盘上的数据还是原来的。
恢复的时候，不完整的binlog可以回滚，但是完整的binlog数据却无法恢复到最新。  
```

<font color=#FF9999>**只有redo log可不可行？**</font>  
```
redo log对于崩溃恢复来说是可以的，但是数据同步呢，还是得依靠bin log。
```

<font color=#FF9999>**数据落盘是怎么样？是从redo log更新还是buffer pool呢？**</font>  
```
1、正常刷盘的时候是直接把内存数据页（脏页）flush到磁盘，也就是buffer pool。  
2、崩溃恢复的时候，磁盘的数据页+redo log得到脏页，也就是上一步
```

<font color=#FF9999>**数据更新的时候先写内存还是先写redo log？**</font>  

```
先写数据页，然后写redo log buffer, 在两阶段提交的第二阶段commit的时候，写redo log。
当然这时候，binlog已经写完了。
```

### [16.“order%20by”是怎么工作的？](https://github.com/geekibli/mysql-study/blob/main/doc/16.%E2%80%9Corder%20by%E2%80%9D%E6%98%AF%E6%80%8E%E4%B9%88%E5%B7%A5%E4%BD%9C%E7%9A%84%EF%BC%9F(1).pdf)

排序的字段如果有索引，就不需要排序了，索引自带排序。mysql为每个线程分配一块内存sort buffer （大小sort_buffer_size）**sort buffer在排序完成后，内存free还给系统。**下面两种排序都是归并排序算法。  
1、全文排序  
innodb索引命中之后，根据主键获取要排序的字段（改字段上没有索引，也没有覆盖索引），然后把**数据（select 的数据）**放到sort buffer，循环这个过程，所有的数据都放到sort buffer，如果sort buffer足够，就在内存进行排序，如果不够，需要借助**磁盘临时文件**，生成许多小文件各自排序，然后归并，获取最终结果。  
2、rowid排序  
和全文排序不同的是，在sort buffer中存的只有主键和需要排序的字段，如果select中还有别的字段，是不可以直接返回的，还需要在回表根据主键把要查询的所有字段查出来，然后返回。

### [17.如何正确地显示随机消息？](https://github.com/geekibli/mysql-study/blob/main/doc/17.%E5%A6%82%E4%BD%95%E6%AD%A3%E7%A1%AE%E5%9C%B0%E6%98%BE%E7%A4%BA%E9%9A%8F%E6%9C%BA%E6%B6%88%E6%81%AF%EF%BC%9F(1).pdf)

1、临时表排序  
`order by rand()` 使用了内存临时表，排序的算法使用的rowid（临时表在内存中，不用考虑回表的磁盘io）。  
2、磁盘临时表  
`tmp_table_size`默认16M，超过这个大小，会使用磁盘临时表（默认是innodb）  
3、优先队列算法  
通过构建堆来实现排序，堆的大小也是受 sort_buffer_size大小控制的，如果只能使用归并排序算法。

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



