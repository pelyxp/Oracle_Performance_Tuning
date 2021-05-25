```

----------------
Btree Index 原理
----------------

1.Oracle中的Btree Index具有3大结构，root节点，branch节点，leaf节点.Root节点始终紧跟索引段头.
  当索引比较小的时候，root节点，branch节点，leaf节点都存储在同一个block中.Branch节点主要存储
  了索引的键值,但是这个键值并不是完整的，它只是完整索引值的部分前缀.同时Branch节点还存储了指向
  leaf节点的指针(DBA)，另外有个主意的是branch节点中还有个叫kdxbrlmc的指针.Leaf节点主要存储了完
  整的索引键值，以及相关索引键值的部分rowid(这个rowid去掉了data object number部分)，同时leaf 
  节点还存储了2个指针(DBA)，他们分别指向上一个leaf节点以及下一个leaf节点.
 
2.Btree Index 是始终平衡的，也就是说 从Root节点到 Leaf 节点的任何一个路径都是等距离的.

3.Btree Index 默认是按照索引值升序排列的，当然了我们可以在创建/重建的时候设置它降序排列.

4.Index Scan 的时候，采用的是 sequential read,并且一次只能读一个block(INDEX FAST FULL SCAN 除外). 

5.Btree Index Update的 时候，先做的是 delete,然后进行insert.

6.Btree Index 不存储 Null值，但是如果组合索引其中一列是非Null的，那么组合索引也会存储Null值.

---------------------------------
Btree Index 存储原理
---------------------------------
SQL> select * from v$version where rownum=1;

BANNER
----------------------------------------------------------------
Oracle Database 10g Enterprise Edition Release 10.2.0.1.0 - Prod

SQL> show user
User is "robinson"

SQL> create table test as select * from dba_objects;

Table created

SQL> select count(*) from test;

  COUNT(*)
----------
     50352

SQL> insert into test select * from dba_objects;

50352 rows inserted

SQL> commit;

Commit complete

SQL> select count(*) from test;

  COUNT(*)
----------
    100704

SQL> create index i_test_object_name on test(object_name);

Index created

------------------------------------------------------------------------------
我们现在来看一下索引的Blevel,什么是Blevel,Blevel可以通过DBA_INDEXES.Blevel获得
------------------------------------------------------------------------------

B*-Tree level: depth of the index from its root block to its leaf blocks. A depth of 0 indicates that the root block and leaf block are the same.
               从 root block 到 leaf block的深度. 如果root block 和 leaf block在同一个块中 那么 Blevel=0

SQL> select index_name,blevel from dba_indexes where index_name='I_TEST_OBJECT_NAME';

INDEX_NAME                         BLEVEL
------------------------------ ----------
I_TEST_OBJECT_NAME                      2

-----------------------------------------------------------------------------------
我们现在来看一下索引的height，索引的高度等于Blevel+1，height可以通过INDEX_STATS获得
-----------------------------------------------------------------------------------

SQL> analyze index I_TEST_OBJECT_NAME validate structure;

Index analyzed

SQL> select name,height from index_stats where name='I_TEST_OBJECT_NAME'; ---查询index_stats之前需要 analyze ... validate structure，否则无数据

NAME                               HEIGHT
------------------------------ ----------
I_TEST_OBJECT_NAME                      3

--------------------------------------------------------------------------------------------
Oracle 提供了分析 Btree index 结构的命令 treedump，在进行treedump之前需要获得索引的object_id
--------------------------------------------------------------------------------------------

SQL> select object_id from dba_objects where object_name='I_TEST_OBJECT_NAME' and owner='ROBINSON';

 OBJECT_ID
----------
     55178
     
SQL> ALTER SESSION SET EVENTS 'immediate trace name treedump level 55178';

Session altered 

------------------------
treedump trace 文件
------------------------

----- begin tree dump
branch: 0x1800014 25165844 (0: nrow: 2, level: 2)
   branch: 0x1800710 25167632 (-1: nrow: 247, level: 1)
      leaf: 0x1800015 25165845 (-1: nrow: 182 rrow: 182)
      leaf: 0x1800016 25165846 (0: nrow: 182 rrow: 182)
      leaf: 0x1800017 25165847 (1: nrow: 186 rrow: 186)
      leaf: 0x1800018 25165848 (2: nrow: 189 rrow: 189)
      leaf: 0x1800019 25165849 (3: nrow: 186 rrow: 186)
      leaf: 0x180001a 25165850 (4: nrow: 190 rrow: 190)
      leaf: 0x180001b 25165851 (5: nrow: 185 rrow: 185)
      leaf: 0x180001c 25165852 (6: nrow: 179 rrow: 179)
      leaf: 0x180001d 25165853 (7: nrow: 187 rrow: 187)
      leaf: 0x180001e 25165854 (8: nrow: 181 rrow: 181)
      leaf: 0x180001f 25165855 (9: nrow: 186 rrow: 186)
      leaf: 0x1800020 25165856 (10: nrow: 189 rrow: 189)
      leaf: 0x1800022 25165858 (11: nrow: 187 rrow: 187)
      leaf: 0x1800023 25165859 (12: nrow: 180 rrow: 180)
      leaf: 0x1800024 25165860 (13: nrow: 182 rrow: 182)
      leaf: 0x1800025 25165861 (14: nrow: 186 rrow: 186)
      leaf: 0x1800026 25165862 (15: nrow: 188 rrow: 188)
      leaf: 0x1800027 25165863 (16: nrow: 187 rrow: 187)
      leaf: 0x1800028 25165864 (17: nrow: 185 rrow: 185)
      leaf: 0x1800029 25165865 (18: nrow: 187 rrow: 187)
      leaf: 0x180002a 25165866 (19: nrow: 185 rrow: 185)
      leaf: 0x180002b 25165867 (20: nrow: 184 rrow: 184)
      leaf: 0x180002c 25165868 (21: nrow: 188 rrow: 188)
      leaf: 0x180002d 25165869 (22: nrow: 189 rrow: 189)
      leaf: 0x180002e 25165870 (23: nrow: 184 rrow: 184)
      leaf: 0x180002f 25165871 (24: nrow: 192 rrow: 192)
      leaf: 0x1800030 25165872 (25: nrow: 187 rrow: 187)
      leaf: 0x1800032 25165874 (26: nrow: 181 rrow: 181)
      leaf: 0x1800033 25165875 (27: nrow: 182 rrow: 182)
      leaf: 0x1800034 25165876 (28: nrow: 192 rrow: 192)
      leaf: 0x1800035 25165877 (29: nrow: 186 rrow: 186)
      leaf: 0x1800036 25165878 (30: nrow: 189 rrow: 189)
      leaf: 0x1800037 25165879 (31: nrow: 181 rrow: 181)
      leaf: 0x1800038 25165880 (32: nrow: 184 rrow: 184)
      leaf: 0x1800039 25165881 (33: nrow: 181 rrow: 181)
      leaf: 0x180003a 25165882 (34: nrow: 189 rrow: 189)
      leaf: 0x180003b 25165883 (35: nrow: 184 rrow: 184)
      leaf: 0x180003c 25165884 (36: nrow: 184 rrow: 184)
      leaf: 0x180003d 25165885 (37: nrow: 183 rrow: 183)
      leaf: 0x180003e 25165886 (38: nrow: 185 rrow: 185)
      leaf: 0x180003f 25165887 (39: nrow: 185 rrow: 185)
      leaf: 0x1800040 25165888 (40: nrow: 189 rrow: 189)
      leaf: 0x1800042 25165890 (41: nrow: 187 rrow: 187)
      leaf: 0x1800043 25165891 (42: nrow: 187 rrow: 187)
      leaf: 0x1800044 25165892 (43: nrow: 186 rrow: 186)
      leaf: 0x1800045 25165893 (44: nrow: 186 rrow: 186)
      leaf: 0x1800046 25165894 (45: nrow: 183 rrow: 183)
      leaf: 0x1800047 25165895 (46: nrow: 188 rrow: 188)
      leaf: 0x1800048 25165896 (47: nrow: 185 rrow: 185)
      leaf: 0x1800049 25165897 (48: nrow: 187 rrow: 187)
      leaf: 0x180004a 25165898 (49: nrow: 186 rrow: 186)
      leaf: 0x180004b 25165899 (50: nrow: 188 rrow: 188)
      leaf: 0x180004c 25165900 (51: nrow: 180 rrow: 180)
      leaf: 0x180004d 25165901 (52: nrow: 185 rrow: 185)
      leaf: 0x180004e 25165902 (53: nrow: 186 rrow: 186)
      leaf: 0x180004f 25165903 (54: nrow: 186 rrow: 186)
      leaf: 0x1800050 25165904 (55: nrow: 184 rrow: 184)
      leaf: 0x1800052 25165906 (56: nrow: 188 rrow: 188)
      leaf: 0x1800053 25165907 (57: nrow: 184 rrow: 184)
      leaf: 0x1800054 25165908 (58: nrow: 187 rrow: 187)
      leaf: 0x1800055 25165909 (59: nrow: 188 rrow: 188)
      leaf: 0x1800056 25165910 (60: nrow: 187 rrow: 187)
      leaf: 0x1800057 25165911 (61: nrow: 189 rrow: 189)
      leaf: 0x1800058 25165912 (62: nrow: 187 rrow: 187)
      leaf: 0x1800059 25165913 (63: nrow: 185 rrow: 185)
      leaf: 0x180005a 25165914 (64: nrow: 183 rrow: 183)
      leaf: 0x180005b 25165915 (65: nrow: 191 rrow: 191)
      leaf: 0x180005c 25165916 (66: nrow: 192 rrow: 192)
      leaf: 0x180005d 25165917 (67: nrow: 189 rrow: 189)
      leaf: 0x180005e 25165918 (68: nrow: 191 rrow: 191)
      leaf: 0x180005f 25165919 (69: nrow: 190 rrow: 190)
      leaf: 0x1800060 25165920 (70: nrow: 189 rrow: 189)
      leaf: 0x1800062 25165922 (71: nrow: 190 rrow: 190)
      leaf: 0x1800063 25165923 (72: nrow: 185 rrow: 185)
      leaf: 0x1800064 25165924 (73: nrow: 186 rrow: 186)
      leaf: 0x1800065 25165925 (74: nrow: 183 rrow: 183)
      leaf: 0x1800066 25165926 (75: nrow: 189 rrow: 189)
      leaf: 0x1800067 25165927 (76: nrow: 189 rrow: 189)
      leaf: 0x1800068 25165928 (77: nrow: 184 rrow: 184)
      leaf: 0x1800069 25165929 (78: nrow: 190 rrow: 190)
      leaf: 0x180006a 25165930 (79: nrow: 193 rrow: 193)
      leaf: 0x180006b 25165931 (80: nrow: 180 rrow: 180)
      leaf: 0x180006c 25165932 (81: nrow: 186 rrow: 186)
      leaf: 0x180006d 25165933 (82: nrow: 189 rrow: 189)
      leaf: 0x180006e 25165934 (83: nrow: 184 rrow: 184)
      leaf: 0x180006f 25165935 (84: nrow: 184 rrow: 184)
      leaf: 0x1800070 25165936 (85: nrow: 188 rrow: 188)
      leaf: 0x1800072 25165938 (86: nrow: 187 rrow: 187)
      leaf: 0x1800073 25165939 (87: nrow: 182 rrow: 182)
      leaf: 0x1800074 25165940 (88: nrow: 188 rrow: 188)
      leaf: 0x1800075 25165941 (89: nrow: 181 rrow: 181)
      leaf: 0x1800076 25165942 (90: nrow: 188 rrow: 188)
      leaf: 0x1800077 25165943 (91: nrow: 190 rrow: 190)
      leaf: 0x1800078 25165944 (92: nrow: 183 rrow: 183)
      leaf: 0x1800079 25165945 (93: nrow: 192 rrow: 192)
      leaf: 0x180007a 25165946 (94: nrow: 187 rrow: 187)
      leaf: 0x180007b 25165947 (95: nrow: 188 rrow: 188)
      leaf: 0x180007c 25165948 (96: nrow: 188 rrow: 188)
      leaf: 0x180007d 25165949 (97: nrow: 188 rrow: 188)
      leaf: 0x180007e 25165950 (98: nrow: 192 rrow: 192)
      leaf: 0x180007f 25165951 (99: nrow: 186 rrow: 186)
      leaf: 0x1800080 25165952 (100: nrow: 185 rrow: 185)
      leaf: 0x1800082 25165954 (101: nrow: 183 rrow: 183)
      leaf: 0x1800083 25165955 (102: nrow: 187 rrow: 187)
      leaf: 0x1800084 25165956 (103: nrow: 191 rrow: 191)
      leaf: 0x1800085 25165957 (104: nrow: 184 rrow: 184)
      leaf: 0x1800086 25165958 (105: nrow: 183 rrow: 183)
      leaf: 0x1800087 25165959 (106: nrow: 180 rrow: 180)
      leaf: 0x1800088 25165960 (107: nrow: 187 rrow: 187)
      leaf: 0x1800609 25167369 (108: nrow: 178 rrow: 178)
      leaf: 0x180060a 25167370 (109: nrow: 189 rrow: 189)
      leaf: 0x180060b 25167371 (110: nrow: 188 rrow: 188)
      leaf: 0x180060c 25167372 (111: nrow: 185 rrow: 185)
      leaf: 0x180060d 25167373 (112: nrow: 185 rrow: 185)
      leaf: 0x180060e 25167374 (113: nrow: 182 rrow: 182)
      leaf: 0x180060f 25167375 (114: nrow: 184 rrow: 184)
      leaf: 0x1800610 25167376 (115: nrow: 187 rrow: 187)
      leaf: 0x180068b 25167499 (116: nrow: 186 rrow: 186)
      leaf: 0x180068c 25167500 (117: nrow: 183 rrow: 183)
      leaf: 0x180068d 25167501 (118: nrow: 184 rrow: 184)
      leaf: 0x180068e 25167502 (119: nrow: 187 rrow: 187)
      leaf: 0x180068f 25167503 (120: nrow: 185 rrow: 185)
      leaf: 0x1800690 25167504 (121: nrow: 179 rrow: 179)
      leaf: 0x1800691 25167505 (122: nrow: 185 rrow: 185)
      leaf: 0x1800692 25167506 (123: nrow: 190 rrow: 190)
      leaf: 0x1800693 25167507 (124: nrow: 187 rrow: 187)
      leaf: 0x1800694 25167508 (125: nrow: 190 rrow: 190)
      leaf: 0x1800695 25167509 (126: nrow: 190 rrow: 190)
      leaf: 0x1800696 25167510 (127: nrow: 184 rrow: 184)
      leaf: 0x1800697 25167511 (128: nrow: 183 rrow: 183)
      leaf: 0x1800698 25167512 (129: nrow: 183 rrow: 183)
      leaf: 0x1800699 25167513 (130: nrow: 186 rrow: 186)
      leaf: 0x180069a 25167514 (131: nrow: 186 rrow: 186)
      leaf: 0x180069b 25167515 (132: nrow: 187 rrow: 187)
      leaf: 0x180069c 25167516 (133: nrow: 182 rrow: 182)
      leaf: 0x180069d 25167517 (134: nrow: 189 rrow: 189)
      leaf: 0x180069e 25167518 (135: nrow: 189 rrow: 189)
      leaf: 0x180069f 25167519 (136: nrow: 188 rrow: 188)
      leaf: 0x18006a0 25167520 (137: nrow: 187 rrow: 187)
      leaf: 0x18006a1 25167521 (138: nrow: 185 rrow: 185)
      leaf: 0x18006a2 25167522 (139: nrow: 189 rrow: 189)
      leaf: 0x18006a3 25167523 (140: nrow: 187 rrow: 187)
      leaf: 0x18006a4 25167524 (141: nrow: 191 rrow: 191)
      leaf: 0x18006a5 25167525 (142: nrow: 181 rrow: 181)
      leaf: 0x18006a6 25167526 (143: nrow: 194 rrow: 194)
      leaf: 0x18006a7 25167527 (144: nrow: 186 rrow: 186)
      leaf: 0x18006a8 25167528 (145: nrow: 192 rrow: 192)
      leaf: 0x18006a9 25167529 (146: nrow: 186 rrow: 186)
      leaf: 0x18006aa 25167530 (147: nrow: 188 rrow: 188)
      leaf: 0x18006ab 25167531 (148: nrow: 187 rrow: 187)
      leaf: 0x18006ac 25167532 (149: nrow: 187 rrow: 187)
      leaf: 0x18006ad 25167533 (150: nrow: 190 rrow: 190)
      leaf: 0x18006ae 25167534 (151: nrow: 187 rrow: 187)
      leaf: 0x18006af 25167535 (152: nrow: 185 rrow: 185)
      leaf: 0x18006b0 25167536 (153: nrow: 184 rrow: 184)
      leaf: 0x18006b1 25167537 (154: nrow: 196 rrow: 196)
      leaf: 0x18006b2 25167538 (155: nrow: 185 rrow: 185)
      leaf: 0x18006b3 25167539 (156: nrow: 189 rrow: 189)
      leaf: 0x18006b4 25167540 (157: nrow: 189 rrow: 189)
      leaf: 0x18006b5 25167541 (158: nrow: 186 rrow: 186)
      leaf: 0x18006b6 25167542 (159: nrow: 184 rrow: 184)
      leaf: 0x18006b7 25167543 (160: nrow: 190 rrow: 190)
      leaf: 0x18006b8 25167544 (161: nrow: 186 rrow: 186)
      leaf: 0x18006b9 25167545 (162: nrow: 188 rrow: 188)
      leaf: 0x18006ba 25167546 (163: nrow: 184 rrow: 184)
      leaf: 0x18006bb 25167547 (164: nrow: 184 rrow: 184)
      leaf: 0x18006bc 25167548 (165: nrow: 183 rrow: 183)
      leaf: 0x18006bd 25167549 (166: nrow: 189 rrow: 189)
      leaf: 0x18006be 25167550 (167: nrow: 189 rrow: 189)
      leaf: 0x18006bf 25167551 (168: nrow: 186 rrow: 186)
      leaf: 0x18006c0 25167552 (169: nrow: 187 rrow: 187)
      leaf: 0x18006c1 25167553 (170: nrow: 186 rrow: 186)
      leaf: 0x18006c2 25167554 (171: nrow: 187 rrow: 187)
      leaf: 0x18006c3 25167555 (172: nrow: 183 rrow: 183)
      leaf: 0x18006c4 25167556 (173: nrow: 184 rrow: 184)
      leaf: 0x18006c5 25167557 (174: nrow: 187 rrow: 187)
      leaf: 0x18006c6 25167558 (175: nrow: 187 rrow: 187)
      leaf: 0x18006c7 25167559 (176: nrow: 184 rrow: 184)
      leaf: 0x18006c8 25167560 (177: nrow: 188 rrow: 188)
      leaf: 0x18006c9 25167561 (178: nrow: 188 rrow: 188)
      leaf: 0x18006ca 25167562 (179: nrow: 188 rrow: 188)
      leaf: 0x18006cb 25167563 (180: nrow: 183 rrow: 183)
      leaf: 0x18006cc 25167564 (181: nrow: 181 rrow: 181)
      leaf: 0x18006cd 25167565 (182: nrow: 184 rrow: 184)
      leaf: 0x18006ce 25167566 (183: nrow: 178 rrow: 178)
      leaf: 0x18006cf 25167567 (184: nrow: 188 rrow: 188)
      leaf: 0x18006d0 25167568 (185: nrow: 189 rrow: 189)
      leaf: 0x18006d1 25167569 (186: nrow: 185 rrow: 185)
      leaf: 0x18006d2 25167570 (187: nrow: 187 rrow: 187)
      leaf: 0x18006d3 25167571 (188: nrow: 192 rrow: 192)
      leaf: 0x18006d4 25167572 (189: nrow: 188 rrow: 188)
      leaf: 0x18006d5 25167573 (190: nrow: 189 rrow: 189)
      leaf: 0x18006d6 25167574 (191: nrow: 190 rrow: 190)
      leaf: 0x18006d7 25167575 (192: nrow: 187 rrow: 187)
      leaf: 0x18006d8 25167576 (193: nrow: 178 rrow: 178)
      leaf: 0x18006d9 25167577 (194: nrow: 189 rrow: 189)
      leaf: 0x18006da 25167578 (195: nrow: 179 rrow: 179)
      leaf: 0x18006db 25167579 (196: nrow: 182 rrow: 182)
      leaf: 0x18006dc 25167580 (197: nrow: 186 rrow: 186)
      leaf: 0x18006dd 25167581 (198: nrow: 186 rrow: 186)
      leaf: 0x18006de 25167582 (199: nrow: 191 rrow: 191)
      leaf: 0x18006df 25167583 (200: nrow: 184 rrow: 184)
      leaf: 0x18006e0 25167584 (201: nrow: 184 rrow: 184)
      leaf: 0x18006e1 25167585 (202: nrow: 183 rrow: 183)
      leaf: 0x18006e2 25167586 (203: nrow: 188 rrow: 188)
      leaf: 0x18006e3 25167587 (204: nrow: 188 rrow: 188)
      leaf: 0x18006e4 25167588 (205: nrow: 189 rrow: 189)
      leaf: 0x18006e5 25167589 (206: nrow: 189 rrow: 189)
      leaf: 0x18006e6 25167590 (207: nrow: 193 rrow: 193)
      leaf: 0x18006e7 25167591 (208: nrow: 183 rrow: 183)
      leaf: 0x18006e8 25167592 (209: nrow: 185 rrow: 185)
      leaf: 0x18006e9 25167593 (210: nrow: 184 rrow: 184)
      leaf: 0x18006ea 25167594 (211: nrow: 186 rrow: 186)
      leaf: 0x18006eb 25167595 (212: nrow: 184 rrow: 184)
      leaf: 0x18006ec 25167596 (213: nrow: 187 rrow: 187)
      leaf: 0x18006ed 25167597 (214: nrow: 188 rrow: 188)
      leaf: 0x18006ee 25167598 (215: nrow: 186 rrow: 186)
      leaf: 0x18006ef 25167599 (216: nrow: 193 rrow: 193)
      leaf: 0x18006f0 25167600 (217: nrow: 185 rrow: 185)
      leaf: 0x18006f1 25167601 (218: nrow: 184 rrow: 184)
      leaf: 0x18006f2 25167602 (219: nrow: 189 rrow: 189)
      leaf: 0x18006f3 25167603 (220: nrow: 184 rrow: 184)
      leaf: 0x18006f4 25167604 (221: nrow: 186 rrow: 186)
      leaf: 0x18006f5 25167605 (222: nrow: 187 rrow: 187)
      leaf: 0x18006f6 25167606 (223: nrow: 188 rrow: 188)
      leaf: 0x18006f7 25167607 (224: nrow: 182 rrow: 182)
      leaf: 0x18006f8 25167608 (225: nrow: 183 rrow: 183)
      leaf: 0x18006f9 25167609 (226: nrow: 186 rrow: 186)
      leaf: 0x18006fa 25167610 (227: nrow: 181 rrow: 181)
      leaf: 0x18006fb 25167611 (228: nrow: 184 rrow: 184)
      leaf: 0x18006fc 25167612 (229: nrow: 185 rrow: 185)
      leaf: 0x18006fd 25167613 (230: nrow: 187 rrow: 187)
      leaf: 0x18006fe 25167614 (231: nrow: 183 rrow: 183)
      leaf: 0x18006ff 25167615 (232: nrow: 182 rrow: 182)
      leaf: 0x1800700 25167616 (233: nrow: 187 rrow: 187)
      leaf: 0x1800701 25167617 (234: nrow: 184 rrow: 184)
      leaf: 0x1800702 25167618 (235: nrow: 186 rrow: 186)
      leaf: 0x1800703 25167619 (236: nrow: 187 rrow: 187)
      leaf: 0x1800704 25167620 (237: nrow: 189 rrow: 189)
      leaf: 0x1800705 25167621 (238: nrow: 186 rrow: 186)
      leaf: 0x1800706 25167622 (239: nrow: 181 rrow: 181)
      leaf: 0x1800707 25167623 (240: nrow: 186 rrow: 186)
      leaf: 0x1800708 25167624 (241: nrow: 190 rrow: 190)
      leaf: 0x180070b 25167627 (242: nrow: 190 rrow: 190)
      leaf: 0x180070c 25167628 (243: nrow: 222 rrow: 222)
      leaf: 0x180070d 25167629 (244: nrow: 200 rrow: 200)
      leaf: 0x180070e 25167630 (245: nrow: 230 rrow: 230)
   branch: 0x180080c 25167884 (0: nrow: 248, level: 1)
      leaf: 0x180070f 25167631 (-1: nrow: 229 rrow: 229)
      leaf: 0x1800711 25167633 (0: nrow: 222 rrow: 222)
      leaf: 0x1800712 25167634 (1: nrow: 226 rrow: 226)
      leaf: 0x1800713 25167635 (2: nrow: 221 rrow: 221)
      leaf: 0x1800714 25167636 (3: nrow: 221 rrow: 221)
      leaf: 0x1800715 25167637 (4: nrow: 249 rrow: 249)
      leaf: 0x1800716 25167638 (5: nrow: 233 rrow: 233)
      leaf: 0x1800717 25167639 (6: nrow: 212 rrow: 212)
      leaf: 0x1800718 25167640 (7: nrow: 256 rrow: 256)
      leaf: 0x1800719 25167641 (8: nrow: 274 rrow: 274)
      leaf: 0x180071a 25167642 (9: nrow: 201 rrow: 201)
      leaf: 0x180071b 25167643 (10: nrow: 199 rrow: 199)
      leaf: 0x180071c 25167644 (11: nrow: 199 rrow: 199)
      leaf: 0x180071d 25167645 (12: nrow: 249 rrow: 249)
      leaf: 0x180071e 25167646 (13: nrow: 286 rrow: 286)
      leaf: 0x180071f 25167647 (14: nrow: 239 rrow: 239)
      leaf: 0x1800720 25167648 (15: nrow: 236 rrow: 236)
      leaf: 0x1800721 25167649 (16: nrow: 222 rrow: 222)
      leaf: 0x1800722 25167650 (17: nrow: 213 rrow: 213)
      leaf: 0x1800723 25167651 (18: nrow: 227 rrow: 227)
      leaf: 0x1800724 25167652 (19: nrow: 204 rrow: 204)
      leaf: 0x1800725 25167653 (20: nrow: 226 rrow: 226)
      leaf: 0x1800726 25167654 (21: nrow: 231 rrow: 231)
      leaf: 0x1800727 25167655 (22: nrow: 211 rrow: 211)
      leaf: 0x1800728 25167656 (23: nrow: 228 rrow: 228)
      leaf: 0x1800729 25167657 (24: nrow: 235 rrow: 235)
      leaf: 0x180072a 25167658 (25: nrow: 222 rrow: 222)
      leaf: 0x180072b 25167659 (26: nrow: 224 rrow: 224)
      leaf: 0x180072c 25167660 (27: nrow: 227 rrow: 227)
      leaf: 0x180072d 25167661 (28: nrow: 222 rrow: 222)
      leaf: 0x180072e 25167662 (29: nrow: 260 rrow: 260)
      leaf: 0x180072f 25167663 (30: nrow: 239 rrow: 239)
      leaf: 0x1800730 25167664 (31: nrow: 244 rrow: 244)
      leaf: 0x1800731 25167665 (32: nrow: 242 rrow: 242)
      leaf: 0x1800732 25167666 (33: nrow: 256 rrow: 256)
      leaf: 0x1800733 25167667 (34: nrow: 259 rrow: 259)
      leaf: 0x1800734 25167668 (35: nrow: 242 rrow: 242)
      leaf: 0x1800735 25167669 (36: nrow: 233 rrow: 233)
      leaf: 0x1800736 25167670 (37: nrow: 249 rrow: 249)
      leaf: 0x1800737 25167671 (38: nrow: 255 rrow: 255)
      leaf: 0x1800738 25167672 (39: nrow: 286 rrow: 286)
      leaf: 0x1800739 25167673 (40: nrow: 311 rrow: 311)
      leaf: 0x180073a 25167674 (41: nrow: 277 rrow: 277)
      leaf: 0x180073b 25167675 (42: nrow: 338 rrow: 338)
      leaf: 0x180073c 25167676 (43: nrow: 277 rrow: 277)
      leaf: 0x180073d 25167677 (44: nrow: 266 rrow: 266)
      leaf: 0x180073e 25167678 (45: nrow: 336 rrow: 336)
      leaf: 0x180073f 25167679 (46: nrow: 358 rrow: 358)
      leaf: 0x1800740 25167680 (47: nrow: 286 rrow: 286)
      leaf: 0x1800741 25167681 (48: nrow: 212 rrow: 212)
      leaf: 0x1800742 25167682 (49: nrow: 199 rrow: 199)
      leaf: 0x1800743 25167683 (50: nrow: 258 rrow: 258)
      leaf: 0x1800744 25167684 (51: nrow: 246 rrow: 246)
      leaf: 0x1800745 25167685 (52: nrow: 245 rrow: 245)
      leaf: 0x1800746 25167686 (53: nrow: 249 rrow: 249)
      leaf: 0x1800747 25167687 (54: nrow: 239 rrow: 239)
      leaf: 0x1800748 25167688 (55: nrow: 236 rrow: 236)
      leaf: 0x1800749 25167689 (56: nrow: 295 rrow: 295)
      leaf: 0x180074a 25167690 (57: nrow: 238 rrow: 238)
      leaf: 0x180074b 25167691 (58: nrow: 234 rrow: 234)
      leaf: 0x180074c 25167692 (59: nrow: 283 rrow: 283)
      leaf: 0x180074d 25167693 (60: nrow: 262 rrow: 262)
      leaf: 0x180074e 25167694 (61: nrow: 266 rrow: 266)
      leaf: 0x180074f 25167695 (62: nrow: 267 rrow: 267)
      leaf: 0x1800750 25167696 (63: nrow: 242 rrow: 242)
      leaf: 0x1800751 25167697 (64: nrow: 244 rrow: 244)
      leaf: 0x1800752 25167698 (65: nrow: 252 rrow: 252)
      leaf: 0x1800753 25167699 (66: nrow: 238 rrow: 238)
      leaf: 0x1800754 25167700 (67: nrow: 241 rrow: 241)
      leaf: 0x1800755 25167701 (68: nrow: 255 rrow: 255)
      leaf: 0x1800756 25167702 (69: nrow: 269 rrow: 269)
      leaf: 0x1800757 25167703 (70: nrow: 260 rrow: 260)
      leaf: 0x1800758 25167704 (71: nrow: 262 rrow: 262)
      leaf: 0x1800759 25167705 (72: nrow: 218 rrow: 218)
      leaf: 0x180075a 25167706 (73: nrow: 253 rrow: 253)
      leaf: 0x180075b 25167707 (74: nrow: 259 rrow: 259)
      leaf: 0x180075c 25167708 (75: nrow: 222 rrow: 222)
      leaf: 0x180075d 25167709 (76: nrow: 227 rrow: 227)
      leaf: 0x180075e 25167710 (77: nrow: 209 rrow: 209)
      leaf: 0x180075f 25167711 (78: nrow: 216 rrow: 216)
      leaf: 0x1800760 25167712 (79: nrow: 219 rrow: 219)
      leaf: 0x1800761 25167713 (80: nrow: 220 rrow: 220)
      leaf: 0x1800762 25167714 (81: nrow: 219 rrow: 219)
      leaf: 0x1800763 25167715 (82: nrow: 222 rrow: 222)
      leaf: 0x1800764 25167716 (83: nrow: 223 rrow: 223)
      leaf: 0x1800765 25167717 (84: nrow: 206 rrow: 206)
      leaf: 0x1800766 25167718 (85: nrow: 255 rrow: 255)
      leaf: 0x1800767 25167719 (86: nrow: 254 rrow: 254)
      leaf: 0x1800768 25167720 (87: nrow: 248 rrow: 248)
      leaf: 0x1800769 25167721 (88: nrow: 246 rrow: 246)
      leaf: 0x180076a 25167722 (89: nrow: 295 rrow: 295)
      leaf: 0x180076b 25167723 (90: nrow: 268 rrow: 268)
      leaf: 0x180076c 25167724 (91: nrow: 281 rrow: 281)
      leaf: 0x180076d 25167725 (92: nrow: 239 rrow: 239)
      leaf: 0x180076e 25167726 (93: nrow: 209 rrow: 209)
      leaf: 0x180076f 25167727 (94: nrow: 306 rrow: 306)
      leaf: 0x1800770 25167728 (95: nrow: 269 rrow: 269)
      leaf: 0x1800771 25167729 (96: nrow: 240 rrow: 240)
      leaf: 0x1800772 25167730 (97: nrow: 254 rrow: 254)
      leaf: 0x1800773 25167731 (98: nrow: 266 rrow: 266)
      leaf: 0x1800774 25167732 (99: nrow: 259 rrow: 259)
      leaf: 0x1800775 25167733 (100: nrow: 269 rrow: 269)
      leaf: 0x1800776 25167734 (101: nrow: 249 rrow: 249)
      leaf: 0x1800777 25167735 (102: nrow: 215 rrow: 215)
      leaf: 0x1800778 25167736 (103: nrow: 230 rrow: 230)
      leaf: 0x1800779 25167737 (104: nrow: 252 rrow: 252)
      leaf: 0x180077a 25167738 (105: nrow: 296 rrow: 296)
      leaf: 0x180077b 25167739 (106: nrow: 226 rrow: 226)
      leaf: 0x180077c 25167740 (107: nrow: 194 rrow: 194)
      leaf: 0x180077d 25167741 (108: nrow: 194 rrow: 194)
      leaf: 0x180077e 25167742 (109: nrow: 194 rrow: 194)
      leaf: 0x180077f 25167743 (110: nrow: 194 rrow: 194)
      leaf: 0x1800780 25167744 (111: nrow: 194 rrow: 194)
      leaf: 0x1800781 25167745 (112: nrow: 217 rrow: 217)
      leaf: 0x1800782 25167746 (113: nrow: 231 rrow: 231)
      leaf: 0x1800783 25167747 (114: nrow: 237 rrow: 237)
      leaf: 0x1800784 25167748 (115: nrow: 276 rrow: 276)
      leaf: 0x1800785 25167749 (116: nrow: 232 rrow: 232)
      leaf: 0x1800786 25167750 (117: nrow: 220 rrow: 220)
      leaf: 0x1800787 25167751 (118: nrow: 239 rrow: 239)
      leaf: 0x1800788 25167752 (119: nrow: 222 rrow: 222)
      leaf: 0x180078b 25167755 (120: nrow: 220 rrow: 220)
      leaf: 0x180078c 25167756 (121: nrow: 238 rrow: 238)
      leaf: 0x180078d 25167757 (122: nrow: 286 rrow: 286)
      leaf: 0x180078e 25167758 (123: nrow: 250 rrow: 250)
      leaf: 0x180078f 25167759 (124: nrow: 256 rrow: 256)
      leaf: 0x1800790 25167760 (125: nrow: 261 rrow: 261)
      leaf: 0x1800791 25167761 (126: nrow: 249 rrow: 249)
      leaf: 0x1800792 25167762 (127: nrow: 246 rrow: 246)
      leaf: 0x1800793 25167763 (128: nrow: 243 rrow: 243)
      leaf: 0x1800794 25167764 (129: nrow: 227 rrow: 227)
      leaf: 0x1800795 25167765 (130: nrow: 216 rrow: 216)
      leaf: 0x1800796 25167766 (131: nrow: 235 rrow: 235)
      leaf: 0x1800797 25167767 (132: nrow: 229 rrow: 229)
      leaf: 0x1800798 25167768 (133: nrow: 230 rrow: 230)
      leaf: 0x1800799 25167769 (134: nrow: 224 rrow: 224)
      leaf: 0x180079a 25167770 (135: nrow: 219 rrow: 219)
      leaf: 0x180079b 25167771 (136: nrow: 268 rrow: 268)
      leaf: 0x180079c 25167772 (137: nrow: 242 rrow: 242)
      leaf: 0x180079d 25167773 (138: nrow: 203 rrow: 203)
      leaf: 0x180079e 25167774 (139: nrow: 200 rrow: 200)
      leaf: 0x180079f 25167775 (140: nrow: 184 rrow: 184)
      leaf: 0x18007a0 25167776 (141: nrow: 177 rrow: 177)
      leaf: 0x18007a1 25167777 (142: nrow: 182 rrow: 182)
      leaf: 0x18007a2 25167778 (143: nrow: 214 rrow: 214)
      leaf: 0x18007a3 25167779 (144: nrow: 217 rrow: 217)
      leaf: 0x18007a4 25167780 (145: nrow: 222 rrow: 222)
      leaf: 0x18007a5 25167781 (146: nrow: 189 rrow: 189)
      leaf: 0x18007a6 25167782 (147: nrow: 190 rrow: 190)
      leaf: 0x18007a7 25167783 (148: nrow: 185 rrow: 185)
      leaf: 0x18007a8 25167784 (149: nrow: 197 rrow: 197)
      leaf: 0x18007a9 25167785 (150: nrow: 203 rrow: 203)
      leaf: 0x18007aa 25167786 (151: nrow: 197 rrow: 197)
      leaf: 0x18007ab 25167787 (152: nrow: 210 rrow: 210)
      leaf: 0x18007ac 25167788 (153: nrow: 211 rrow: 211)
      leaf: 0x18007ad 25167789 (154: nrow: 195 rrow: 195)
      leaf: 0x18007ae 25167790 (155: nrow: 209 rrow: 209)
      leaf: 0x18007af 25167791 (156: nrow: 192 rrow: 192)
      leaf: 0x18007b0 25167792 (157: nrow: 192 rrow: 192)
      leaf: 0x18007b1 25167793 (158: nrow: 193 rrow: 193)
      leaf: 0x18007b2 25167794 (159: nrow: 208 rrow: 208)
      leaf: 0x18007b3 25167795 (160: nrow: 200 rrow: 200)
      leaf: 0x18007b4 25167796 (161: nrow: 206 rrow: 206)
      leaf: 0x18007b5 25167797 (162: nrow: 216 rrow: 216)
      leaf: 0x18007b6 25167798 (163: nrow: 186 rrow: 186)
      leaf: 0x18007b7 25167799 (164: nrow: 183 rrow: 183)
      leaf: 0x18007b8 25167800 (165: nrow: 202 rrow: 202)
      leaf: 0x18007b9 25167801 (166: nrow: 200 rrow: 200)
      leaf: 0x18007ba 25167802 (167: nrow: 199 rrow: 199)
      leaf: 0x18007bb 25167803 (168: nrow: 184 rrow: 184)
      leaf: 0x18007bc 25167804 (169: nrow: 188 rrow: 188)
      leaf: 0x18007bd 25167805 (170: nrow: 188 rrow: 188)
      leaf: 0x18007be 25167806 (171: nrow: 182 rrow: 182)
      leaf: 0x18007bf 25167807 (172: nrow: 192 rrow: 192)
      leaf: 0x18007c0 25167808 (173: nrow: 198 rrow: 198)
      leaf: 0x18007c1 25167809 (174: nrow: 210 rrow: 210)
      leaf: 0x18007c2 25167810 (175: nrow: 195 rrow: 195)
      leaf: 0x18007c3 25167811 (176: nrow: 186 rrow: 186)
      leaf: 0x18007c4 25167812 (177: nrow: 183 rrow: 183)
      leaf: 0x18007c5 25167813 (178: nrow: 183 rrow: 183)
      leaf: 0x18007c6 25167814 (179: nrow: 209 rrow: 209)
      leaf: 0x18007c7 25167815 (180: nrow: 197 rrow: 197)
      leaf: 0x18007c8 25167816 (181: nrow: 201 rrow: 201)
      leaf: 0x18007c9 25167817 (182: nrow: 184 rrow: 184)
      leaf: 0x18007ca 25167818 (183: nrow: 184 rrow: 184)
      leaf: 0x18007cb 25167819 (184: nrow: 184 rrow: 184)
      leaf: 0x18007cc 25167820 (185: nrow: 180 rrow: 180)
      leaf: 0x18007cd 25167821 (186: nrow: 178 rrow: 178)
      leaf: 0x18007ce 25167822 (187: nrow: 179 rrow: 179)
      leaf: 0x18007cf 25167823 (188: nrow: 179 rrow: 179)
      leaf: 0x18007d0 25167824 (189: nrow: 179 rrow: 179)
      leaf: 0x18007d1 25167825 (190: nrow: 180 rrow: 180)
      leaf: 0x18007d2 25167826 (191: nrow: 183 rrow: 183)
      leaf: 0x18007d3 25167827 (192: nrow: 178 rrow: 178)
      leaf: 0x18007d4 25167828 (193: nrow: 179 rrow: 179)
      leaf: 0x18007d5 25167829 (194: nrow: 186 rrow: 186)
      leaf: 0x18007d6 25167830 (195: nrow: 191 rrow: 191)
      leaf: 0x18007d7 25167831 (196: nrow: 203 rrow: 203)
      leaf: 0x18007d8 25167832 (197: nrow: 190 rrow: 190)
      leaf: 0x18007d9 25167833 (198: nrow: 186 rrow: 186)
      leaf: 0x18007da 25167834 (199: nrow: 185 rrow: 185)
      leaf: 0x18007db 25167835 (200: nrow: 183 rrow: 183)
      leaf: 0x18007dc 25167836 (201: nrow: 198 rrow: 198)
      leaf: 0x18007dd 25167837 (202: nrow: 208 rrow: 208)
      leaf: 0x18007de 25167838 (203: nrow: 184 rrow: 184)
      leaf: 0x18007df 25167839 (204: nrow: 183 rrow: 183)
      leaf: 0x18007e0 25167840 (205: nrow: 186 rrow: 186)
      leaf: 0x18007e1 25167841 (206: nrow: 189 rrow: 189)
      leaf: 0x18007e2 25167842 (207: nrow: 181 rrow: 181)
      leaf: 0x18007e3 25167843 (208: nrow: 181 rrow: 181)
      leaf: 0x18007e4 25167844 (209: nrow: 193 rrow: 193)
      leaf: 0x18007e5 25167845 (210: nrow: 197 rrow: 197)
      leaf: 0x18007e6 25167846 (211: nrow: 186 rrow: 186)
      leaf: 0x18007e7 25167847 (212: nrow: 189 rrow: 189)
      leaf: 0x18007e8 25167848 (213: nrow: 201 rrow: 201)
      leaf: 0x18007e9 25167849 (214: nrow: 201 rrow: 201)
      leaf: 0x18007ea 25167850 (215: nrow: 183 rrow: 183)
      leaf: 0x18007eb 25167851 (216: nrow: 196 rrow: 196)
      leaf: 0x18007ec 25167852 (217: nrow: 178 rrow: 178)
      leaf: 0x18007ed 25167853 (218: nrow: 183 rrow: 183)
      leaf: 0x18007ee 25167854 (219: nrow: 180 rrow: 180)
      leaf: 0x18007ef 25167855 (220: nrow: 196 rrow: 196)
      leaf: 0x18007f0 25167856 (221: nrow: 197 rrow: 197)
      leaf: 0x18007f1 25167857 (222: nrow: 197 rrow: 197)
      leaf: 0x18007f2 25167858 (223: nrow: 194 rrow: 194)
      leaf: 0x18007f3 25167859 (224: nrow: 190 rrow: 190)
      leaf: 0x18007f4 25167860 (225: nrow: 193 rrow: 193)
      leaf: 0x18007f5 25167861 (226: nrow: 207 rrow: 207)
      leaf: 0x18007f6 25167862 (227: nrow: 202 rrow: 202)
      leaf: 0x18007f7 25167863 (228: nrow: 194 rrow: 194)
      leaf: 0x18007f8 25167864 (229: nrow: 207 rrow: 207)
      leaf: 0x18007f9 25167865 (230: nrow: 204 rrow: 204)
      leaf: 0x18007fa 25167866 (231: nrow: 194 rrow: 194)
      leaf: 0x18007fb 25167867 (232: nrow: 185 rrow: 185)
      leaf: 0x18007fc 25167868 (233: nrow: 208 rrow: 208)
      leaf: 0x18007fd 25167869 (234: nrow: 218 rrow: 218)
      leaf: 0x18007fe 25167870 (235: nrow: 194 rrow: 194)
      leaf: 0x18007ff 25167871 (236: nrow: 192 rrow: 192)
      leaf: 0x1800800 25167872 (237: nrow: 200 rrow: 200)
      leaf: 0x1800801 25167873 (238: nrow: 191 rrow: 191)
      leaf: 0x1800802 25167874 (239: nrow: 186 rrow: 186)
      leaf: 0x1800803 25167875 (240: nrow: 181 rrow: 181)
      leaf: 0x1800804 25167876 (241: nrow: 182 rrow: 182)
      leaf: 0x1800805 25167877 (242: nrow: 182 rrow: 182)
      leaf: 0x1800806 25167878 (243: nrow: 190 rrow: 190)
      leaf: 0x1800807 25167879 (244: nrow: 191 rrow: 191)
      leaf: 0x1800808 25167880 (245: nrow: 180 rrow: 180)
      leaf: 0x180080b 25167883 (246: nrow: 55 rrow: 55)
----- end tree dump

------------------
解释 treedump 输出
------------------

branch 表示的是 branch block ，它后面跟了一个十六进制表示的DBA(data block address)，以及用10进制表示的DBA
DBA 之后表示在同一层次的相对位置(root 从0开始,branch 以及leaf从 -1开始)
nrow  表示块中包含了多少条目(包括delete的条目)
rrow  表示块中包含的实际条目(不包括delete的条目)
level 表示从该block到leaf的深度(leaf没有 level)

branch: 0x1800014 25165844 (0: nrow: 2, level: 2)

这个 branch block 的 level 为2，也就是说 从这个branch block 到 leaf block 的深度为2,根据前面的查询，这个索引的Blevel为2,所以这个
branch其实是 root block. 其实根据 nrow:2 也可以看出来它是root block，因为nrow:2 说明它只包含了2个条目，那么这2个条目其实就是dump
文件中的其他2个 branch block 的条目

现在我来验证一下 branch: 0x1800014 25165844 (0: nrow: 2, level: 2) 是不是 root block , 我查询这个 branch 的 DBA

SQL> select dbms_utility.data_block_address_file('25165844') FILE_ID,
  2  dbms_utility.data_block_address_block('25165844') BLOCK_ID from dual;

   FILE_ID   BLOCK_ID
---------- ----------
         6         20

Btree 索引的 root block总是segment header+1,所以我查询该索引的段头

SQL> select header_file,header_block from dba_segments where segment_name='I_TEST_OBJECT_NAME';

HEADER_FILE HEADER_BLOCK
----------- ------------
          6           19

那么现在已经证明了 branch: 0x1800014 25165844 (0: nrow: 2, level: 2) 是root block, 其实 treedump第一个 branch block 就是 root block

branch: 0x1800710 25167632 (-1: nrow: 247, level: 1)

这个branch 的 DBA为0x1800710(十六进制)，25167632(十进制) -1 表示它是与它在同一个深度的 branch block中的第一个 branch block
nrow: 247表示它有247个 leaf block. level: 1 表示 这个 branch block 到 leaf block的深度为1
 
leaf: 0x1800015 25165845 (-1: nrow: 182 rrow: 182)

leaf 表示它是一个 leaf block  0x1800015 25165845 分别是 这个 leaf block 的 十六进制/十进制的 DBA -1 表示它是 leaf block 中的第一个 block
nrow: 182 表示它一共有182条记录 rrow: 182 表示它实际有182条记录

-----------------
branch block dump
-----------------

这里选择dump branch: 0x1800710 25167632 (-1: nrow: 247, level: 1)

SQL> select dbms_utility.data_block_address_file('25167632') FILE_ID,
  2  dbms_utility.data_block_address_block('25167632') BLOCK_ID from dual;

   FILE_ID   BLOCK_ID
---------- ----------
         6       1808

SQL> alter system dump datafile 6 block 1808;

System altered.

部分 DUMP 文件

Block header dump:  0x01800710
 Object id on Block? Y
 seg/obj: 0xd78a  csc: 0x00.17bbef0  itc: 1  flg: E  typ: 2 - INDEX
     brn: 0  bdba: 0x1800709 ver: 0x01 opc: 0
     inc: 0  exflg: 0
 
 Itl           Xid                  Uba         Flag  Lck        Scn/Fsc
0x01   0xffff.000.00000000  0x00000000.0000.00  C---    0  scn 0x0000.017bbef0
 
Branch block dump
=================
header address 156329548=0x951664c
kdxcolev 1
KDXCOLEV Flags = - - -
kdxcolok 0
kdxcoopc 0x80: opcode=0: iot flags=--- is converted=Y
kdxconco 2
kdxcosdc 0
kdxconro 246
kdxcofbo 520=0x208
kdxcofeo 549=0x225
kdxcoavs 29
kdxbrlmc 25165845=0x1800015
kdxbrsno 0
kdxbrbksz 8060 
kdxbr2urrc 15
row#0[8027] dba: 25165846=0x1800016
col 0; len 24; (24): 
 2f 31 30 64 65 32 32 63 36 5f 43 6c 61 73 73 54 79 70 65 49 6d 70 6c 32
col 1; len 3; (3):  01 80 03
row#1[7988] dba: 25165847=0x1800017
col 0; len 30; (30): 
 2f 31 31 64 35 30 39 31 32 5f 44 61 74 65 46 6f 72 6d 61 74 5a 6f 6e 65 44
 61 74 61 5f 7a
col 1; len 3; (3):  01 80 04
row#2[7977] dba: 25165848=0x1800018
col 0; len 5; (5):  2f 31 32 61 32
col 1; TERM
..................................省略..................................

row#245[549] dba: 25167630=0x180070e
col 0; len 10; (10):  41 4c 4c 24 4f 4c 41 50 5f 44
col 1; TERM
----- end of branch block dump -----
End dump data blocks tsn: 7 file#: 6 minblk 1808 maxblk 1808

----------------
解释部分dump输出
----------------

kdxcolev 1  --该block到leaf block的深度(leaf block 为0).这里branch block 的level 为1 与前面查询相吻合 
KDXCOLEV Flags = - - -
kdxcolok 0  --表示是否有事务lock了这个branch block，如果有 有多少事务
kdxcoopc 0x80: opcode=0: iot flags=--- is converted=Y
kdxconco 2  --索引值条目. 这里表示有2个条目
kdxcosdc 0  --这个block的结构被更改次数.这里0表示没有更改
kdxconro 246 --索引条目(不包含kdxbrlmc 指针)
kdxcofbo 520=0x208 --空闲空间的起始偏移量
kdxcofeo 549=0x225 --空闲空间的末尾偏移量
kdxcoavs 29  --block中的空闲空间=kdxcofeo-kdxcofbo
kdxbrlmc 25165845=0x1800015 --如果index value小于row#0，指向该 block 的地址
kdxbrsno 0   --最后被更改的索引条目
kdxbrbksz 8060 --块中的可用空间
kdxbr2urrc 15

row#2[7977] dba: 25165848=0x1800018  --row# 表示索引条目数，从0开始，紧接着就是十进制和十六进制的DBA，该DBA指向 leaf block
col 0; len 5; (5):  2f 31 32 61 32   --列的行号，从0开始，紧接着的就是列的长度以及列的值，那么这个值称之为separator key，
                                       这个separator key 可以区分真实的索引值，所以从这里我们也知道 branch block不会存储完整的索引值，只要能区分就行
col 1; TERM

---------------
leaf block DUMP
---------------

这里选择 dump row#2[7977] dba: 25165848=0x1800018 ，因为它包含的索引键很少

dump 之前先看一下  row#2[7977] 存储了什么索引键

2f 表示 /
31 表示 1
32 表示 2
61 表示 a

具体转换过程如下

SQL> declare n varchar2(2000);
  2       begin
  3       dbms_stats.convert_raw_value('2f',n);
  4       dbms_output.put_line(n);
  5       end;
  6  /
/

PL/SQL procedure successfully completed.

SQL> select dbms_utility.data_block_address_file('25165848') FILE_ID,
  2   dbms_utility.data_block_address_block('25165848') BLOCK_ID from dual;

   FILE_ID   BLOCK_ID
---------- ----------
         6         24

SQL> alter system dump datafile 6 block 24;

System altered.

部分DUMP文件

Block header dump:  0x01800018
 Object id on Block? Y
 seg/obj: 0xd78a  csc: 0x00.17bbef0  itc: 2  flg: E  typ: 2 - INDEX
     brn: 0  bdba: 0x1800011 ver: 0x01 opc: 0
     inc: 0  exflg: 0
 
 Itl           Xid                  Uba         Flag  Lck        Scn/Fsc
0x01   0x0000.000.00000000  0x00000000.0000.00  ----    0  fsc 0x0000.00000000
0x02   0xffff.000.00000000  0x00000000.0000.00  C---    0  scn 0x0000.017bbef0
 
Leaf block dump
===============
header address 146564708=0x8bc6664
kdxcolev 0
KDXCOLEV Flags = - - -
kdxcolok 0
kdxcoopc 0x80: opcode=0: iot flags=--- is converted=Y
kdxconco 2
kdxcosdc 0
kdxconro 189
kdxcofbo 414=0x19e
kdxcofeo 1248=0x4e0
kdxcoavs 834
kdxlespl 0
kdxlende 0
kdxlenxt 25165849=0x1800019
kdxleprv 25165847=0x1800017
kdxledsz 0
kdxlebksz 8036
row#0[7996] flag: ------, lock: 0, len=40
col 0; len 30; (30): 
 2f 31 32 61 32 37 33 65 36 5f 49 6e 74 65 72 6e 61 74 69 6f 6e 61 6c 46 6f
 72 6d 61 74 74
col 1; len 6; (6):  01 80 00 e2 00 16
row#1[7956] flag: ------, lock: 0, len=40
col 0; len 30; (30): 
 2f 31 32 61 32 37 33 65 36 5f 49 6e 74 65 72 6e 61 74 69 6f 6e 61 6c 46 6f
 72 6d 61 74 74
col 1; len 6; (6):  01 80 00 e2 00 17
row#2[7916] flag: ------, lock: 0, len=40
col 0; len 30; (30): 
 2f 31 32 61 32 37 33 65 36 5f 49 6e 74 65 72 6e 61 74 69 6f 6e 61 6c 46 6f
 72 6d 61 74 74
col 1; len 6; (6):  01 80 03 b5 00 25
row#3[7876] flag: ------, lock: 0, len=40
col 0; len 30; (30): 
 2f 31 32 61 32 37 33 65 36 5f 49 6e 74 65 72 6e 61 74 69 6f 6e 61 6c 46 6f
 72 6d 61 74 74
col 1; len 6; (6):  01 80 03 b5 00 26
row#4[7836] flag: ------, lock: 0, len=40

..................................省略..................................

row#188[1248] flag: ------, lock: 0, len=40
col 0; len 30; (30): 
 2f 31 33 38 35 32 32 37 66 5f 4d 61 70 52 65 67 69 6f 6e 43 6f 6e 74 61 69
 6e 6d 65 6e 74
col 1; len 6; (6):  01 80 01 6a 00 00
----- end of leaf block dump -----
End dump data blocks tsn: 7 file#: 6 minblk 24 maxblk 24

----------------
解释部分dump输出
----------------

kdxcolev 0  --该block到leaf block的深度(leaf block 为0).
KDXCOLEV Flags = - - -
kdxcolok 0   --表示是否有事务lock了这个branch block，如果有 有多少事务
kdxcoopc 0x80: opcode=0: iot flags=--- is converted=Y
kdxconco 2   --索引值条目. 这里表示有2个条目
kdxcosdc 0   --这个block的结构被更改次数.这里0表示没有更改
kdxconro 189   --索引条目 ,这里有189个索引条目 它等于 row#188-row#0+1
kdxcofbo 414=0x19e  --空闲空间的起始偏移量
kdxcofeo 1248=0x4e0 --空闲空间的末尾偏移量
kdxcoavs 834        --block中的空闲空间=kdxcofeo-kdxcofbo
kdxlespl 0   --block split的时候没有commit的记录的大小(byte)
kdxlende 0   --被删除的条目
kdxlenxt 25165849=0x1800019 --指向下一个 leaf block的指针(DBA)
kdxleprv 25165847=0x1800017 --指向上一个 leaf block的指针(DBA)
kdxledsz 0   --被删除的空间
kdxlebksz 8036 --可用空间

row#0[7996] flag: ------, lock: 0, len=40
col 0; len 30; (30): 
 2f 31 32 61 32 37 33 65 36 5f 49 6e 74 65 72 6e 61 74 69 6f 6e 61 6c 46 6f
 72 6d 61 74 74
col 1; len 6; (6):  01 80 00 e2 00 16

row# 表示索引值的条目，从0开始  lock: 0 表示ITL中的锁信息 0表示没有被锁 len=40 表示索引值长度
col 表示列号，从0开始 那么接下来就是索引的键值 以及 rowid中后12位值

下面我们来看一下row#0存储的值是什么

SQL> declare n varchar2(2000);
  2       begin
  3       dbms_stats.convert_raw_value('2f31326132373365365f496e7465726e6174696f6e616c466f726d617474',n);
  4       dbms_output.put_line(n);
  5       end;
  6  /
/12a273e6_InternationalFormatt

PL/SQL procedure successfully completed.

ROWID一共用18位表示
最前6位表示 data object number
之后后3位表示 datafile number
之后后6位表示 datablock number
最后3位表示 row number

所以这里的rowid其实只是除去了 data object number的一部分而已

SQL> select to_number('18000e2','xxxxxxxxxxxxxxxx') from dual;

TO_NUMBER('18000E2','XXXXXXXXX
------------------------------
                      25166050

SQL> select dbms_utility.data_block_address_file('25166050') FILE_ID,dbms_utility.data_block_address_block('25166050') BLOCK_ID from dual;

   FILE_ID   BLOCK_ID
---------- ----------
         6        226

SQL> select dbms_rowid.rowid_relative_fno(rowid)file_id, dbms_rowid.rowid_block_number(rowid)block_id,dbms_rowid.rowid_row_number(rowid) row#
  2  from test where object_name='/12a273e6_InternationalFormatt';

   FILE_ID   BLOCK_ID       ROW#
---------- ---------- ----------
         6        226         22
         6        226         23
         6        949         37
         6        949         38

根据这个还原的值我来查询一下

SQL> select owner,object_name,rowid from test where object_name='/12a273e6_InternationalFormatt';

OWNER                          OBJECT_NAME                    ROWID
------------------------------ ------------------------------ ------------------
SYS                            /12a273e6_InternationalFormatt AAANeJAAGAAAADiAAW
PUBLIC                         /12a273e6_InternationalFormatt AAANeJAAGAAAADiAAX
SYS                            /12a273e6_InternationalFormatt AAANeJAAGAAAAO1AAl
PUBLIC                         /12a273e6_InternationalFormatt AAANeJAAGAAAAO1AAm

SQL> select owner,object_name,dump(rowid,16) from test where object_name='/12a273e6_InternationalFormatt';

OWNER      OBJECT_NAME                         DUMP(ROWID,16)
---------- ----------------------------------- --------------------------------------------------------------------------------
SYS        /12a273e6_InternationalFormatt      Typ=69 Len=10: 0,0,d7,89,1,80,0,e2,0,16
PUBLIC     /12a273e6_InternationalFormatt      Typ=69 Len=10: 0,0,d7,89,1,80,0,e2,0,17
SYS        /12a273e6_InternationalFormatt      Typ=69 Len=10: 0,0,d7,89,1,80,3,b5,0,25
PUBLIC     /12a273e6_InternationalFormatt      Typ=69 Len=10: 0,0,d7,89,1,80,3,b5,0,26



回忆一下 branch block 

row#2[7977] dba: 25165848=0x1800018
col 0; len 5; (5):  2f 31 32 61 32     -------它正是真正的索引键的前缀
col 1; TERM

row#0[7996] flag: ------, lock: 0, len=40
col 0; len 30; (30): 
 2f 31 32 61 32 37 33 65 36 5f 49 6e 74 65 72 6e 61 74 69 6f 6e 61 6c 46 6f
 72 6d 61 74 74
col 1; len 6; (6):  01 80 00 e2 00 16

之所以Oracle在 Branch block中只记录 索引键值的前缀，而不是所有值，是因为这样可以节约空间，从而能够存储更多的 索引条目
同时，我们也能理解了为什么 查询使用 like '%xxx' 这种方法不会走Btree 索引，因为Branch block 存储的是前缀.


```

