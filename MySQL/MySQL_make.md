<!--
 * @由于个人水平有限, 难免有些错误, 还请指点:  
 * @Author: cpu_code
 * @Date: 2020-09-23 17:51:28
 * @LastEditTime: 2020-09-23 18:47:19
 * @FilePath: \notes\MySQL\MySQL_make.md
 * @Gitee: [https://gitee.com/cpu_code](https://gitee.com/cpu_code)
 * @Github: [https://github.com/CPU-Code](https://github.com/CPU-Code)
 * @CSDN: [https://blog.csdn.net/qq_44226094](https://blog.csdn.net/qq_44226094)
 * @Gitbook: [https://923992029.gitbook.io/cpucode/](https://923992029.gitbook.io/cpucode/)
-->


# MySQL 配置, 避坑



## 当然第一步就是要下载`mysql` 啦



mysql的官方下载地址 , 无法保证这个链接一直有效, 建议你自己google 一下 mysql 下载: 

[https://dev.mysql.com/downloads/mysql/](https://dev.mysql.com/downloads/mysql/)

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200923180504.png" alt="image-20200923180337747" style="zoom: 67%;" />



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200923181253.png" alt="image-20200923181253405" style="zoom:50%;" />





下载完毕, 就解压到一个目录下, 我就是解压到 D盘



![image-20200923180638597](https://gitee.com/cpu_code/picture_bed/raw/master//20200923180638.png)





然后在这个目录下, 



## 新创建一个 `my.ini` 文件



```javascript
[client]
# 设置mysql客户端默认字符集
default-character-set=utf8
 
[mysqld]
# 设置3306端口
port = 3306
# 设置mysql的安装目录 , 只要修改这个地方就可以, 改成自己解压的目录
basedir=D:\\Program Files\\MySQL\\mysql-8.0.21-winx64
# 设置 mysql数据库的数据的存放目录，MySQL 8+ 不需要以下配置，系统自己生成即可，否则有可能报错
# datadir=
# 允许最大连接数
max_connections=20
# 服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
```



![image-20200923180837337](https://gitee.com/cpu_code/picture_bed/raw/master//20200923180837.png)





## 启动下 MySQL 数据库：





注意 注意 注意 !!!!!!!!!!!!!!!!!!!!!!

**管理员身份**打开 cmd 命令行工具，切换目录：



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200923181553.png" alt="image-20200923181553599" style="zoom:67%;" />



切换到自己的解压文件的目录bin文件下 , 可以用 `tab`  进行 补充



![image-20200923181724718](https://gitee.com/cpu_code/picture_bed/raw/master//20200923181724.png)



初始化数据库：

```mysql
mysqld --initialize --console
```



![image-20200923182248908](https://gitee.com/cpu_code/picture_bed/raw/master//20200923182248.png)



执行完成后，会输出 root 用户的初始默认密码，如：



?????????????????????????????????????????????????????????@localhost: 



`*rnaDstKY4kk` 就是初始密码，一定要保存一下, 搞不好就忘记了, 后续登录需要用到，后面我也教怎么改密码



输入以下安装命令：

```
mysqld install
```



![image-20200923182324951](https://gitee.com/cpu_code/picture_bed/raw/master//20200923182325.png)



启动输入以下命令即可：

```
net start mysql
```



![image-20200923182349467](https://gitee.com/cpu_code/picture_bed/raw/master//20200923182349.png)



如果我们要登录本机的 MySQL 数据库，只需要输入以下命令即可：



```mysql
mysql -u root -p
```

按回车确认, 如果安装正确且 MySQL 正在运行, 会得到以下响应:





![image-20200923182430184](https://gitee.com/cpu_code/picture_bed/raw/master//20200923182430.png)





### 修改密码 :



![image-20200923183745955](https://gitee.com/cpu_code/picture_bed/raw/master//20200923183746.png)

```mysql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '登陆密码, 改成自己的密码';
```



退出 :



以 **mysq>** 加一个闪烁的光标等待命令的输入, 输入 **exit** 或 **quit** 退出登录。



![image-20200923182549936](https://gitee.com/cpu_code/picture_bed/raw/master//20200923182550.png)







### 修改环境变量,  以后就不要切换到解压目录下 进行命令启动mysql



win 搜索  **环境变量**



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200923182656.png" alt="image-20200923182655902" style="zoom: 80%;" />



点击环境变量,  新建  , 设置环境变量 

变量名 : MySQL



变量值 : 

然后点击确认

![image-20200923182919119](https://gitee.com/cpu_code/picture_bed/raw/master//20200923182919.png)



点击 Path 环境

新建

```my
%MySQL%\bin
```

点击确认



![image-20200923183147572](https://gitee.com/cpu_code/picture_bed/raw/master//20200923183147.png)





然后打开命令 输入

```my
mysql -u root -p
```



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200923183424.png"/>





#sqlyog连接mysql错误码2058



 sqlyog配置新连接报错：

错误号码 2058，是因为mysql 密码加密方法变了。



先登录你的数据库，



```my
mysql -u root -p
```



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200923183424.png"/>



然后执行 





```mysql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '登陆密码, 改成自己的密码';
```



![image-20200923183745955](https://gitee.com/cpu_code/picture_bed/raw/master//20200923183746.png)





### 查看mysql的端口号：



```mysql
show global variables like 'port';
```



![image-20200923184245550](https://gitee.com/cpu_code/picture_bed/raw/master//20200923184245.png)



![image-20200923184514705](https://gitee.com/cpu_code/picture_bed/raw/master//20200923184514.png)





![image-20200923184456800](https://gitee.com/cpu_code/picture_bed/raw/master//20200923184456.png)