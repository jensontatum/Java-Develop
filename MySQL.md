## MySQL

### MySQL基础语法

### MySQL性能调优

#### 一、事务

##### 1.什么是数据库事务

-  事务是一个不可分割的数据库操作序列，也是数据库并发控制的基本单位，其执行的结果必须使数据库从一种一致性状态变到另一种一致性状态。事务是逻辑上的一组操作，要么都执行，要么都不执行。 

##### 2.事务的ACID属性

- **原子性（Atomicity）**
  原子性是指事务是一个不可分割的工作单位，事务中的操作要么都发生，要么都不发生。 

- **一致性（Consistency）**
  事务必须使数据库从一个一致性状态变换到另外一个一致性状态。

- **隔离性（Isolation）**
  事务的隔离性是指一个事务的执行不能被其他事务干扰，即一个事务内部的操作及使用的数据对并发的其他事务是隔离的，并发执行的各个事务之间不能互相干扰。

- **持久性（Durability）**
  持久性是指一个事务一旦被提交，它对数据库中数据的改变就是永久性的，接下来的其他操作和数据库故障不应该对其有任何影响。

##### 3.数据库的并发问题

- 对于同时运行的多个事务, 当这些事务访问数据库中相同的数据时, 如果没有采取必要的隔离机制, 就会导致各种并发问题:
  - **脏读**: 对于两个事务 T1, T2, T1 读取了已经被 T2 更新但还**没有被提交**的字段。之后, 若 T2 回滚, T1读取的内容就是临时且无效的。
  - **不可重复读**: 对于两个事务T1, T2, T1 读取了一个字段, 然后 T2 **更新**了该字段。之后, T1再次读取同一个字段, 值就不同了。
  - **幻读**: 对于两个事务T1, T2, T1 从一个表中读取了一个字段, 然后 T2 在该表中**插入**了一些新的行。之后, 如果 T1 再次读取同一个表, 就会多出几行。

- **数据库事务的隔离性**: 数据库系统必须具有隔离并发运行各个事务的能力, 使它们不会相互影响, 避免各种并发问题。

- 一个事务与其他事务隔离的程度称为隔离级别。数据库规定了多种事务隔离级别, 不同隔离级别对应不同的干扰程度, **隔离级别越高, 数据一致性就越好, 但并发性越弱。**

##### 4.数据库的隔离级别

![1615884846924](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\1615884846924.png)

Mysql 默认的事务隔离级别为: **REPEATABLE READ。**

#### 二、MySQL锁机制

##### 1.锁的概述

- 锁是计算机协调多个进程或线程并发访问某一资源的机制。当数据库有并发事务的时候，可能会产生数据的不一致，这时候需要锁机制来保证数据的一致性。

##### 2.锁的分类

###### 2.1 从对数据操作的类型（读\写）分

- **读锁(共享锁)**：针对同一份数据，多个读操作可以同时进行而不会互相影响。
- **写锁（独占锁）**：当前写操作没有完成前，它会阻断其他写锁和读锁。

###### 2.2 从对数据操作的粒度分

- 为了尽可能提高数据库的并发度，每次锁定的数据范围越小越好，理论上每次只锁定当前操作的数据的方案会得到最大的并发度，但是管理锁是很耗资源的事情（涉及获取，检查，释放锁等动作），因此数据库系统需要在高并发响应和系统性能两方面进行平衡，这样就产生了“锁粒度（Lock granularity）”的概念。

- 一种提高共享资源并发发性的方式是让锁定对象更有选择性。尽量只锁定需要修改的部分数据，而不是所有的资源。更理想的方式是，只对会修改的数据片进行精确的锁定。任何时候，在给定的资源上，锁定的数据量越少，则系统的并发程度越高，只要相互之间不发生冲突即可。

- 从这个角度，数据库的锁分为表锁和行锁，下面为MyISAM和InnoDB的存储引擎区别

  | 对比项 | MyISAM                     | InnoDB         |
  | ------ | -------------------------- | -------------- |
  | 事务   | 不支持                     | 支持           |
  | 行表锁 | 表锁                       | 行锁（高并发） |
  | 缓存   | 只缓存索引，不缓存真实数据 | 都缓存         |
  | 关注点 | 性能                       | 事务           |

