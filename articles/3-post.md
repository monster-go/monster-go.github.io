查询表级锁争用情况
=================

可以通过检查table_locks_waited和table_locks_immediate状态变量来分析系统上的表锁定争夺：
如果Table_locks_waited的值比较高，则说明存在着较严重的表级锁争用情况。
```mysql
mysql> show status like 'table%';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| Table_locks_immediate | 2979  |
| Table_locks_waited    | 0     |
+-----------------------+-------+
2 rows in set (0.00 sec))
```

MySQL表级锁的锁模式
==================
![MyISAM锁模式](/assets/images/table-lock-mode.jpg)

并发插入（Concurrent Inserts)
=============================

上文提到过MyISAM表的读和写是串行的，但这是就总体而言的。在一定条件下，MyISAM表也支持查询和插入操作的并发进行，也就是这种情况：S1获得 read local锁,S2能在表的尾部插入数据， 当然这得涉及一个变量。

###MyISAM存储引擎有一个系统变量concurrent_insert，专门用以控制其并发插入的行为，其值分别可以为0、1或2。

* 当concurrent_insert设置为0时，不允许并发插入。
* 当concurrent_insert设置为1时，如果MyISAM表中没有空洞（即表的中间没有被删除的行），MyISAM允许在一个进程读表的同时，另一个进程从表尾插入记录。这也是MySQL的默认设置。
* 当concurrent_insert设置为2时，无论MyISAM表中有没有空洞，都允许在表尾并发插入记录。

MyISAM的锁调度
=============

MyISAM存储引擎的读锁和写锁是互斥的，读写操作是串行的。那么，一个进程请求某个MyISAM表的读锁，同时另一个进程也请求同一表的写锁，MySQL如何处理呢？答案是写进程先获得锁。不仅如此，即使读请求先到锁等待队列，写请求后到，写锁也会插到读锁请求之前！这是因为MySQL认为写请求一般比读请求要重要。这也正是MyISAM表不太适合于有大量更新操作和查询操作应用的原因，因为，大量的更新操作会造成查询操作很难获得读锁，从而可能永远阻塞。这种情况有时可能会变得非常糟糕！幸好我们可以通过一些设置来调节MyISAM的调度行为。

   * 通过指定启动参数low-priority-updates，使MyISAM引擎默认给予读请求以优先的权利。
   * 通过执行命令SET LOW_PRIORITY_UPDATES=1，使该连接发出的更新请求优先级降低。
   * 通过指定INSERT、UPDATE、DELETE语句的LOW_PRIORITY属性，降低该语句的优先级。

