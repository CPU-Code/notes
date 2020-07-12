<!--
 * @Author: cpu_code
 * @Date: 2020-07-12 15:14:55
 * @LastEditTime: 2020-07-12 21:08:32
 * @FilePath: \note\android_bottom\AOSP.md
 * @Gitee: https://gitee.com/cpu_code
 * @Github: https://github.com/CPU-Code
 * @CSDN: https://blog.csdn.net/qq_44226094
 * @Gitbook: https://923992029.gitbook.io/cpucode/
--> 


# AOSP源码开发

 * @Author: cpu_code
 * @Date: 2020-07-11 16:18:27
 * @LastEditTime: 2020-07-12 21:08:41
 * @FilePath: \note\android_bottom\summary.md
 * @Gitee: https://gitee.com/cpu_code
 * @Github: https://github.com/CPU-Code
 * @CSDN: https://blog.csdn.net/qq_44226094
 * @Gitbook: https://923992029.gitbook.io/cpucode/

android源码之前分为四层  :

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200712151542.png" alt="Android四层" style="zoom: 150%;" />

新的android源码分为五层， 增加了HAL层 :

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200712151605.png" alt="androidw的五层" style="zoom: 50%;" />



##　源码级开发（ 系统开发）  

源码级开发就是基于**AOSP**（ Android Open Source Project） 环境的开发工作， 主要开发场景为**Android系统定制**， 比如手机设备的MIUI, Flyme， Smartisan OS,  基于电视的LetvUI， 甚至还有针对投影仪， 路由器等传统物联设备的Android系统定制开发。  

### Android系统分层

**HAL层**： （ Hardware Abstract Layer） **硬件抽象层**。 Android系统里封装内核驱动程序的接口层。 对上层提供接口， 屏蔽底层驱动实现细节.  

为了摆脱 GNU License（ 开源感染性） ，Google将Linux内核中跟底层硬件操作相关的逻辑封装成HAL层接口， 厂商基于接口去实现， 不直接在内核空间实现驱动。 因为Android系统遵循 Apache License， 不强制开源。  



### 三方应用开发与源码级开发的区别  

三方应用开发是基于**Android SDK开发**。 主要技术方向为**APP**及**混合APP开发**， 数据库， 网络协议， 应用架构等， 服务于商业APP需求。  

源码级开发是基于**AOSP环境开发**， 主要技术方向为**系统应用开发**， Framework开发， 底层浏览器内核开发， 音视频编解码开发， 虚拟机开发， 底层驱动开发等。 服务于系统定制需求。  



### AOSP官网  

AOSP官网提供系统开发相关指导， 比如源码的环境搭建， 下载， 编译， 维护， 更新版本， 开放驱动的下载等  