##### 3.表锁（偏读）

###### 3.1 特点

- 偏向MyISAM存储引擎，开销小，加锁快；无死锁；锁定粒度大，发生锁冲突的概率最高,并发度最低。

###### 3.2 结论

- MyISAM在执行查询语句（SELECT）前，会自动给涉及的整张表加读锁，在执行增删改操作前，会自动给涉及的表加写锁。读锁会阻塞写，但是不会堵塞读。而写锁则会把读和写都堵塞。
- MySQL的表级锁有两种模式：
  - 表共享读锁（Table Read Lock）
  - 表独占写锁（Table Write Lock）

###### 3.3 表锁分析

- 查看哪些表被加锁了：  mysql>show open tables;

- 如何分析表锁定：

  - 可以通过检查table_locks_waited和table_locks_immediate状态变量来分析系统上的表锁定：mysql>show status like 'table%';

    ![](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\1615886541402.png)

  - 这里有两个状态变量记录MySQL内部表级锁定的情况，两个变量说明如下：
    Table_locks_immediate：产生表级锁定的次数，表示可以立即获取锁的查询次数，每立即获取锁值加1 ；
    Table_locks_waited：出现表级锁定争用而发生等待的次数(不能立即获取锁的次数，每等待一次锁值加1)，**此值高则说明存在着较严重的表级锁争用情况**；

  - **此外，Myisam的读写锁调度是写优先，这也是myisam不适合做写为主表的引擎。因为写锁后，其他线程不能做任何操作，大量的更新会使查询很难得到锁，从而造成永远阻塞**

##### 4.行锁（偏写）

###### 4.1 特点

- 偏向InnoDB存储引擎，开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低,并发度也最高。
- InnoDB与MyISAM的最大不同有两点：一是支持事务（TRANSACTION）；二是采用了行级锁

###### 4.2 结论

-  行锁只针对当前操作的行进行加锁 
-  Innodb存储引擎由于实现了行级锁定，虽然在锁定机制的实现方面所带来的性能损耗可能比表级锁定会要更高一些，但是在整体并发处理能力方面要远远优于MyISAM的表级锁定的。当系统并发量较高的时候，Innodb的整体性能和MyISAM相比就会有比较明显的优势了。

###### 4.3 行锁分析

- 【如何分析行锁定】
  通过检查InnoDB_row_lock状态变量来分析系统上的行锁的争夺情况
  mysql>show status like 'innodb_row_lock%';

  ![image-20210317091224532](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210317091224532.png)

- 对各个状态量的说明如下：

  - Innodb_row_lock_current_waits：当前正在等待锁定的数量；
  - Innodb_row_lock_time：从系统启动到现在锁定总时间长度；
  - Innodb_row_lock_time_avg：每次等待所花平均时间；
  - Innodb_row_lock_time_max：从系统启动到现在等待最常的一次所花的时间；
  - Innodb_row_lock_waits：系统启动后到现在总共等待的次数；
  - 对于这5个状态变量，**比较重要的主要是**
    - **Innodb_row_lock_time_avg**（等待平均时长），
    - **Innodb_row_lock_waits**（等待总次数）
    - **Innodb_row_lock_time**（等待总时长）这三项。
    - 尤其是当等待次数很高，而且每次等待时长也不小的时候，我们就需要分析系统中为什么会有如此多的等待，然后根据分析结果着手指定优化计划。

###### 4.5 MySQL中InnoDB引擎的行锁是怎么实现的

- 答：InnoDB是基于索引来完成行锁

- 例: select * from tab_with_index where id = 1 for update;  for update 可以根据条件来完成行锁锁定，并且 id 是有索引键的列，如果 id 不是索引键那么InnoDB将升级成表锁，并发将无从谈起

###### 4.6 间隙锁

