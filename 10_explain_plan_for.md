### Explain Plan for

explain plan for

```
SQL> explain plan for select ename,deptno from emp where deptno in (select deptno from dept where dname='CHICAGO');

Explained.

SQL> select * from table(dbms_xplan.display);

PLAN_TABLE_OUTPUT
--------------------------------------------------------------------------------
Plan hash value: 615168685

---------------------------------------------------------------------------
| Id  | Operation          | Name | Rows  | Bytes | Cost (%CPU)| Time     |
---------------------------------------------------------------------------
|   0 | SELECT STATEMENT   |      |     1 |    42 |     6   (0)| 00:00:01 |
|*  1 |  HASH JOIN         |      |     1 |    42 |     6   (0)| 00:00:01 |
|*  2 |   TABLE ACCESS FULL| DEPT |     1 |    22 |     3   (0)| 00:00:01 |
|   3 |   TABLE ACCESS FULL| EMP  |    14 |   280 |     3   (0)| 00:00:01 |
---------------------------------------------------------------------------


PLAN_TABLE_OUTPUT
--------------------------------------------------------------------------------
Predicate Information (identified by operation id):
---------------------------------------------------

   1 - access("DEPTNO"="DEPTNO")
   2 - filter("DNAME"='CHICAGO')

Note
-----
   - dynamic statistics used: dynamic sampling (level=2)

20 rows selected.

SQL> 
```

explain plan for 不执行sql语句，只是把执行计划写入plan_table

plan_table是临时表，只对当前会话有用

explain plan for没有统计信息

#### 查看执行计划的高级方法。

用hint改进执行计划，就有这种方法。

```
SQL> select * from table(dbms_xplan.display(format=>'ADVANCED -PROJECTION'));

PLAN_TABLE_OUTPUT
--------------------------------------------------------------------------------
Plan hash value: 615168685

---------------------------------------------------------------------------
| Id  | Operation          | Name | Rows  | Bytes | Cost (%CPU)| Time     |
---------------------------------------------------------------------------
|   0 | SELECT STATEMENT   |      |     1 |    42 |     6   (0)| 00:00:01 |
|*  1 |  HASH JOIN         |      |     1 |    42 |     6   (0)| 00:00:01 |
|*  2 |   TABLE ACCESS FULL| DEPT |     1 |    22 |     3   (0)| 00:00:01 |
|   3 |   TABLE ACCESS FULL| EMP  |    14 |   280 |     3   (0)| 00:00:01 |
---------------------------------------------------------------------------


PLAN_TABLE_OUTPUT
--------------------------------------------------------------------------------
Query Block Name / Object Alias (identified by operation id):
-------------------------------------------------------------

   1 - SEL$5DA710D3
   2 - SEL$5DA710D3 / DEPT@SEL$2
   3 - SEL$5DA710D3 / EMP@SEL$1

Outline Data
-------------

  /*+

PLAN_TABLE_OUTPUT
--------------------------------------------------------------------------------
      BEGIN_OUTLINE_DATA
      USE_HASH(@"SEL$5DA710D3" "EMP"@"SEL$1")
      LEADING(@"SEL$5DA710D3" "DEPT"@"SEL$2" "EMP"@"SEL$1")
      FULL(@"SEL$5DA710D3" "EMP"@"SEL$1")
      FULL(@"SEL$5DA710D3" "DEPT"@"SEL$2")
      OUTLINE(@"SEL$2")
      OUTLINE(@"SEL$1")
      UNNEST(@"SEL$2")
      OUTLINE_LEAF(@"SEL$5DA710D3")
      ALL_ROWS
      DB_VERSION('12.2.0.1')

PLAN_TABLE_OUTPUT
--------------------------------------------------------------------------------
      OPTIMIZER_FEATURES_ENABLE('12.2.0.1')
      IGNORE_OPTIM_EMBEDDED_HINTS
      END_OUTLINE_DATA
  */

Predicate Information (identified by operation id):
---------------------------------------------------

   1 - access("DEPTNO"="DEPTNO")
   2 - filter("DNAME"='CHICAGO')


PLAN_TABLE_OUTPUT
--------------------------------------------------------------------------------
Note
-----
   - dynamic statistics used: dynamic sampling (level=2)

47 rows selected.

SQL> 
```



Yet Another xxxx

