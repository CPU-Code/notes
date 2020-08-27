## Linux内核开发移植

### Linux内核版本变迁及其获得

`Linux`是最受欢迎的自由电脑操作**系统内核**， 是一个用`C`语言写成， 并且符合`POSIX`标准的类`Unix`操作系统  

`Linux`是由芬兰黑客`Linus Torvalds`开发的， 目的是尝试在英特尔`x86`架构上提供自由免费的类`Unix`操作系统  

`Linux`操作系统的诞生、 发展、 和成长过程依赖于五个重要支柱： `unix`操作系统、 `minix`操作系统、 `GNU`计划、 `POSIX`标准和互联网  

内核版本号从Linux1.0以后主要分为两个阶段：  

Linux1.0-2.6， 数字包括四部分“ A.B.C.D”  

*   A代表主版本号， 如1994年的1.0， 1996年的2.0， 2011年的3.0
*   B代表次版本号， 表示一些重大的修改， 偶数表示稳定版， 奇数表示开发版
*   C代表末版本号， 代表着一个新版本的发布(包括安全补丁、 bug修复、 新增功能、 或驱动程序)， 一般数字变化范围比较大
*   D代表一些BUG修复， 对已经加入的安全补丁、 bug修复、 新增功能或驱动程序做的微调或添加新的特性， 2.6版本之后经常出现  

Linux2.6之后的版本， 数字主要包括三部分"A.B.C"或"A.B.C-rcn"  

*   A代表主版本号
*   B代表次版本号， 随着一个新版本的发布而增加， 与上一阶段中的数字” C“功能类似， 但不再代表稳定与否
*   C代表稳定版本号， 如果只有数字则代表稳定版本， 如果带"-rcn"表示最终稳定版本之前的候选版本(Release Candidate)， n代表候选版本号  

