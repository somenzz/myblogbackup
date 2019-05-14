---
layout: post
title: "我是一个索引"
date: 2019-04-26 23:00:57
comments: true
reward: true
tags: 
	- db2
	- 索引
---

在关系型数据库中，我是一名索引 (Index)。

大家都知道，通常情况下我都会带来查询性能的提高。

需要指出的是，我并不是多多益善。

我类似于一本书的目录，只不过书的内容是静态的，而数据是动态变化的。可以想像，如果书中的内容页频繁变化，那么更新书的目录也会花掉不少成本。所以说，我不是多多益善。

我是数据库中一个排序的数据结构，以协助快速查询、更新数据库表中数据。如果深入了解我，并加以合理的运用，可以对查询性能有成百上千倍的提高。

今天，你需要知道，哪些 SQL 中的条件有可能走索引，哪些一定不会走索引，建索引时时如何确定字段的顺序？

<!-- more -->

### 匹配索引扫描
比如有这样的一张表 CUSTOMER ，表的定义如下：
```sql
 CREATE TABLE CUSTOMER ( 
  C_CUSTOMER_ID  CHAR(16) NOT NULL, 
  C_FIRST_NAME  CHAR(20) NOT NULL, 
  C_LAST_NAME  CHAR(30) NOT NULL, 
  C_BIRTH_DAY  INTEGER NOT NULL, 
  C_BIRTH_MONTH  INTEGER NOT NULL, 
  C_BIRTH_YEAR  INTEGER NOT NULL, 
  …
  )
```

假设 CUSTOMER 表上只有如下的索引：

```sql
 CREATE INDEX CUSTOMER_IDX_01 ON CUSTOMER 
  (C_FIRST_NAME ASC)

```
那么语句

