---
layout: post
title: "如何写出更快的 SQL"
date: 2018-10-13 23:00:57
comments: true
reward: true
tags: 
    - db2
    - sql
---

在数据库开发的初期，或者在系统刚上线的初期，由于数据量比较少，一些查询 SQL 语句、视图、存储过程编写等体会不出 SQL 语句各种写法的性能优劣，但是随着数据库中数据的增加，像数据仓库这种 TB 级别的海量数据，劣质SQL语句和优质SQL语句之间的速度差别可以达到上百倍，因此写 sql 不能简单的能查出相应的数据即可，而是要写出高质量的 SQL 语句，提高 SQL 语句的执行速度。

<!-- more -->

下面我就自己的工作经验，分享一下如何写出更快的 SQL 

## 一、查看执行计划来选择更快的 SQL

在写 SQL 的初期，你可能不知道到底是使用 UNION ALL  好还是 FULL JOIN 好，是使用 EXISTS 好，还是使用 IN 好，那么不防将这些语句都写出来，看看数据库的执行计划怎么说。

首先要明白什么是执行计划

>执行计划是数据库根据 SQL 语句和相关表的统计信息作出的一个查询方案，这个方案是由查询优化器自动分析产生的，比如一条 SQL 语句如果用来从一个 10 万条记录的表中查 1 条记录，那查询优化器会选择索引查找方式，如果该表进行了归档，当前只剩下 5000 条记录了，那查询优化器就会改变方案，采用全表扫描方式。

可见，执行计划并不是固定的，它是个性化的。产生一个正确的“执行计划”有两点很重要：
(1)    SQL语句是否清晰地告诉查询优化器它想干什么？
(2)    查询优化器得到的数据库统计信息是否是最新的、正确的？


比如现在有个这样的需求：有两个客户信息表 custinfo_a、 custinfo_b ，主健都是客户号 custid，现要求对这两个表的信息进行整合，要求合并后的表主健仍是 custid，如果同一个 custid 在这两个表都存在，优先取 custinfo_a 表的信息。

此时你可能会想到三种写法：

写法一：使用 FULL JOIN 

```sql
SELECT 
NVL(A.CUSTID,B.CUSTID)   AS USTID,--使用NVL优先取A表信息为准
NVL(A.CUSTNAME,B.CUSTNAME) AS CUSTNAME
FROM CUSTINFO_A A
FULL JOIN CUSTINFO_B  B ON A.CUSTID = B.CUSTID
```
在 db2 的说明查询中查看其成本：
 
