# SQL基础

## SQLite 数据库简介

SQLite 是一个开源的、 内嵌式的关系型数据库， 第一个版本诞生于 2000 年 5 月， 目前最高版本为 SQLite3。

下载地址： [https://www.sqlite.org/download.html](https://www.sqlite.org/download.html)

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

* 零配置
* 灵活
* 可移植
* 自由的授权
* 紧凑
* 可靠
* 简单
* 易用  

## SQL 语句基础

SQL 是一种结构化查询语言（Structured Query Language） 的缩写， SQL 是一种专门用来与数据库通信的语言。

SQL 目前已成为应用最广的数据库语言。

SQL 已经被众多商用数据库管理系统产品所采用， 不同的数据库管理系统在其实践过程中都对 SQL 规范作了某些编改和扩充。 故不同数据库管理系统之间的 SQL 语言不能完全相互通用。

SQLite 数据类型 :

一般数据采用固定的静态数据类型， 而 SQLite 采用的是动态数据类型， 会根据存入值自动判断。

SQLite 具有以下五种基本数据类型 ：

* `integer`： 带符号的整型（最多 64 位） 。
* `real`： 8 字节表示的浮点类型。
* `text`： 字符类型， 支持多种编码（如 UTF-8、 UTF-16） ， 大小无限制。
* `blob`： 任意类型的数据， 大小无限制。
* `BLOB`\(binary large object\)二进制大对象， 使用二进制保存数据  
* `null`： 表示空值  

对数据库文件 SQL 语句：

创建、 打开数据库：

当\*.db 文件不存在时， sqlite 会创建并打开数据库文件。

当\*.db 文件存在时， sqlite 会打开数据库文件。

```text
sqlite3 test.db
```

![image-20200809124512574](https://gitee.com/cpu_code/picture_bed/raw/master//20200809124512.png)

SQL 的语句格式：

所有的 SQL 语句都是以分号结尾的， SQL 语句不区分大小写。 两个减号“--” 则代表注释。

关系数据库的核心操作：

* 创建、 修改、 删除表  
* 添加、 修改、 删除行  
* 查表  

### 创建表： create 语句

语法：

```text
create table 表名称 (列名称1 数据类型, 列名称2 数据类型, 列名称3 数据类型, ...);
```

创建一表格该表包含 3 列， 列名分别是： “id” 、 “name” 、 “addr” 。

```text
create table cpucode (id integer, name text, addr text);
```

![image-20200809125437496](https://gitee.com/cpu_code/picture_bed/raw/master//20200809125437.png)

### 创建表： create 语句 \(设置主键\)

在用 sqlite 设计表时， 每个表都可以通过 `primary key` 手动设置**主键,** **每个表只能有一个主键**， 设置为主键的列数据不可以重复。

语法：

```text
create table 表名称 ( 列名称1 数据类型 primary key, 列名称2 数据类型,列名称3 数据类型, ...);
```

```text
create table test (id integer primary key, name text, addr text);
```

![image-20200809124801066](https://gitee.com/cpu_code/picture_bed/raw/master//20200809124801.png)

### 查看表： .table

查看数据表的结构：

```text
.schema[表名]
```

```text
.tables
```

![image-20200809124825297](https://gitee.com/cpu_code/picture_bed/raw/master//20200809124825.png)

```text
.schema
```

![image-20200809125503551](https://gitee.com/cpu_code/picture_bed/raw/master//20200809125503.png)

### 退出数据库命令

```text
.quit
```

```text
.exit
```

![image-20200809125554172](https://gitee.com/cpu_code/picture_bed/raw/master//20200809125554.png)

图形化的软件查看表的结构 :

```text
sqlitebrowser test.db
```

![image-20200809125700156](https://gitee.com/cpu_code/picture_bed/raw/master//20200809125700.png)

![image-20200809125806604](https://gitee.com/cpu_code/picture_bed/raw/master//20200809125806.png)

### 修改表： alter 语句

在已有的表中**添加**或**删除列**以及**修改表名**。\(添加、 删除-sqlite3 暂不支持、 重命名\)

语法 :

```text
alter table 表名 add 列名 数据类型;
```

```text
alter table cpucode add sex text;
```

![image-20200809130120229](https://gitee.com/cpu_code/picture_bed/raw/master//20200809130120.png)

语法： （alter 修改表名）

```text
alter table 表名 rename to 新表名;
```

```text
.tables
```

```text
alter table cpucode rename to new_cpucode;
```

```text
.tables
```

![image-20200809130242617](https://gitee.com/cpu_code/picture_bed/raw/master//20200809130242.png)

### 删除表： drop table 语句

用于删除表（表的结构、 属性以及表的索引也会被删除）

语法：

```text
drop table 表名称;
```

```text
drop table new_cpucode;
```

![image-20200809131750952](https://gitee.com/cpu_code/picture_bed/raw/master//20200809131751.png)

### 插入新行： insert into 语句\(全部赋值\)

给一行中的所有列赋值。

当列值为字符串时要加上`‘ ’` 号。

语法：

```text
insert into 表名 values (列值 1, 列值 2, 列值 3， 列值 4, ...);
```

```text
create table cpucode (id integer, name text, addr text);
```

![image-20200809132249261](https://gitee.com/cpu_code/picture_bed/raw/master//20200809132249.png)

```text
insert into cpucode values (1, 'code', 'changsha');
```

![image-20200809132551917](https://gitee.com/cpu_code/picture_bed/raw/master//20200809132551.png)

```bash
sqlitebrowser test.db
```

![image-20200809132524349](https://gitee.com/cpu_code/picture_bed/raw/master//20200809132524.png)

![image-20200809132446540](https://gitee.com/cpu_code/picture_bed/raw/master//20200809132446.png)

### 插入新行： insert into 语句部分赋值\)

给一行中的部分列赋值

语法：

```text
insert into 表名 (列名 1, 列名 2, ...) values (列值 1, 列值 2, ...);
```

```text
insert into cpucode (id, name) values (1, 'cpu');
```

![image-20200809133320623](https://gitee.com/cpu_code/picture_bed/raw/master//20200809133320.png)

```bash
sqlitebrowser test.db
```

![image-20200809133413766](https://gitee.com/cpu_code/picture_bed/raw/master//20200809133413.png)

![image-20200809133347328](https://gitee.com/cpu_code/picture_bed/raw/master//20200809133347.png)

### 修改表中的数据： update 语句

使用 where 根据匹配条件， 查找一行或多行， 根据查找的结果修改表中相应行的列值\(修改哪一列由列名指定\)。

语法：

```text
update 表名 set 列 1 = 值1 [, 列2 = 值2, ...] [匹配条件];
```

匹配： where 子句

where 子句用于规定匹配的条件。

| 操作数 | 描述 |
| :---: | :---: |
| = | 等于 |
| &lt;&gt; | 不等于 |
| &gt; | 大于 |
| &lt; | 小于 |
| &gt;= | 大于等于 |
| &lt;= | 小于等于 |

匹配条件语法：

```text
where 列名 操作符 列值
```

```text
update cpucode set id=2, addr='shenzhen' where name='cpu';
```

![image-20200809140409888](https://gitee.com/cpu_code/picture_bed/raw/master//20200809140409.png)

```bash
sqlitebrowser test.db
```

![image-20200809140353950](https://gitee.com/cpu_code/picture_bed/raw/master//20200809140354.png)

当表中有多列、 多行符合匹配条件时会修改相应的多行。 当匹配条件为空时则匹配所有。

![image-20200809140325506](https://gitee.com/cpu_code/picture_bed/raw/master//20200809140325.png)

当表中有多列、 多行符合匹配条件时会修改相应的多行 :

查询

```text
select * from cpucode;
```

![image-20200810095855079](https://gitee.com/cpu_code/picture_bed/raw/master//20200810095855.png)

插入

```text
insert into cpucode values (3, 'test', 'changsha');
```

查询

```text
select * from cpucode;
```

![image-20200810095748022](https://gitee.com/cpu_code/picture_bed/raw/master//20200810095748.png)

修改

```text
update cpucode set name='cpu' where addr='changsha';
```

![image-20200810111341800](https://gitee.com/cpu_code/picture_bed/raw/master//20200810111342.png)

查看

```text
select * from cpucode;
```

当匹配条件为空时则匹配所有 :

修改 :

```text
update cpucode set addr='shenzhen';
```

![image-20200810111523513](https://gitee.com/cpu_code/picture_bed/raw/master//20200810111523.png)

### 删除表中的数据： delete 语句

使用 where 根据匹配条件， 查找一行或多行， 根据查找的结果删除表中的查找到的行。

当表中有多列、 多行符合匹配条件时会删除相应的多行。

语法：

```text
delete from 表名 [匹配条件];
```

删除

```bash
delete from cpucode where name='cpu';
```

查看

```text
select * from cpucode;
```

![image-20200810112210025](https://gitee.com/cpu_code/picture_bed/raw/master//20200810112210.png)

```text
insert into cpucode values (1, 'code', 'changsha');
```

```text
insert into cpucode values (2, 'cpu', 'shenzhen');
```

```text
insert into cpucode values (3, 'test', 'beijing');
```

![image-20200810112452676](https://gitee.com/cpu_code/picture_bed/raw/master//20200810112452.png)

### 查询： select 语句

用于从表中选取数据， 结果被存储在一个结果表中（称为结果集） 。

星号（\*） 是选取所有列的通配符

语法：

```text
 select * from 表名 [匹配条件];
```

```text
select 列名 1[, 列名 2, ...] from 表名 [匹配条件];
```

```text
select * from cpucode
```

![image-20200810112826421](https://gitee.com/cpu_code/picture_bed/raw/master//20200810112826.png)

查看

```text
select * from cpucode where id=2;
```

![image-20200810113213821](https://gitee.com/cpu_code/picture_bed/raw/master//20200810113214.png)

```text
select name from cpucode;
```

![image-20200810113232766](https://gitee.com/cpu_code/picture_bed/raw/master//20200810113232.png)

```text
select name from cpucode where id = 1;
```

![image-20200810113321806](https://gitee.com/cpu_code/picture_bed/raw/master//20200810113321.png)

列名显示

```text
.headers on
```

左对齐

```text
.mode column
```

```text
select * from cpucode where id = 3;
```

![image-20200810113454155](https://gitee.com/cpu_code/picture_bed/raw/master//20200810113454.png)

### 匹配条件语法

数据库提供了丰富的操作符配合 where 子句实现了多种多样的匹配方法。

* in 操作符
* and 操作符
* or 操作符
* between and 操作符
* like 操作符
* not 操作符  

`in` 允许我们在 where 子句中规定多个值。

匹配条件语法：

```text
where 列名 in (列值 1, 列值 2, ...)
```

```text
select * from 表名 where 列名 in (值 1, 值 2, ...);
```

```text
select 列名 1[,列名 2,...] from 表名 where 列名 in (列值 1, 列值 2, ...);
```

```text
select * from cpucode where id in (1, 2);
```

![image-20200810113901131](https://gitee.com/cpu_code/picture_bed/raw/master//20200810113901.png)

```text
select name from cpucode where id in(2, 3);
```

![image-20200810113915934](https://gitee.com/cpu_code/picture_bed/raw/master//20200810113916.png)

`and` 可在 where 子语句中把两个或多个条件结合起来（多个条件之间是与的关系） 。

匹配条件语法：

```text
where 列 1 = 值 1 [and 列 2 = 值 2 and ...]
```

```text
select * from 表名 where 列 1 = 值 1 [and 列 2 = 值 2 and ...];
```

```text
select 列名 1[, 列名 2, ...] from 表名 where 列 1 = 值 1 [and 列 2 = 值 2 and ...];
```

```text
select * from cpucode where     id =1 and addr = 'changsha';
```

![image-20200810114705475](https://gitee.com/cpu_code/picture_bed/raw/master//20200810114705.png)

```text
select addr from cpucode where id = 2 and name = 'cpu';
```

![image-20200810114823444](https://gitee.com/cpu_code/picture_bed/raw/master//20200810114823.png)

`or` 可在 where 子语句中把两个或多个条件结合起来（多个条件之间是或的关系） 。

匹配条件语法：

```text
where 列 1 = 值 1 [or 列 2 = 值 2 or ...]
```

```text
select * from 表名 where 列 1 = 值 1 [or 列 2 = 值 2 or ...];
```

```text
select 列名 1[,列名 2,...] from 表名 列 1 = 值 1 [or 列 2 = 值 2 or ...];
```

```text
select * from cpucode where id = 1 or addr = 'beijing';
```

![image-20200810115842372](https://gitee.com/cpu_code/picture_bed/raw/master//20200810115842.png)

```text
select name from cpucode where id = 3 or addr = 'shenzhen';
```

![image-20200810115958013](https://gitee.com/cpu_code/picture_bed/raw/master//20200810115958.png)

`between A and B` 会选取介于 A、 B 之间的数据范围。 这些值可以是数值、 文本或者日期。

匹配字符串时会以 ascii 顺序匹配。

不同的数据库对 between A and B 操作符的处理方式是有差异的。

* 有些数据库包含 A 不包含 B。  
* 有些包含 B 不包含 A  
* 有些既不包括 A 也不包括 B。  
* 有些既包括 A 又包括 B  

匹配条件语法：

```text
where 列名 between A and B
```

```text
select * from 表名 where 列名 between A and B;
```

```text
select 列名 1[,列名 2,...] from 表名 where 列名 between A and B;
```

```text
select * from cpucode where id between 1 and 3;
```

![image-20200810120415025](https://gitee.com/cpu_code/picture_bed/raw/master//20200810120415.png)

```text
select * from cpucode where addr between 'a' and 'f';
```

![image-20200810120425867](https://gitee.com/cpu_code/picture_bed/raw/master//20200810120426.png)

`like` 用于模糊查找

匹配条件语法：

若列值为数字 , 相当于列名＝列值

若列值为字符串 , 可以用通配符“ % ” 代表缺少的字符（一个或多个） 。

```text
where 列名 like 列值
```

```text
select * from cpucode where id like 2;
```

![image-20200810120638870](https://gitee.com/cpu_code/picture_bed/raw/master//20200810120639.png)

```text
select * from cpucode where name like '%u%';
```

![image-20200810120653093](https://gitee.com/cpu_code/picture_bed/raw/master//20200810120653.png)

`not` 可取出原结果集的补集

匹配条件语法：

```text
where 列名 not in 列值等
```

```text
where 列名 not in (列值 1, 列值 2, ...)
```

```text
where not (列 1 = 值 1 [and 列 2 = 值 2 and ...])
```

```text
where not (列 1 = 值 1 [or 列 2 = 值 2 or ...])
```

```text
 where 列名 not between A and B
```

```text
where 列名 not like 列值
```

```text
select * from cpucode where id not in (1);
```

![image-20200810123016883](https://gitee.com/cpu_code/picture_bed/raw/master//20200810123017.png)

```text
select * from cpucode where addr not like '%zhen';
```

![image-20200810123116343](https://gitee.com/cpu_code/picture_bed/raw/master//20200810123116.png)

order by 语句

根据指定的列对结果集进行排序。

默认按照升序对结果集进行排序， 可使用 `desc` 关键字按照降序对结果集进行排序。

升序

```text
select * from 表名 order by 列名;
```

降序

```text
select * from 表名 order by 列名 desc;
```

```text
select * from cpucode order by name;
```

![image-20200810123322828](https://gitee.com/cpu_code/picture_bed/raw/master//20200810123323.png)

```text
select * from cpucode order by id;
```

![image-20200810123351083](https://gitee.com/cpu_code/picture_bed/raw/master//20200810123351.png)

```text
select * from cpucode order by addr;
```

![image-20200810123412496](https://gitee.com/cpu_code/picture_bed/raw/master//20200810123412.png)

```text
select * from cpucode order by id desc;
```

![image-20200810123337554](https://gitee.com/cpu_code/picture_bed/raw/master//20200810123337.png)

### 事务

事务（Transaction） 可以使用 `BEGIN TRANSACTION` 命令或简单的 `BEGIN` 命令来启动。 此类事务通常会持续执行下去， 直到遇到下一个 `COMMIT` 或 `ROLLBACK` 命令。 不过在数据库关闭或发生错误时， 事务处理也会回滚。 以下是启动一个事务的简单语法：

在 SQLite 中， 默认情况下， 每条 SQL 语句自成事务。

`begin`： 开始一个事务， 之后的所有操作都可以取消

`commit`： 使 begin 后的所有命令得到确认。

`rollback`： 取消 begin 后的所有操作。

```text
begin;

delete from cpucode;

rollback;

select * from cpucode;
```

![image-20200810123809245](https://gitee.com/cpu_code/picture_bed/raw/master//20200810123809.png)

