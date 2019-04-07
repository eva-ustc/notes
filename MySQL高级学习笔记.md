# MySQL高级学习笔记

## 主要配置文件

1.二进制日志log-bin（主要用于主从复制）

2.错误日志log-error（默认是关闭的，记录严重的警告和错误信息，每次启动和关闭的详细信息等）

3.查询日志log（默认是关闭的，记录查询的sql语句，如果开启会减低mysql的整体性能，因为记录日志也是需要消耗系统资源）

4.数据文件（目录/var/lib/mysql，frm文件存放表结构，myd文件存表表数据，myi文件存放表索引）

mysql读sql从from开始解析，重组。

![img](https://www.e-learn.cn/sites/default/files/ueditor/1/upload/image/20180604/1528084230308433.png)

第一层客户端连接层

第二层管理服务层（包含连接池、sql接口、解析器&转化器、优化器、缓冲器）

第三层可拔插的数据库引擎层

第四层 文件系统存储层

 **总结**：和其他数据库相比，MySQL有点与众不同，他的架构可以在多种不同场景中应用并发挥良好作用。主要体现在存储引擎的

架构上，**插件式的存储引擎架构将查询处理和其他的系统任务以及数据的存储提取相分离**。这种架构可以根据业务的需求和实际需要选

择合适的存储引擎。



![7种join](http://image.bubuko.com/info/201807/20180702173156580987.jpg)



![join图解](img/join图解.jpg)

## 索引

 BTree索引、Hash索引、full-text全文索引、R-Tree索引等

​	**数据库索引采用B+树**的主要原因是 B树在提高了磁盘IO性能的同时并没有解决**元素遍历**的效率低下的问题。正是为了解决这个问题，B+树[应运而生](https://www.baidu.com/s?wd=%E5%BA%94%E8%BF%90%E8%80%8C%E7%94%9F&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)。B+树只要遍历叶子节点就可以实现整棵树的遍历。而且在数据库中基于范围的查询是非常频繁的，而B树不支持这样的操作（或者说效率太低）

​	至于**MongoDB为什么使用B-树**而不是B+树，可以从它的设计角度来考虑，它并不是传统的关系性数据库，而是以**Json格式**作为存储的nosql，目的就是高性能，高可用，易扩展。首先它摆脱了关系模型，上面所述的优点2需求就没那么强烈了，其次Mysql由于使用B+树，数据都在叶节点上，每次查询都需要访问到叶节点，而MongoDB使用B-树，所有节点都有Data域，只要找到指定索引就可以进行访问，无疑单次查询平均快于Mysql（但侧面来看Mysql至少平均查询耗时差不多）。

​	**总体来说，Mysql选用B+树和MongoDB选用B-树还是以自己的需求来选择的。**

### BTree索引检索原理

![img](https://www.e-learn.cn/sites/default/files/ueditor/1/upload/image/20180604/1528084232798887.png)

#### 初始化介绍

 一颗b+树，浅蓝色的块我们称之为一个磁盘块，可以看到每个磁盘块包含几个数据项（深蓝色所示）和指针（黄色所示），如磁盘

块1包含数据项17和35，包含指针P1、P2、P3，P1表示小于17的磁盘块，P2表示17和25之间的磁盘块，P3表示大于35的磁盘块。

 **真实的数据存在于叶子节点**即3、5、9、10、15、28、29、36、60、75、79、90、99。

 **非叶子节点不存储真是的数据，只存储指引搜索方向的数据项**，如17、35并不真是存在于数据表中。

#### 查找过程

 如果要查找数据项29，那么首先会把磁盘块1由磁盘加载到内存，此时发生一次IO，在内存中用二分查找确定29在17和35之间，锁定

磁盘块1的P2指针，内存时间因为非常短（相比磁盘的IO）可以忽略不计，通过磁盘块1的P2指针的磁盘地址把磁盘块3加载到内存，发生第二

次IO，29在26和30之间，锁定磁盘块3的P2指针，通过指针加载磁盘块8到内存，发生第三次IO，同时内存中做二分查找找到29，结束查询，

总计三次IO。

### 索引利弊

#### 哪些情况需要创建索引

1.主键自动建立唯一索引

2.频繁作为查询条件的字段应该创建索引

3.查询中与其他表关联的字段，外键关系建立索引

4.频繁更新的字段不合适创建索引

#### 哪些情况不要创建索引

1.表记录太少

2.经常增删改的表

3.字段重复的数据比较多（索引的选择性越接近1就可以建索引，如果选择性越小就不要建索引）

## MySQL性能分析

#### MySQL查询优化器

![img](https://www.e-learn.cn/sites/default/files/ueditor/1/upload/image/20180604/1528084235163652.png)

#### MySQL的SQL解析流程

![img](https://www.e-learn.cn/sites/default/files/ueditor/1/upload/image/20180604/1528084236681386.png)

#### MySQL引擎

##### MyISAM和InnoDB引擎

![img](https://www.e-learn.cn/sites/default/files/ueditor/1/upload/image/20180604/1528084237322584.png)

##### 阿里巴巴所用MySQL引擎

![img](https://www.e-learn.cn/sites/default/files/ueditor/1/upload/image/20180604/1528084239183672.png)

#### MySQL常见瓶颈

![img](https://www.e-learn.cn/sites/default/files/ueditor/1/upload/image/20180604/1528084240781157.png)

#### EXPLAIN

 查看执行计划

 Explain + SQL语句

![img](https://www.e-learn.cn/sites/default/files/ueditor/1/upload/image/20180604/1528084241948416.png)

##### **id**

 id值相同，表的执行顺序自上而下顺序执行

 id值不同，id值越大优先级越高

 id值有相同有不同，id值越大越优先执行，id值相同自上而下顺序执行。

##### search_type

 	**SIMPLE**--简单的select查询，查询中不包含子查询或者UNION

 	**PRIMARY**--查询中若包含任何复杂的子部分，最外层查询则被标记为PRIMARY（就是最外层那个）

 	**SUBQUERY**--在SELECT或WHERE列表中包含了子查询

 	**DERIVED**--在FROM列表中包含的子查询被标记为DERIVED（衍生）MySQL会递归执行这些子查询，把结果放在临时表里

 	**UNION**--若第二个SELECT出现在UNION关键字之后，则search_type被标记为UNION；若UNION关键字包含在FROM子句的子查询中，外层SELECT被标记为：DERIVED

 	**UNION RESULT**--从UNION表获取结果的SELECT

##### table

 显示这行数据是关于哪张表的

##### **type**

 从最好到最差依次是：**system>const>eq_ref>ref>range>index>ALL**

 	**system**--表只有一行记录（等于系统表），这是const类型的特例，平时不会出现，这个也可以忽略不计

​	 **const**--表示通过**索引一次**就找到了，const用于比较**primary key**或者**unique索引**。因为只匹配一行数据，所以很快，如将主键置于where列表中，MySQL就能将该查询转换为一个常量

 	**eq_ref**--**唯一性索引扫描**，对于每个索引键，表中只有**一条记录**与之匹配。常见于主键或唯一索引扫描

​	 **ref**--**唯一性索引扫描**，返回**匹配某个单独值的所有行**。本质上也是一种索引访问，它返回所有匹配某个单独值的行，然而，它可能会找到多个符合条件的行，所以他应该属于查找和扫描的混合体

​	 **range**-- 只检索给定范围的行，使用一个索引来选择行。key列显示使用了哪个索引，一般就是在你的where语句中出现了**between、<、>、in**等的查询，这种**范围扫描索引扫描比全表扫描要好**，因为它只需要开始于索引的某一点，而结束于另一点，不用扫描全部索引

 	**index**--Full Index Scan，index与ALL区别为index类型只遍历索引树。这通常比ALL快，因为索引文件通常比数据文件小。（**也就是说虽然all和Index都是读全表，**但index是从索引中读取的，而all是从硬盘中读取的）

​	 **ALL**--Full Table Scan，将遍历全表以找到匹配的行

 **备注：一般来说，得保证查询至少达到range级别，最好能达到ref**



##### possible_keys

 	显示可能应用在这张表中的索引，一个或多个。查询涉及到的字段上若存在索引，则该索引将被列出，但不一定被查询实际使用



##### **key**

​	 实际使用的索引。如果为NULL，则没有使用索引，**查询中若使用了覆盖索引，则该索引仅出现在key列表中**

​	 在possible_keys中可能没有索引，但是在key中有使用索引，说明可能使用了覆盖索引，覆盖索引是指查询结果列中刚好匹配对应顺序所建立的索引

##### key_len

 	表示索引中使用的字节数，可通过该列计算查询中使用的索引的长度。在不损失精确性的情况下，长度越短越好，key_len显示的值为索引字段的最大可能长度，**并非实际使用长度**，即key_len是根据表定义计算而得，不是通过表内检索出的

##### ref

 	显示索引的哪一列被使用了，如果可能的话，是一个常数。哪些列或常量被用于查找索引列上的值

##### rows

 	根据表统计信息及索引选用情况，大致估算出找到所需的记录所需要读取的行数

##### **Extra**

```
包含不适合在其他列中显示但十分重要的额外信息
```

![img](https://www.e-learn.cn/sites/default/files/ueditor/1/upload/image/20180604/1528084241581443.png)

​	**USING filesort**--说明mysql会对数据使用一个外部的索引排序(**内存中使用算法排序**)，而不是按照表内的索引顺序进行读取。**MySQL中无法利用索引完成的排序操作称为“文件内排序”**

![img](https://www.e-learn.cn/sites/default/files/ueditor/1/upload/image/20180604/1528084242305785.png)

​	**USING temporary**--使用了**临时表**保存中间结果，MySQL在对查询结果排序时使用临时表。**常见于排序order by和分组查询group by。**

![img](https://www.e-learn.cn/sites/default/files/ueditor/1/upload/image/20180604/1528084243322404.png)

​	**USING index**--表示相应的select操作中使用了**覆盖索引**（Covering Index），避免访问了表的数据行，效率不错！如果同时出现usering

​	where，表明索引被用来执行索引键值的查找；如果没有同时出现using where，表明索引用来读取数据而非执行查找动作

![img](https://www.e-learn.cn/sites/default/files/ueditor/1/upload/image/20180604/1528084244852097.png)

​	覆盖索引（Covering Index），一说为索引覆盖。

​	理解方式一：就是select的数据列只用从索引中就能够取得，不必读取数据行，MySQL可以利用索引返回select列表中的地段，而不必根据索引再次读取数据文件，**换句话说查询列要被所建的索引覆盖**

![img](https://www.e-learn.cn/sites/default/files/ueditor/1/upload/image/20180604/1528084246657795.png)

查询案例分析:

![查询案例](img/查询案例.png)

第五行(执行顺序5) : 代表从union的临时表中读取行的阶段,table列的<union1,4>表示用第一个和第四个的查询结果进行union操作.





## 索引优化

### 索引分析

​	单表: **range**索引字段后面的索引字段失效;

​	两表索引: 左右外连接**反向建索引**;

 	尽可能减少Join语句中的NestedLoop(**嵌套循环**)的循环总次数：**“永远用小结果集驱动大的结果集”**；

​	 **优先优化NestedLoop的内层循环；**

​	 **保证Join语句中被驱动表上Join条件字段已经被索引**；

 	当无法保证被驱动表的Join条件字段被索引且内存资源充足的前提下，**不要太吝惜JoinBuffer的设置**；

​	like: %开头索引会失效,解决办法是%写右边或者使用**覆盖索引**;

### 索引失效（应该避免）

![img](https://www.e-learn.cn/sites/default/files/ueditor/1/upload/image/20180604/1528084247712052.png)



最佳左前缀匹配,匹配几个用几个,直到不匹配;

索引列上不做任何操作(计算,函数,类型转换),会导致索引失效;

范围条件('>','<' '!=' 等)右边的索隐列无效;

is null is not null也会导致索引失效( null列索引多1字节,可变类型多2字节)

like 以通配符%开头会导致索引失效( 解决办法: 使用覆盖索引);

字符串不加单引号会导致类型转换,索引失效;

使用or也会导致索引失效;





## MySQL调优



![索引优化总结](img/索引优化总结.png)



### MySQL优化原则：小表驱动大表

 以MySQL关键字in与exists作为例子，如下：

![img](https://www.e-learn.cn/sites/default/files/ueditor/1/upload/image/20180604/1528084250810802.png)

 in与exists首先，写法上就不一样，如上图，

​	in查询是查询A中id在是B表的id数据，**以B表为驱动(即先查B表,再拿A表套条件)**的查询，

​	而exists是A表中A表的id与B表的id一样则exists为true的数据返回，是**以A表为驱动**（exists后面的内容返回true或false，exists子句select后面的查询列可以是随便一个常量，mysql最终都会忽略该列返回一个true或false）

 所以，根据优化原则小表驱动大表，

​	**当B表数据集小于A表的数据集时，in性能优于exists；**

​	**当A表数据集小于B表的数据集时，exists的性能优于in。**



MySQL调优一般分为4个步骤：**

1. 慢查询的开启与捕获
2. explain+慢SQL分析
3. show profile查询SQL在Mysql服务器里面的执行细节和生命周期情况
4. SQL数据库服务器的参数调优

### ORDER BY排序优化

​	 ORDER BY子句，尽量使用Index方式排序，避免使用FileSort方式排序，尽可能在索引列上完成排序操作，遵照索引建的最佳左前缀

**原则**

1. MySQL支持两种方式的排序，**FileSort和Index**。

  ​Index效率高，它指MySQL扫描索引本身完成排序。

  ​FileSort方式效率较低。

2. ORDER BY满足两种情况，会使用Index方法排序：

   ​	**ORDER BY语句使用索引最左前列;**

   ​	**使用Where子句与ORDER BY子句条件列组合满足索引最左前列;**

3. 如果不在索引列上，filesort有两种算法：mysql就要启动双路排序和单路排序

   **双路排序：**

```
	在MySQL4.1之前是使用双路排序，字面意思就是两次扫描磁盘，最终得到数据，读取行指针和order by列，对他们进行排序，然后扫描已经排序好的列表，按照列表中的值重新从列表中读取对应的数据输出。即：从磁盘取排序字段，在buff进行排序，在从磁盘取其他字段。
	 取一批数据，要对磁盘进行两次扫描，众所周知，I/O是很耗时的，所以在mysql4.1之后，出现了第二种改进的算法，就是单路排序。
```

 	**单路排序：**

```
	从磁盘读取查询需要的所有列，按照order by列在buffer对它们进行排序，然后扫描排序后的列表进行输出，它的效率更快一些，避免了第二次读取数据。并且把随机IO变成了顺序IO，但是它会使用更多的空间，因为他把每一行都保存在内存中了。即：一次性从磁盘中将所有字段取出来，只进行一次I/O。
	单路排序是后出的，总体而言好过双路排序，但是，单路排序有可能取出来的数据的总大小超出了sort_buffer的容量，导致每次只能取sort_buffer容量大小的数据，进行排序（创建tmp文件，多路合并），排完再取sort_buffer容量大小，再排......从而多次I/O。本来想省一次I/O操作，反而导致了大量的I/O操作，反而得不偿失。
```

**优化策略：**

![img](https://www.e-learn.cn/sites/default/files/ueditor/1/upload/image/20180604/1528084252706816.png)

总结：

![img](https://www.e-learn.cn/sites/default/files/ueditor/1/upload/image/20180604/1528084254965034.png)

### group by分组优化

![img](https://www.e-learn.cn/sites/default/files/ueditor/1/upload/image/20180604/1528084255428681.png)

### 慢日志查询

 **默认情况下，MySQL数据库没有开启慢查询日志**，需要我们手动来设置这个参数。

 **当然，如果不是调优需要的话，一般不建议启动该参数**，因为开启慢查询日志会或多或少带来一定的性能影响。慢查询日志支

持将日志记录写入文件。

#### 开启慢查询日志

```
#查看是否开启慢查询日志
SHOW VARIABLES LIKE '%slow_query_log%';
#开启慢查询日志
set global slow_query_log=1;
```

使用set global slow_query_log=1;开启了慢查询日志**只对当前数据库生效**，如果MySQL重启后会失效。

如果要永久生效，就必须修改配置文件my.cnf（其他系统变量也是如此）

修改my.cnf文件。[mysqld]下增加或修改参数

slow_query_log和slow_query_log_file后，然后重启MySQL服务器。也即将如下两行配置进my.cnf文件

```
slow_query_log=1
slow_query_log_file=/var/lib/mysql/xxx-slow.log
```

关于慢查询的参数slow_query_log_file，它指定慢查询日志文件的存放路径，**系统默认会给一个缺省的文件host_name-slow.log**（如果

没有指定参数slow_query_log_file的话）

开启了慢查询日志功能之后，mysql会根据参数long_query_time判断是否为慢SQL，默认情况下long_query_time的值为10秒，命令：

```
#查看mysql慢查询的配置时长
SHOW VARIABLES LIKE 'long_query_time%';
```

可以使用命令修改，也可以在my.cnf参数里面修改。

加入运行时间正好等于long_query_time的情况并不会被记录下来。也就是说，在mysql源码里面是**判断大于long_query_time，而非大于等

于**。

```
#设置mysql慢查询的配置时长
set global long_query_time=3;
```

**注意：修改完成之后，需要重新连接或新开一个会话才能看到修改值。**

查看当前系统中有多少条慢查询记录

```
#查看当前系统中有多少条慢查询记录
show global status like '%Slow_queries%'
```

配置my.cnf开启慢查询日志小总结：

```
slow_query_log=1
slow_query_log_file=/var/lib/mysql/xxx-slow.log
long_query_time=3
log_output=FILE
```

##### 日志分析工具mysqldumpslow

![img](https://www.e-learn.cn/sites/default/files/ueditor/1/upload/image/20180604/1528084255262041.png)

![img](https://www.e-learn.cn/sites/default/files/ueditor/1/upload/image/20180604/1528084257891408.png)

### 用Show Profile分析sql

 show profile是mysql提供可以用来分析当前会话中语句执行的资源消耗情况。可以用于SQL的调优测量。

 官网：<http://dev.mysql.com/doc/refman/5.5/en/show-profile.html>

 默认情况下，参数处于关闭状态，并保存最近15次的运行结果

```
SHOW VARIABLES LIKE 'profiling'
SET profiling=on;
```

查看结果

```
show profiles;
```

诊断SQL

```
#查看show profiles列出来的10号id的sql执行生命周期
show profile cpu,block io for query 10;
```

如出现以下结论，诊断的SQL可能需要进行优化

![img](https://www.e-learn.cn/sites/default/files/ueditor/1/upload/image/20180604/1528084258812652.png)

### 全局查询日志

配置全局查询日志

```
set global general_log=1;
set global log_output='TABLE';
```

伺候，你所编写的sql语句，将会记录到mysql库里的general_log表，可以用下面的命令查看

```
select * from mysql.general_log;
```

**永远不要在生产环境开启这个功能。**

## 数据库锁

### 锁的分类

从对数据操作的类型（读/写）分

- 读锁（共享锁）：针对同一份数据，多个读操作可以同时进行而不会相互影响
- 写锁（排它锁）：当写操作没有完成前，它会阻塞其他写锁和读锁

从对数据操作的粒度分

- 表锁
- 行锁
- 页锁

### 加锁&解锁

```
#给表加锁
lock table mytable1 read, mytable2 write; 
#查看表上加过的锁
show open tables;
#释放表锁
unlock tables;
```

### MyISAM表锁

#### 加读锁

| session1                     | session2                           |
| ---------------------------- | ---------------------------------- |
| 对table1表加读锁                  | 连接终端                               |
| 当前session可以查询该表记录            | 其他session也可以查询该表记录                 |
| 当前session**不能查询其他没有锁定的表**    | 其他session可以查询或更新未锁定的表              |
| 当前session中插入或更新锁定的**都会提示错误** | 其他session插入或者更新锁定表**会阻塞，会一直等待获得锁** |
| 释放锁                          | 获得锁，插入操作完成                         |

#### 加写锁

| session1                      | session2                               |
| ----------------------------- | -------------------------------------- |
| 对table1表加写锁                   | 连接终端                                   |
| 当前session对锁定表的查询+更新+插入操作都可以执行 | 其他session对锁定表的查询被阻塞，需要等待锁被释放（备注：如果可以，请 |

换成不同的id来进行测试，因为mysql有缓存，第2次的条件会从缓存中取，影响锁效果） |

#### 读写锁总结

 **简而言之，就是读锁会阻塞写，但是不会堵塞读。而写锁则会把读和写都阻塞。**

![img](https://www.e-learn.cn/sites/default/files/ueditor/1/upload/image/20180604/1528084260723623.png)

#### 查询分析表锁定

 	此外，MyISAM的读写锁调度是写优先，这也是MyISAM不适合做写为主表的引擎。因为写锁后，其他线程不能做任何操作，大量的更新会使查询很难得到锁，从而造成永远阻塞。

![img](https://www.e-learn.cn/sites/default/files/ueditor/1/upload/image/20180604/1528084261823248.png)

### InnoDB行锁

 MySQL行锁的特点，偏向InnoDB存储引擎，开销大，加锁慢；会出现思索：锁定粒度最小，发生锁冲突的概率最低，并发度也最高。

InnoDB与MyISAM的最大不同有两点：一是支持事务（TRANSACTION）；二是采用了行级锁。

> 行锁支持事务（Transaction）--ACID
>
> 并发事务处理带来的问题：更新丢失、脏读、不可重复读、幻读

![img](https://www.e-learn.cn/sites/default/files/ueditor/1/upload/image/20180604/1528084264222678.png)

```
begin;
select * from mytable where id=xxx for update;
commit;
```

#### 索引失效--行锁变表锁

 **注意：索引失效会导致行锁变表锁。如：当索引列类型转换会导致索引失效，此时会将行锁变为表锁，更新或插入操作会锁表**

#### 间隙锁及其危害

 当我们用范围条件而不是相等条件检索数据，并请求共享或排他锁时，InnoDB会给符合条件的已有数据的索引项加锁；对于键值在

条件范围内但并不存在的记录，叫做“间隙（GAP）”，InnoDB也会对这个“间隙”加锁，这种锁机制就是所谓的间隙锁（Next-Key锁）。

> 危害：
>
>  因为Query执行过程中通过范围查找的话，他会锁定整个范围内所有的索引键值，即使这个键值并不存在。
>
>  间隙锁有一个比较致命的弱点，就是当锁定一个范围键值之后，即使某些不存在的键值也会被无辜的锁定，而造成在锁定的时候无

法插入锁定值范围内的任何数据。在某些场景下这可能会对性能造成很大的危害。

#### 行锁总结

 Innodb存储引擎由于事先了行级锁定，虽然在锁定机制的实现方面锁带来的性能损耗可能比表级锁定会要更高一些，但是在整体并

发处理能力方面要远远优于MyISAM的表级锁定。当系统并发量较高的时候，Innodb的整体性能和MyISAM相比就会有比较明显的优势了。

 但是Innodb的行级锁定同样也有其脆弱的一面，当我们使用不当的时候，可能会让Innodb的整体性能表现不仅不能比MyISAM高，甚

至可能会更差。（索引失效行锁变表锁）

#### 行锁分析

```
show status like 'innodb_row_lock%';
```

![img](https://www.e-learn.cn/sites/default/files/ueditor/1/upload/image/20180604/1528084264783220.png)

各个状态量的说明如下：

Innodb_row_lock_current_waits：当前正在等待锁定的数量

Innodb_row_lock_time：从系统启动到现在锁定总时间长度

Innodb_row_lock_time_avg：每次等待所花平均时间

Innodb_row_lock_time_max：从系统启动到现在等待最长的一次所花的时间

Innodb_row_lock_waits：系统启动后到现在总共等待的次数

对于这5个状态变量，**比较重要的主要是**

**Innodb_row_lock_time_avg（等待平均时长），Innodb_row_lock_waits（等待总次数），Innodb_row_lock_time（等待总时长）这三项。

**

尤其是当等待次数很高，而且每次等待时长也不小的时候，我们就需要分析系统中为什么会有如此多的等待，然后根据分析结果着手制定优

化计划。**（show profiles）**

#### **优化建议**

> 1.尽可能让所有数据检索都通过索引来完成，避免无索引行锁升级为表锁。
>
> 2.合理设计索引，尽量缩小锁的范围
>
> 3.尽可能较少检索条件，避免间隙锁
>
> 4.尽量控制事务大小，减少锁定资源量和时间长度
>
> 5.尽可能低级别事务隔离

### 页锁

 开销和加锁时间界于表锁和行锁之间；会出现死锁；锁定粒度界于表锁和行锁之间，并发一般。（现在基本不用，了解即可）

## MySQL主从复制

![img](https://www.e-learn.cn/sites/default/files/ueditor/1/upload/image/20180604/1528084265207477.png)

### 原理

![img](https://www.e-learn.cn/sites/default/files/ueditor/1/upload/image/20180604/1528084266543189.png)

MySQL复制过程分成三步：

1.master将改变记录到二进制日志（binary log）。这些记录过程叫做二进制日志时间，binary log events；

2.slave将master的binary log events拷贝到它的中继日志（relay log）；

3.slave重做中继日志中的事件，将改变应用到自己的数据库中。MySQL复制是异步的且串行化的。

### 具体实现

步骤：

- 主服务器：
- 开启二进制日志
- 配置唯一的server-id
- 获得master二进制日志文件名及位置
- 创建一个用于slave和master通信的用户账号
- 从服务器：
- 配置唯一的server-id
- 使用master分配的用户账号读取master二进制日志
- 启用slave服务

具体实现如下：

**一、准备工作：**

1.主从数据库版本最好一致

2.主从数据库内数据保持一致

主数据库：182.92.172.80 /linux

从数据库：123.57.44.85 /linux

**二、主数据库master修改：**

**1.修改mysql配置**

找到主数据库的配置文件my.cnf(或者my.ini)，我的在/etc/mysql/my.cnf,在[mysqld]部分插入如下两行：

```
[mysqld]
log-bin=mysql-bin #开启二进制日志
server-id=1 #设置server-id
```

**2.重启mysql，创建用于同步的用户账号**

打开mysql会话shell>mysql -hlocalhost -uname -ppassword

创建用户并授权：用户：rel1密码：slavepass

```
mysql> CREATE USER 'repl'@'123.57.44.85' IDENTIFIED BY 'slavepass';#创建用户
mysql> GRANT REPLICATION SLAVE ON *.* TO 'repl'@'123.57.44.85';#分配权限
mysql>flush privileges;   #刷新权限
```

**3.查看master状态，记录二进制文件名(mysql-bin.000003)和位置(73)：**

```
mysql > SHOW MASTER STATUS;
+------------------+----------+--------------+------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+------------------+----------+--------------+------------------+
| mysql-bin.000003 | 73       | test         | manual,mysql     |
+------------------+----------+--------------+------------------+
```

**三、从服务器slave修改：**

**1.修改mysql配置**

同样找到my.cnf配置文件，添加server-id

```
[mysqld]
server-id=2 #设置server-id，必须唯一
```

**2.重启mysql，打开mysql会话，执行同步SQL语句**(需要主服务器主机名，登陆凭据，二进制文件的名称和位置)：

```
mysql> CHANGE MASTER TO
    ->     MASTER_HOST='182.92.172.80',
    ->     MASTER_USER='rep1',
    ->     MASTER_PASSWORD='slavepass',
    ->     MASTER_LOG_FILE='mysql-bin.000003',
    ->     MASTER_LOG_POS=73;
```

**3.启动slave同步进程：**

```
mysql>start slave;
```

**4.查看slave状态：**

```
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 182.92.172.80
                  Master_User: rep1
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000013
          Read_Master_Log_Pos: 11662
               Relay_Log_File: mysqld-relay-bin.000022
                Relay_Log_Pos: 11765
        Relay_Master_Log_File: mysql-bin.000013
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
        ...
```

当Slave_IO_Running和Slave_SQL_Running都为YES的时候就表示主从同步设置成功了。接下来就可以进行一些验证了，比如在主master数据

库的test数据库的一张表中插入一条数据，在slave的test库的相同数据表中查看是否有新增的数据即可验证主从复制功能是否有效，还可以

关闭slave（mysql>stop slave;）,然后再修改master，看slave是否也相应修改（停止slave后，master的修改不会同步到slave），就可以

完成主从复制功能的验证了。

还可以用到的其他相关参数：

master开启二进制日志后默认记录所有库所有表的操作，可以通过配置来指定只记录指定的数据库甚至指定的表的操作，具体在mysql配置文

件的[mysqld]可添加修改如下选项：

```
# 不同步哪些数据库  
binlog-ignore-db = mysql  
binlog-ignore-db = test  
binlog-ignore-db = information_schema  
  
# 只同步哪些数据库，除此之外，其他不同步  
binlog-do-db = game  
```

如之前查看master状态时就可以看到只记录了test库，忽略了manual和mysql库。

## 后记：批量插入数据脚本

> 函数与存储过程都是数据库编程的脚本，可以类比于java的方法，而函数与存储过程的区别在于函数有返回值，而存储过程没有返回值。

创建函数，假如报错：This function has none of DETERMINISTIC......

```
#由于开启过慢查询日志，因为我们开启了bin-log，我们就必须为我们的function指定一个参数
SHOW VARIABLES LIKE 'log_bin_trust_function_creators';

set global log_bin_trust_function_creators=1;

#这样添加了参数以后，如果mysqld重启，上述参数又会消失，永久方法还是在my.cnf中添加该参数键值对

log_bin_trust_function_creators=1
```

创建产生随机数函数，样例：

![img](https://www.e-learn.cn/sites/default/files/ueditor/1/upload/image/20180604/1528084267196408.png)

创建批量插入数据存储过程，样例：

![img](https://www.e-learn.cn/sites/default/files/ueditor/1/upload/image/20180604/1528084268676358.png)