```sql
SELECT * FROM CUSTOMER WHERE C_FIRST_NAME ='MARIA'
```
谓词 C_FIRST_NAME ='MARIA'中指定了 C_FIRST_NAME 列的值，所以 DB2 可以利用索引 CUSTOMER_IDX_01 直接定位到叶节点，再访问表的对应的数据页。
![1](https://www.ibm.com/developerworks/cn/data/library/techarticle/dm-1106qinw/image004.jpg)

这种方式叫**匹配索引扫描**。



### 非匹配索引扫描
再看另外一个 SQL 语句：

```sql
 SELECT * 
 FROM DB2ADMIN.CUSTOMER 
 WHERE C_FIRST_NAME = 'MARIA' 
		 AND C_BIRTH_YEAR = 1977
```
假设 CUSTOMER 表上只有如下的索引 CUSTOMER_IDX_02：

```sql
 CREATE INDEX CUSTOMER_IDX_02 ON CUSTOMER"
  (C_FIRST_NAME ASC,   
   C_LAST_NAME ASC, 
   C_BIRTH_YEAR ASC)
```
DB2 访问索引的时候会做如下步骤：

先利用索引的第一个键 C_FIRST_NAME 限定范围 C_FIRST_NAME = 'MARIA'，
对于另一个谓词 C_BIRTH_YEAR = 1977，由于 C_BIRTH_YEAR 是索引第三个键，所以 DB2 无法根据它直接找到对应的索引叶节点，而只能从满足条件 C_FIRST_NAME='MARIA' 的全部索引叶节点中扫描选取满足 C_BIRTH_YEAR = 1977 的叶节点。


![2](https://www.ibm.com/developerworks/cn/data/library/techarticle/dm-1106qinw/image006.jpg)


这种方式叫**非匹配索引扫描**。

### 只做索引扫描

再看另外一个例子，给定如下 SQL 语句：
```sql
 SELECT C_BIRTH_YEAR 
 FROM CUSTOMER 
 WHERE C_BIRTH_YEAR>1977
```
并且假设 CUSTOMER 表上只有索引 CUSTOMER_IDX_03：
```sql
 CREATE INDEX CUSTOMER_IDX_03 ON CUSTOMER 
  (C_BIRTH_YEAR ASC)
```
可以看出查询返回的数据恰恰就是 CUSTOMER_IDX_03 中的索引键，此时 DB2 不用访问磁盘上表的数据页，只需要扫描索引就可以得到对应列的值。

![3](https://www.ibm.com/developerworks/cn/data/library/techarticle/dm-1106qinw/image008.jpg)


这种访问方式就是**只做索引扫描**。


### 建索引时如何确定顺序？

如果where 条件（谓词）中全部是 =，那么对此索引的访问可以一直进行索引匹配访问；但是当其中包含了 <、 >、LIKE 这种范围操作谓词时，只有第一个范围操作谓词可以进行匹配索引扫描，之后所有的谓词，即使是 = 的谓词，也只能进行非匹配索引扫描。我们称这种谓词为停止匹配谓词。显而易见，我们希望进行更多的索引匹配访问操作，因此要把所有停止匹配的谓词放在索引的最后面。


比如对于如下 SQL 语句：
```sql
 Select C_COMMENT 
 From CUSTOMER 
 Where C_ACCTBAL > 10000 
	 AND UCASE(C_NAME)= ’ IBM ’
	 AND C_CUSTKEY < > C_NATIONKEY 
	 AND C_MKTSEGMENT = ‘ CHINA 
	 AND C_PHONE LIKE ‘ 135010% ’
	 AND C_ADDRESS= ’ BEIJING ’
```
在这条查询里面，在 C_ACCTBAL、C_CUSTKEY、C_PHONE 上的 3 个谓词均为范围操作的谓词，也就是说它们是都是停止匹配谓词，我们在设计时要把他们放在索引的最后面。对这条查询，一种可能的索引设计为 (C_MKTSEGMENT, C_ADDRESS, C_ACCTBAL)。

上述 where 条件中 UCASE(C_NAME)= ’ IBM ’ 一定不会走索引。
C_CUSTKEY < > C_NATIONKEY  一定不会走索引。
其他的，则可能走索引。不会走索引的字段没有必须建索引，可能走索引的字段我们可以建索引，在实际编写SQL时，尽量少用不走索引的谓词。

对于查询
```sql
 Select C_COMMENT 
 From CUSTOMER 
 Where C_ACCTBAL > 10000 
 AND C_PHONE LIKE ‘ 135010% ’
 AND C_ADDRESS= ’ BEIJING ’
```
可以考虑建立的索引为 (C_ADDRESS, C_ACCTBAL, C_PHONE)。注意一定要把 C_ADDRESS 放在索引的第一位，这样 DB2 才能在这个键上进行匹配访问操作，C_ACCTBAL 即为停止匹配的谓词，在其之后的 C_PHONE 只能使用非匹配访问的访问方式。



还有一种方法，假如有下语句

```sql
 Select C_COMMENT 
 From CUSTOMER 
 Where C_NAME= ’ IBM ’
    AND C_MKTSEGMENT = 'CHINA'
	 AND C_ADDRESS= 'BEIJING'
```
这时要估算单位使用各条件时哪个条件查询的结果较少，较少的键放前面，保证首次索引取得的数据量最少。

假如单独限制 
C_MKTSEGMENT = 'CHINA' 得到 1000 条，单独限制 C_NAME= 'IBM' 得到 10000 条，单独限制 C_ADDRESS= 'BEIJING' 得 8000 条，那么一种可能的索引设计为 (C_MKTSEGMENT, ADDRESS,C_NAME)。

### 如何判断这个谓词是否走索引

如果一个谓词为假，那么整个 where 条件的值都为假，那么这个谓词对 where 条件相当于一个开关的作用，这种谓词叫作布尔项（Boolean-term）。对于数据库表中被处理的每一条数据记录（Row），一旦该数据记录不满足 Boolean-term 的判断条件，那么这条数据记录就被认为是不满足整个 WHERE 子句的判断条件。下面通过具体的 SQL 例子来说明这个概念。

在下面这条 SQL 语句中：

```sql
SELECT * FROM EMP 
 WHERE WORKDEPT = 'A00'
 AND (SALARY > 40000 OR BONUS > 800)
```
可以看到三个基本谓词：WORKDEPT = 'A00'，SALARY > 40000 以及 BONUS > 800。其中只有 WORKDEPT = 'A00' 满足 Boolean-term 的定义，也就是说对于 EMP 表中的每一条数据记录，如果它不满足 WORKDEPT = 'A00'的条件，那么它就不满足整个 WHERE 子句的条件，从而也就不会被作为这条 SQL 查询的结果被返回。而对于谓词 SALARY > 40000 来说，即使某条数据记录不满足这个条件，该记录也有可能因为满足 BONUS > 800 和 WORKDEPT = 'A00 而被作为查询结果返回，所以谓词 SALARY > 40000 不是 Boolean-term。同理，谓词 BONUS > 800 也不是 Boolean-term。通过这样的例子可以看出，在 SQL 形式上，Boolean-term 是不能出现在用 OR 连接的谓词里面，它必须是用 AND 与其他的谓词相连接。

在上面这个例子中，因为 SALARY > 40000 和 BONUS > 800 都不是 Boolean-term，所以即使存在某个索引包括 SALARY 列或者 BONUS 列，DB2 也不会选择这个索引来进行索引匹配扫描（Matching index scan）。正因如此，在设计新索引的时候，我们也就无需去考虑这种不是 Boolean-term 的谓词。


分析 SQL 语句最基本的一步，就是在 WHERE 子句的所有 Boolean-term 中找到所有的可以使用索引的谓词（ Indexable predicates），并根据其中引用到的列来设计索引键（Index Key）。从逻辑上来说，按照这种谓词中给定的条件，DB2 数据库可以用索引访问的方式来在索引树中快速找到一个或多个相匹配的记录。需要注意的是，可以使用索引的谓词 这个概念关注的是谓词本身的写法使得通过索引来访问数据成为可能，而它并不能保证在数据库中合适的索引是存在的，也不能保证 DB2 数据库在运行时一定会通过索引访问的方式来筛选满足这个谓词条件的数据；但是反过来，如果一个谓词不是 Indexable 的形式，那么数据库则肯定不能通过索引来筛选满足条件的数据。换而言之，“谓词是 Indexable的形式”是“数据库能使用索引访问来筛选数据”的必要非充分条件。那么什么样的谓词是“可以使用索引的谓词”？表 1 列出了每一种谓词的形式，并标明了它是否属于 Indexable predicate。


|谓词类型|Indexable|
| -- | -- |
|value BETWEEN COL1 AND COL2|N|
|COL BETWEEN COL1 AND COL2|N|
|COL <> value|N|
|COL <> noncol expr|N|
|COL NOT BETWEEN value1 AND value2|N|
|COL NOT BETWEEN noncol expr1 AND noncol expr2|N|
|value NOT BETWEEN COL1 AND COL2|N|
|COL NOT IN (list)|N|
|COL NOT LIKE ' char'|N|
|COL LIKE '%char'|N|
|COL LIKE '_char'|N|
|T1.COL <> T2 col expr|N|
|T1.COL1 = T1.COL2|N|
|T1.COL1 op T1.COL2|N|
|(COL1,...COLn) IN (cor subq)|N|
|COL NOT IN (cor subq)|N|
|(COL1,...COLn) NOT IN (cor subq)|N|
|COL IS DISTINCT FROM value|N|
|COL IS DISTINCT FROM noncol expr|N|
|T1.COL1 IS DISTINCT FROM T2.COL2|N|
|T1.COL1 IS NOT DISTINCT FROM T2.COL2|N|
|T1.COL1 IS DISTINCT FROM T2 col expr|N|
|COL IS DISTINCT FROM (noncor subq)|N|
|COL IS DISTINCT FROM ANY (noncor subq)|N|
|COL IS NOT DISTINCT FROM ANY (noncor subq)|N|
|COL IS DISTINCT FROM ALL (noncor subq)|N|
|COL IS NOT DISTINCT FROM ALL (noncor subq)|N|
|COL IS NOT DISTINCT FROM (cor subq)|N|
|COL IS DISTINCT FROM ANY (cor subq)|N|
|COL IS DISTINCT FROM ANY (cor subq)|N|
|COL IS NOT DISTINCT FROM ANY (cor subq)|N|
|COL IS DISTINCT FROM ALL (cor subq)|N|
|COL IS NOT DISTINCT FROM ALL (cor subq)|N|
|EXISTS (subq)|N|
|NOT EXISTS (subq)|N|
|expression = value|N|
|expression <> value|N|
|COL <> (noncor subq)|N|
|COL <> ANY (noncor subq)|N|
|COL <> ALL (noncor subq)|N|
|COL NOT IN (noncor subq)|N|
|(COL1,...COLn) NOT IN (noncor subq)|N|
|COL = (cor subq)|N|
|COL = ALL (cor subq)|N|
|COL op (cor subq)|N|
|COL op ANY (cor subq)|N|
|COL op ALL (cor subq)|N|
|COL <> (cor subq)|N|
|COL <> ANY (cor subq)|N|
|COL <> ALL (cor subq)|N|
|NOT XMLEXISTS|N|
|COL = ALL (noncor subq)|N|
|expression op (subq)|N|
|expression op value|N|
|T1.COL1 <> T1.COL2|N|
|COL = value|Y|
|COL = noncol expr|Y|
|COL IS NULL|Y|
|COL op value|Y|
|COL op noncol expr|Y|
|COL BETWEEN value1 AND value2|Y|
|COL BETWEEN noncol expr1 AND noncol expr2|Y|
|COL BETWEEN expression1 AND expression2|Y|
|COL LIKE 'pattern'|Y|
|COL IN (list)|Y|
|COL IS NOT NULL|Y|
|COL LIKE host variable|Y|
|T1.COL = T2 col expr|Y|
|T1.COL op T2 col expr|Y|
|COL IN (cor subq)|Y|
|COL IS NOT DISTINCT FROM value|Y|
|COL IS NOT DISTINCT FROM noncol expr|Y|
|T1.COL1 IS NOT DISTINCT FROM T2 col expr|Y|
|COL IS NOT DISTINCT FROM (noncor subq)|Y|
|COL op (noncor subq)|Y|
|COL op ANY (noncor subq)|Y|
|COL op ALL (noncor subq)|Y|
|COL IN (noncor subq)|Y|
|(COL1,...COLn) IN (noncor subq)|Y|
|COL = ANY (cor subq)|Y|
|XMLEXISTS|Y|
|COL = ANY (noncor subq)|Y|
|COL=(noncor subq)|Y|



在实际使用中，请尽可能使用 Indexable 为 Y 的谓词作查询。

在分析得到 SQL 语句里所有 Boolean-term 中可以使用索引的谓词后，就可以根据这些谓词中的列来设计索引了。以下面 SQL 语句为例：

```sql
 SELECT C_COMMENT 
 FROM CUSTOMER 
 WHERE C_ACCTBAL > 10000 
  AND UCASE(C_NAME)= ’ IBM ’
	 AND C_CUSTKEY < > C_NATIONKEY 
	 AND C_MKTSEGMENT = ‘ CHINA 
	 AND C_PHONE LIKE ‘ 135010% ’
	 AND C_ADDRESS= ’ BEIJING ’
```
对照上表可以知道，C_ACCTBAL，C_MKTSEGMENT，C_PHONE 和 C_ADDRESS 都是合适的索引列的候选者，如果要设计单键索引（Single-key Index），它们任意一个都可以构成索引；如果要设计多键索引（Multiple-keys Index）, 它们之间的前后顺序是下一个需要考虑的问题，详细讨论见后文的“索引键顺序的选择”。而相对应的，C_NAME，C_CUSTKEY 和 C_NATIONKEY 则不是合格的索引列候选者。



### 如果还慢

1、请把 SELECT 从句中的所有列也加上索引，使查询成为只使用索引的访问方式。

2、把 GROUP BY 和 ORDER BY 从句中的所有列加上，可以减少访问计划中的排序操作。

3、把表关联上的键也加索引，但要注意加在哪个表上很重要。假如 A LEFT JOIN B, 那么请确保 A 的数据量比 B 的大，如果A、B数据量差不多，尽量在 B 表的建索引。比如：

嵌套循环连接过程的伪代码示意如下：
```sql
 For each i in 外表 : 
	 For each j in 内表 : 
		如果 (i,j) 满足约束条件
  将（i,j）放入结果集
```
从中可以看到，外表只需要做一次完整的全表扫描，索引对这种访问是不起作用的；而内表需要被多次扫描，并且每次扫描都是利用连接谓词进行一次查询操作，对于此种访问方式，在内表相关的列上面建立索引就是相当有必要的了。



### 我的另一面
建立索引会降低更新（update）， 插入（insert）， 删除（delete）表中数据的速度。因为此时 DB2 需要同时更新表上的索引，若同一张表上有多个索引，情况会更糟。

### 索引不被采用的原因分析

在查看了访问计划之后可能会发现，我们设计出来的索引并没有被优化器所选中。造成这种结果的原因可能是多方面的，常见情况可能是如下几种：

最通常的情况，是设计的索引存在一些问题，比如没有考虑清楚最优的表连接顺序，或者是索引中有 stop-matching 的键存在。假如确定是此种原因，那么就需要返回前几步重新设计索引。

另一种可能是数据库中的统计信息不对，甚至是根本不存在的。在这种情况下，DB2 往往无法选出最优的访问计划，因此有可能设计的索引并不会被使用。这种情况一般重新执行 DB2 RUNSTATS 命令即可解决。
此外，如果 DB2 判断出需要从表中读取的数据的比例很高（比如有超过 90% 表里面的记录需要被返回），那么 DB2 很有可能选择全表扫描来代替使用索引，因为这样能够减少一次对索引树的读取。假如是此种情况，在表上建立索引实际上并不能提高性能。



（完）

欢迎订阅个人公众号 somenzz ，专注有价值的技术分享。