- **什么是间隙锁**
       当我们用范围条件而不是相等条件检索数据，并请求共享或排他锁时，InnoDB会给符合条件的已有数据记录的索引项加锁；对于键值在条件范围内但并不存在的记录，叫做“间隙（GAP)”，InnoDB也会对这个“间隙”加锁，这种锁机制就是所谓的间隙锁（Next-Key锁）。
- **危害**
       因为Query执行过程中通过过范围查找的话，他会锁定整个范围内所有的索引键值，即使这个键值并不存在。
  间隙锁有一个比较致命的弱点，就是当锁定一个范围键值之后，即使某些不存在的键值也会被无辜的锁定，而造成在锁定的时候无法插入锁定键值范围内的任何数据。在某些场景下这可能会对性能造成很大的危害

###### 4.4 优化建议

- 尽可能让所有数据检索都通过索引来完成，避免无索引行锁升级为表锁。
- 合理设计索引，尽量缩小锁的范围
- 尽可能减少检索条件，避免间隙锁
- 尽量控制事务大小，减少锁定资源量和时间长度
- 尽可能低级别事务隔离

##### 5.页锁

- 开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般。了解即可

##### 6.面：什么是死锁？怎么解决？

- 死锁是指两个或多个事务在同一资源上相互占用，并请求锁定对方的资源，从而导致恶性循环的现象。

- 常见的解决死锁的方法
  - 1、如果不同程序会并发存取多个表，尽量约定以相同的顺序访问表，可以大大降低死锁机会。
  - 2、在同一个事务中，尽可能做到一次锁定所需要的所有资源，减少死锁产生概率；
  - 3、对于非常容易产生死锁的业务部分，可以尝试使用升级锁定颗粒度，通过表级锁定来减少死锁产生的概率；

### 三、查询与索引优化分析

#### 1.常见通用的Join查询

###### 1）SQL执行顺序

- 手写

  ![image-20210317111512977](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210317111512977.png)

- 机读

  ![image-20210317111525817](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210317111525817.png)

- 总结

  ![image-20210317111542230](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210317111542230.png)

###### 2）Join图

![image-20210317111244952](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210317111244952.png)

###### 3）7中Join语句

```sql
 
1 A、B两表共有
 select * from tbl_emp a inner join tbl_dept b on a.deptId = b.id;
 
2 A、B两表共有+A的独有
 select * from tbl_emp a left join tbl_dept b on a.deptId = b.id;
 
3 A、B两表共有+B的独有
 select * from tbl_emp a right join tbl_dept b on a.deptId = b.id;
 
4 A的独有 
select * from tbl_emp a left join tbl_dept b on a.deptId = b.id where b.id is null; 
 
5 B的独有
 select * from tbl_emp a right join tbl_dept b on a.deptId = b.id where a.deptId is null; #B的独有
 
6 AB全有
#MySQL Full Join的实现 因为MySQL不支持FULL JOIN,下面是替代方法
 #left join + union(可去除重复数据)+ right join
SELECT * FROM tbl_emp A LEFT JOIN tbl_dept B ON A.deptId = B.id
UNION
SELECT * FROM tbl_emp A RIGHT JOIN tbl_dept B ON A.deptId = B.id
 
7 A的独有+B的独有
SELECT * FROM tbl_emp A LEFT JOIN tbl_dept B ON A.deptId = B.id WHERE B.`id` IS NULL
UNION
SELECT * FROM tbl_emp A RIGHT JOIN tbl_dept B ON A.deptId = B.id WHERE A.`deptId` IS NULL;

```

#### 2.索引

##### 1）索引是什么

- 在数据之外，数据库系统还维护着满足特定查找算法的数据结构，这些数据结构以某种方式引用（指向）数据，
  这样就可以在这些数据结构上实现高级查找算法。这种数据结构，就是索引。
- 索引（Index）是帮助MySQL高效获取数据的数据结构。

##### 2）使用索引的好处

- 可以大大加快数据的检索速度，这也是创建索引的最主要的原因
- 通过索引列对数据进行排序，降低数据排序的成本，降低了CPU的消耗

##### 3）使用索引的缺点

