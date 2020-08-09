

# SQLite 数据库基础



----------------------------



## SQLite 数据库简介  



SQLite 是一个开源的、 内嵌式的关系型数据库， 第一个版本诞生于 2000 年 5 月， 目前最高版本为 SQLite3。  



下载地址：  [https://www.sqlite.org/download.html](https://www.sqlite.org/download.html)



菜鸟教程 : [https://www.runoob.com/sqlite/sqlite-tutorial.html](https://www.runoob.com/sqlite/sqlite-tutorial.html)



Linux 下 字符界面

```bash
sudo apt-get install sqlite3
```



Linux 下 图形界面  

```bash
sudo apt-get install sqlitebrowser
```



该教程没有使用这个, 因为我下载时找不到

```bash
sudo apt-get install sqliteman
```



SQLite 特性：

*   零配置
*   灵活
*   可移植
*   自由的授权
*   紧凑
*   可靠
*   简单
*   易用  



-------------------------



## SQL 语句基础  



SQL 是一种结构化查询语言（Structured Query Language） 的缩写， SQL 是一种专门用来与数据库通信的语言。  



SQL 目前已成为应用最广的数据库语言。  



SQL 已经被众多商用数据库管理系统产品所采用， 不同的数据库管理系统在其实践过程中都对 SQL 规范作了某些编改和扩充。 故不同数据库管理系统之间的 SQL 语言不能完全相互通用。  



SQLite 数据类型  :



一般数据采用固定的静态数据类型， 而 SQLite 采用的是动态数据类型， 会根据存入值自动判断。  



SQLite 具有以下五种基本数据类型 ：  

*   `integer`： 带符号的整型（最多 64 位） 。
*   `real`： 8 字节表示的浮点类型。
*    `text`： 字符类型， 支持多种编码（如 UTF-8、 UTF-16） ， 大小无限制。
*   `blob`： 任意类型的数据， 大小无限制。
*   `BLOB`(binary large object)二进制大对象， 使用二进制保存数据  
*   `null`： 表示空值  



对数据库文件 SQL 语句：  



创建、 打开数据库：  



当*.db 文件不存在时， sqlite 会创建并打开数据库文件。  

当*.db 文件存在时， sqlite 会打开数据库文件。  

```sqlite
sqlite3 test.db
```

![image-20200809124512574](https://gitee.com/cpu_code/picture_bed/raw/master//20200809124512.png)









SQL 的语句格式：  



所有的 SQL 语句都是以分号结尾的， SQL 语句不区分大小写。 两个减号“--” 则代表注释。  



关系数据库的核心操作：  

*   创建、 修改、 删除表  
*   添加、 修改、 删除行  
*   查表  



----------------------------



### 创建表： create 语句  



语法：   

```sqlite
create table 表名称 (列名称1 数据类型, 列名称2 数据类型, 列名称3 数据类型, ...);
```



创建一表格该表包含 3 列， 列名分别是： “id” 、 “name” 、 “addr” 。  

```sqlite
create table cpucode (id integer, name text, addr text);
```



![image-20200809125437496](https://gitee.com/cpu_code/picture_bed/raw/master//20200809125437.png)



-----------------------------



### 创建表： create 语句 (设置主键)  



在用 sqlite 设计表时， 每个表都可以通过 `primary key` 手动设置**主键,** **每个表只能有一个主键**， 设置为主键的列数据不可以重复。  



语法： 

```sqlite
create table 表名称 ( 列名称1 数据类型 primary key, 列名称2 数据类型,列名称3 数据类型, ...);
```



```sqlite
create table test (id integer primary key, name text, addr text);
```



![image-20200809124801066](https://gitee.com/cpu_code/picture_bed/raw/master//20200809124801.png)



--------------



### 查看表： .table  



查看数据表的结构： 

```sqlite
.schema[表名]
```



```sqlite
.tables
```



![image-20200809124825297](https://gitee.com/cpu_code/picture_bed/raw/master//20200809124825.png)



```sqlite
.schema
```



![image-20200809125503551](https://gitee.com/cpu_code/picture_bed/raw/master//20200809125503.png)



--------------------------



### 退出数据库命令

```sqlite
.quit
```



```sqlite
.exit
```



![image-20200809125554172](https://gitee.com/cpu_code/picture_bed/raw/master//20200809125554.png)





图形化的软件查看表的结构  :



```sqlite
sqlitebrowser test.db
```



![image-20200809125700156](https://gitee.com/cpu_code/picture_bed/raw/master//20200809125700.png)



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200809125806.png" alt="image-20200809125806604" style="zoom:80%;" />







-------------------------------



### 修改表： alter 语句  



在已有的表中**添加**或**删除列**以及**修改表名**。(添加、 删除-sqlite3 暂不支持、 重命名)  



语法  :



```sqlite
alter table 表名 add 列名 数据类型;
```



```sqlite
alter table cpucode add sex text;
```



![image-20200809130120229](https://gitee.com/cpu_code/picture_bed/raw/master//20200809130120.png)





语法： （alter 修改表名）  



```sqlite
alter table 表名 rename to 新表名;
```



```sqlite
.tables
```



```sqlite
alter table cpucode rename to new_cpucode;
```



```sqlite
.tables
```



![image-20200809130242617](https://gitee.com/cpu_code/picture_bed/raw/master//20200809130242.png)



-----------------------



### 删除表： drop table 语句  



用于删除表（表的结构、 属性以及表的索引也会被删除）  



语法：  



```sqlite
drop table 表名称;
```



```sqlite
drop table new_cpucode;
```



![image-20200809131750952](https://gitee.com/cpu_code/picture_bed/raw/master//20200809131751.png)





-------------------------------



### 插入新行： insert into 语句(全部赋值)  



给一行中的所有列赋值。  



当列值为字符串时要加上`‘ ’` 号。  



语法：  

```sqlite
insert into 表名 values (列值 1, 列值 2, 列值 3， 列值 4, ...);
```



```sqlite
create table cpucode (id integer, name text, addr text);
```



![image-20200809132249261](https://gitee.com/cpu_code/picture_bed/raw/master//20200809132249.png)



```sqlite
insert into cpucode values (1, 'code', 'changsha');
```



![image-20200809132551917](https://gitee.com/cpu_code/picture_bed/raw/master//20200809132551.png)



```bash
sqlitebrowser test.db
```



![image-20200809132524349](https://gitee.com/cpu_code/picture_bed/raw/master//20200809132524.png)



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200809132446.png" alt="image-20200809132446540" style="zoom:67%;" />









----------------------------------



### 插入新行： insert into 语句部分赋值)  



给一行中的部分列赋值  



语法： 



```sqlite
insert into 表名 (列名 1, 列名 2, ...) values (列值 1, 列值 2, ...);
```



```sqlite
insert into cpucode (id, name) values (1, 'cpu');
```



![image-20200809133320623](https://gitee.com/cpu_code/picture_bed/raw/master//20200809133320.png)



```bash
sqlitebrowser test.db
```



![image-20200809133413766](https://gitee.com/cpu_code/picture_bed/raw/master//20200809133413.png)



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200809133347.png" alt="image-20200809133347328" style="zoom:67%;" />







--------------------------



### 修改表中的数据： update 语句  



使用 where 根据匹配条件， 查找一行或多行， 根据查找的结果修改表中相应行的列值(修改哪一列由列名指定)。  



语法：  

```sqlite
update 表名 set 列 1 = 值1 [, 列2 = 值2, ...] [匹配条件];
```



匹配： where 子句

where 子句用于规定匹配的条件。  



| 操作数 |   描述   |
| :----: | :------: |
|   =    |   等于   |
|   <>   |  不等于  |
|   >    |   大于   |
|   <    |   小于   |
|   >=   | 大于等于 |
|   <=   | 小于等于 |



匹配条件语法： 

```sqlite
where 列名 操作符 列值
```



```sqlite
update cpucode set id=2, addr='shenzhen' where name='cpu';
```



![image-20200809140409888](https://gitee.com/cpu_code/picture_bed/raw/master//20200809140409.png)



```bash
sqlitebrowser test.db
```



![image-20200809140353950](https://gitee.com/cpu_code/picture_bed/raw/master//20200809140354.png)



当表中有多列、 多行符合匹配条件时会修改相应的多行。 当匹配条件为空时则匹配所有。  



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200809140325.png" alt="image-20200809140325506" style="zoom:67%;" />





当表中有多列、 多行符合匹配条件时会修改相应的多行  :



```sqlite
update cpucode set addr='' where name='';
```







当匹配条件为空时则匹配所有  :



```sqlite
update cpucode set addr='';
```







-------------------------



### 删除表中的数据： delete 语句  



使用 where 根据匹配条件， 查找一行或多行， 根据查找的结果删除表中的查找到的行。  

当表中有多列、 多行符合匹配条件时会删除相应的多行。  



语法：  

```sqlite
delete from 表名 [匹配条件];
```



```bash
delete from cpucode where name='cpu';
```





















-------------------------



### 查询： select 语句  



用于从表中选取数据， 结果被存储在一个结果表中（称为结果集） 。  



星号（*） 是选取所有列的通配符  



语法：  



```sqlite
 select * from 表名 [匹配条件];
```



```sqlite
select 列名 1[, 列名 2, ...] from 表名 [匹配条件];
```



```sqlite
insert into cpucode values (1, '');
```



```sqlite
select * from cpucode
```























---



### 匹配条件语法



数据库提供了丰富的操作符配合 where 子句实现了多种多样的匹配方法。  



`in` 允许我们在 where 子句中规定多个值。  



匹配条件语法：  

```sqlite
where 列名 in (列值 1, 列值 2, ...)
```



```sqlite
select * from 表名 where 列名 in (值 1, 值 2, ...);
```



```sqlite
select 列名 1[,列名 2,...] from 表名 where 列名 in (列值 1, 列值 2, ...);
```





`and` 可在 where 子语句中把两个或多个条件结合起来（多个条件之间是与的关系） 。  





















----------------------



### 事务  







