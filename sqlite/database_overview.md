# 数据库概述



## 数据与数据管理  



信息 : 对现实世界存在方式 或 运动状态的反映。  



数据 : 存储在某一媒体上， 能够被识别的物理符号;

数据的概念在数据处理领域已经被大为拓宽， 不仅包括字符组成的文本形式的数据， 而且还包括其他类型的数据（如音频、 视频等）  



信息与数据的关系：

信息与数据是**相互依赖存在**的， 数据是信息的**载体**， 信息是数据的**内涵**。



数据处理 : 数据及信息相互转换的过程。

从数据处理的角度而言， 信息是一种被加工成特定形式的数据。  





数据处理：  



数据处理的核心是**数据管理**技术， 其中数据管理技术是指对数据的**分类**、 **组织**、 **编码**、 **存储**、 **检索**和**维护**的技术。  



数据管理技术的发展经历了如下几个阶段：  



1、 人工管理阶段  



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200807165233.png" alt="image-20200807165233305" style="zoom: 67%;" />



2、 文件系统阶段  



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200807165255.png" alt="image-20200807165255498" style="zoom:67%;" />



3、 数据库系统阶段  



`DBMS` : `data base management system` (数据库管理系统)  



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200807165307.png" alt="image-20200807165307358" style="zoom:67%;" />





## 数据库系统  



定义： 数据库系统（DBS） 是指计算机中引入数据库后的系统构成。  



组成： 数据库、 操作系统、 数据库管理系统、 数据库管理应用系统、 数据库管理员（DBA） 、 用户  



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200807165356.png" alt="image-20200807165356625" style="zoom: 80%;" />



## 数据库模型  



数据库管理系统根据**数据模型**对数据进行**存储**和**管理**  



数据模型应满足三方面要求：	

*   能比较真实地模拟现实世界;
*   容易为人所理解
*   便于在计算机上实现



**数据结构**、 **数据操作**和**完整性约束**是构成数据模型的三要素

完整性约束：

数据完整性约束 : 是为了**防止不符合规范的数据进入数据库**。

在用户对数据进行插入、 修改、 删除等操作时， DBMS 自动按照**一定的约束条件**对数据进行**监测**， 使**不符合规范**的数据**不能进入数据库**， 以确保数据库中存储的数据正确、 有效、 相容。  



数据库管理系统数据模型：

数据库管理系统采用的数据模型主要有： **层次模型**、 **网状模型** 和 **关系模型**  



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200807165454.png" alt="image-20200807165454751" style="zoom:67%;" />



层次模型是种典型的**树形结构**

特点：

*   有且仅有一个节点无父节点， 这个节点被称为**根节点**。
*   其他节点有且仅有一个父节点。
*   同一父节点的子节点被称为**兄弟节点**。
*   没有子节点的节点称为**叶节点**



在现实世界中， 事物之间的联系更多的是非层次关系的， 用层次模型表示非树型结构是很不直观的， 网状模型则可以克服这一缺点。  



**网状模型**构成了比层次结构模型更加复杂的网状结构

特点：

*   允许一个以上的节点无父节点
*    一个节点可以有多个的父节点  



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200807165532.png" alt="image-20200807165532348" style="zoom: 50%;" />



**关系模型**数据的逻辑结构是一张二维表

一行为一个对象成员

每一列为对象的一个属性  



特点： 

*   每一列中的分量是类型相同的数据。
*   列的顺序可以是任意的。
*   行的顺序可以是任意的。
*   表中的分量是不可再分割的最小数据项， 即表中不允许有子表。



关系数据库采用**关系模型**作为数据的组织方式



关系数据库因其严格的数学理论、 使用简单灵活、 数据独立性强等特点， 而被公认为最有前途的一种数据库管理系统。



它目前已成为占据主导地位的数据库管理系统



自 20 世纪 80 年代以来， 作为商品推出的数据库管理系统几乎都是关系型的， 例如： Oracle、 Sybase 等。  





---------------



## 常用的数据库  



常用的数据库有：

ORACLE、 Mysql、 SQL server、 Access、 Sybase、 SQLite



1、 ORACLE：

ORACLE 是甲骨文公司开发的一款数据库， 是一种适用于大型、 中型和微型计算机的关系数据库管理系统， 它使用 SQL 语言作为它的数据库语言。



2、 MySQL：

MySQL 是一个开放源码的小型关系型数据库管理系统， 开发者为瑞典 MySQL AB 公司， 92HeZu 网免费赠送 MySQL。 目前 MySQL 被广泛地应用在 Internet 上的中小型网站 . 提供由于其体积小、 速度快、 总体拥有成本低， 尤其是开放源码这一特点， 许多中小型网站为了降低网站总体拥有成本而选择了 MySQL 作为网站数据库。



3、 SQL server：

真正的客户机/服务器体系结构。 微软 Microsoft 出品的一款数据库软件图形化用户界面， 使系统管理和数据库管理更加直观/简单具有很好的伸缩性， 可跨越从运行 Windows 95/98 型电脑到运行 Windows 2000 的大型多处理器等多种平台使用。



4、 Access：

Access 是由微软发布的关系数据库管理系统。它结合了 MicrosoftJet Database Engine 和 图形用户界面两项特点， 是 Microsoft Office 的系统程序之一Access 是一种桌面数据库， 只适合数据量少的应用， 在处理少量数据和单机访问的数据库时是很好的，效率也很高. 但是它的同时访问客户端不能多于 4 个。access 数据库有一定的极限， 如果数据达到 100M 左右， 很容易造成服务器 iis 假死， 或者消耗掉服务器的内存导致服务器崩溃。



5、 Sybase：

sybase 公司 1987 年推出了 Sybase 数据库产品



Sybase 主要有三种版本， 一是 UNIX 操作系统下运行的版本， 二是 Novell Netware 环境下运行的版本，三是 Windows NT 环境下运行的版本。



Windows NT：

Microsoft Windows NT(New Technology)是 Microsoft 在 1993 年推出的面向工作站、 网络服务器和大型计算机的网络操作系统， 也可做 PC 操作系统



Sybase 数据库特点：

*   基于**客户/服务器**体系结构的数据库。
*   它是真正开放的数据库 ， 容易移植。
*   它是一种**高性能**的数据库。  