- 实际上索引也是一张表，该表保存了主键与索引字段，并指向实体表的记录，所以索引列也是要占用空间的
- 虽然索引大大提高了查询速度，同时却会降低更新表的速度，如对表进行INSERT、UPDATE和DELETE更新表时，MySQL不仅要更新数据，还要更新一下索引文件信息。

##### 4）索引分类

- 主键索引： 数据列不允许重复，不允许为NULL，一个表只能有一个主键

- 唯一索引： 数据列不允许重复，允许为NULL值

- 普通索引：基本的索引类型，没有唯一性的限制，允许为NULL值。

- 基本语法：

  - 创建：

    ```sql
    CREATE  [UNIQUE ] INDEX indexName ON mytable(columnname(length)); 
    ALTER mytable ADD  [UNIQUE ]  INDEX [indexName] ON (columnname(length))
    ```

  - 删除

    ```sql
    DROP INDEX [indexName] ON mytable; 
    ```

  - 查看

    ```sql
    SHOW INDEX FROM table_name\G
    ```

  - 使用ALTER命令

    ```sql
    有四种方式来添加数据表的索引：
    ALTER TABLE tbl_name ADD PRIMARY KEY (column_list): 该语句添加一个主键，这意味着索引值必须是唯一的，且不能为NULL。
     
    ALTER TABLE tbl_name ADD UNIQUE index_name (column_list): 这条语句创建索引的值必须是唯一的（除了NULL外，NULL可能会出现多次）。
     
    ALTER TABLE tbl_name ADD INDEX index_name (column_list): 添加普通索引，索引值可出现多次。
     
    ALTER TABLE tbl_name ADD FULLTEXT index_name (column_list):该语句指定了索引为 FULLTEXT ，用于全文索引。
    ```

##### 5）MySQL索引的数据结构

-  索引的数据结构和具体存储引擎的实现有关，在MySQL中使用较多的索引有Hash索引，B+树索引等，经常使用的InnoDB存储引擎的默认索引实现为：B+树索引。对于哈希索引来说，底层的数据结构就是哈希表，因此在绝大多数需求为单条记录查询的时候，可以选择哈希索引，查询性能最快；其余大部分场景，建议选择BTree索引


###### ①面：B树和B+树的区别

- 在B树中，键和值存放在内部节点和叶子节点；但在B+树中，内部节点都是键，没有值，只有叶子节点同时存放键和值。

- B+树的所有叶子节点有一条链相连，而B树的叶子节点各自独立。

  ![img](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAxOC85LzIxLzE2NWZiNjgyZTc1OWNmMTI?x-oss-process=image/format,png)

###### ②面：数据库为什么使用B+树而不是B树

- B树只适合随机检索，而B+树同时支持随机检索和顺序检索；
- B+树空间利用率更高，可减少I/O次数，磁盘读写代价更低。一般来说，索引本身也很大，不可能全部存储在内存中，因此索引往往以索引文件的形式存储的磁盘上。这样的话，索引查找过程中就要产生磁盘I/O消耗。B+树的内部结点并没有指向关键字具体信息的指针，只是作为索引使用，其内部结点比B树小，盘块能容纳的结点中关键字数量更多，一次性读入内存中可以查找的关键字也就越多，相对的，IO读写次数也就降低了。而IO读写次数是影响索引检索效率的最大因素；
- B+树的叶子节点使用指针顺序连接在一起，只要遍历叶子节点就可以实现整棵树的遍历。而且在数据库中基于范围的查询是非常频繁的，而B树不支持这样的操作。
- 增删文件（节点）时，效率更高。因为B+树的叶子节点包含所有关键字，并以有序的链表结构存储，这样可很好提高增删效率。

###### ③面：Hash索引和B+树索引的区别

- Hash索引和B+树索引的底层实现原理不同：

  ​          hash索引底层就是hash表，进行查找时，调用一次hash函数就可以获取到相应的键值，之后进行回表查询获得实际数据。B+树底层实现是多路平衡查找树。对于每一次的查询都是从根节点出发，查找到叶子节点方可以获得所查键值，然后根据查询判断是否需要回表查询数据。

