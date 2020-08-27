

# SQL 语句进阶



------------------------------



## 函数和聚合



函数：  

SQL 语句支持利用函数来处理数据， 函数一般是在数据上执行的， 它给数据的转换和处理提供了方便常用的文本处理函数：  



常用的文本处理函数：  



```c
// 返回字符串的长度
length();
```



```c
//将字符串转换为小写
lower();
```



```c
// 将字符串转换为大写
upper();
```



语法： 

```sqlite
select 函数名(列名) from 表名;
```



```sqlite
select * from cpucode;
```



![image-20200810143450457](https://gitee.com/cpu_code/picture_bed/raw/master//20200810143450.png)



```sqlite
select id upper(name) from cpucode;
```



![image-20200810143440630](https://gitee.com/cpu_code/picture_bed/raw/master//20200810143440.png)



常用的聚集函数：  

使用聚集函数， 用于检索数据， 以便分析和报表生成  



```sqlite
--返回某列的平均值
avg() 
```



```sqlite
--返回某列的行数
count()
```



```sqlite
-- 返回某列的最大值
max() 
```



```sqlite
-- 返回某列的最小值
min() 
```



```sqlite
-- 返回某列值之和
sum() 
```



插入一列分数 score   

```sqlite
alter table cpucode add score integer;
```



![image-20200810143923933](https://gitee.com/cpu_code/picture_bed/raw/master//20200810143924.png)



修改内容  



```sqlite
update cpucode set score=66 where name='cpu';
```



```sqlite
update cpucode set score=77 where name='code';
```



```sqlite
update cpucode set score=88 where name='test';
```



![image-20200810144150470](https://gitee.com/cpu_code/picture_bed/raw/master//20200810144150.png)



```sqlite
select max(score) from cpucode;
```



![image-20200810144730589](https://gitee.com/cpu_code/picture_bed/raw/master//20200810144730.png)



```sqlite
select avg(score) from cpucode;
```



![image-20200810144806248](https://gitee.com/cpu_code/picture_bed/raw/master//20200810144806.png)



```sqlite
select count(*) from cpucode;
```



![image-20200810144817464](https://gitee.com/cpu_code/picture_bed/raw/master//20200810144817.png)



判断数据库中是否有 `cpucode` 这张表  

`sqlite_master` 是数据库自带的一个表。 当用户创建一张表时， 数据库会将用户新建的表的信息存放在 `sqlite_master` 这张表中  

```sqlite
select count(*) from sqlite_master where type = 'table' and name = 'cpucode';
```



![image-20200810152755590](https://gitee.com/cpu_code/picture_bed/raw/master//20200810152755.png)



------------------------



## 数据分组 group by  



分组数据， 以便能汇总表内容的子集， 常和聚集函数搭配使用。 



```sqlite
select 列名 1[, 列名 2, ...] from 表名 group by 列名
```



```sqlite
alter table cpucode add class text;

update cpucode set class='class_a' where name='cpu';
update cpucode set class='class_b' where name='code';
update cpucode set class='class_b' where name='test';
```



![image-20200810155059089](https://gitee.com/cpu_code/picture_bed/raw/master//20200810155059.png)



```sqlite
select class, count(*) from cpucode group by class;
```



![image-20200810155155413](https://gitee.com/cpu_code/picture_bed/raw/master//20200810155155.png)



```sqlite
select class, avg(score) from cpucode group by class;
```



![image-20200810155236737](https://gitee.com/cpu_code/picture_bed/raw/master//20200810155236.png)



group by 子句必须出现在 where 子句之后  



```sqlite
select class, avg(score) from cpucode where class='class_a' group by class;
```



![image-20200810155318529](https://gitee.com/cpu_code/picture_bed/raw/master//20200810155318.png)



-------------------------



## 过滤分组 having  



除了能用 `group by` 分组数据外， 还可以包括哪些分组， 排除哪些分组。 

通过 having 实现  



语法：  



```sqlite
select 函数名（列名 1） [, 列名 2, ...] from 表名 group by 列名 having 函数名 限制值
```



```sqlite
select class, avg(score) from cpucode group by class having avg(score) >=80; 
```



![image-20200810155539900](https://gitee.com/cpu_code/picture_bed/raw/master//20200810155540.png)



------------------------



## 约束  



管理如何插入或处理数据库数据的规则  



常用约束分类 :

**主键**、 **唯一约束**、 **检查约束**  



主键：

惟一的标识一行( 一张表中只能有一个主键 )

主键应当是对用户没有意义的（ 常用于索引 ）

永远不要更新主键， 否则违反对用户没有意义原则

主键不应包含动态变化的数据， 如时间戳、 创建时间列、 修改时间列等

主键应当有计算机自动生成（ 保证唯一性 ）  



语法：  

```sqlite
create table 表名称 (列名称1 数据类型 primary key, 列名称2 数据 类型,列名称3 数据类型, ...)；
```



唯一约束：

用来保证一个列（或一组列） 中数据唯一， 类似于主键， 但跟主键有区别

表可包含多个唯一约束， 但只允许一个主键

唯一约束列可修改或更新

创建表时， 通过 unique 来设置  



语法:  

```sqlite
create table 表名 (列名称 1 数据类型 unique[， 列名称 2 数据类型 unique,...]);
```



```sqlite
create table test (id integer primary key, name text unique);
```



![image-20200810160122651](https://gitee.com/cpu_code/picture_bed/raw/master//20200810160122.png)



```sqlite
insert into test values(1, 'cpu');
```



```sqlite
insert into test values(1, 'code');
```



![image-20200810164549086](https://gitee.com/cpu_code/picture_bed/raw/master//20200810164549.png)



```sqlite
insert into test values(2, 'cpu');
```



![image-20200810164626151](https://gitee.com/cpu_code/picture_bed/raw/master//20200810164626.png)



检查约束：  

用来保证一个列（或一组列） 中的数据满足一组指定的条件。  

指定范围， 检查最大或最小范围， 通过 check 实现



```sqlite
create table 表名 (列名 数据类型 check (判断语句));  
```



```sqlite
create table test2 (id integer, age integer check(age > 0));
```



![image-20200810164838299](https://gitee.com/cpu_code/picture_bed/raw/master//20200810164838.png)



```sqlite
insert into test2 values(1, 30);
```



![image-20200810164927303](https://gitee.com/cpu_code/picture_bed/raw/master//20200810164927.png)



```sqlite
insert into cpucode values(1, -20);
```



![image-20200810164941123](https://gitee.com/cpu_code/picture_bed/raw/master//20200810164941.png)



-------------------------------



## 联结表（多表操作）  



概念：  

保存数据时往往不会将所有数据保存在一个表中， 而是在多个表中存储， 联结表就是从多个表中查询数据。  



在一个表中不利于分解数据， 也容易使相同数据出现多次， 浪费存储空间； 使用联结表查看各个数据更直观，这使得在处理数据时更简单。  



例如： 学生每年的考试成绩， 学生个人信息基本固定(包括学号、 姓名、 地址等)； 把所有信息放在同一个表中必然会造成学生的学号等基本信息重复。  



对比：

学生信息和成绩在一个表中  





单表缺点：  

*   每年记录的成绩都需要添加重复的学生信息， 如： name， addr  
*   lucy 的地址(addr)修改， 整个表所有的关于 lucy 的 addr 都需更改， 处理复杂。  



学生信息和成绩在不同的表中

学生信息(persons)：  





学生成绩(grade)：  



每个人的信息只需保存一份， 没有重复， 成绩 id 与学生信息 id 相同， 作为关联， 用于查找相应学生的成绩  



lucy 的地址(addr)修改， 只需修改 persons 表中的 addr  



分表优点：  

将学生信息和成绩分开存储， 节省空间， 处理简单， 效率更高， 在处理大量数据时尤为明显。  



使用关系型数据库存储数据， 各个表的设计是非常重要的， 良好的表设计， 能够简化数据的处理， 提高效率， 提高数据库的健壮性。  





使用联结：  



通过 select 语句将要联结的所有表以及它们如何关联  



```sqlite
select 列名 1,列名 2,.. from 表 1,表 2,.. where 判断语句;
```



```sqlite
select name , addr, score, year from cpucode, grade where cpucode.id = grade.id;
```



在联结两个表时， 实际上是将第一个表中的每一行与第二个表中每一行配对， where 子句作为过滤条件，只有满足条件的才显示出来  



匹配语句: persons.id = grade.id

完全限定列名， 用一个点(.)分隔表名和列名



终端输入(输出指定学生的信息和分数)：  



```sqlite
select name, addr, score, year from cpucode, grade where cpucode.id = greade.id and name='cpu';
```







select 语句中可以联结的表的数目没有限制

当前面指定列名二义性时， 需要通过完全限定名引用  



-------------------------



## 视图（虚拟的表）  



重用 SQL 语句

简化复杂的 SQL 操作(如:多表查询)  







使用视图， 将整个查询包装成一个名为 PersonsGrade 的虚拟表， 简化了查询的 SQL 语句：  





创建视图：  



--------------------------------



## 触发器  















-------------------------------------



## 查询优化-索引  































