> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.aneasystone.com](https://www.aneasystone.com/archives/2017/11/solving-dead-locks-two.html)

> 在上一篇博客中，我们学习了事务以及事务并发时可能遇到的问题，并介绍了四种不同的隔离级别来解决这些并发问题，在隔离级别的实现一节中，我们提到了锁的概念，锁是实现事务并发的关键。其实，锁的概念不仅仅...

在上一篇博客中，我们学习了事务以及事务并发时可能遇到的问题，并介绍了四种不同的隔离级别来解决这些并发问题，在隔离级别的实现一节中，我们提到了锁的概念，锁是实现事务并发的关键。其实，锁的概念不仅仅出现在数据库中，在大多数的编程语言中也存在，譬如 Java 中的 synchronized，C# 中的 lock 等，所以对于开发同学来说应该是不陌生的。但是数据库中的锁有很多花样，一会是行锁表锁，一会是读锁写锁，又一会是记录锁意向锁，概念真是层出不穷，估计很多同学就晕了。

在讨论传统的隔离级别实现的时候，我们就提到：通过对锁的类型（读锁还是写锁），锁的粒度（行锁还是表锁），持有锁的时间（临时锁还是持续锁）合理的进行组合，就可以实现四种不同的隔离级别；但是上一篇博客中并没有对锁做更深入的介绍，我们这一篇就来仔细的学习下 MySQL 中常见的锁类型。

这是《解决死锁之路》系列博文中的一篇，你还可以阅读其他几篇：

1.  [学习事务与隔离级别](https://www.aneasystone.com/archives/2017/10/solving-dead-locks-one.html)
2.  了解常见的锁类型
3.  [掌握常见 SQL 语句的加锁分析](https://www.aneasystone.com/archives/2017/12/solving-dead-locks-three.html)
4.  [死锁问题的分析和解决](https://www.aneasystone.com/archives/2018/04/solving-dead-locks-four.html)

一、表锁 vs. 行锁
-----------

在 MySQL 中锁的种类有很多，但是最基本的还是表锁和行锁：表锁指的是对一整张表加锁，一般是 DDL 处理时使用，也可以自己在 SQL 中指定；而行锁指的是锁定某一行数据或某几行，或行和行之间的间隙。行锁的加锁方法比较复杂，但是由于只锁住有限的数据，对于其它数据不加限制，所以并发能力强，通常都是用行锁来处理并发事务。表锁由 MySQL 服务器实现，行锁由存储引擎实现，常见的就是 InnoDb，所以通常我们在讨论行锁时，隐含的一层意义就是数据库的存储引擎为 InnoDb ，而 MyISAM 存储引擎只能使用表锁。

### 1.1 表锁

表锁由 MySQL 服务器实现，所以无论你的存储引擎是什么，都可以使用。一般在执行 DDL 语句时，譬如 **ALTER TABLE** 就会对整个表进行加锁。在执行 SQL 语句时，也可以明确对某个表加锁，譬如下面的例子：

```
mysql> lock table products read;
Query OK, 0 rows affected (0.00 sec)
 
mysql> select * from products where id = 100;
 
mysql> unlock tables;
Query OK, 0 rows affected (0.00 sec)
```

上面的 SQL 首先对 products 表加一个表锁，然后执行查询语句，最后释放表锁。表锁可以细分成两种：读锁和写锁，如果是加写锁，则是 `lock table products write` 。详细的语法可以参考 [MySQL 的官网文档](https://dev.mysql.com/doc/refman/5.7/en/lock-tables.html)。

关于表锁，我们要了解它的加锁和解锁原则，要注意的是它使用的是 **一次封锁** 技术，也就是说，我们会在会话开始的地方使用 lock 命令将后面所有要用到的表加上锁，在锁释放之前，我们只能访问这些加锁的表，不能访问其他的表，最后通过 unlock tables 释放所有表锁。这样的好处是，不会发生死锁！所以我们在 MyISAM 存储引擎中，是不可能看到死锁场景的。对多个表加锁的例子如下：

```
mysql> lock table products read, orders read;
Query OK, 0 rows affected (0.00 sec)
 
mysql> select * from products where id = 100;
 
mysql> select * from orders where id = 200;
 
mysql> select * from users where id = 300;
ERROR 1100 (HY000): Table 'users' was not locked with LOCK TABLES
 
mysql> update orders set price = 5000 where id = 200;
ERROR 1099 (HY000): Table 'orders' was locked with a READ lock and can't be updated
 
mysql> unlock tables;
Query OK, 0 rows affected (0.00 sec)
```

可以看到由于没有对 users 表加锁，在持有表锁的情况下是不能读取的，另外，由于加的是读锁，所以后面也不能对 orders 表进行更新。MySQL 表锁的加锁规则如下：

*   对于读锁
    
    *   持有读锁的会话可以读表，但不能写表；
    *   允许多个会话同时持有读锁；
    *   其他会话就算没有给表加读锁，也是可以读表的，但是不能写表；
    *   其他会话申请该表写锁时会阻塞，直到锁释放。
*   对于写锁
    
    *   持有写锁的会话既可以读表，也可以写表；
    *   只有持有写锁的会话才可以访问该表，其他会话访问该表会被阻塞，直到锁释放；
    *   其他会话无论申请该表的读锁或写锁，都会阻塞，直到锁释放。

锁的释放规则如下：

*   使用 UNLOCK TABLES 语句可以显示释放表锁；
*   如果会话在持有表锁的情况下执行 LOCK TABLES 语句，将会释放该会话之前持有的锁；
*   如果会话在持有表锁的情况下执行 START TRANSACTION 或 BEGIN 开启一个事务，将会释放该会话之前持有的锁；
*   如果会话连接断开，将会释放该会话所有的锁。

### 1.2 行锁

表锁不仅实现和使用都很简单，而且占用的系统资源少，所以在很多存储引擎中使用，如 MyISAM、MEMORY、MERGE 等，MyISAM 存储引擎几乎完全依赖 MySQL 服务器提供的表锁机制，查询自动加表级读锁，更新自动加表级写锁，以此来解决可能的并发问题。但是表锁的粒度太粗，导致数据库的并发性能降低，为了提高数据库的并发能力，InnoDb 引入了行锁的概念。行锁和表锁对比如下：

*   表锁：开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低；
*   行锁：开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。

行锁和表锁一样，也分成两种类型：读锁和写锁。常见的增删改（INSERT、DELETE、UPDATE）语句会自动对操作的数据行加写锁，查询的时候也可以明确指定锁的类型，SELECT ... LOCK IN SHARE MODE 语句加的是读锁，SELECT ... FOR UPDATE 语句加的是写锁。

行锁这个名字听起来像是这个锁加在某个数据行上，实际上这里要指出的是：在 MySQL 中，行锁是加在索引上的。所以要深入了解行锁，还需要先了解下 MySQL 中索引的结构。

#### 1.2.1 MySQL 的索引结构

我们知道，数据库中索引的作用是方便服务器根据用户条件快速查找数据库中的数据。在一堆有序的数据集中查找某条特定的记录，通常我们会使用[二分查找算法（Binary search）](https://zh.wikipedia.org/wiki/%E4%BA%8C%E5%88%86%E6%90%9C%E7%B4%A2%E7%AE%97%E6%B3%95)，使用该算法查询一组固定长度的数组数据是没问题的，但是如果数据集是动态增减的，使用扁平的数组结构就变得不那么方便了。所以，后来又发明了一种新的数据结构：[二叉查找树（Binary search tree）](https://zh.wikipedia.org/wiki/%E4%BA%8C%E5%85%83%E6%90%9C%E5%B0%8B%E6%A8%B9)，又叫做排序二叉树（Sorted binary tree），使用树形结构的好处是，可以大大的提高数据插入和删除的复杂度。二叉查找树查找算法的复杂度依赖于树的高度，为 O(log n)，譬如下面这样的一颗 3 层的二叉查找树，查找任意元素最多不超过 3 次比较就可以找到。

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2025/02/27/3887402864.png)

当然这是最理想的一种情况，我们考虑下最糟糕的一种情况，当数据本身就是有序的时候，生成的二叉树将会退化为线性表，复杂度就变成了 O(n)，如下图所示：

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2025/02/27/1895065164.png)

为了解决这个问题，人们又想出了一种新的解决方法，那就是[平衡树（Balanced search tree）](https://zh.wikipedia.org/wiki/%E5%B9%B3%E8%A1%A1%E6%A0%91)，其实平衡树有很多种，但是通常我们讨论的是 [AVL 树](https://zh.wikipedia.org/wiki/AVL%E6%A0%91)，这个名字来自于它的发明者：G.M. Adelson-Velsky 和 E.M. Landis。通过平衡树的 **树旋转** 操作，可以使得任何情况下二叉树的任意两个子树高度最大差为 1，也就是说让二叉树一直保持着矮矮胖胖的身材，这样保证对树的所有操作最坏复杂度都是 O(log n)。

那这些树结构和 MySQL 的索引有什么关系呢？其实，无论是 InnoDb 还是 MyISAM 存储引擎，它们的索引采用的数据结构都是 [B+ 树](https://zh.wikipedia.org/wiki/B%2B%E6%A0%91)，而 B+ 树又是从 [B 树](https://zh.wikipedia.org/wiki/B%E6%A0%91)演变而来。二叉树虽然查找效率很高，但是也有着一些局限性，特别是当数据存储于外部设备时（如数据库或文件系统），因为这个时候不仅需要考虑算法本身的复杂度，还需要考虑程序与外部设备之间的读写效率。在二叉树中，每一个树节点只保存一条数据，并且最多有两个子节点。程序在查找数据时，读取一条数据，比较，再读取另一条数据，再比较，再读取，如此往复。每读取一次，都涉及到程序和外部设备之间的 IO 开销，而这个开销将大大降低程序的查找效率。于是，便有人提出了增加节点保存的数据条数的想法，譬如 [2-3 树](https://zh.wikipedia.org/wiki/2-3%E6%A0%91)（每个节点保存 1 条或 2 条数据）、[2-3-4 树](https://zh.wikipedia.org/wiki/2-3-4%E6%A0%91)（每个节点保存 1 条、2 条或 3 条数据）等，当然也不用限定得这么死，数值范围可以 1 - n 条，这就是 B 树。在实际应用中，会根据硬盘上一个 page 的大小来调整 n 的数值，这样可以让一次 IO 操作就读取到 n 条数据，减少了 IO 开销，并且，树的高度显著降低了，查找时只需几次 page 的 IO 即可定位到目标（page 翻译为中文为页，表示 InnoDB 每次从磁盘（data file）到内存（buffer pool）之间传送数据的大小）。一颗典型的 B 树如下图所示（[图片来源](http://novoland.github.io/%E6%95%B0%E6%8D%AE%E5%BA%93/2014/07/26/MySQL%E6%80%BB%E7%BB%93.html)）：

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2025/02/27/3506926970.png)

不过在现实场景里几乎没有地方在使用 B 树，这是因为 B 树没有很好的伸缩性，它将多条数据都保存在节点里，如果数据中某个字段太长，一个 page 能容纳的数据量将受到限制，最坏的情况是一个 page 保存一条数据，这个时候 B 树退化成二叉树；另外 B 树无法修改字段最大长度，除非调整 page 大小，重建整个数据库。于是，B+ 树横空出世，在 B+ 树里，内节点（非叶子节点）中不再保存数据，而只保存用于查找的 key，并且所有的叶子节点按顺序使用链表进行连接，这样可以大大的方便范围查询，只要先查到起始位置，然后按链表顺序查找，一直查到结束位置即可。如下图所示（[图片来源](http://novoland.github.io/%E6%95%B0%E6%8D%AE%E5%BA%93/2014/07/26/MySQL%E6%80%BB%E7%BB%93.html)）：

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2025/02/27/2847462571.png)

那么在 B+ 树中，数据保存在什么地方呢？关于这一点，InnoDb 和 MyISAM 的实现是不一样的，InnoDb 将数据保存在叶子节点中，而 MyISAM 将数据保存在独立的文件中，MyISAM 有三种类型的文件：*.frm 用于存储表的定义，*.MYI 用于存放表索引，*.MYD 用于存放数据。MYD 文件中的数据是以堆表的形式存储的，所以像 MyISAM 这样以堆形式存储数据的我们通常把它叫做 **堆组织表（Heap organized table，简称 HOT）**，而像 InnoDb 这种将数据保存在叶子节点中，叫做 **索引组织表（Index organized table，简称 IOT）**。

MyISAM 索引结构如下图所示（[图片来源](http://novoland.github.io/%E6%95%B0%E6%8D%AE%E5%BA%93/2014/07/26/MySQL%E6%80%BB%E7%BB%93.html)）：

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2025/02/27/1183701961.png)

InnoDb 索引结构如下图所示（[图片来源](http://novoland.github.io/%E6%95%B0%E6%8D%AE%E5%BA%93/2014/07/26/MySQL%E6%80%BB%E7%BB%93.html)）：

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2025/02/27/1483339495.png)

可以看到，MyISAM 索引的 B+ 树中，非叶子节点中保存 key，叶子节点中保存着数据的地址，指向数据文件中数据的位置；InnoDb 索引的 B+ 树中，非叶子节点和 MyISAM 一样保存 key，但是叶子节点直接保存数据。所以，MyISAM 在通过索引查找数据时，必须通过两步才能拿到数据（先获取数据的地址，再读取数据文件），InnoDb 在通过索引查找数据时，可以直接读取数据。

注意上面两张图都是对应着 Primary Key 的情况，MySQL 有两种索引类型：主键索引（Primary Index）和非主键索引（Secondary Index，又称为二级索引、辅助索引），MyISAM 存储引擎对两种索引的存储没有区别，InnoDb 存储引擎的数据是保存在主键索引里的，非主键索引里保存着该节点对应的主键。所以 InnoDb 的主键索引有时候又被称为 **聚簇索引（Clustered Index）**，二级索引被称为 **非聚簇索引（Nonclustered Index）**。如果没有主键，InnoDB 会试着使用一个非空的唯一索引（Unique nonnullable index）代替；如果没有这种索引，会定义一个隐藏的主键。所以 InnoDb 的表一定会有主键索引。关于聚簇索引和二级索引，可以参看这里的 [MySQL 文档](https://dev.mysql.com/doc/refman/5.7/en/innodb-index-types.html)。（疑惑：MyISAM 如果没有索引，会怎么样？会定义隐藏的主键吗？）

#### 1.2.2 MySQL 加锁流程

关于 MySQL 的索引是一个很大的话题，譬如，增删改查时 B+ 树的调整算法是怎样实现的，如何通过索引加快 SQL 的执行速度，如何优化索引，等等等等。我们这里为了加强对锁的理解，只需要了解索引的数据结构即可。当执行下面的 SQL 时（id 为 students 表的主键），我们要知道，InnoDb 存储引擎会在 id = 49 这个主键索引上加一把 X 锁。

```
mysql> update students set score = 100 where id = 49;
```

当执行下面的 SQL 时（name 为 students 表的二级索引），InnoDb 存储引擎会在 name = 'Tom' 这个索引上加一把 X 锁，同时会通过 name = 'Tom' 这个二级索引定位到 id = 49 这个主键索引，并在 id = 49 这个主键索引上加一把 X 锁。

```
mysql> update students set score = 100 where name = 'Tom';
```

加锁过程如下图所示：

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2025/02/27/1423030488.png)

像上面这样的 SQL 比较简单，只操作单条记录，如果要同时更新多条记录，加锁的过程又是什么样的呢？譬如下面的 SQL（假设 score 字段为二级索引）：

```
mysql> update students set level = 3 where score >= 60;
```

下图展示了当用户执行这条 SQL 时，MySQL Server 和 InnoDb 之间的执行流程：

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2025/02/27/201797556.png)

从图中可以看到当 UPDATE 语句被发给 MySQL 后，MySQL Server 会根据 WHERE 条件读取第一条满足条件的记录，然后 InnoDB 引擎会将第一条记录返回并加锁（current read），待 MySQL Server 收到这条加锁的记录之后，会再发起一个 UPDATE 请求，更新这条记录。一条记录操作完成，再读取下一条记录，直至没有满足条件的记录为止。因此，MySQL 在操作多条记录时 InnoDB 与 MySQL Server 的交互是一条一条进行的，加锁也是一条一条依次进行的，先对一条满足条件的记录加锁，返回给 MySQL Server，做一些 DML 操作，然后在读取下一条加锁，直至读取完毕。理解这一点，对我们后面分析复杂 SQL 语句的加锁过程将很有帮助。

### 1.2.3 行锁种类

根据锁的粒度可以把锁细分为表锁和行锁，行锁根据场景的不同又可以进一步细分，在 MySQL 的源码里，定义了四种类型的行锁，如下：

```
#define LOCK_TABLE  16  /* table lock */
#define LOCK_REC    32  /* record lock */
 
/* Precise modes */
#define LOCK_ORDINARY   0   
#define LOCK_GAP    512 
#define LOCK_REC_NOT_GAP 1024   
#define LOCK_INSERT_INTENTION 2048
```

*   LOCK_ORDINARY：也称为 **Next-Key Lock**，锁一条记录及其之前的间隙，这是 RR 隔离级别用的最多的锁，从名字也能看出来；
*   LOCK_GAP：间隙锁，锁两个记录之间的 GAP，防止记录插入；
*   LOCK_REC_NOT_GAP：只锁记录；
*   LOCK_INSERT_INTENSION：插入意向 GAP 锁，插入记录时使用，是 LOCK_GAP 的一种特例。

这四种行锁将是理解并解决数据库死锁的关键，我们下面将深入研究这四种锁的特点。但是在介绍这四种锁之前，让我们再来看下 MySQL 下锁的模式。

二、读锁 vs. 写锁
-----------

MySQL 将锁分成两类：锁类型（lock_type）和锁模式（lock_mode）。锁类型就是上文中介绍的表锁和行锁两种类型，当然行锁还可以细分成记录锁和间隙锁等更细的类型，锁类型描述的锁的粒度，也可以说是把锁具体加在什么地方；而锁模式描述的是到底加的是什么锁，譬如读锁或写锁。锁模式通常是和锁类型结合使用的，锁模式在 MySQL 的源码中定义如下：

```
/* Basic lock modes */
enum lock_mode {
    LOCK_IS = 0, /* intention shared */
    LOCK_IX,    /* intention exclusive */
    LOCK_S,     /* shared */
    LOCK_X,     /* exclusive */
    LOCK_AUTO_INC,  /* locks the auto-inc counter of a table in an exclusive mode*/
    ...
};
```

*   LOCK_IS：读意向锁；
*   LOCK_IX：写意向锁；
*   LOCK_S：读锁；
*   LOCK_X：写锁；
*   LOCK_AUTO_INC：自增锁；

将锁分为读锁和写锁主要是为了提高读的并发，如果不区分读写锁，那么数据库将没办法并发读，并发性将大大降低。而 IS（读意向）、IX（写意向）只会应用在表锁上，方便表锁和行锁之间的冲突检测。LOCK_AUTO_INC 是一种特殊的表锁。下面依次进行介绍。

### 2.1 读写锁

读锁和写锁都是最基本的锁模式，它们的概念也比较容易理解。读锁，又称共享锁（Share locks，简称 S 锁），加了读锁的记录，所有的事务都可以读取，但是不能修改，并且可同时有多个事务对记录加读锁。写锁，又称排他锁（Exclusive locks，简称 X 锁），或独占锁，对记录加了排他锁之后，只有拥有该锁的事务可以读取和修改，其他事务都不可以读取和修改，并且同一时间只能有一个事务加写锁。（注意：这里说的读都是当前读，快照读是无需加锁的，记录上无论有没有锁，都可以快照读）

在其他的数据库系统中（譬如 MSSQL），我们可能还会看到一种基本的锁模式：更新锁（Update locks，简称 U 锁），MySQL 暂时不支持 U 锁，所以这里只是稍微了解一下。这个锁主要是用来防止死锁的，因为多数数据库在加 X 锁的时候是先获取 S 锁，获取成功之后再升级成 X 锁，如果有两个事务同时获取了 S 锁，然后又同时尝试升级 X 锁，就会发生死锁。增加 U 锁表示有事务对该行有更新意向，只允许一个事务拿到 U 锁，该事务在发生写后 U 锁变 X 锁，未写时看做 S 锁。（疑问：MySQL 更新的时候是直接申请 X 锁么？）

### 2.2 读写意向锁

表锁锁定了整张表，而行锁是锁定表中的某条记录，它们俩锁定的范围有交集，因此表锁和行锁之间是有冲突的。譬如某个表有 10000 条记录，其中有一条记录加了 X 锁，如果这个时候系统需要对该表加表锁，为了判断是否能加这个表锁，系统需要遍历表中的所有 10000 条记录，看看是不是某条记录被加锁，如果有锁，则不允许加表锁，显然这是很低效的一种方法，为了方便检测表锁和行锁的冲突，从而引入了意向锁。

意向锁为表级锁，也可分为读意向锁（IS 锁）和写意向锁（IX 锁）。当事务试图读或写某一条记录时，会先在表上加上意向锁，然后才在要操作的记录上加上读锁或写锁。这样判断表中是否有记录加锁就很简单了，只要看下表上是否有意向锁就行了。意向锁之间是不会产生冲突的，也不和 AUTO_INC 表锁冲突，它只会阻塞表级读锁或表级写锁，另外，意向锁也不会和行锁冲突，行锁只会和行锁冲突。

下面是各个表锁之间的兼容矩阵：

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2025/02/27/1431433403.png)

这个矩阵看上去有点眼花缭乱，其实很简单，因为是斜对称的，所以我们用一条斜线把表格分割成两个部分，只需要看左下角的一半即可。总结起来有下面几点：

*   意向锁之间互不冲突；
*   S 锁只和 S/IS 锁兼容，和其他锁都冲突；
*   X 锁和其他所有锁都冲突；
*   AI 锁只和意向锁兼容；

### 2.3 AUTO_INC 锁

AUTO_INC 锁又叫自增锁（一般简写成 AI 锁），它是一种特殊类型的表锁，当插入的表中有自增列（AUTO_INCREMENT）的时候可能会遇到。当插入表中有自增列时，数据库需要自动生成自增值，在生成之前，它会先为该表加 AUTO_INC 表锁，其他事务的插入操作阻塞，这样保证生成的自增值肯定是唯一的。AUTO_INC 锁具有如下特点：

*   AUTO_INC 锁互不兼容，也就是说同一张表同时只允许有一个自增锁；
*   自增锁不遵循二段锁协议，它并不是事务结束时释放，而是在 INSERT 语句执行结束时释放，这样可以提高并发插入的性能。
*   自增值一旦分配了就会 +1，如果事务回滚，自增值也不会减回去，所以自增值可能会出现中断的情况。

显然，AUTO_INC 表锁会导致并发插入的效率降低，为了提高插入的并发性，MySQL 从 5.1.22 版本开始，引入了一种可选的轻量级锁（mutex）机制来代替 AUTO_INC 锁，我们可以通过参数 `innodb_autoinc_lock_mode` 控制分配自增值时的并发策略。参数 `innodb_autoinc_lock_mode` 可以取下列值：

*   innodb_autoinc_lock_mode = 0 （traditional lock mode）
    
    *   使用传统的 AUTO_INC 表锁，并发性比较差。
*   innodb_autoinc_lock_mode = 1 （consecutive lock mode）
    
    *   MySQL 默认采用这种方式，是一种比较折中的方法。
    *   MySQL 将插入语句分成三类：`Simple inserts、Bulk inserts、Mixed-mode inserts`。通过分析 INSERT 语句可以明确知道插入数量的叫做 `Simple inserts`，譬如最经常使用的 INSERT INTO table VALUE(1,2) 或 INSERT INTO table VALUES(1,2), (3,4)；通过分析 INSERT 语句无法确定插入数量的叫做 `Bulk inserts`，譬如 INSERT INTO table SELECT 或 LOAD DATA 等；还有一种是不确定是否需要分配自增值的，譬如 INSERT INTO table VALUES(1,'a'), (NULL,'b'), (5, 'C'), (NULL, 'd') 或 INSERT ... ON DUPLICATE KEY UPDATE，这种叫做 `Mixed-mode inserts`。
    *   Bulk inserts 不能确定插入数使用表锁；Simple inserts 和 Mixed-mode inserts 使用轻量级锁 mutex，只锁住预分配自增值的过程，不锁整张表。Mixed-mode inserts 会直接分析语句，获得最坏情况下需要插入的数量，一次性分配足够的自增值，缺点是会分配过多，导致浪费和空洞。
    *   这种模式的好处是既平衡了并发性，又能保证同一条 INSERT 语句分配的自增值是连续的。
*   innodb_autoinc_lock_mode = 2 （interleaved lock mode）
    
    *   全部都用轻量级锁 mutex，并发性能最高，按顺序依次分配自增值，不会预分配。
    *   缺点是不能保证同一条 INSERT 语句内的自增值是连续的，这样在复制（replication）时，如果 binlog_format 为 statement-based（基于语句的复制）就会存在问题，因为是来一个分配一个，同一条 INSERT 语句内获得的自增值可能不连续，主从数据集会出现数据不一致。所以在做数据库同步时要特别注意这个配置。

可以参考 MySQL 的这篇文档 [AUTO_INCREMENT Handling in InnoDB](https://dev.mysql.com/doc/refman/5.7/en/innodb-auto-increment-handling.html) 了解自增锁，InnoDb 处理自增值的方式，以及在不同的复制模式下可能遇到的问题。

三、细说 MySQL 锁类型
--------------

前面在讲行锁时有提到，在 MySQL 的源码中定义了四种类型的行锁，我们这一节将学习这四种锁。在我刚接触数据库锁的概念时，我理解的行锁就是将锁锁在行上，这一行记录不能被其他人修改，这种理解其实很肤浅，因为行锁也有可能并不是锁在行上而是行与行之间的间隙上，事实上，我理解的这种锁是最简单的行锁模式：记录锁。

### 3.1 记录锁（Record Locks）

[**记录锁**](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking.html#innodb-record-locks) 是最简单的行锁，并没有什么好说的。譬如下面的 SQL 语句（id 为主键）：

```
mysql> UPDATE accounts SET level = 100 WHERE id = 5;
```

这条 SQL 语句就会在 id = 5 这条记录上加上记录锁，防止其他事务对 id = 5 这条记录进行修改或删除。记录锁永远都是加在索引上的，就算一个表没有建索引，数据库也会隐式的创建一个索引。如果 WHERE 条件中指定的列是个二级索引，那么记录锁不仅会加在这个二级索引上，还会加在这个二级索引所对应的聚簇索引上（参考上面的加锁流程一节）。

注意，如果 SQL 语句无法使用索引时会走主索引实现全表扫描，这个时候 MySQL 会给整张表的所有数据行加记录锁。如果一个 WHERE 条件无法通过索引快速过滤，存储引擎层面就会将所有记录加锁后返回，再由 MySQL Server 层进行过滤。不过在实际使用过程中，MySQL 做了一些改进，在 MySQL Server 层进行过滤的时候，如果发现不满足，会调用 unlock_row 方法，把不满足条件的记录释放锁（显然这违背了二段锁协议）。这样做，保证了最后只会持有满足条件记录上的锁，但是每条记录的加锁操作还是不能省略的。可见在没有索引时，不仅会消耗大量的锁资源，增加数据库的开销，而且极大的降低了数据库的并发性能，所以说，更新操作一定要记得走索引。

### 3.2 间隙锁（Gap Locks）

还是看上面的那个例子，如果 id = 5 这条记录不存在，这个 SQL 语句还会加锁吗？答案是可能有，这取决于数据库的隔离级别。

还记得我们在上一篇博客中介绍的数据库并发过程中可能存在的问题吗？其中有一个问题叫做 **幻读**，指的是在同一个事务中同一条 SQL 语句连续两次读取出来的结果集不一样。在 read committed 隔离级别很明显存在幻读问题，在 repeatable read 级别下，标准的 SQL 规范中也是存在幻读问题的，但是在 MySQL 的实现中，使用了间隙锁的技术避免了幻读。

[**间隙锁**](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking.html#innodb-gap-locks) 是一种加在两个索引之间的锁，或者加在第一个索引之前，或最后一个索引之后的间隙。有时候又称为范围锁（Range Locks），这个范围可以跨一个索引记录，多个索引记录，甚至是空的。使用间隙锁可以防止其他事务在这个范围内插入或修改记录，保证两次读取这个范围内的记录不会变，从而不会出现幻读现象。很显然，间隙锁会增加数据库的开销，虽然解决了幻读问题，但是数据库的并发性一样受到了影响，所以在选择数据库的隔离级别时，要注意权衡性能和并发性，根据实际情况考虑是否需要使用间隙锁，大多数情况下使用 read committed 隔离级别就足够了，对很多应用程序来说，幻读也不是什么大问题。

回到这个例子，这个 SQL 语句在 RC 隔离级别不会加任何锁，在 RR 隔离级别会在 id = 5 前后两个索引之间加上间隙锁。

值得注意的是，间隙锁和间隙锁之间是互不冲突的，间隙锁唯一的作用就是为了防止其他事务的插入，所以加间隙 S 锁和加间隙 X 锁没有任何区别。

### 3.3 Next-Key Locks

[**Next-key 锁**](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking.html#innodb-next-key-locks) 是记录锁和间隙锁的组合，它指的是加在某条记录以及这条记录前面间隙上的锁。假设一个索引包含  
10、11、13 和 20 这几个值，可能的 Next-key 锁如下：

*   (-∞, 10]
*   (10, 11]
*   (11, 13]
*   (13, 20]
*   (20, +∞)

通常我们都用这种左开右闭区间来表示 Next-key 锁，其中，圆括号表示不包含该记录，方括号表示包含该记录。前面四个都是 Next-key 锁，最后一个为间隙锁。和间隙锁一样，在 RC 隔离级别下没有 Next-key 锁，只有 RR 隔离级别才有。继续拿上面的 SQL 例子来说，如果 id 不是主键，而是二级索引，且不是唯一索引，那么这个 SQL 在 RR 隔离级别下会加什么锁呢？答案就是 Next-key 锁，如下：

*   (a, 5]
*   (5, b)

其中，a 和 b 是 id = 5 前后两个索引，我们假设 a = 1、b = 10，那么此时如果插入一条 id = 3 的记录将会阻塞住。之所以要把 id = 5 前后的间隙都锁住，仍然是为了解决幻读问题，因为 id 是非唯一索引，所以 id = 5 可能会有多条记录，为了防止再插入一条 id = 5 的记录，必须将下面标记 ^ 的位置都锁住，因为这些位置都可能再插入一条 id = 5 的记录：

1 ^ 5 ^ 5 ^ 5 ^ 10 11 13 15

可以看出来，Next-key 锁确实可以避免幻读，但是带来的副作用是连插入 id = 3 这样的记录也被阻塞了，这根本就不会引起幻读问题的。

关于 Next-key 锁，有一个比较有意思的问题，比如下面这个 orders 表（id 为主键，order_id 为二级非唯一索引）：

```
+-----+----------+
|  id | order_id |
+-----+----------+
|   1 |        1 |
|   3 |        2 |
|   5 |        5 |
|   7 |        5 |
|  10 |        9 |
+-----+----------+
```

事务 A 执行下面的 SQL：

```
mysql> begin;
mysql> select * from orders where order_id = 5 for update;
+-----+----------+
|  id | order_id |
+-----+----------+
|   5 |        5 |
|   7 |        5 |
+-----+----------+
2 rows in set (0.00 sec)
```

这个时候不仅 order_id = 5 这条记录会加上 X 记录锁，而且这条记录前后的间隙也会加上锁，加锁位置如下：

1 2 ^ 5 ^ 5 ^ 9

可以看到 (2, 9) 这个区间都被锁住了，这个时候如果插入 order_id = 4 或者 order_id = 8 这样的记录肯定会被阻塞，这没什么问题，那么现在问题来了，如果插入一条记录 order_id = 2 或者 order_id = 9 会被阻塞吗？答案是可能阻塞，也可能不阻塞，这取决于插入记录主键的值，感兴趣的读者可以参考[这篇博客](http://blog.sina.com.cn/s/blog_a1e9c7910102vnrj.html)。

### 3.4 插入意向锁（Insert Intention Locks）

[**插入意向锁**](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking.html#innodb-insert-intention-locks) 是一种特殊的间隙锁（所以有的地方把它简写成 II GAP），这个锁表示插入的意向，只有在 INSERT 的时候才会有这个锁。注意，这个锁虽然也叫意向锁，但是和上面介绍的表级意向锁是两个完全不同的概念，不要搞混淆了。插入意向锁和插入意向锁之间互不冲突，所以可以在同一个间隙中有多个事务同时插入不同索引的记录。譬如在上面的例子中，id = 1 和 id = 5 之间如果有两个事务要同时分别插入 id = 2 和 id = 3 是没问题的，虽然两个事务都会在 id = 1 和 id = 5 之间加上插入意向锁，但是不会冲突。

插入意向锁只会和间隙锁或 Next-key 锁冲突，正如上面所说，间隙锁唯一的作用就是防止其他事务插入记录造成幻读，那么间隙锁是如何防止幻读的呢？正是由于在执行 INSERT 语句时需要加插入意向锁，而插入意向锁和间隙锁冲突，从而阻止了插入操作的执行。

### 3.5 行锁的兼容矩阵

下面我们对这四种行锁做一个总结，它们之间的兼容矩阵如下图所示：

![](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAADIAAAAyCAYAAAAeP4ixAAACbklEQVRoQ+2aMU4dMRCGZw6RC1CSSyQdLZJtKQ2REgoiRIpQkCYClCYpkgIESQFIpIlkW+IIcIC0gUNwiEFGz+hlmbG9b1nesvGW++zxfP7H4/H6IYzkwZFwQAUZmpJVkSeniFJKA8ASIi7MyfkrRPxjrT1JjZ8MLaXUDiJuzwngn2GJaNd7vyP5IoIYY94Q0fEQIKIPRGS8947zSQTRWh8CwLuBgZx479+2BTkHgBdDAgGAC+fcywoyIFWqInWN9BSONbTmFVp/AeA5o+rjKRJ2XwBYRsRXM4ZXgAg2LAPzOCDTJYQx5pSIVlrC3EI45y611osMTHuQUPUiYpiVooerg7TWRwDAlhSM0TuI+BsD0x4kGCuFSRVzSqkfiLiWmY17EALMbCAlMCmI6IwxZo+INgQYEYKBuW5da00PKikjhNNiiPGm01rrbwDwofGehQjjNcv1SZgddALhlJEgwgJFxDNr7acmjFLqCyJuTd6LEGFttpmkYC91Hrk3s1GZFERMmUT01Xv/sQljjPlMRMsxO6WULwnb2D8FEs4j680wScjO5f3vzrlNJszESWq2LYXJgTzjZm56MCHf3zVBxH1r7ftU1splxxKYHEgoUUpTo+grEf303rPH5hxENJqDKQEJtko2q9zGeeycWy3JhpKhWT8+NM/sufIhBwKI+Mta+7pkfxKMtd8Qtdbcx4dUQZcFCQ2I6DcAnLUpf6YMPxhIDDOuxC4C6djoQUE6+tKpewWZ1wlRkq0qUhXptKTlzv93aI3jWmE0Fz2TeujpX73F9TaKy9CeMk8vZusfBnqZ1g5GqyIdJq+XrqNR5AahKr9CCcxGSwAAAABJRU5ErkJggg==)

其中，**第一行表示已有的锁，第一列表示要加的锁**。这个矩阵看起来很复杂，因为它是不对称的，如果要死记硬背可能会晕掉。其实仔细看可以发现，不对称的只有插入意向锁，所以我们先对插入意向锁做个总结，如下：

*   插入意向锁不影响其他事务加其他任何锁。也就是说，一个事务已经获取了插入意向锁，对其他事务是没有任何影响的；
*   插入意向锁与间隙锁和 Next-key 锁冲突。也就是说，一个事务想要获取插入意向锁，如果有其他事务已经加了间隙锁或 Next-key 锁，则会阻塞。

了解插入意向锁的特点之后，我们将它从矩阵中移去，兼容矩阵就变成了下面这个样子：

![](https://raw.githubusercontent.com/bloatfan/PicGo/master/2025/02/27/3787975353.png)

这个看起来就非常简单了，可以得出下面的结论：

*   间隙锁不和其他锁（不包括插入意向锁）冲突；
*   记录锁和记录锁冲突，Next-key 锁和 Next-key 锁冲突，记录锁和 Next-key 锁冲突；

### 3.6 在 MySQL 中观察行锁

为了更好的理解不同的行锁，下面我们在 MySQL 中对不同的锁实际操作一把。有两种方式可以在 MySQL 中观察行锁，第一种是通过下面的 SQL 语句：

```
mysql> select * from information_schema.innodb_locks;
```

这个命令会打印出 InnoDb 的所有锁信息，包括锁 ID、事务 ID、以及每个锁的类型和模式等其他信息。第二种是使用下面的 SQL 语句：

```
mysql> show engine innodb status\G
```

这个命令并不是专门用来查看锁信息的，而是用于输出当前 InnoDb 引擎的状态信息，包括：BACKGROUND THREAD、SEMAPHORES、TRANSACTIONS、FILE I/O、INSERT BUFFER AND ADAPTIVE HASH INDEX、LOG、BUFFER POOL AND MEMORY、ROW OPERATIONS 等等。其中 TRANSACTIONS 部分会打印当前 MySQL 所有的事务，如果某个事务有加锁，还会显示加锁的详细信息。如果发生死锁，也可以通过这个命令来定位死锁发生的原因。不过在这之前需要先打开 Innodb 的锁监控：

```
mysql> set global innodb_status_output = ON;
mysql> set global innodb_status_output_locks = ON;
```

打开锁监控之后，使用 `show engine innodb status` 命令，会输出大量的信息，我们在其中可以找到 **TRANSACTIONS** 部分，这里面就包含了每个事务及相关锁的信息，如下所示：

```
------------
TRANSACTIONS
------------
Trx id counter 3125
Purge done for trx's n:o < 3106 undo n:o < 0 state: running but idle
History list length 17
LIST OF TRANSACTIONS FOR EACH SESSION:
---TRANSACTION 3124, ACTIVE 10 sec
4 lock struct(s), heap size 1136, 3 row lock(s)
MySQL thread id 19, OS thread handle 6384, query id 212 localhost ::1 root
TABLE LOCK table `accounts` trx id 3124 lock mode IX
RECORD LOCKS space id 53 page no 5 n bits 72 index createtime of table `accounts` trx id 3124 lock_mode X
Record lock, heap no 4 PHYSICAL RECORD: n_fields 2; compact format; info bits 0
 0: len 4; hex 5a119f98; asc Z   ;;
 1: len 4; hex 80000005; asc     ;;
 
RECORD LOCKS space id 53 page no 3 n bits 80 index PRIMARY of table `accounts` trx id 3124 lock_mode X locks rec but not gap
Record lock, heap no 4 PHYSICAL RECORD: n_fields 7; compact format; info bits 0
 0: len 4; hex 80000005; asc     ;;
 1: len 6; hex 000000000c1c; asc       ;;
 2: len 7; hex b70000012b0110; asc     +  ;;
 3: len 4; hex 80000005; asc     ;;
 4: len 4; hex 80000005; asc     ;;
 5: len 0; hex ; asc ;;
 6: len 4; hex 5a119f98; asc Z   ;;
 
RECORD LOCKS space id 53 page no 5 n bits 72 index createtime of table `accounts` trx id 3124 lock_mode X locks gap before rec
Record lock, heap no 5 PHYSICAL RECORD: n_fields 2; compact format; info bits 0
 0: len 4; hex 5a119fa1; asc Z   ;;
 1: len 4; hex 8000000a; asc     ;;
```

可以看出，`show engine innodb status` 的输出比较晦涩，要读懂它还需要学习一些其他知识，我们这里暂且不提，后面再专门对其进行介绍。这里使用 `information_schema.innodb_locks` 表来体验一下 MySQL 中不同的行锁。

要注意的是，只有在两个事务出现锁竞争时才能在这个表中看到锁信息，譬如你执行一条 UPDATE 语句，它会对某条记录加 X 锁，这个时候 `information_schema.innodb_locks` 表里是没有任何记录的。

另外，只看这个表只能得到当前持有锁的事务，至于是哪个事务被阻塞，可以通过 `information_schema.innodb_lock_waits` 表来查看。

#### 3.6.1 记录锁

根据上面的行锁兼容矩阵，记录锁和记录锁或 Next-key 锁冲突，所以想观察到记录锁，可以让两个事务都对同一条记录加记录锁，或者一个事务加记录锁另一个事务加 Next-key 锁。

事务 A 执行：

```
mysql> begin;
mysql> select * from accounts where id = 5 for update;
+----+----------+-------+
| id |     name | level |
+----+----------+-------+
|  5 | zhangsan |     7 |
+----+----------+-------+
1 row in set (0.00 sec)
```

事务 B 执行：

```
mysql> begin;
mysql> select * from accounts where id = 5 lock in share mode;
```

事务 B 阻塞，出现锁竞争，查看锁状态：

```
mysql> select * from information_schema.innodb_locks;
+-------------+-------------+-----------+-----------+------------+------------+------------+-----------+----------+-----------+
| lock_id     | lock_trx_id | lock_mode | lock_type | lock_table | lock_index | lock_space | lock_page | lock_rec | lock_data |
+-------------+-------------+-----------+-----------+------------+------------+------------+-----------+----------+-----------+
| 3108:53:3:4 | 3108        | S         | RECORD    | `accounts` | PRIMARY    |         53 |         3 |        4 | 5         |
| 3107:53:3:4 | 3107        | X         | RECORD    | `accounts` | PRIMARY    |         53 |         3 |        4 | 5         |
+-------------+-------------+-----------+-----------+------------+------------+------------+-----------+----------+-----------+
2 rows in set, 1 warning (0.00 sec)
```

#### 3.6.2 间隙锁

根据兼容矩阵，间隙锁只和插入意向锁冲突，而且是先加间隙锁，然后加插入意向锁时才会冲突。

事务 A 执行（id 为主键，且 id = 3 这条记录不存在）：

```
mysql> begin;
mysql> select * from accounts where id = 3 lock in share mode;
Empty set (0.00 sec)
```

事务 B 执行：

```
mysql> begin;
mysql> insert into accounts(id, name, level) value(3, 'lisi', 10);
```

事务 B 阻塞，出现锁竞争，查看锁状态：

```
mysql> select * from information_schema.innodb_locks;
+-------------+-------------+-----------+-----------+------------+------------+------------+-----------+----------+-----------+
| lock_id     | lock_trx_id | lock_mode | lock_type | lock_table | lock_index | lock_space | lock_page | lock_rec | lock_data |
+-------------+-------------+-----------+-----------+------------+------------+------------+-----------+----------+-----------+
| 3110:53:3:4 | 3110        | X,GAP     | RECORD    | `accounts` | PRIMARY    |         53 |         3 |        4 | 3         |
| 3109:53:3:4 | 3109        | S,GAP     | RECORD    | `accounts` | PRIMARY    |         53 |         3 |        4 | 3         |
+-------------+-------------+-----------+-----------+------------+------------+------------+-----------+----------+-----------+
2 rows in set, 1 warning (0.00 sec)
```

#### 3.6.3 Next-key 锁

根据兼容矩阵，Next-key 锁和记录锁、Next-key 锁或插入意向锁冲突，但是貌似很难制造 Next-key 锁和记录锁冲突的场景，也很难制造 Next-key 锁和 Next-key 锁冲突的场景（如果你能找到这样的例子，还望不吝赐教）。所以还是用 Next-key 锁和插入意向锁冲突的例子，和上面间隙锁的例子几乎一样。

事务 A 执行（level 为二级索引）：

```
mysql> begin;
mysql> select * from accounts where level = 7 lock in share mode;
+----+----------+-------+
| id |     name | level |
+----+----------+-------+
|  5 | zhangsan |     7 |
|  9 |   liusan |     7 |
+----+----------+-------+
2 rows in set (0.00 sec)
```

事务 B 执行：

```
mysql> begin;
mysql> insert into accounts(name, level) value('lisi', 7);
```

事务 B 阻塞，出现锁竞争，查看锁状态：

```
mysql> select * from information_schema.innodb_locks;
+-------------+-------------+-----------+-----------+------------+------------+------------+-----------+----------+----------------+
| lock_id     | lock_trx_id | lock_mode | lock_type | lock_table | lock_index | lock_space | lock_page | lock_rec | lock_data      |
+-------------+-------------+-----------+-----------+------------+------------+------------+-----------+----------+----------------+
| 3114:53:5:5 | 3114        | X,GAP     | RECORD    | `accounts` |      level |         53 |         5 |        5 | 0x5A119FA1, 10 |
| 3113:53:5:5 | 3113        | S,GAP     | RECORD    | `accounts` |      level |         53 |         5 |        5 | 0x5A119FA1, 10 |
+-------------+-------------+-----------+-----------+------------+------------+------------+-----------+----------+----------------+
2 rows in set, 1 warning (0.00 sec)
```

可以看到除了锁住的索引不同之外，Next-key 锁和间隙锁之间几乎看不出任何差异。

四、乐观锁 vs. 悲观锁
-------------

关于 MySQL 下的锁类型到这里就告一段落了。在结束这边博客之前，我认为还有必要介绍下乐观锁和悲观锁的概念，这两个概念听起来很像是一种特殊的锁，但实际上它并不是什么具体的锁，而是一种锁的思想。这种思想无论是在操作数据库时，还是在编程项目中，都非常实用。

我们知道，不同的隔离级别解决不同的并发问题，MySQL 能够根据设置的隔离级别自动管理事务内的锁，不需要开发人员关心就能避免那些并发问题的发生，譬如在 RC 级别下，开发人员不用担心会出现脏读问题，只要正常的写 SQL 语句就可以了。但对于当前隔离级别无法解决的并发问题，我们就需要自己来处理了，譬如在 MySQL 的 RR 级别下（不是标准的 RR 级别），你肯定会遇到丢失更新问题，对于这种问题，通常有两种解决思路。其实在讲 MVCC 的时候也提到过，解决并发问题的方式除了锁，还可以利用时间戳或者版本号等等手段。前一种处理数据的方式通常叫做 **悲观锁（Pessimistic Lock）**，第二种无锁方式叫做 **乐观锁（Optimistic Lock）**。

*   悲观锁，顾名思义就是很悲观，每次拿数据时都假设有别人会来修改，所以每次在拿数据的时候都会给数据加上锁，用这种方式来避免跟别人冲突，虽然很有效，但是可能会出现大量的锁冲突，导致性能低下。
*   乐观锁则是完全相反，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有改过这个数据，可以使用版本号等机制来判断。

我们在上一篇博客中丢失更新那一节举了一个商品库存的例子，我们分别用悲观锁和乐观锁来解决这个问题，以此来体会两个思想之间的区别以及优缺点。这个例子如下：

> 譬如商品表的库存字段，每次下单之后库存值需要减 1，大概的流程如下：
> 
> *   SELECT name, stock FROM product WHERE id = 100;
> *   判断 stock 值是否足够，如果足够，则下单：if (stock> n) process order;
> *   更新 stock 值，减去下单的商品数量：new_stock = stock - n;
> *   UPDATE product SET stock = new_stock WHERE id = 100;
> 
> 如果两个线程同时下单，很可能就会出现下面这样的情况：
> 
> *   线程 A 获取库存值为 10；
> *   线程 B 获取库存值为 10；
> *   线程 A 需要买 5 个商品，校验通过，并下单；
> *   线程 B 需要买 5 个商品，校验通过，并下单；
> *   线程 A 下单完成，更新库存值为 10 - 5 = 5；
> *   线程 B 下单完成，更新库存值为 10 - 5 = 5；

如果采用悲观锁的思想，我们在线程 A 获取商品库存的时候对该商品记录加 X 锁，并在最后下单完成并更新库存值之后再释放锁，这样线程 B 在获取库存值时就会等待，从而也不会出现线程 B 并发修改库存的情况了。

如果采用乐观锁的思想，我们不对记录加锁，于是线程 A 获取库存值为 10，线程 B 获取库存也为 10，然后线程 A 更新库存为 5 的时候使用类似于这样的 SQL 来校验当前库存值是否被修改过：`UPDATE product SET stock = new_stock WHERE id = 100 AND stock = 10;` 如果 UPDATE 成功则认为没有修改过，下单成功；同样线程 B 更新库存为 5 的时候也用同样的方式校验，很显然校验失败，这个时候我们可以重新查询最新的库存值并下单，或者直接抛出异常提示下单失败。这种带条件的更新就是乐观锁。但是要注意的是，这种带条件的更新还可能会遇到 [ABA 问题](https://en.wikipedia.org/wiki/ABA_problem)（关于 ABA 问题可以参考 [CAS](https://en.wikipedia.org/wiki/Compare-and-swap)），解决方法则是为每一条记录增加一个唯一的版本号字段，使用版本号字段来进行判断。再举一个很现实的例子，我们通常使用 svn 更新和提交代码，也是使用了乐观锁的思想，当用户提交代码时，会根据你提交的版本号和代码仓库中最新的版本号进行比较，如果一致则允许提交，如果不一致，则提示用户更新代码到最新版本。

总的来说，**悲观锁需要使用数据库的锁机制来实现，而乐观锁是通过程序的手段来实现**，这两种锁各有优缺点，不可认为一种好于另一种，像乐观锁适用于读多写少的情况下，即冲突真的很少发生，这样可以省去锁的开销，加大系统的吞吐量。但如果经常产生冲突，上层应用不断的进行重试，这样反倒是降低了性能，所以这种情况下用悲观锁更合适。虽然使用带版本检查的乐观锁能够同时保持高并发和高可伸缩性，但它也不是万能的，譬如它不能解决脏读问题，所以在实际应用中还是会和数据库的隔离级别一起使用。

不仅仅是数据库，其实在分布式系统中，我们也可以看到悲观锁和乐观锁的影子。譬如酷壳上的这篇文章[《多版本并发控制 (MVCC) 在分布式系统中的应用》](https://coolshell.cn/articles/6790.html)中提到的案例，就是一个典型的提交覆盖问题，可以通过悲观锁或者乐观锁来解决。

参考
--

1.  [MySQL 5.7 Reference Manual - LOCK TABLES and UNLOCK TABLES Syntax](https://dev.mysql.com/doc/refman/5.7/en/lock-tables.html)
2.  [MySQL 5.7 Reference Manual - Internal Locking Methods](https://dev.mysql.com/doc/refman/5.7/en/internal-locking.html)
3.  [MySQL 5.7 Reference Manual - InnoDB Locking](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking.html)
4.  [MySQL 5.7 Reference Manual - AUTO_INCREMENT Handling in InnoDB](https://dev.mysql.com/doc/refman/5.7/en/innodb-auto-increment-handling.html)
5.  [MySQL 5.7 Reference Manual - Clustered and Secondary Indexes](https://dev.mysql.com/doc/refman/5.7/en/innodb-index-types.html)
6.  [克鲁斯卡尔的博客 - MySQL 总结](http://novoland.github.io/%E6%95%B0%E6%8D%AE%E5%BA%93/2014/07/26/MySQL%E6%80%BB%E7%BB%93.html)
7.  [克鲁斯卡尔的博客 - InnoDB 锁](http://novoland.github.io/%E6%95%B0%E6%8D%AE%E5%BA%93/2015/08/17/InnoDB%20%E9%94%81.html)
8.  [MySQL InnoDB 锁机制之 Gap Lock、Next-Key Lock、Record Lock 解析](http://blog.sina.com.cn/s/blog_a1e9c7910102vnrj.html)
9.  [MySQL 加锁分析](http://www.fanyilun.me/2017/04/20/MySQL%E5%8A%A0%E9%94%81%E5%88%86%E6%9E%90/)
10.  [mysql、innodb 和加锁分析](https://liuzhengyang.github.io/2016/09/25/mysqlinnodb/)
11.  [MySQL 中的锁（表锁、行锁） 并发控制锁](https://github.com/MrLining/mysql/wiki/MySQL%E4%B8%AD%E7%9A%84%E9%94%81%EF%BC%88%E8%A1%A8%E9%94%81%E3%80%81%E8%A1%8C%E9%94%81%EF%BC%89-%E5%B9%B6%E5%8F%91%E6%8E%A7%E5%88%B6%E9%94%81)
12.  [《深入浅出 MySQL：数据库开发、优化与管理维护》](https://book.douban.com/subject/25817684/)
13.  [MySQL 乐观锁与悲观锁](http://www.jianshu.com/p/f5ff017db62a)

![](data:image/png;base64,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)

扫描二维码，在手机上阅读！