- hash索引进行等值查询更快，但是却无法进行范围查询，也没法使用索引进行排序
            因为在hash索引中经过hash函数建立索引之后，索引的顺序与原顺序无法保持一致，不能支持范围查询。而B+树的的所有节点皆遵循(左节点小于父节点，右节点大于父节点，多叉树也类似)，天然支持范围。

- hash索引不支持模糊查询以及多列索引的最左前缀匹配。原理也是因为hash函数的不可预测。AAAA和AAAAB的索引没有相关性。

- hash索引任何时候都避免不了回表查询数据，而B+树在符合某些条件(聚簇索引，覆盖索引等)的时候可以只通过索引完成查询。

- hash索引虽然在等值查询上较快，但是不稳定。性能不可预测，当某个键值存在大量重复的时候，发生hash碰撞，此时效率可能极差。而B+树的查询效率比较稳定，对于所有的查询都是从根节点到叶子节点，且树的高度较低。

###### ④面：联合索引是什么？为什么需要注意联合索引中的顺序？

- MySQL可以使用多个字段同时建立一个索引，叫做联合索引。在联合索引中，如果想要命中索引，需要按照建立索引时的字段顺序挨个使用，否则无法命中索引。

  具体原因为:

  MySQL使用索引时需要索引有序，假设现在建立了"name，age，school"的联合索引，那么索引的排序为: 先按照name排序，如果name相同，则按照age排序，如果age的值也相等，则按照school进行排序。

  当进行查询时，此时索引仅仅按照name严格有序，因此必须首先使用name字段进行等值查询，之后对于匹配到的列而言，其按照age字段严格有序，此时可以使用age字段用做索引查找，以此类推。因此在建立联合索引的时候应该注意索引列的顺序，一般情况下，将查询需求频繁或者字段选择性高的列放在前面。此外可以根据特例的查询或者表结构进行单独的调整

##### 6）创建索引的情况

- 1.主键自动建立唯一索引
- 2.频繁作为查询条件的字段应该创建索引
- 3.查询中与其它表关联的字段，外键关系建立索引
- 4.频繁更新的字段不适合创建索引:因为每次更新不单单是更新了记录还会更新索引，加重了IO负担
- 5.Where条件里用不到的字段不创建索引
- 6.单键/组合索引的选择问题，who？(在高并发下倾向创建组合索引)
- 7.查询中排序的字段，排序字段若通过索引去访问将大大提高排序速度
- 8.查询中统计或者分组字段

##### 7）不需要创建索引的情况

- 1.表记录太少 ----> MySQL官方文档说明是可以高效维护5-8百万的数据量，实际情况下真实数据达到3百万MySQL性能开始下降。

- 2.经常增删改的表  ----> Why:提高了查询速度，同时却会降低更新表的速度，如对表进行INSERT、UPDATE和DELETE。因为更新表时，MySQL不仅要保存数据，还要保存一下索引文件

- 3.数据重复且分布平均的表字段，因此应该只为最经常查询和最经常排序的数据列建立索引。注意，如果某个数据列包含许多重复的内容，为它建立索引就没有太大的实际效果。

  ![image-20210317133434100](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210317133434100.png)

#### 3.性能分析：Explain + SQL语句

##### 慢查询日志：

- MySQL的慢查询日志是MysQL提供的一种日志记录，它用来记录在MySQL中响应时间超过阀值的语句，具体指运行时间超过long-query_time值的SQL，则会被记录到慢查询日志中。
- 具体指运行时间超过long-query_time值的SQL，则会被记录到慢查询日志中。long_query_time的默认值为10，意思是运行10秒以上的语句。
- 由他来查看哪些SQL超出了我们的最大忍耐时间值，比如一条sql执行超过5秒钟，我们就算慢SQL，希望能收集超过5秒的sql，结合之前explain进行全面分析。

##### 1）explain概念

- 使用EXPLAIN关键字可以模拟优化器执行SQL查询语句，从而知道MySQL是如何处理你的SQL语句，分析查询语句，包含的信息如下图

  ![image-20210317151439301](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210317151439301.png)

##### 2）各字段解释

