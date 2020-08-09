

# SQL 语句进阶



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



```sqlite
select id upper(name) from cpucode;
```



常用的聚集函数：  

使用聚集函数， 用于检索数据， 以便分析和报表生成  



```c
avg() 返回某列的平均值
```





```sqlite
alter table cpucode add score integer;
```







## 数据分组 group by  



分组数据， 以便能汇总表内容的子集， 常和聚集函数搭配使用。 例如查询每个班级中的人数、 平均分  



```sqlite
select 列名 1[, 列名 2, ...] from 表名 group by 列名
```



```sqlite
alter table cpucode add class text;

update cpucode set class='class_a' where name='cpu';
update cpucode set class='class_b' where name='code';
update cpucode set class='class_c' where name='test';
```





```sqlite
select * from cpucode;

select class,count(*) from cpucode group by class;

select class,avg(score) from cpucode group by class;
```



group by 子句必须出现在 where 子句之后  



```sqlite
select class,avg(score) from cpucode where class='class_a' group by class;
```





## 过滤分组 having  



除了能用 group by 分组数据外， 还可以包括哪些分组， 排除哪些分组。 例如： 查看班级平均分大于 90 的班级  



通过 having 实现  



语法：  



```sqlite
select 函数名（列名 1） [, 列名 2, ...] from 表名 group by 列名 having 函数名 限制值
```



```sqlite
select class,avg(score) from cpucode group by class having avg(score) >=90; 
```







## 约束  



管理如何插入或处理数据库数据的规则  



常用约束分类 :

主键、 唯一约束、 检查约束  



主键：

惟一的标识一行(一张表中只能有一个主键)

主键应当是对用户没有意义的（常用于索引）

永远不要更新主键， 否则违反对用户没有意义原则

主键不应包含动态变化的数据， 如时间戳、 创建时间列、 修改时间列等

主键应当有计算机自动生成（保证唯一性）  



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
create table stu (id integer primary key, name text unique);
```



```sqlite
insert into stu values(1, 'cpu');
```



```sqlite
insert into stu values(1, 'code');
```



```sqlite
insert into stu values(2, 'cpu');
```



检查约束：  

用来保证一个列（或一组列） 中的数据满足一组指定的条件。  

指定范围， 检查最大或最小范围， 通过 check 实现



```sqlite
create table 表名 (列名 数据类型 check (判断语句));  
```



```sqlite
create table cpucode (id integer, age integer check(age > 0));
```



```sqlite
insert into cpucode values(1, 30);
```



```sqlite
insert into cpucode values(1, -20);
```







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



## 视图（虚拟的表）  



重用 SQL 语句

简化复杂的 SQL 操作(如:多表查询)  







使用视图， 将整个查询包装成一个名为 PersonsGrade 的虚拟表， 简化了查询的 SQL 语句：  





创建视图：  







## 触发器  



















## 查询优化-索引  































