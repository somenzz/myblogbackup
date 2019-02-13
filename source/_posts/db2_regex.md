---
layout: post
title: "给 db2 添加正则表达式函数"
date: 2018-11-13 23:00:57
comments: true
reward: true
tags: 
	- db2
	- 正则表达式
---

正则表达式实在太强大了，理论上它可以将任何字符串变成你想要的结果，使用方法可参考上一篇文章[学会正则表达式，玩弄文本于股掌之中](https://somenzz.github.io/2018/11/09/Regular-expression-of-work-essential-technology/)。
<!-- more -->

相信有不少朋友是喜欢使用正则表达式来解决问题的，像一些主流的数据库 mysql 、oracle 是原生支持正则表式的。如 mysql 中 查找 name 字段中以元音字符开头或以 'ok' 字符串结尾的所有数据：
```sql
mysql> SELECT name FROM person_tbl WHERE name REGEXP '^[aeiou]|ok$';
```
如 oracle 10g 提供的四个正则表达式函数
- REGEXP_LIKE(srcstr, pattern [, match_option]) ：比较一个字符串是否与正则表达式匹配。
- REGEXP_INSTR(srcstr, pattern [, position [, occurrence [, return_option [, match_option]]]])：在字符串中查找正则表达式，并且返回匹配的位置。
- REGEXP_SUBSTR (srcstr, pattern [, position [, occurrence [, match_option]]])：(提取) 返回与正则表达式匹配的子字符串 。
- REGEXP_REPLACE(srcstr, pattern [, replacestr [, position [, occurrence [, match_option]]]])：(替换)搜索并且替换匹配的正则表达式。

在实际应用有不少应用的数据库是 db2 数据库，据我所知 db2 并未自带正则表达式函数，需要我们动手去添加，官方已经给出了两种解决方案：
一类是 java 实现的正则表达式函数 [https://www.ibm.com/developerworks/cn/data/library/techarticle/dm-1011db2luwpatternmatch/](https://www.ibm.com/developerworks/cn/data/library/techarticle/dm-1011db2luwpatternmatch/)。

一类是 C 实现的正则表达式函数 [https://www.ibm.com/developerworks/cn/data/library/techarticles/0301stolze/0301stolze.html](https://www.ibm.com/developerworks/cn/data/library/techarticles/0301stolze/0301stolze.html)

个人比较了以上两个方法，JAVA 版的提供了 4 个函数，同 oracle 那 4 个函数，而 C 版的只有两个函数 ，一个是判断字段否匹配正则表达式的，一个是生将匹配结果生成表的，感觉 java 版的更实用一些，如果认为 C 更快的，可以深入研究一下，改写源码以满足个性需求，也是可以的。

官方文档比较长，如果了解相关细节可以看下，如果只想快速安装正则表达式函数可参考下面快速安装步骤：
1. 下载官网提供的 db2-regex.zip [https://www.ibm.com/developerworks/apps/download/index.jsp?contentid=630922&filename=db2-regex.zip&method=http&locale=zh_CN](https://www.ibm.com/developerworks/apps/download/index.jsp?contentid=630922&filename=db2-regex.zip&method=http&locale=zh_CN) 并解压至一个位置，假如为 /home/xx/db2-regex 目录下。
2. 修改 /home/xx/db2-regex/scripts/sql/db2_regex_functions.sql 文件，修改 
```sql
CALL SQLJ.INSTALL_JAR('file:C:\Tivoli\db2_regex\lib\db2_regex.jar', db2_regex);
```
为
```sql
CALL SQLJ.INSTALL_JAR('file:/home/xx/db2-regex/lib/db2_regex.jar', db2_regex);
```
3. 在 db2 命令窗口执行：
```sh
     \>db2 connect to <my_db> user <uid> using <pwd>
     \>db2 set current schema='REGEXP'
     \>db2 -td@ -vf /home/xx/db2-regex/scripts/sql/db2_regex_functions.sql
```
至此，你的数据库上已经有 4 个函数了：
```sql
INTEGER REGEXP_LIKE(SOURCE VARCHAR(3000), REGEX VARCHAR(512), MODE VARCHAR(3))

VARCHAR(3000) REGEXP_REPLACE(SOURCE VARCHAR(3000), 
                             REGEX VARCHAR(512), 
                             REPLACEMENT VARCHAR(3000), 
                             POSITION INTEGER,
                             OCCURRENCE INTEGER,
                             MODES VARCHAR(3))

VARCHAR(3000) REGEXP_SUBSTR(SOURCE VARCHAR(3000),
                             REGEX VARCHAR(512),
                             POSITION INTEGER,
                             OCCURRENCE INTEGER,
                             MODES VARCHAR(3))

INTEGER REGEXP_INSTR(SOURCE VARCHAR(3000),
                     REGEX VARCHAR(512),
                     POSITION INTEGER,
                     OCCURRENCE INTEGER,
                     ROPT INTEGER,
                     MODES VARCHAR(3))
```
如果你修改改了源 java 代码，想重新安装，那么，先编译生成 jar 文件并放在相应位置，然后在命令行执行：
```sh
db2 connect t to <my_db> user <uid> using <pwd>
db2 set current schema='REGEXP'
db2 drop function REGEXP_LIKE
db2 drop function REGEXP_REPLACE
db2 drop function REGEXP_SUBSTR
db2 drop function REGEXP_INSTR
db2 call SQLJ.REMOVE_JAR(db2_regex)
db2stop force
db2start
db2 connect to <my_db> user <uid> using <pwd>
db2 set current schema='REGEXP'
db2 -td@ -f /home/xx/db2-regex/scripts/sql/db2_regex_functions.sql
```
即可。

4个函数的使用方法如下：
```sql
select ID from REGEXP.REGEXP_STRINGS where REGEXP_LIKE(STRING, '^.EF[ ]+SAVEALIAS[ ]+[0-9]+', 'c') > 0

select ID from REGEXP.REGEXP_STRINGS where REGEXP_REPLACE(STRING, '^.EF[]+SAVEALIAS[ ]+[0-9]+', 'XX', 1, 1, 'c')='XX'

select ID from REGEXP.REGEXP_STRINGS where REGEXP_SUBSTR(STRING, '^.EF[ ]+SAVEALIAS[ ]+[0-9]+', 1, 1, 'c')='DEF SAVEALIAS 2210'

select ID from REGEXP.REGEXP_STRINGS where REGEXP_INSTR(STRING, '^.EF[ ]+SAVEALIAS[ ]+[0-9]+',1, 1, 1, 'c') > 0
```
至此可以愉快地使用正则表达式了。需要注意地是，如果处理大量数据，为了防止查询过慢最好不好直接使用正则表达式函数，因为这样会失去索引的价值，最好是先使用 where 条件过滤掉一部分数据，然后再使用正则表达式处理过滤后的数据，关于如何写出更快的 SQL 请参考我的历史文章 [如何写出更快的 SQL (db2)](https://somenzz.github.io/2018/10/13/write_fast_sql/)

（完）

![somenzz的公众号](https://upload-images.jianshu.io/upload_images/12989993-bef2bc509f59955d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