###### ① id（重要）

- select查询的序列号,包含一组数字，表示查询中执行select子句或操作表的顺序,分三种情况
  - id相同，执行顺序由上至下
  - id不同，如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行
  - id同时存在相同不同，不同的id大先执行，相同的顺序执行

###### ② select_type

- 查询的类型，主要是用于区别普通查询、联合查询、子查询等的复杂查询

  ![image-20210317152246753](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210317152246753.png)

###### ③ table

- 显示这一行的数据是关于哪张表的

###### ④ type（重要）

- type显示的是查询使用的访问类型，是较为重要的一个指标，结果值从最好到最坏依次是： system > const > eq_ref > ref > range > index > ALL ，一般来说，得保证查询至少**达到range级别，最好能达到ref**。

- 1.system

  - 表只有一行记录（等于系统表），这是const类型的特列，平时不会出现，这个也可以忽略不计

- 2.const

  - 表示通过索引一次就找到了,const用于比较primary key或者unique索引。因为只匹配一行数据，所以很快如将主键置于where列表中，MySQL就能将该查询转换为一个常量

- 3.eq_ref

  - 唯一性索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或唯一索引扫描

- 4.ref

  - 非唯一性索引扫描，返回匹配某个单独值的所有行.
  - 本质上也是一种索引访问，它返回所有匹配某个单独值的行，然而，
  - 它可能会找到多个符合条件的行，所以他应该属于查找和扫描的混合体

- 5.range

  - 只检索给定范围的行,使用一个索引来选择行。key 列显示使用了哪个索引

  - 一般就是在你的where语句中出现了between、<、>、in等的查询

  - 这种范围扫描索引扫描比全表扫描要好，因为它只需要开始于索引的某一点，而结束语另一点，不用扫描全部索引。

    ![image-20210317153020297](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210317153020297.png)

- 6.index:    

  - Full Index Scan，index与ALL区别为index类型只遍历索引树。这通常比ALL快，因为索引文件通常比数据文件小。（也就是说虽然all和Index都是读全表，但index是从索引中读取的，而all是从硬盘中读的）

- 7.ALL:   Full Table Scan，将遍历全表以找到匹配的行

###### ⑤ possible_keys

- 显示可能应用在这张表中的索引，一个或多个。查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询实际使用

###### ⑥ key(重要)

- 实际使用的索引。如果为NULL，则没有使用索引;查询中若使用了覆盖索引，则该索引仅出现在key列表中

  ![image-20210317153520182](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210317153520182.png)

###### ⑦ key_len

- 表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。在不损失精确性的情况下，长度越短越好

###### ⑧ ref

- 显示索引的哪一列被使用了，如果可能的话，是一个常数。哪些列或常量被用于查找索引列上的值

###### ⑨ rows

- 根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数

###### ⑩ Extra（包含3个重要参数）

- **Using filesort**:
  - MySQL会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取。当MySQL中无法利用已建索引完成的排序操作称为“文件排序”
- **Using temporary**:
  - 使用了临时表保存中间结果,MySQL在对查询结果排序时使用临时表。常见于排序 order by 和分组查询 group by。
- **Using Index**:
  - 表示相应的select操作中使用了覆盖索引(Covering Index)，避免访问了表的数据行，效率不错！
  - 如果同时出现using where，表明索引被用来执行索引键值的查找;
  - 如果没有同时出现using where，表明索引用来读取数据而非执行查找动作。
  - 覆盖索引（Covering Index）,一说为索引覆盖。
    - 就是select的数据列只用从索引中就能够取得，不必读取数据行，MySQL可以利用索引返回select列表中的字段，而不必根据索引再次读取数据文件,换句话说查询列要被所建的索引覆盖。
    - 如果要使用覆盖索引，一定要注意select列表中只取出需要的列，不可select *，
      因为如果将所有字段一起做索引会导致索引文件过大，查询性能下降。 

##### 3) 举例

![image-20210317155013343](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210317155013343.png)

