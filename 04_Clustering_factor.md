# Clustering Factor

## Index

`create table test as select * from dba_objects;`



rowid就是某一行的**物理地址**

rownum，top

对object_id建立索引，就是存储object_id+rowid

Oracle表默认是堆表，不排序，随机的

IOT是默认升序排序

索引是排序，object_id， rowid也跟着排序了

rowid能定位到物理地址，数据文件号，块号

同一个块的rowid是递增的。物理的连续的块组成extent，一个extent里面的rowid是递增的。



`select object_id,rowid from test order by object_id;`

| OBJECT_ID | ROWID              |
| --------- | ------------------ |
| 134       | ADvSJbADDAAAAbLAAA |
| 143       | ADvSJbADDAAAAbLAAB |
| 144       | ADvSJbADDAAAAbLAAC |
| 428       | ADvSJbADDAAAAbLAAD |
| 520       | ADvSJbADDAAAAbLAAE |
| 532       | ADvSJbADDAAAAbLAAF |
| 535       | ADvSJbADDAAAAbLAAG |
| 536       | ADvSJbADDAAAAbLAAH |
| 537       | ADvSJbADDAAAAbLAAI |

前后相邻的两行来比较，比如上图，

143，144的rowid来比较，是不是同一个块，

144和428的rowid比

428和520的rowid比 

集群因子针对索引的概念。没有索引就没有集群因子。

```sql
select object_id,rowid,dbms_rowid.rowid_relative_fno(rowid) file#,
dbms_rowid.rowid_block_number(rowid) block#,
dbms_rowid.rowid_row_number(rowid) row#
from test;
```

| OBJECT_ID | ROWID              | FILE# | BLOCK# | ROW# |
| --------- | ------------------ | ----- | ------ | ---- |
| 134       | ADvSJbADDAAAAbLAAA | 195   | 1739   | 0    |
| 143       | ADvSJbADDAAAAbLAAB | 195   | 1739   | 1    |
| 144       | ADvSJbADDAAAAbLAAC | 195   | 1739   | 2    |
| 428       | ADvSJbADDAAAAbLAAD | 195   | 1739   | 3    |
| 520       | ADvSJbADDAAAAbLAAE | 195   | 1739   | 4    |
| 532       | ADvSJbADDAAAAbLAAF | 195   | 1739   | 5    |
| 535       | ADvSJbADDAAAAbLAAG | 195   | 1739   | 6    |
| 536       | ADvSJbADDAAAAbLAAH | 195   | 1739   | 7    |
| 537       | ADvSJbADDAAAAbLAAI | 195   | 1739   | 8    |
| 538       | ADvSJbADDAAAAbLAAJ | 195   | 1739   | 9    |
| 541       | ADvSJbADDAAAAbLAAK | 195   | 1739   | 10   |
| 1401      | ADvSJbADDAAAAbLAAL | 195   | 1739   | 11   |
| 1750      | ADvSJbADDAAAAbLAAM | 195   | 1739   | 12   |
| 1752      | ADvSJbADDAAAAbLAAN | 195   | 1739   | 13   |
| 1753      | ADvSJbADDAAAAbLAAO | 195   | 1739   | 14   |
| 1763      | ADvSJbADDAAAAbLAAP | 195   | 1739   | 15   |
| 1765      | ADvSJbADDAAAAbLAAQ | 195   | 1739   | 16   |
| 1767      | ADvSJbADDAAAAbLAAR | 195   | 1739   | 17   |
| 1769      | ADvSJbADDAAAAbLAAS | 195   | 1739   | 18   |
| 1771      | ADvSJbADDAAAAbLAAT | 195   | 1739   | 19   |
| 1773      | ADvSJbADDAAAAbLAAU | 195   | 1739   | 20   |
| 1775      | ADvSJbADDAAAAbLAAV | 195   | 1739   | 21   |
| 1777      | ADvSJbADDAAAAbLAAW | 195   | 1739   | 22   |
| 1779      | ADvSJbADDAAAAbLAAX | 195   | 1739   | 23   |
| 1781      | ADvSJbADDAAAAbLAAY | 195   | 1739   | 24   |
| 1783      | ADvSJbADDAAAAbLAAZ | 195   | 1739   | 25   |
| 1785      | ADvSJbADDAAAAbLAAa | 195   | 1739   | 26   |
| 1787      | ADvSJbADDAAAAbLAAb | 195   | 1739   | 27   |
| 1789      | ADvSJbADDAAAAbLAAc | 195   | 1739   | 28   |
| 1791      | ADvSJbADDAAAAbLAAd | 195   | 1739   | 29   |
| 1793      | ADvSJbADDAAAAbLAAe | 195   | 1739   | 30   |
| 1795      | ADvSJbADDAAAAbLAAf | 195   | 1739   | 31   |
| 1797      | ADvSJbADDAAAAbLAAg | 195   | 1739   | 32   |
| 1799      | ADvSJbADDAAAAbLAAh | 195   | 1739   | 33   |
| 1801      | ADvSJbADDAAAAbLAAi | 195   | 1739   | 34   |
| 1803      | ADvSJbADDAAAAbLAAj | 195   | 1739   | 35   |
| 1805      | ADvSJbADDAAAAbLAAk | 195   | 1739   | 36   |
| 1807      | ADvSJbADDAAAAbLAAl | 195   | 1739   | 37   |
| 1809      | ADvSJbADDAAAAbLAAm | 195   | 1739   | 38   |
| 1811      | ADvSJbADDAAAAbLAAn | 195   | 1739   | 39   |
| 1813      | ADvSJbADDAAAAbLAAo | 195   | 1739   | 40   |
| 1815      | ADvSJbADDAAAAbLAAp | 195   | 1739   | 41   |
| 1817      | ADvSJbADDAAAAbLAAq | 195   | 1739   | 42   |
| 1819      | ADvSJbADDAAAAbLAAr | 195   | 1739   | 43   |
| 1821      | ADvSJbADDAAAAbLAAs | 195   | 1739   | 44   |
| 1823      | ADvSJbADDAAAAbLAAt | 195   | 1739   | 45   |
| 1825      | ADvSJbADDAAAAbLAAu | 195   | 1739   | 46   |
| 1827      | ADvSJbADDAAAAbLAAv | 195   | 1739   | 47   |
| 1829      | ADvSJbADDAAAAbLAAw | 195   | 1739   | 48   |
| 1831      | ADvSJbADDAAAAbLAAx | 195   | 1739   | 49   |

都在1个块，集群因子=0

集群因子，介于标的总行数和总块数之间。集群因子，越接近于总行数，说明列的值越分散，分布的块越多。越接近总块数，说明表的存储越集中。

`select * from test where object_id < 1000;`

走索引么？总行数7万多。<1000,最多返回1000行，低于5%，走索引

test表里面的object_id没有排序，索引的object_id排序了。

```
2 --- rowid
3 --- rowid
4 --- rowid
```

最理想的情况下2,3,4,5,6,...999,1000行都在同一个块
索引里面有rowid，通过rowid来访问表，（**回表**）
通过索引访问，要回表，999次，返回多少行数据，要回多少次，
如果都在同一个块，第一次回表是物理io，后面都是逻辑io。

最坏的情况，999行在999个块，999次物理IO。

集群因子用来干嘛的，用来衡量索引回表要多少物理io的。