各版本之间的关系及命名规则可以参考网站：[https://zh.wikipedia.org/wiki/Linux%E5%86%85%E6%A0%B8](https://zh.wikipedia.org/wiki/Linux内核)



注意： page9中的EOL： end of life： 指从此往后不在对应的版本上进行更新。 下面展示了Linux内核的版本变迁过程：  

![image-20200701153655101](https://gitee.com/cpu_code/picture_bed/raw/master//20200701153655.png)

### Linux内核结构

Linux内核是Linux系统软件的核心， 它的性能对整个系统的性能起决定作用  

Linux内核文件数目将近4万个， 他们分别位于顶层目录下的20个子目录中， 了解Linux内核的工作过程，必须从了解其目录结构做起  

我们后面研究并移植的内核为:3.4.39， 首先我们从官网上下载3.4.39的内核源码linux-3.4.39.tar.bz2将其在虚拟机上解压， 使用tree命令可以在虚拟机下查看其目录结构  

Linux内核源代码主要包含以下子目录：  

![image-20200701155535511](https://gitee.com/cpu_code/picture_bed/raw/master//20200701155535.png)

*   **arch** 与体系结构相关的代码。 对应于每个支持的体系结构， 有一个相应的子目录如`x86`、 `arm`等与之对应， 相应目录下有对应的芯片与之对应  
*   **crypto** 常用加密及散列算法， 和一些压缩及`CRC`校验算法  
*   **Documentation** 内核相关文档， 关于内核各部分的通用解释和注释  
*   **drivers** 设备驱动代码， 占整个内核代码量的一半以上， 里面的每个子目录对应一类驱动程序， 如: `block`:块设备、 `char`:字符设备、 `net`:网络设备等  
*   **fs** 文件系统代码， 每个支持的文件系统有相应的子目录， 如`cramfs`，`yaffs`， `jffs2`等  
*   **include** 这里包括编译内核所需的大部分头文件 , 与平台无关的头文件放在`include/linux`子目录下 , 各类驱动或功能部件的头文件（ `/media`、 `/mtd`、 `/net`等）,  与体系相关的头文件 `arch/arm/include/` ,  与平台相关的头文件路径`arch/arm/mach-s5p6818/include/mach`  
*   **init** 内核初始化代码,其中的`main.c`中的`start_kernel`函数是系统引导起来后运行的第1个函数， 这是研究内核工作过程的起点  
*   **ipc** 内核进程间通信代码  
*   **kernel** 内核管理的核心代码， 此目录下的文件实现了大多数`linux`系统的内核函数， 体系结构相关的代码在`arch/arm/kernel`  
*   **lib** 与体系结构无关的内核库代码,与系统调用相关 , 与体系结构相关的内核库代码在`arch/arm/lib`下  
*   **mm** 与体系结构无关的内存管理代码 , 与处理器体系结构相关的代码在 `arch/arm/mm`下  
*   **net** 网络部分代码， 其每个目录对应于网络的一个方面  
*   **scripts** 存放一些脚本文件， 如配置内核时用到的`make menuconfig`命令等  



Linux kernel组成：  

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200701154144.png" alt="img" style="zoom:150%;" />

![image-20200701160147862](https://gitee.com/cpu_code/picture_bed/raw/master//20200701160148.png)

Linux内核部分模块：  

进程管理、 进程调度、 系统调用、 中断处理、 下半部和工作队列、 内核同步、 定时管理、 内存管理、虚拟文件系统、 块设备、 网络子系统、 进程地址空间、 模块机制等等...  

Linux编程风格  :

像所有其他大型软件项目一样， Linux制定了一套编码风格  

跟选择一个唯一确定的风格相比， 到底选择什么样的风格反而显得不是那么重要  

编码风格的主要规范伴随着Linus一贯的幽默，都记录在内核源码的Documentation/Coding Style中了。  

制定编码风格的目的， 是为了提高编程效率， 吸引更多的开发者眼球  

缩进  

*   采用制表符(Tab)进行缩进， 而不是空格
*   禁止制表符和空格混合使用  

花括号使用如下：  

```c
if (flag) {
	fun();
	//...
}

void fun (void)
{ 
	//..
}

if (flag)
	fun();
//...
```

命名规范  

*   Linux规定名称中不允许使用混合大小写字符
*   单词之间用下划线分隔
*   避免取有疑惑的简单名称， 如pad()， 应该写成  platform_add_devices()  

代码长度  

*   每行尽量不超过80个字符(可进行有意义分行切割)
*   函数体代码长度尽量不超过两屏
*   函数局部变量尽量不超过十个
*   根据函数使用频率和函数体大小可以使用inline声明， 以提高调用效率  

注释  

*   一般情况用于描述代码可以做什么和为什么要做， 尽量不写实现方式
*   函数的修改和维护日志统一集中在文件开头  

在程序中对ifdef的处理  

一般不这么用：

```c
//...
#ifdef CONFIG_FUN
	fun();
#endif
//...
```

而是在声明处使用ifdef  

```c
#ifdef CONFIG_FUN
void fun (void)
{
	//...
} 
#else
inline void fun (void) { }
#endif
```

其他  

*   指针中的“ *” 号应靠近变量名， 而不是类型关键字
*   函数之间用空行隔开
*   函数导出申明紧跟在函数定义的下面  

代码风格的事后修正

 indent命令是大多数Linux系统中都带有的工具， 当得到一段与内核编码风格大相径庭的代码时， 可以通过这个工具进行
调整：  

```c
#indent -kr -i8 -ts8 -sob -l80 -ss -bs -psl
<filename>
```



### Linux内核启动引导过程

了解Linux镜像的格式及其产生过程  

Linux内核有多种镜像格式：  

Image :  直接生成并且无头部未压缩的内核， 一般用于PC机  

zImage :  Image的压缩版， 采用gzip进行压缩， 比Image体积小，但启动时需要进行自解压， 嵌入式系统中一般采用此种方法  

uImage :  是u-boot专用的一种内核镜像格式， 它是在zImage的基础上又添加了一个长度为64字节的标签头， 在u-boot启动时会去掉此头信息， 仍按zImage启动， 头信息主要用于区分不同格式的内核镜像  

xipImage :  片上执行的未压缩内核， (如norflash等)  



zImage镜像产生过程：  

>   vmlinux -> Image -> compressed / vmlinux -> zImage  

vmlinux 是由以下内核代码生成的非压缩镜像  (arch/arm/kernel/head.s、 kernel/、 mm/、 fs/、 ipc/、 crypto/、 lib/、drivers/、 net/等等)  

Image 是使用objcopy工具对vmlinux进行二进制化处理得到的镜像  

`arch/arm/boot/compressed/vmlinux`由压缩的`Image`和 `compressed/head.S`、 `misc.c`等文件组成  

`zImage`是由`compressed/vmlinux`使用`objcopy`工具二进制化得到  

再对`zImage`加上头部就成为了`uImage`  

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200701161524.png" alt="image-20200701161524245" style="zoom: 80%;" />

boot.img的生成过程  :

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200701161847.png" alt="image-20200701161846753" style="zoom: 67%;" />

`boot.img`是使用一种文件系统打包工具制作的， 相当于把各种非必须文件打包在一起成为一整个文件， 然后被烧写进设备  



Linux内核的启动过程大体上可以分为3个阶段：  

内核解压（ 汇编+C）  

*   主要由`arch/arm/boot/compressed/`对`zImage`完成解压(C语言)，并跳转到下阶段代码  

板级引导阶段（ 汇编）  

*   主要进行对`cpu`和体系结构的检查、 `cpu`本身的初始化以及页表的建立等  

通用内核启动阶段（ C语言）  

*   进入`init/main.c`文件， 从`start_kernel`开始进行内核初始化工作，最后调用`rest_init`  

由`rest_init()`创建并进入内核第一个线程， 线程函数为 `kernel_init()`  

在`kernel_init()`中会初始化各种驱动并建立起标准输入/ 标准输出/错误输出， 最后调用

在`init_post()`中会释放初始化内存段， 标志着内核启动完成， 并努力寻找一个用户进程， 通过kernel_execve()函数加载， 将该进程作为系统的第一个用户进程init，进程号为1  

内核启动完成， 接下来就是用户的事儿了  



### Linux内核的配置与编译  

Linux内核配置裁剪过程  

```shell
make menuconfig
```

Linux内核编译过程  

根据配置裁剪的结果配合`Makefile`完成内核编译  



Linux内核这一配置编译机制  

把驱动程序v_motor_driver.c(振动马达驱动)添加 到内核源码来验证整个过程  

在Linux2.6以后的版本中， 文件的组织是通过 Kconfig 和 Makefile 来实现的  

通过每层目录的 Kconfig 和 Makefile 实现了整个Linux 内核的分布式配置  

*   Kconfig： 对应内核模块的配置菜单  
*   Makefile： 对应内核模块的编译选项  



Kconfig包含了当前目录下所有模块的配置  

子目录的Makefile在被顶层Makefile调用时， 会负责编译当前目录下的所有模块  

当有新的模块被加入时， 需要更改当前目录下的Kconfig文件， 并且更改对应的Makefile文件， 这样在最终编译之前， 可以通过  

make menuconfig 对需要添加的模块进行配置和添加  

当保存make menuconfig时， 系统除了会自动更新.config外， 还会将选项以宏的形式保存在内核根目录下的include/generated/autoconf.h文件下  

当执行`make menuconfig`时， 配置工具会自动根据根目录下的`ARCH`变量读取`arch/$(ARCH)/Kconfig`文件来生成配置界面， 这个文件是所有文件的总入口， 其它目录的`Kconfig`都由它来引用， 如图所示：  

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200701165024.png" alt="image-20200701165024280" style="zoom:67%;" />



`ARM`平台为例讲解其具体配置过程  

当执行`make menuconfig`时， 系统首先读取 `arch/arm/Kconfig`生成整个配置界面  

在读取配置界面的同时， 系统会读取顶层目录下的`.config`文件生成所有配置选项的默认值  

当修改完配置并保存后， 系统会更新顶层目录下的`.config`文件  

当执行`make`时， 各层的`Makefile`会根据`.config`文件中的编译选项来决定哪些文件会被编译到内核中， 或编译成模块  

`Kconfig`语法格式可以参考具体文件， 如：`drivers/char/Kconfig`  

```makefile
# SPDX-License-Identifier: GPL-2.0
#
# Character device configuration

# comment： 注释信息， 菜单和.config文件中都会出现
# string： 字符串; hex： 16进制的数; int： 10进制的数
# choice/endchoice： 表示选择性的菜单条目

# menu/endmenu： 表示生成一个菜单
menu "Character devices"
# source： 用于包含其它Kconfig
source "drivers/tty/Kconfig"
#config代表一个选项的开始,最终会出现在.config中(会自动增加一个CONFIG_前缀)
config DEVMEM
	#bool代表此选项仅能选中或不选中,bool后面的字符串代表此选项在make menuconfig中的名字
	bool "/dev/mem virtual device support"
	#default： 默认选项值
	default y
	help
	  Say Y here if you want to support the /dev/mem device.
	  The /dev/mem device is used to access areas of physical
	  memory.
	  When in doubt, say "Y".

config DEVKMEM
	bool "/dev/kmem virtual device support"
	# On arm64, VMALLOC_START < PAGE_OFFSET, which confuses kmem read/write
	# depends on:依赖其余的选项
	depends on !ARM64
	help
	  Say Y here if you want to support the /dev/kmem device. The
	  /dev/kmem device is rarely used, but can be used for certain
	  kind of kernel debugging operations.
	  When in doubt, say "N".

source "drivers/tty/serial/Kconfig"
source "drivers/tty/serdev/Kconfig"

config TTY_PRINTK
	#tristate： 代表可以选择编译、 不编译、 编译成模块
	tristate "TTY driver to output user messages via printk"
	depends on EXPERT && TTY
	default n
	---help---
	  If you say Y here, the support for writing user messages (i.e.
	  console messages) via printk is available.

	  The feature is useful to inline user messages with kernel
	  messages.
	  In order to use this feature, you should output user messages
	  to /dev/ttyprintk or redirect console to this TTY.

	  If unsure, say N.

config VIRTIO_CONSOLE
	tristate "Virtio console"
	depends on VIRTIO && TTY
	# select： 表示当前config被选中时， 此选项也被选中
	select HVC_DRIVER
	help
	  Virtio console for use with hypervisors.

	  Also serves as a general-purpose serial device for data
	  transfer between the guest and host.  Character devices at
	  /dev/vportNpn will be created when corresponding ports are
	  found, where N is the device number and n is the port number
	  within that device.  If specified by the host, a sysfs
	  attribute called 'name' will be populated with a name for
	  the port which can be used by udev scripts to create a
	  symlink to the device.

//....

endmenu
```



将自己开发的内核代码加入到Linux内核中， 需要有3个步骤：  

把自己编写的代码放到内核中合适的位置  

把自己开发的功能增加到Linux内核的配置选项中， 使用户能够选中这项功能并编译

构建或修改Makefile， 根据用户的选择， 将相应的代码编译到最终生成的Linux内核中去    





将 振动马达的驱动添加进内核为例 ,  讲解配置机制的具体应用  

添加步骤如下：  

Step1： 将`v_motor_driver.c`拷到`drivers/char/`目录下

Step2: `vi driver/char/Kconfig,在Kconfig`文件结尾， 在`endmenu`的前面加入一个`config`选项  

```makefile
config 6818_VMOTOR
	bool "The Vibration motor driver ofr S5P6818"
	default y
	help
		The driver for Vibration motor device

endmenu
```



Step3： make menuconfig(添加配置选项)

>   Device driver ---> character devices ---> [*] The Vibration motor driver for S5P6818

Step4: vi driver/char/Makefile添加内容如下：  

```makefile
obj-$(CONFIG_6818_VMOTOR)		+= v_motor_driver.c
```



Step5： make (更新内核镜像到开发板)  

Step6： 交叉编译 测试程序,  并放到开发板运行  

```shell
arm-linux-gcc v_motor_test.c -o v_motor_test

./v_motor_test 1
```



### Linux3.4.39内核移植

以Linux3.4.39内核在开发板上的移植为例， 学习如何从一个纯净的Linux内核编译、 配置得到适合在一个特定平台运行的镜像。 具体移植步骤如下:  

*   源码解压
*   添加`BSP`
*   修改顶层和`BSP`下`Makefile`
*   修改链接地址机器`ID`号并拷贝默认配置文件
*   移植串口驱动支持
*   `emmc`驱动移植
*   `ext4`文件系统的支持移植
*   内核配置、 编译  



练习: 

*   添加键盘驱动到Linux-3.4.39
*   Linux-3.4.39内核移植
*   Linux-3.4.39内核LCD移植
*   Linux-3.4.39内核修改开机logo  