- 第一行（执行顺序4）：id列为1，表示是union里的第一个select，select_type列的primary表 示该查询为外层查询，table列被标记为<derived3>，表示查询结果来自一个衍生表，其中derived3中3代表该查询衍生自第三个select查询，即id为3的select。【select d1.name......】
- 第二行（执行顺序2）：id为3，是整个查询中第三个select的一部分。因查询包含在from中，所以为derived。【select id,name from t1 where other_column=''】
- 第三行（执行顺序3）：select列表中的子查询select_type为subquery，为整个查询中的第二个select。【select id from t3】
- 第四行（执行顺序1）：select_type为union，说明第四个select是union里的第二个select，最先执行【select name,id from t2】
- 第五行（执行顺序5）：代表从union的临时表中读取行的阶段，table列的<union1,4>表示用第一个和第四个select的结果进行union操作。【两个结果union操作】

#### 4.索引优化

##### 1）多表连接优化

- 两表连接：

  ```sql
  EXPLAIN SELECT * FROM class LEFT JOIN book ON class.card = book.card;
  
  ALTER TABLE `book` ADD INDEX Y ( `card`);
  
  建索引：左连接建右表、右连接建左表
  ```

- 三表连接：

  ```sql
  EXPLAIN SELECT * FROM class LEFT JOIN book ON class.card=book.card LEFT JOIN phone ON book.card = phone.card;
  
  ALTER TABLE `phone` ADD INDEX z ( `card`);
  ALTER TABLE `book` ADD INDEX Y ( `card`);
  
  建索引：左连接建右表、右连接建左表
  ```

##### 2)  索引失效情况（避免）

- 1.全值匹配我最爱

  ```sql
  EXPLAIN SELECT * FROM staffs WHERE NAME = 'July';
  EXPLAIN SELECT * FROM staffs WHERE NAME = 'July' AND age = 25;
  EXPLAIN SELECT * FROM staffs WHERE NAME = 'July' AND age = 25 AND pos = 'dev';
  ```

  ![image-20210317175358787](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210317175358787.png)

- 2.最佳左前缀法则

  - 如果索引了多列，要遵守最左前缀法则。指的是查询从索引的**最左前列开始并且不跳过索引中的列**。

  ![image-20210317175541637](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210317175541637.png)

- 3.不在索引列上做任何操作（计算、函数、(自动or手动)类型转换），会导致索引失效而转向全表扫描

  ![image-20210317175729104](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210317175729104.png)

  - 索引列上使用了表达式，如where substr(a, 1, 3) = 'hhh'，where a = a + 1，表达式是一大忌讳，再简单mysql也不认。

- 4.存储引擎不能使用索引中范围条件右边的列

  -  in,between and,like,>等比较范围的关键字会使后面的索引失效

    ![image-20210317175905506](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210317175905506.png)

- 5.尽量使用覆盖索引(只访问索引的查询(索引列和查询列一致))，减少select *

  ![image-20210317180307368](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210317180307368.png)

- 6.mysql 在使用不等于(!= 或者<>)的时候无法使用索引会导致全表扫描

  ![image-20210317180504904](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210317180504904.png)

- 7.is null ,is not null 也无法使用索引

  ![image-20210317180522014](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210317180522014.png)

- 8.like以通配符开头('%abc...')mysql索引失效会变成全表扫描的操作

  ![image-20210317180544098](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210317180544098.png)

- 9.字符串不加单引号索引失效

  ![image-20210317180627552](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210317180627552.png)

- 10.少用or,用它来连接时会索引失效

  ![image-20210317180655005](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210317180655005.png)

- 11.总结

  ![image-20210317180752503](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210317180752503.png)

  ![image-20210317180809682](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210317180809682.png)