![image.png](https://upload-images.jianshu.io/upload_images/12989993-024f2e586825c82f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到总成本为 9796.56，这里不用关心这个数字的单位是什么，只要知道它越大，查询的就越慢。
其中的 TBSCAN 代表整表扫描，IXSCAN 代表索引扫描，可以看出 IXSCAN 的成本是很低的。

写法二：使用 UNION ALL 和 NOT EXISTS

```sql
SELECT A.CUSTID,        a.CUSTNAME
FROM CUSTINFO_A A
UNION  ALL
SELECT b.CUSTID,       b.CUSTNAME
FROM CUSTINFO_b b
WHERE NOT EXISTS     (SELECT '1'      FROM CUSTINFO_A A      WHERE A.CUSTID = b.CUSTID)
```
在 db2 的说明查询中查看其成本：
![](https://upload-images.jianshu.io/upload_images/12989993-7dbe4d9a83dbfe90.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到总成本为 6375.67。

写法三：使用 row_nubmer() over() 过滤

```sql
SELECT CUSTID,        CUSTNAME
FROM
(SELECT CUSTID,        CUSTNAME, 
          ROW_NUMBER()OVER(PARTITION BY CUSTID ORDER BY PRIORITY) BH 
    FROM
     (SELECT A.CUSTID, A.CUSTNAME, '1' AS PRIORITY       FROM CUSTINFO_A A
      UNION  ALL SELECT B.CUSTID, B.CUSTNAME, '2' AS PRIORITY      FROM CUSTINFO_B B
      )
)
WHERE BH =1
```
在 db2 的说明查询中查看其成本：
![image.png](https://upload-images.jianshu.io/upload_images/12989993-2d35571da48f2199.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到总成本为 6147.56

因此追求快速响应的的可以使用 方法三。

**那么如何使用 db2 的执行计划呢？**
**windows 用户**，可以在程序中找到 **控制中心**，图标如下图所示：
![image.png](https://upload-images.jianshu.io/upload_images/12989993-420afe22b73aeb7b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

点击打开后，查找相应的数据库，右键选择说明查询，如下图所示：
![image.png](https://upload-images.jianshu.io/upload_images/12989993-38d3ee703b6eb68e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
再将查询的 SQL 粘贴到输入框中，确定即可看到上面所示的执行计划图，如果未登陆会要求让你输入用户名密码。

**LINUX 或 AIX 用户**
在下面的脚本中的 SQL 语句替换为你自己的 SQL ，执行此 shell 脚本，即可生成 explain.out ，查看 explain.out 可以看到和 windows 下一样效果的文本信息和更多的 CPU 、I/O 消耗等。
```sh
#!/bin/sh
db2 connect to edwdb user dsadm using dsadm 
db2 -tvf /home/edwinst/sqllib/misc/EXPLAIN.DDL
db2 set current explain mode explain

db2 "
SELECT 
NVL(A.CUSTID,B.CUSTID)   AS USTID,
NVL(A.CUSTNAME,B.CUSTNAME) AS CUSTNAME
 FROM edw.CUSTINFO_A A
FULL JOIN edw.CUSTINFO_B  B ON A.CUSTID = B.CUSTID
";

db2 set current explain mode no
db2exfmt -d edwdb -g TIC -w -1 -l -# 0 -s % -n % -o explain.out  #输出信息到文件
#db2exfmt -d edwdb -g TIC -w -1 -l -# 0 -s % -n % -t  #输出信息到终端
db2 terminate
```
注意  /home/edwinst/sqllib/ 是 db2 的 HOME 路径。

## 二、一些原则和经验

- 避免全表扫描

Where 条件中尽可能**少用否定**，如 NOT、!=、<>、!<、!>、NOT EXISTS、NOT IN、NOT LIKE，它们会引起全表扫描。那些可以过滤掉最大数量记录的条件写在 Where 子句的末尾。

- 避免Select * 

Selcet 中每少提取一个字段，数据的提取速度就会有相应的提升。提升的速度还要看您舍弃的字段的大小来判断。应避免使用Select * ，就算查询记录数，也不要使用 *，可以使用 select 1 from tablename 。

- 用 Where 子句替代 having 子句 

避免使用 having 子句，having 只会在检索出所有记录之后才对结果集进行过滤。

- exists 代替 in 

当（）中的数据量较大时使用 exists() ，较少时可以使用 in ()。

- IS NULL 与 IS NOT NULL 

数据库不能用 NULL 作索引，任何包含 NULL 值的列都将不会被包含在索引中。即使索引有多列这样的情况下，只要这些列中有一列含有 NULL ，该列就会从索引中排除。也就是说如果某列存在 NULL 值，即使对该列建索引也不会提高性能。任何在 where 子句中使用 IS NULL 或 IS NULL 的语句优化器是不使用索引的。 

- 联接列 

对于有联接的列，即使最后的联接值为一个静态值，优化器是不会使用索引的。 
like ‘%xx%’ 不会执行索引 
like ‘y%xx%’ 会执行索引 

- 用 TRUNCATE 替代 DELETE 来清空一个表

当删除表中的记录时，在通常情况下, 回滚段 (rollback segments ) 用来存放可以被恢复的信息。如果你没有COMMIT 事务，db2 可以将数据恢复到删除之前的状态，而当运用 TRUNCATE 时, 回滚段不再存放任何可被恢复的信息，当命令运行后，数据不能被恢复，因此很少的资源被调用，执行时间也会很短，TRUNCATE 只在删除全表适用。

- 用 EXISTS 替代 IN、用 NOT EXISTS 替代 NOT IN： 

在许多基于基础表的查询中，为了满足一个条件，往往需要对另一个表进行联接。在这种情况下， 使用EXISTS（或 NOT EXISTS）通常将提高查询的效率. 在子查询中，NOT IN 子句将执行一个内部的排序和合并。无论在哪种情况下，NOT IN 都是最低效的（因为它对子查询中的表执行了一个全表遍历）。为了避免使用NOT IN ,我们可以把它改写成外连接(Outer Joins)或NOT EXISTS. 
例子： 

```sql
（高效）SELECT * FROM EMP (基础表) WHERE EMPNO > 0 AND EXISTS (SELECT ‘X’ FROM DEPT WHERE DEPT.DEPTNO = EMP.DEPTNO AND LOC = ‘MELB’) 
（低效）SELECT * FROM EMP (基础表) WHERE EMPNO > 0 AND DEPTNO IN(SELECT DEPTNO FROM DEPT WHERE LOC = ‘MELB’)
```

- 用索引提高效率

使用索引同样能提高效率，但是我们也必须注意到它的代价，索引需要空间来存储，也需要定期维护，每当有记录在表中增减或索引列被修改时，索引本身也会被修改。这意味着每条记录的 INSERT、DELETE 、UPDATE 将为此多付出 4 , 5 次的磁盘 I/O 。因为索引需要额外的存储空间和处理，那些不必要的索引反而会使查询反应时间变慢，定期的重构索引是有必要的：
```sql
ALTER INDEX REBUILD
```

- 用 EXISTS 替换 DISTINCT

当提交一个包含一对多表信息（比如部门表和雇员表）的查询时，避免在SELECT 子句中使用 DISTINCT， 一般可以考虑用 EXIST 替换, EXISTS 使查询更为迅速，因为 RDBMS 核心模块将在子查询的条件一旦满足后，立刻返回结果。
```SQL
(低效): SELECT  DISTINCT  DEPT_NO,DEPT_NAME  FROM  DEPT D , EMP E 
WHERE  D.DEPT_NO = E.DEPT_NO 
(高效): SELECT  DEPT_NO,DEPT_NAME  FROM  DEPT D  WHERE  EXISTS ( SELECT ‘X' 
FROM  EMP E  WHERE E.DEPT_NO = D.DEPT_NO); 
```


- 避免在索引列上使用 NOT

我们要避免在索引列上使用 NOT ， NOT 会产生在和在索引列上使用函数相同的影响，会导致使用索引转而执行全表扫描。 

- 避免在索引列上使用计算 

WHERE 子句中，如果索引列是函数的一部分．优化器将不使用索引而使用全表扫描． 

- 用>=替代> 
```sql
高效: 
 SELECT * FROM  EMP  WHERE  DEPTNO >=4 
低效: 
 SELECT * FROM  EMP  WHERE  DEPTNO >3 
```
两者的区别在于， 前者 DBMS 将直接跳到第一个 DEPT 等于 4 的记录而后者将首先定位到 DEPTNO =3 的记录并且向前扫描到第一个 DEPT 大于 3 的记录。

- 用 UNION 替换 OR (适用于索引列) 

通常情况下， 用 UNION 替换 WHERE 子句中的 OR 将会起到较好的效果，对索引列使用 OR 将造成全表扫描。注意， 以上规则只针对多个索引列有效。如果有 column 没有被索引， 查询效率可能会因为你没有选择 OR 而降低。在下面的例子中， LOC _ ID 和 REGION 上都建有索引：

```sql
高效: 
 SELECT  LOC _ ID ， LOC _ DESC ， REGION 
 FROM  LOCATION 
 WHERE  LOC _ ID = 10 
 UNION 
 SELECT  LOC _ ID ， LOC _ DESC ， REGION 
 FROM  LOCATION 
 WHERE  REGION = “ MELBOURNE ” 
低效: 
 SELECT  LOC _ ID ， LOC _ DESC ， REGION 
 FROM  LOCATION 
 WHERE  LOC _ ID = 10 OR  REGION = “ MELBOURNE ” 
```
如果你坚持要用 OR ， 那就需要返回记录最少的索引列写在最前面。


- 总是使用索引的第一个列

如果索引是建立在多个列上， 只有在它的第一个列（leading  column）被 where 子句引用时，优化器才会选择使用该索引。这也是一条简单而重要的规则，当仅引用索引的第二个列时，优化器使用了全表扫描而忽略了索引 。

- 用 UNION - ALL 替换 UNION ( 如果有可能的话)

UNION  ALL 将重复输出两个结果集合中相同记录，UNION 将对结果集合排序，这个操作会使用到 SORT_AREA_SIZE 这块内存. 对于这块内存的优化也是相当重要的。

- 用 WHERE 替代 ORDER  BY ： 
 
ORDER  BY 子句只在两种严格的条件下使用索引。
ORDER  BY 中所有的列必须包含在相同的索引中并保持在索引中的排列顺序。
ORDER  BY 中所有的列必须定义为非空。
WHERE 子句使用的索引和 ORDER  BY 子句中所使用的索引不能并列。


- 避免使用耗费资源的操作: 

带有 DISTINCT ， UNION ， MINUS ， INTERSECT ， ORDER  BY 的 SQL 语句会启动 SQL 引擎 
执行耗费资源的排序( SORT )功能. DISTINCT 需要一次排序操作， 而其他的至少需要执行两次排序。通常， 带有 UNION ， MINUS ， INTERSECT 的 SQL 语句都可以用其他方式重写，如果你的数据库的 SORT_AREA_SIZE 调配得好， 使用 UNION ， MINUS ， INTERSECT 也是可以考虑的， 毕竟它们的可读性很强。

（完）

![somenzz.png](https://upload-images.jianshu.io/upload_images/12989993-db09bca07c33df69.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


