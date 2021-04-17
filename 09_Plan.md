# 执行计划

## 执行计划是什么

如何执行一条sql，多个路径。选一个

## 执行计划怎么看

sqlplus 

pl/sql developer F5

### autotrace

```
SQL> set autot
Usage: SET AUTOT[RACE] {OFF | ON | TRACE[ONLY]} [EXP[LAIN]] [STAT[ISTICS]]
set autot on ----执行SQL，并显示执行计划和统计信息 
set autot trace  ----执行SQL，但不显示执行记过，显示执行计划和统计信息
Set autot trace exp ---如果是select，就不执行SQL(DML执行)，只显示执行计划
set autot trace only --执行sql，只显示统计信息。
```

```
SQL> set autot trace
SQL> select count(*) from test;      


Execution Plan
----------------------------------------------------------
Plan hash value: 1950795681

-------------------------------------------------------------------
| Id  | Operation          | Name | Rows  | Cost (%CPU)| Time     |
-------------------------------------------------------------------
|   0 | SELECT STATEMENT   |      |     1 |   395   (1)| 00:00:01 |
|   1 |  SORT AGGREGATE    |      |     1 |            |          |
|   2 |   TABLE ACCESS FULL| TEST | 72632 |   395   (1)| 00:00:01 |
-------------------------------------------------------------------


Statistics
----------------------------------------------------------
          2  recursive calls
          5  db block gets
       1432  consistent gets
       1416  physical reads
          0  redo size
        544  bytes sent via SQL*Net to client
        608  bytes received via SQL*Net from client
          2  SQL*Net roundtrips to/from client
          0  sorts (memory)
          0  sorts (disk)
          1  rows processed
          
          
SQL> 734: unknown command beginning "
SQL> /               


Execution Plan
----------------------------------------------------------
Plan hash value: 1950795681

-------------------------------------------------------------------
| Id  | Operation          | Name | Rows  | Cost (%CPU)| Time     |
-------------------------------------------------------------------
|   0 | SELECT STATEMENT   |      |     1 |   395   (1)| 00:00:01 |
|   1 |  SORT AGGREGATE    |      |     1 |            |          |
|   2 |   TABLE ACCESS FULL| TEST | 72632 |   395   (1)| 00:00:01 |
-------------------------------------------------------------------


Statistics
----------------------------------------------------------
          0  recursive calls
          5  db block gets
       1431  consistent gets
          0  physical reads
          0  redo size
        544  bytes sent via SQL*Net to client
        608  bytes received via SQL*Net from client
          2  SQL*Net roundtrips to/from client
          0  sorts (memory)
          0  sorts (disk)
          1  rows processed
          
```



全表扫描究竟是如何扫描数据的？

按区扫描的。

Oracle最底层的存储单位是块。物理上连续的块组成了区Extent，Extent组成了Segments，区与区之间不一定联系。

全表扫描一个表，其实就是扫描表的所有区。全表扫描，一次性可以读多个块。

全表扫描只会读高水位以下的块



|                                                              |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| 2  recursive calls                                           | 递归调用次数<br />迎接写的时候 recursive calls > 0<br />第二次跑sql，软解析，recursive_call=0<br />如果sql里面有自定义函数，那么recursive_calls永远不为0<br />第一个执行。跑一些内部查询语句，就会有递归调用。<br />一班优化不参加。 |
| 5  db block gets                                             | 多少个块被修改了。DML语句才能修改块。<br/>一般情况下，Select语句，db block gets 为0<br/>延迟快清除的情况下，Select语句的db block gets=0<br/>ref:tom写的编程艺术。 |
| 1432  consistent gets                                        | **逻辑读**<br />一个块在buffer cache里面被访问一次，逻辑读就加1<br />一个表有100个块，逻辑读是否可能变成1000,<br />可能的。<br />一个表有100个块，所有块都给修改过N次，这些块都在undo中。<br />去读这个表的时候，要从undo读取，还要构造之前的环境。会有大量的逻辑读。<br />很高并发的dml，select查询一个sql，<br />逻辑读惠子在不繁忙的情况高几倍 |
| 1416  physical reads                                         | 物理读，把磁盘的块写入buffer cache。<br />逻辑读包含物理读<br />第二次访问物理读为0 |
| 0 redo size                                                  | db blocks get=0<br />那么redo size=0<br />                   |
| 544  bytes sent via SQL*Net to client<br/>        608  bytes received via SQL*Net from client | 网络通信<br />尽量减少列                                     |
| 0  sorts (memory)<br/>          0  sorts (disk)              | 没用                                                         |
| 1  rows processed                                            | 最有用<br />                                                 |
|                                                              |                                                              |

最重要

```
1、rows processed
1 rows processed可以优化<br />10000000 rows processed，不能优化
2、consistent gets 逻辑读减少了，物理读也减少

```