- 举例

  ```sql
  【建索引】
  create index idx_test03_c1234 on test03(c1,c2,c3,c4);
  show index from test03;
   
  问题：我们创建了复合索引idx_test03_c1234 ,根据以下SQL分析下索引使用情况？
   
  explain select * from test03 where c1='a1';
  explain select * from test03 where c1='a1' and c2='a2';
  explain select * from test03 where c1='a1' and c2='a2' and c3='a3';
  explain select * from test03 where c1='a1' and c2='a2' and c3='a3' and c4='a4';
   
   
  1）explain select * from test03 where c1='a1' and c2='a2' and c3='a3' and c4='a4';       用4个
  2）explain select * from test03 where c1='a1' and c2='a2' and c4='a4' and c3='a3';       用4个
  3）explain select * from test03 where c1='a1' and c2='a2' and c3>'a3' and c4='a4';      用到3个
  4）explain select * from test03 where c1='a1' and c2='a2' and c4>'a4' and c3='a3';      4个，MySQL自动优化
  5）explain select * from test03 where c1='a1' and c2='a2' and c4='a4' order by c3;        用到2个查找， c3作用在排序而不是查找，c4不起作用
  6）explain select * from test03 where c1='a1' and c2='a2' order by c3;                          同5）     
  7）explain select * from test03 where c1='a1' and c2='a2' order by c4;             
      出现了filesort
  8） 
  8.1 explain select * from test03 where c1='a1' and c5='a5' order by c2,c3; 
   只用c1一个字段索引，但是c2、c3用于排序,无filesort
   
  8.2 explain select * from test03 where c1='a1' and c5='a5' order by c3,c2;
   出现了filesort，我们建的索引是1234，它没有按照顺序来，3 2 颠倒了
   
  9） explain select * from test03 where c1='a1' and c2='a2' order by c2,c3;                 用c1、c2两个字段索引，但是c2、c3用于排序,无filesort
  10）explain select * from test03 where c1='a1' and c2='a2' and c5='a5' order by c2,c3;         
        用c1、c2两个字段索引，但是c2、c3用于排序,无filesort
   explain select * from test03 where c1='a1' and c2='a2' and c5='a5' order by c3,c2;       本例有常量c2的情况，和8.2对比，区别于8.2，原因是排序用的c2是个常量
   explain select * from test03 where c1='a1' and c5='a5' order by c3,c2;                 filesort
  11）explain select * from test03 where c1='a1' and c4='a4' group by c2,c3;               用到1个
  12）explain select * from test03 where c1='a1' and c4='a4' group by c3,c2;   
      Using where; Using temporary; Using filesort 
  
  ```

  

##### 3) order by 关键字优化

- ORDER BY子句，尽量使用Index方式排序,避免使用FileSort方式排序

- MySQL支持二种方式的排序，FileSort和Index，Index效率高.它指MySQL扫描索引本身完成排序。FileSort方式效率较低

- ORDER BY满足两情况，会使用Index方式排序

  - ORDER BY 语句使用索引最左前列

  - 使用Where子句与Order BY子句条件列组合满足索引最左前列

    ![image-20210318101333415](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210318101333415.png)

    ![image-20210318101350600](C:\Users\admin\AppData\Roaming\Typora\typora-user-images\image-20210318101350600.png)

- 如果不在索引列上，就要使用filesort，mysql中有两种算法：双路排序和单路排序
  - 双路排序：读取行指针和order by列，对他们进行排序，然后扫描已经排序好的列表，重新从列表中读取对应的数据输出。
    - 缺点是：需要读取记录两次，IO开销大，成本高。
  - 单路排序：原理是：按照筛选条件把SQL中涉及的字段全部读入排序缓冲区(sort buffer)里，然后依据排序字段进行排序，如果排序缓冲不够，会将临时排序结果写入到一个临时文件里，最后合并临时排序文件，直接返回已经排序好的结果集。
    - 优点是：不需要读取记录两次，相对于双路排序，减少I/O开销。
    - 缺点是：由于要读入所有字段，排序缓冲可能不够，需要额外的临时文件协助进行排序，导致增加额外的I/O成本。
  - 优化策略
    - 增大sort_buffer_size参数的设置，增大max_length_for_sort_data参数的设置，尽量使用单路排序

##### 4）group by 关键字优化

- group by实质是先排序后进行分组，遵照索引建的最佳左前缀
- 当无法使用索引列，增大max_length_for_sort_data参数的设置+增大sort_buffer_size参数的设置
- where高于having，能写在where限定的条件就不要去having限定了