[https://source.android.google.cn/](https://source.android.google.cn/)



----------------------------------------------



## Ubuntu系统安装与介绍  

AOSP环境要求操作系统为Linux或者Mac OS。 

如果想在Window下进行AOSP开发， 通过通过虚拟机软件安装一个Linux系统， 比如Ubuntu或CentOS。 



### 安装虚拟机

VirtualBox   ,  VMWare  



### Ubuntu系统的安装与介绍  	

Ubuntu系统是开发使用比较广泛的一个Linux发行版 ,  带LTS（ Long Term Support） ， 代表是官方长期维护的版本  



------------------------------------



## 常见Linux命令  

Linux命令是对Linux系统进行管理的命令。 对于Linux系统来说， 无论是中央处理器、 内存、磁盘驱动器、 键盘、 鼠标， 还是用户等都是文件。 命令就是去操作文件， 以达到管理系统的目的。

Linux包含系统内置**shell**命令（ 如 `cd`） 及安装的**其他Linux命令**（ 如 `git`） 。

相比Window系统的强项-图形化， Linux的命令则是它的强项。 熟练掌握常见Linux命令， 是做Linux环境下开发的基本技能。

PS： Linux命令大部分在**Mac OS** 下也是通用的， 它们都属于**类Unix系统**。  



### 目录操作

```shell
# 获取当前目录路径
pwd 
```

```shell
# 查看文件列表 eg: ls dirname
ls
```

```shell
# 查看所有文件， 包括隐藏文件
-a 
```

```shell
# 查看文件详细信息
-l 
```

```shell
# 代表当前目录
.
```

```shell
# 代表上一级目录
..
```

```shell
# 进入某一个目录 eg: cd dirname
cd 
```

```shell
# 进入上一级目录
cd .. 
```

```shell
# 进入上次所在目录
cd - 
```

```shell
# 创建目录
mkdir 
```

```shell
# 创建多级目录
-p 
```

```shell
# 拷贝文件或者目录 eg: cp src1 src2... dest
cp 
```

```shell
#  拷贝目录 eg: cp -r src dest
-r
```

```shell
# 移动文件或者目录 eg:mv src dest (如果srcfile和destfile在同一目录下， 就是重命名的效果)
mv
```

```shell
# 删除文件 eg: rm filename1 filename2 ...
rm 
```

```shell
# 删除目录 eg: rm -r dirname
-r 
```



### 文件查找  

```shell
# 查找某一个文件， find [path] -name <filename>
find 
```

```shell
# 从某一个文件/目录下所有文件 中匹配字符串， grep [-r -n ]<string> path
grep 
```

```shell
# 遍历目录查找
-r 
```

```shell
# 显示行号
-n 
```



### 系统操作  

```shell
# 切换到某一个用户(需要输入要切换的用户的密码) eg: su username
su 
```

```shell
# 非root用户强制执行一些需要root权限的操作(需要输入当前用户的密码) eg: sudo rm filename
sudo
```

```shell
# Ubuntu的软件管家， 进行软件的更新， 卸载与维护。
apt-get
```

```shell
# 终端下的文本编辑器。
vim 
```

```shell
# 进入编辑模式
i 
```

```shell
#  跳到命令模式
ESC
```

```shell
# 删除一行
dd 
```

```shell
# 保存退出
:wq
```

```shell
# 不保存退出
:q!
```

```shell
# 查看文件,将文本内容打印到控制台。
cat 
```



### vim

```shell
# 安装VIM
sudo apt-get install vim

# 卸载VIM
sudo apt-get remove vim

# 更新到最新软件列表
sudo apt-get update
```

### chmod

```shell
# 修改文件权限， eg： chmod a+x xxx.sh chmod 777 xxx.sh
# 权限设定可以使用字串[ugoa][+-=][rwx]或者数字 (r=4,w=2,x=1,-=0)，
# u 拥有者
# g 用户组
# o 其他用户
# a 所有人
# + 表示增加权限
# - 表示取消权限
# = 表示直接指定权限
chmod
```



## AOSP源码工作环境  

源码工作环境， 就是安装所依赖的一些软件库， 以及开发调试时需要进行的一些配置。 主要分为编译环境准备， AOSP源码下载， 源码预编译等。 这里主要介绍在Ubuntu14.04 LTS下开发Android6.0代码的工作环境准备。  



### 编译环境搭建（ Ubuntu14.04）

JDK和依赖包下载

```shell
#获取源的更新
sudo apt-get update 

#下载openjdk7
sudo apt-get install openjdk-7-jdk 
```

sunjdk 采用Java研究许可（ Java Research License， 简称JRL） 许可证书， 部分开源， 仅限研究。 openjdk采用GNU许可证书， 完全开源。 sunjdk中私有APIs用类似功能的开源代码替换/重新实现。   

安装其他依赖  

```shell
sudo apt-get install git-core gnupg flex bison gperf build-essential zip curl zlib1g -dev gcc-multilib g++-multilib libc6-dev-i386 lib32ncurses5-dev x11proto-core-dev libx 11-dev lib32z-dev ccache libgl1-mesa-dev libxml2-utils xsltproc unzip
```



USB设备权限配置  

以root身份在/etc/udev/rules.d/ 目录下创建一个rules规则文件， AOSP提供的rules里配置了常见Android设备  

```shell
# 下载并配置AOSP的usb设备权限规则模板
wget -S -O - [http://source.android.com/source/51-android.rules](http://source.andro id.com/source/51-android.rules) | sed "s/<username>/$USER/" | sudo tee >/dev/null /etc /udev/rules.d/51-android.rules; sudo udevadm control --reload-rules
```

执行adb devices的时候可能会提示permission denied， 原因就是因为自己的Android设备没有加入到这个配置文件当中  

```shell
# 列出当前系统识别到的USB设备
lsusb 
```

假设出现

```shell
Bus 001 Device 007: ID 0403:cb48 Future Technology Devices International, Ltd
```

idVendor是0403,  idProduct是cb48  

51-android.rules  

```shell
SUBSYSTEM=="usb", ATTR{idVendor}=="0403", ATTR{idProduct}=="cb48", MODE="0600", OWNER="<username>"
```





### 源码下载与管理  





### 源码预编译



### 知识扩展



### AOSP常见的命令， 目录介绍  





## Android Build System入门



### 什么是makefile  



### ABS的工作流程





## AOSP下进行系统开发  



### Android的启动流程简述  



### 修改系统APP代码



### 定制framework





### 进一步修改native层的代码  





