# Linux\_driver

* @Author: cpu\_code
* @Date: 2020-06-27 16:43:33
* @LastEditTime: 2020-06-29 20:39:03
* @FilePath: \md\Linux\Linux设备驱动开发详解.md
* @Gitee: [https://gitee.com/cpu\_code](https://gitee.com/cpu_code)
* @CSDN: [https://blog.csdn.net/qq\_44226094](https://blog.csdn.net/qq_44226094)

文件目录 :

## [粉丝不够W](https://blog.csdn.net/qq_44226094)

## Linux设备驱动开发详解\( 宋宝华 \)笔记

### Linux设备驱动概述及开发环境构建

#### 设备驱动的作用

设备驱动提供了硬件和应用软件之间的纽带

应用软件时只需 调用系统软件的应用编程接口（API） 就可让硬件去完成要求的工作

#### 无操作系统时的设备驱动

不是任何一个计算机系统都一定要有操作系统

循环 对设备中断的检测 或 对设备的轮询

```text
 //单任务软件架构
 int main(int argc, char* argv[])
 {
     while(1)
     {
         /*有串口中断*/
         if(serialInt == 1)
         {
             ProcessSerialInt(); /* 处理串口中断 */
             serialInt = 0;  /* 中断标志变量清0 */
         }
         /* 有按键中断 */
         if(keyInt == 1)
         {
             ProcessKeyInt(); /* 处理按键中断 */
             keyInt = 0; /* 中断标志变量清0 */
         }
         
         status = CheckXXX();
         switch (status)
         {
             //...
         }
         //...
     }
 }
```

```text
 //无操作系统情况下串口的驱动
 /**********************
  * serial.h
  **********************/
 extern void SerialInit(void);
 extern void SerialSend(const char buf*, int count);
 extern void SerialRecv(char buf*, int count);
 ​
 /**********************
  * serial.c文件
  **********************/
 ​
  /* 初始化串口 */
 void SerialInit(void)
 {
     //...
 }
 ​
 /* 串口发送 */
 void SerialSend(const char *buf, int count)
 {
     //...
 }
 ​
  /* 串口接收 */
 void SerialRecv(char *buf, int count)
 {
     //...
 }
 ​
 /* 串口中断处理函数 */
 void SerialIsr(void)
 {
     //...
     serialInt = 1;
 }
```

无操作系统时硬件、 设备驱动和应用软件的关系 :

![&#x65E0;&#x64CD;&#x4F5C;&#x7CFB;&#x7EDF;&#x65F6;&#x786C;&#x4EF6;&#x3001; &#x8BBE;&#x5907;&#x9A71;&#x52A8;&#x548C;&#x5E94;&#x7528;&#x8F6F;&#x4EF6;&#x7684;&#x5173;&#x7CFB;](https://gitee.com/cpu_code/picture_bed/raw/master//20200627171455.png)

驱动与应用高耦合的不合理设计 :

![image-20200627172554374](https://gitee.com/cpu_code/picture_bed/raw/master//20200627172554.png)

应用直接访问硬件的不合理设计:

![image-20200627172626039](https://gitee.com/cpu_code/picture_bed/raw/master//20200627172626.png)

#### 有操作系统时的设备驱动

操作系统 把单一的“ 驱使硬件设备行动 ”变成了操作系统内与硬件交互的模块， 设为操作系统的API， 给应用程序提供接口

硬件、驱动、操作系统和应用程序的关系 :

![image-20200627172833099](https://gitee.com/cpu_code/picture_bed/raw/master//20200627172851.png)

操作系统给我们提供内存管理机制 :

MMU的32位处理器 , 可以让 每个进程都可以独立地访问 4GB 的内存空间

应用程序将可使用**统一的系统调用**接口来访问各种设备

#### Linux设备驱动

**设备的分类及特点**

计算机系统的硬件主要由 CPU、 存储器 和 外设组成

一般的处理器都集成了UART、I2C控制器、 SPI控制器、 USB控制器、 SDRAM控制器等， 有的处理器还集成了GPU（图形处理器） 、 视频编解码器

Linux将 存储器 和 外设 分为3个基础大类 :

**字符设备** : 以串行顺序依次进行访问的设备， 如触摸屏、 磁带驱动器、 鼠标等。

**块设备** : 按任意顺序进行访问， 以块为单位进行操作，如硬盘、 eMMC等

使用文件系统的操作接口 `open()` 、 `close()` 、`read()` 、 `write()` 等进行访问

**网络设备** : 面向数据包的 接收 和 发送 , 使用套接字接口

**Linux设备驱动与整个软硬件系统的关系**

字符设备 与 块设备都 被映射到 Linux文件系统的文件和目录， 通过文件系统的系统调用接口 `open()` 、 `write()` 、`read()`、 `close()` 等 即可访问 字符设备 和 块设备

Linux的块设备有两种访问方法 :

类似 dd命令对应的原始块设备， 如“ /dev/sdb1 ”等

块设备上建立FAT、EXT4、 BTRFS等文件系统， 然后以文件路径 , 如“ /home/barry/hello.txt ”的形式进行访问

在Linux中， 对 NOR、 NAND 等提供了独立的内存技术设备（Memory Technology Device， **MTD**） 子系统， 可以运行YAFFS2、 JFFS2、UBIFS等具备 擦除 和 负载均衡 能力的文件系统。

Linux设备驱动与整个软硬件系统的关系 :

![image-20200627175735717](https://gitee.com/cpu_code/picture_bed/raw/master//20200627175736.png)

C库函数是通过系统调用接口而实现， 如C库函数 `fopen()` 、 `fwrite()` 、 `fread()` 、 `fclose()` 分别会调用操作系统的API `open()` 、 `write()` 、 `read()` 、 `close()`

为了代码可移植性的目的 , 应用程序最好使用C库函数

**Linux设备驱动的重点、 难点**

 懂得SRAM、Flash、 SDRAM、 磁盘的读写方式， UART、 I2C、 USB等设备的接口以及轮询、 中断、 DMA的原理， PCI总线工作方式以及CPU的内存管理单元（MMU）

运用C语言的结构体、 指针、 函数指针及内存动态申请和释放等

有一定的Linux内核基础，至少要明白驱动与内核的接口。 尤其是对于块设备、 网络设备、 Flash设备、 串口设备等复杂设备

有 多任务并发控制 和 同步 的基础， 因为在驱动中有大量的自旋锁、 互斥、 信号量、 等待队列等 并发与同步机制

#### Linux设备驱动的开发环境构建

**PC上的Linux环境**

**QEMU实验平台**

**源代码阅读和编辑**

[http://lxr.free-electrons.com](http://lxr.free-electrons.com)、 [http://lxr.oss.org.cn/](%20http://lxr.oss.org.cn/) , 这样的网站提供了Linux内核源代码的交叉索引

Linux主机上阅读和编辑Linux源码的常用方式是 `vim` + `cscope`或者 `vim`+ `ctags`， `vim` 是一个文本编辑器， 而 `cscope` 和 `ctags` 则可建立代码索引

 vscode使用

#### 设备驱动Hello World： LED驱动

**无操作系统时的LED驱动**

GPIO一般由 两组寄存器控制， 一组**控制寄存器** 和 一组**数据寄存器**

**控制寄存器**可设置 `GPIO` 口的工作方式为 **输入** 或 **输出** 。

当引脚被设置为输出时， 向 **数据寄存器** 的对应位写入 1 和 0 会分别在引脚上产生 高电平和 低电平

当引脚设置为输入时， 读取 **数据寄存器** 的对应位可获得引脚上的电平为 高 或 低

如：

在　`GPIO_REG_CTRL`　物理地址中**控制寄存器**处的第 `n` 位写入 `1` 可设置 `GPIO` 口为输出，

在地址 `GPIO_REG_DATA` 物理地址中**数据寄存器**的第 `n` 位写入`1` 或 `0` 可在引脚上产生 高 或 低电平．

```text
 //无操作系统时的LED驱动
 //启动硬件MMU后，将寄存器的 物理地址 转化为 虚拟地址
 #define reg_gpio_ctrl *(volatile int *)(ToVirtual(GPIO_REG_CTRL))
 #define reg_gpio_data *(volatile int *)(ToVirtual(GPIO_REG_DATA))
 ​
 /* 初始化LED */
 void LightInit(void)
 {
     reg_gpio_ctrl |= (1 << n); /* 设置GPIO为输出 */
 }
 ​
 /* 点亮LED */
 void LightOn(void)
 {
     reg_gpio_data |= (1 << n); /* 在GPIO上输出高电平 */
 }
 ​
 /* 熄灭LED */
 void LightOff(void)
 {
     reg_gpio_data &= ～(1 << n); /* 在GPIO上输出低电平 */
 }
```

**Linux下的LED驱动**

内核中实际实现了一个提供sysfs节点的GPIO LED驱动， 位于 `drivers/leds/leds-gpio.c` 中

```text
 #include //.../* 包含内核中的多个头文件 */
 ​
 /* 设备结构体 */
 struct light_dev 
 {
     struct cdev cdev;    /* 字符设备cdev结构体 */
     unsigned char vaule; /* LED亮时为1， 熄灭时为0， 用户可读写此值 */
 };
 ​
 struct light_dev *light_devp;
 int light_major = LIGHT_MAJOR;
 /* 打开和关闭函数 */
 int light_open(struct inode *inode, struct file *filp)
 {
     struct light_dev *dev;
     
     /* 获得设备结构体指针 */
     dev = container_of(inode->i_cdev, struct light_dev, cdev);
     /* 让设备结构体作为设备的私有信息 */
     filp->private_data = dev;
     
     return 0;
 }
 ​
 int light_release(struct inode *inode, struct file *filp)
 {
     return 0;
 }
 ​
 /* 读写设备:可以不需要 */
 ssize_t light_read(struct file *filp, char __user *buf, size_t count, loff_t *f_pos)
 {
     struct light_dev *dev = filp->private_data; /* 获得设备结构体 */
     
     if(copy_to_user(buf, &(dev->value), 1))
     {
         return -EFAULT;
     }
     
     return 1;
 }
 ​
 ssize_t light_write(struct file *filp, const char __user *buf, size_t count, loff_t *f_pos)
 {
     struct light_dev *dev = filp->private_data;
     
     if(copy_from_user(&(dev->value), buf, 1))
     {
         return -EFAULT;
     }
     
     /* 根据写入的值点亮和熄灭LED */
     if(dev->value == 1)
     {
         light_on();
     }
     else
     {
         light_off()
     }
     
     return 1;
 }
 ​
 /* ioctl函数 */
 int light_ioctl(struct file *filp, unsigned int cmd, unsigned long arg)
 {
     struct light_dev *dev = filp->private_data;
     
     switch(cmd) 
     {
         case LIGHT_ON:
             dev->value = 1;
             light_on();
             break;
             
         case LIGHT_OFF:
             dev->value = 0;
             light_off();
             break;
 ​
         default:
             /* 不能支持的命令 */
             return -ENOTTY;
     }
    
     return 0;
 }
 ​
 struct file_operations light_fops = {
     .owner = THIS_MODULE,
     .read = light_read,
     .write = light_write,
     .unlocked_ioct1 = light_ioctl,
     .open = light_open,
     .release = light_release,
 };
 ​
 /* 设置字符设备cdev结构体 */
 static void light_setup_cdev(struct light_dev *dev, int index)
 {
     int err;
     int devno = MKDEV(light_major, index);
     
     cdev_init(&dev->cdev, &light_fops);
     dev->cdev.owner = THIS_MODULE;
     dev->cdev.ops = &light_fops;
     
     err = cdev_add(&dev->cdev, devno, 1);
     if(err)
     {
         printk(KERN_NOTICE "Error %d adding LED%d", err, index);
     }
 }
 ​
 /* 模块加载函数 */
 int light_init(void)
 {
     int result;
     dev_t dev = MKDEV(light_major, 0);
     /* 申请字符设备号 */
     if(light_major)
     {
         //字符设备注册
         result = register_chrdev_region(dev, 1, "LED");
     }
     else
     {
         // 分配
         result = alloc_chrdev_region(&dev, 0, 1, "LED");
         light_major = MAJOR(dev);
     }
     
     if((result < 0)
     {
         return result;
     }
        
     /* 分配设备结构体的内存 */
     light_devp = kmalloc(sizeof(struct light_dev), GFP_KERNEL);
     if(!light_devp)
     {
         result = -ENOMEM;
         goto fail_malloc;
     }
        
        
 fail_malloc:
     unregister_chrdev_region(dev, light_devp);
     return result;
 }
        
 /* 模块卸载函数 */
 void light_cleanup(void)
 {
     /* 删除字符设备结构体 */
     cdev_del(&light_devp->cdev);
     /* 释放在light_init中分配的内存 */
     kfree(light_devp);
     /* 删除字符设备 */
     unregister_chrdev_region(MKDEV(light_major, 0), 1); 
 }
 ​
 MODULE_AUTHOR("cpucode");
 MODULE_LICENSE("GPL");
```

### 驱动设计的硬件基础

#### 处理器

**通用处理器**

很多的通用处理器（GPP） 采用SoC（片上系统） 的芯片设计方法， 集成了各种功能模块， 每一种功能都是由**硬件描述语言**设计程序，然后在SoC内由电路实现的

ARM SoC范例： Snapdragon 810 :

![image-20200627232016367](https://gitee.com/cpu_code/picture_bed/raw/master//20200627232036.png)

ARM移动处理芯片供应商包括 高通（Qualcomm） 、 三星（Samsung） 、 英伟达（Nvidia） 、 美满（Marvell） 、 联发科（MTK） 、 海思（HiSilicon） 、 展讯（Spreadtrum） 等

中央处理器的体系结构可以分为两类 :

**冯·诺依曼**结构\( 普林斯顿结构 \)，将**程序指令存储器**和**数据存储器**合并在一起的存储器结构 , 程序指令和数据的**宽度相同**。 如 : Intel公司的中央处理器

**哈佛**结构 , 将 **程序指令** 和 **数据** 分开存储， 指令和数据是**不同的数据宽度** 如 : Cortex A系列

哈佛结构还采用了独立的**程序总线**和**数据总线**， 分别为**CPU** 与 每个**存储器**之间的**专用通信路径**， 提高了执行效率

冯· 诺依曼结构 与 哈佛结构 :

![image-20200627232522881](https://gitee.com/cpu_code/picture_bed/raw/master//20200627232523.png)

改进的哈佛架构， 有独立的**地址总线**和**数据总线** , 两条总线由 **程序存储器** 和 **数据存储器** 分时共用

改进的哈佛结构对 程序 和 数据， 使用**公用数据总线**来完成 程序存储模块 或 数据存储模块 与 CPU 之间的数据传输， 公用的地址总线来寻址 程序 和 数据

改进的哈佛结构 :

![image-20200627232605530](https://gitee.com/cpu_code/picture_bed/raw/master//20200627232605.png)

**指令集**的角度来讲， 中央处理器也可以分为两类 : RISC（精简指令集计算机） 和CISC（复杂指令集计算机）

CSIC 强调增强指令的能力、 减少目标代码的数量， 但 指令复杂， 指令周期长；

而RISC强调尽量减少指令集、 指令单周期执行， 但 目标代码会更大

**数字信号处理器**

数字信号处理器（DSP） 针对通信、 图像、 语音和视频处理等领域的算法而设计

**DSP**分为两类， **定点DSP**， **浮点DSP**。

浮点DSP的浮点运算用**硬件**来实现， 可以在单周期内完成， 因此浮点运算处理速度高于定点DSP。

而定点DSP只能用定点运算**模拟**浮点运算

**网络处理器**器件内部通常由 n个**微码处理器** 和 n个**硬件协处理器** 组成， 多个微码处理器在网络处理器内部**并行处理**， 通过**预先编制**的微码来控制处理流程

处理器分类 :

```text
处理器按应用领域分类融合,数字信号控制器_DSC_通用处理器_GPP_微控制器_MCU_简称单片机微处理器__MPU数字信号处理器_DSP_定点DSP浮点DSP专门处理器_ASO_及ASIC网络处理器音视频编解码器按体系结构分类冯诺依曼结构哈佛结构按指令集分类RISCCSIC
```

MCU 处理图形用户界面和用户的按键输入并运行**多任务操作系统**

DSP进行音视频编解码

射频方面则采用ASIC

#### 存储器

存储器主要可分类为**只读**储存器（ROM） 、 **闪存**（Flash） 、 **随机**存取存储器（RAM） 、 **光/磁介质**储存器。

ROM可细分为 **不可编程**ROM、 **可编程**ROM（PROM） 、 **可擦除可编程**ROM（EPROM） 和**电可擦除可编程**ROM（E2PROM） ， E2PROM完全可以用软件来擦写

Flash闪存技术两种主要的是 **NOR**（或非） 和 **NAND**（与非）

NOR Flash的特点是可芯片内执行（eXecute InPlace， XIP） ， 程序可以直接在NOR内运行

NAND Flash以 **块方式**进行访问， 不支持芯片内执行

典型的类SRAM接口 :

![image-20200628100348132](https://gitee.com/cpu_code/picture_bed/raw/master//20200628100348.png)

公共闪存接口（Common Flash Interface， CFI） 是一个从NOR Flash器件中读取数据的公开、 标准接口

如 芯片不支持CFI， 可使用JEDEC（Joint Electron Device EngineeringCouncil， 电子电器设备联合会）, 需要读出 制造商ID 和 设ID， 来获取 Flash的大小

NAND Flash的主要接口

* I/O总线： 地址、 指令 和 数据通过这组总线传输， 一般为8位或16位
* 芯片启动（Chip Enable， CE\#） ： 如果没有检测到CE信号， NAND器件就保持待机模式， 不对任何控制信号做出响应
* 写使能（Write Enable， WE\#） ： WE\#负责将数据、 地址或指令写入NAND之中
* 读使能（Read Enable， RE\#） ： RE\#允许数据输出
* 指令锁存使能（Command Latch Enable， CLE） ： 当CLE为高电平时，在WE\#信号的上升沿， 指令将被锁存到NAND指令寄存器中
* 地址锁存使能（Address Latch Enable， ALE） ： 当ALE为高电平时， 在WE\#信号的上升沿， 地址将被锁存到NAND地址寄存器中。
* 就绪/忙（Ready/Busy， R/B\#） ： 如果NAND器件忙， R/B\#信号将变为低电平。 该信号是漏极开路， 需要采用上拉电阻

使用NAND Flash的同时， 应采用 错误探测 / 错误更正（EDC / ECC） 算法

Flash的编程原理都是 **只能将1写为0**， 而 **不能将0写为1**

RAM可分为**静态**RAM（SRAM） 和 **动态**RAM（DRAM）

**DRAM**以电荷形式进行存储， 数据存储在电容器中 , 需要定期刷新

**SRAM**是静态的， 只要供电它就会保持一个值， SRAM没有刷新周期

**DPRAM**： 双端口RAM

DPRAM的优点是通信速度快、 实时性强、 接口简单， 而且两边处理器都可主动进行数据传输

双端口RAM :

![image-20200628100700440](https://gitee.com/cpu_code/picture_bed/raw/master//20200628100739.png)

**CAM**： 内容寻址RAM

CAM是以内容进行寻址的存储器 , 输入要查询的数据， 输出就是 数据地址 和 匹配标志。 若匹配（即搜寻到数据） ， 则输出数据地址。

CAM的输入与输出 :

![image-20200628100651583](https://gitee.com/cpu_code/picture_bed/raw/master//20200628100732.png)

**FIFO**： 先进先出队列

FIFO存储器的特点是先进先出， 进出有序， FIFO多用于数据缓冲 .

存储器分类 :

```text
存储器非易失性存储器_NVM_ROM_ROM_PROMEPROME2PROMFlashNORFlashNANDFlash光/磁介质存储器掉电丢失数据的RAMSRAMDRAM_SDRM_DDR_SDRAM_特定类型的RAM一般采用SRAMNVRAMDPRAMCAMFIFO
```

#### 接口与总线

**串口**

RS-232、 RS-422 与 RS-485 都是串行数据接口标准

**RS-232** : 命名为 EIA-232-E

**RS-422** : 定义了一种平衡通信接口， 它是一种单机发送、 多机接收的单向、 平衡传输规范， 被命名为 TIA/EIA-422-A 标准

**RS-485** : 增加了多点、 双向通信能力， 允许多个发送器连接到同一条总线上，同时增加了 发送器的驱动能力 和 冲突保护特性， 并扩展了 总线共模范围， 被命名为TIA/EIA-485-A标准。

RS-232C : 为连接 **DTE**（数据终端设备） 与 **DCE**（数据通信设备） 而制定 .

RS-232C标准接口有25条线（4条数据线、 11条控制线、 3条定时线、 7条备用和未定义线）, 常用的只有9根， 它们是**RTS/CTS**（请求发送/清除发送流控制） 、**RxD/TxD**（数据收发） 、 **DSR/DTR**（数据设置就绪/数据终端就绪流控制） 、**DCD**（数据载波检测， 也称RLSD， 即接收线信号检出） 、 **Ringing-RI**（振铃指示） 、 **SG**（信号地） 信号。

* RTS： 用来表示 DTE请求DCE发送数据， 当终端要发送数据时， 使该信号有效
* CTS： 用来表示DCE准备好接收DTE发来的数据， 是对RTS的响应信号。
* RxD： DTE通过RxD接收从DCE发来的串行数据。
* TxD： DTE通过TxD将串行数据发送到DCE。
* DSR： 有效（ON状态） 表明DCE可以使用。
* DTR： 有效（ON状态） 表明DTE可以使用。
* DCD： 当本地DCE设备收到对方DCE设备送来的载波信号时， 使DCD有效， 通知DTE准备接收， 并且由DCE将接收到的载波信号解调为数字信号， 经RxD线送给DTE。
* Ringing-RI： 当调制解调器收到交换台送来的振铃呼叫信号时， 使该信号有效（ON状态） ， 通知终端， 已被呼叫。

RS-232C串口电路原理 :

![image-20200628145348941](https://gitee.com/cpu_code/picture_bed/raw/master//20200628145349.png)

**I2C**

I2C（ 内置集成电路） 总线是由Philips公司开发的两线式**串行总线**

I2C总线支持多主控（ Multi-Mastering） 模式， 任何能够进行发送和接收的设备都可以成为主设备。

两个信号为数据线SDA和时钟SCL

总线的输出端必须是开漏输出\( CMOS \)或 集电极开路\( TTL \)输出的结构。

总线空闲时， 上拉电阻使 SDA 和 SCL线都保持高电平。

根据 开漏输出 或 集电极开路输出信号的“ **线与** ”逻辑， I2C总线上任意器件输出 **低电平** 都会使相应总线上的**信号线变低**

当 SCL稳定在高电平时， SDA 由高到低 的变化将产生一个**开始位**， 而 由低到高 的变化则产生一个**停止位**

I2C总线的开始位和停止位 :

![image-20200628145547766](https://gitee.com/cpu_code/picture_bed/raw/master//20200628145547.png)

开始位 和 停止位 都由I2C主设备产生。 在选择从设备时， 如果从设备采用7位地址， 则主设备在发起传输过程前， 需先发送1字节的地址信息， 前7位为**设备地址**， 最后1位为**读写标志**。 数据传输， 每次传输的数据也是1字节， 从MSB开始传输。 每个字节传完后， 在SCL的第9个上升沿到来之前， **接收方**应该发出1个**ACK位**。

I2C总线的时序 :

![image-20200628145603514](https://gitee.com/cpu_code/picture_bed/raw/master//20200628145603.png)

**SPI**

SPI（ Serial Peripheral Interface， 串行外设接口） 总线系统是一种同步串行外设接口 , 使CPU与各种外围设备以**串行方式**进行通信以交换信息。一般主控SoC作为SPI的“ 主 ”， 而外设作为SPI的“ 从 ”

SPI接口一般使用4条线： **串行时钟线**（ SCLK） 、 **主机输入/从机输出数据线** MISO、 **主机输出/从机输入数据线** MOSI 和 低电平有效的**从机选择线**SS

SPI主、 从硬件连接图 :

![image-20200628145616617](https://gitee.com/cpu_code/picture_bed/raw/master//20200628145616.png)

在SPI总线的传输中， SS信号是低电平有效的， 当我们要与某外设通信的时候， 需要将该外设上的SS线置低。 SPI从设备支持的SPI总线最高时钟频率（ 决定了SCK的频率） 决定了数据与时钟之间的偏移 , 外设的CPHA决定了采样的时刻、CPOL模式决定了触发的边沿是上升沿还是下降沿

SPI模块为了和外设进行数据交换， 根据外设工作要求， 其输出串行同步时钟极性（CPOL） 和相位（CPHA） 可以进行配置。

如 CPOL = 0， 串行同步时钟的空闲状态 = 低电平；

如 CPOL = 1， 串行同步时钟的空闲状态 = 高电平。

如 CPHA = 0， 在串行同步时钟的第一个跳变沿（上升或下降） 数据被采样；

如 CPHA = 1， 在串行同步时钟的第二个跳变沿（上升或下降） 数据被采样

SPI总线的时序 :

![image-20200628145641748](https://gitee.com/cpu_code/picture_bed/raw/master//20200628145641.png)

**USB**

USB（ 通用串行总线） 具有数据传输率高、 易扩展、 支持即插即用 和 热插拔的优点

USB 1.1包含**全速**和**低速**两种模式， 低速模式的速率为1.5Mbit/s， 支持小数据吞吐量 和 高实时性的设备， 如鼠标等。 全速模式为12Mbit/s， 可以外接速率更高的外设。

在USB 2.0中， 增加了一种高速方式，数据传输率达到480Mbit/s， 半双工， 满足更高速外设的需要。 采用4芯的屏蔽线， 一对**差分**线（ D+、 D-） 传送信号， 另一对（ VBUS、 电源地） 传送+5V的直流电 . 增添的 Bulk模式支持1个**数据流**

而USB3.0（ 也被认为是Super Speed USB） 的最大传输带宽高达5.0Gbit/s（ 即640MB/s） ， 全双工。 设计了8条内部线路， 除VBUS、 电源地之外， 其余3对均为 数据传输线路。 其中保留了 **D+** 与 **D-** 这两条兼容USB 2.0的线路， 新增了SSRX与SSTX专为USB 3.0所设的线路 . 增加了 **Bulk Streams**传输模式 支持**多个数据流**， 每个数据流被分配一个**Stream ID**（SID） ， 每个SID与一个主机缓冲区对应 .

开发板需要挂接USB设备， 则需提供**USB主机（ Host） 控制器**和连接器；

作为USB设备， 则需提供**USB设备适配**器和连接器

USB的物理拓扑结构 :

![image-20200628145659602](https://gitee.com/cpu_code/picture_bed/raw/master//20200628145659.png)

每一个USB设备会有一个或者多个逻辑连接点在里面， 每个连接点叫端点

以下是端点传输方式 :

**控制**（Control） 传输方式

控制传输是双向传输， 数据量通常较小， 主要用来进行查询、 配置和给USB设备发送通用命令。 所有USB设备必须支持标准请求（StandardRequest） ， 控制传输方式 和 端点0。

**同步**（Isochronous） 传输方式 \( Streaming Real-time \)

同步传输提供了确定的**带宽**和**间隔时间**， 它用于时间要求严格并具有较强**容错性**的流数据传输， 或 用于要求恒定数据传送率的即时应用 . 如: 语音业务传输

**中断**（Interrupt） 传输方式

中断方式传送是单向的， 对于USB主机而言， 只有输入。 中断传输方式主要用于定时查询设备是否有中断数据要传送， 该传输方式应用在少量分散的、 不可预测的数据传输场合， 如: 键盘、 游戏杆和 鼠标

**批量**（Bulk） 传输方式

批量传输主要应用在没有带宽、 间隔时间要求的批量数据的传送和接收中， 它要求保证传输。 如 : 打印机和扫描仪

**集线器**负责检测设备的**连接**和**断开**， 利用其**中断IN端点**（Interrupt IN Endpoint） 来向主机报告。 一旦获悉有新设备连接上来， 主机就会发送一系列请求给设备所挂载的集线器， 再由集线器建立起一条连接**主机和设备**之间的**通信通道**。 然后主机以**控制传输**的方式， 通过**端点0**对设备发送各种请求， 设备收到主机发来的请求后回复相应的信息， 进行**枚举**（Enumerate） 操作。 因此USB总线具备**热插拔**的能力

**以太网接口**

以太网接口由**MAC**（以太网媒体接入控制器） 和 **PHY**（物理接口收发器） 组成

以太网MAC由 **IEEE 802.3** 以太网标准定义， 实现了数据链路层

MAC 和 PHY之间采用 MII（媒体独立接口） 连接， 它是 IEEE-802.3定义的以太网行业标准， 包括 1个**数据**接口 与 MAC 和 PHY 之间的1个**管理**接口

数据接口包括分别用于 **发送** 和 **接收** 的两条独立信道， 每条信道都有自己的 数据、时钟 和 控制信号， MII数据接口总共需要16个信号。

MII管理接口有两个信号， **时钟**信号， **数据**信号。 通过管理接口， 上层能 监视 和 控制PHY

以太网接口的硬件电路原理 :

![image-20200628145720172](https://gitee.com/cpu_code/picture_bed/raw/master//20200628145720.png)

**PCI和PCI-E**

PCI（外围部件互连） 是由 Intel 推出的一种局部总线， 作为一种通用的总线接口标准

PCI总线具有如下特点 :

* 数据总线为32位， 可扩充到64位
* 可进行突发（Burst） 模式传输。 突发方式传输是指取得总线控制权后连续进行多个数据的传输。 突发传输时， 只需要给出目的地的首地址， 访问第1个数据后， 第2～n个数据会在首地址的基础上按一定规则自动寻址和传输。 与突发方式对应的是单周期方式， 它在1个总线周期只传送1个数据。
* 总线操作与处理器——存储器子系统操作并行
* 采用中央集中式总线仲裁
* 支持全自动配置、 资源分配， PCI卡内有设备信息寄存器组为系统提供卡的信息， 可实现即插即用
* PCI总线规范独立于微处理器， 通用性好
* PCI设备可以完全作为主控设备控制总线

基于PCI总线的计算机系统逻辑示意图 :

![image-20200628145740570](https://gitee.com/cpu_code/picture_bed/raw/master//20200628145740.png)

系统的各个部分通过**PCI总线**和**PCI-PCI桥**连接在一起

CPU和RAM通过**PCI桥**连接到PCI总线0（即主PCI总线） ， 而具有PCI接口的**显卡**则可以直接连接到主PCI总线上。

PCI-PCI桥是一个特殊的PCI设备， 它负责将 **PCI总线0** 和 **PCI总线1**（即从PCI主线） 连接在一起， 通常PCI总线1称为 PCI-PCI桥的**下游**（Downstream） ， 而PCI总线0则称为 PCI-PCI桥的**上游**（Upstream） 。

为了兼容旧的ISA总线标准， PCI总线还可以通过 **PCI-ISA桥** 来连接ISA总线， 从而支持以前的ISA设备。

PCI配置空间 :

![image-20200628145759043](https://gitee.com/cpu_code/picture_bed/raw/master//20200628145759.png)

PCI配置空间共为256字节 = \(3CH + 04H\)\* 4 :

* 制造商标识（Vendor ID） ： 由PCI组织分配给厂家
* 设备标识（Device ID） ： 按产品分类给本卡的编号
* 分类码（Class Code） ： 本卡功能的分类码， 如图卡、 显示卡、 解压卡等
* 申请存储器空间： PCI卡内有存储器 或 以**存储器编址**的寄存器和I/O空间， 为使**驱动程序**和**应用程序**能访问它们， 需申请CPU的一段**存储区域**以进行定位。 配置空间的**基地址寄存器**用于此目的
* 申请I/O空间： 配置空间中的**基地址**寄存器用来进行系统I/O空间的申请
* 中断资源申请： 配置空间中的**中断引脚**和**中断线**用来向系统**申请中断资源**。 偏移 3D H 处为中断引脚寄存器， 其值表明PCI设备使用了哪一个中断引脚， 对应关系为1—INTA\#、 2—INTB\#、 3—INTC\#、 4—INTD\#

PCI-E（ PCI Express） 是Intel公司提出的新一代的总线接口， PCI Express采用了目前业内流行的**点对点串行**连接， 比起PCI以及更早的计算机总线的共享并行架构， 每个设备都有自己的专用连接， 采用**串行**方式传输数据， 并把数据传输率提高到一个很高的频率， 达到PCI所不能提供的高带宽。

**SD和SDIO**

SD（Secure Digital） 是一种关于Flash存储卡的标准， 也就是一般常见的SD记忆卡， 在设计上与MMC（Multi-Media Card） 保持了兼容

SDHC（SDHigh Capacity） 是大容量SD卡， 支持的最大容量为32GB。

SDXC（SD eXtended Capacity） 则支持最大2TB大小的容量

SDIO（Secure Digital Input and Output Card， 安全数字输入输出卡） 在SD标准的基础上， 定义了除存储卡以外的外设接口。

SDIO主要有两类应用——**可移动**和**不可移动**。

不可移动设备遵循相同的电气标准， 但不要求符合物理标准。

有很多的手机或者手持装置都支持SDIO的功能， 以连接WiFi、 蓝牙、 GPS等模块。

SD/SDIO的传输模式有：

* SPI模式
* 1位模式
* 4位模式

SDIO接口引脚定义 :

![image-20200628145832178](https://gitee.com/cpu_code/picture_bed/raw/master//20200628145832.png)

CLK为时钟引脚， 每个时钟周期传输一个**命令** 或 **数据位**；

CMD是命令引脚， 命令在CMD线上**串行传输**， 是**双向半双工**的（命令从主机到从卡， 而命令的响应是从卡发送到主机）

DAT\[0\] ~ DAT\[3\]为**数据线**引脚；

在SPI模式中， 第8脚位被当成**中断信号**

SDIO单模块读、 写的典型时序 :

![image-20200628145852582](https://gitee.com/cpu_code/picture_bed/raw/master//20200628145852.png)

**eMMC**（ Embedded Multi Media Card） 是当前移动设备本地存储的主流解决方案， 目的在于简化手机存储器的设计。

 eMMC就是**NAND Flash**、 **闪存控制芯片**和**标准接口封装**的集合， 它把NAND和控制芯片封装成为一个多芯片封装（ Multi-Chip Package， MCP） 芯片。

 eMMC支持 DAT\[0\] ~ DAT\[7\] 8位的数据线。

上电或者复位后， 默认处于1位模式， 只使用DAT\[0\]， 后续可以配置为4位或者8位模式。

#### CPLD和FPGA

CPLD（复杂可编程逻辑器件） 由 完全可编程的**与或门阵列** 以及 **宏单元**构成。

CPLD中的基本逻辑单元是**宏单元**，

宏单元由一些“ 与或 ”阵列加上 **触发器** 构成， 其中“ 与或 ”阵列完成 **组合逻辑** 功能， 触发器完成 **时序逻辑** 功能。

宏单元中与阵列的输出称为**乘积项**， 其数量标示着CPLD的容量。

**乘积项阵列**实际上就是一个“**与或**”阵列， 每一个交叉点都是一个可编程熔丝， 如 导通就是实现“**与**”逻辑。

在“ **与** ”阵列后一般还有一个“ **或** ”阵列， 用以完成**最小逻辑表达式**中的“ **或** ”关系。

典型的CPLD的单个宏单元结构

![image-20200628145916715](https://gitee.com/cpu_code/picture_bed/raw/master//20200628145916.png)

CPLD由 **LAB**（逻辑阵列模块， 由多个宏单元组成） 通过**PIA**（可编程互连阵列） 互连组成， 而CPLD与外部的接口则由**I/O控制**模块提供。

宏单元的输出会经**I/O控制块**送至**I/O引脚**， I/O控制块控制每一个I/O引脚的**工作模式**， 决定其为**输入**、 **输出**还是**双向引脚**， 并决定其**三态输出**的**使能端**控制。

典型的CPLD整体架构

![image-20200628145943802](https://gitee.com/cpu_code/picture_bed/raw/master//20200628145943.png)

FPGA由IOC（输入/输出控制模块） 、 EAB（嵌入式阵列块） 、 LAB和快速通道互连构成

IOC是内部信号到I/O引脚的接口， 它位于快速通道的行和列的末端， 每个IOC包含一个双向I/O缓冲器和一个既可作为输入寄存器也可作为输出寄存器的触发器

EAB（嵌入式存储块） 是一种输入输出端带有寄存器的非常灵活的RAM。 EAB不仅可以用作存储器， 还可以事先写入查表值以用来构成如乘法器、 纠错逻辑等电路。 当用于RAM时， EAB可配制成8位、 4位、 2位和1位长度的数据格式

LAB主要用于逻辑电路设计， 一个LAB包括多个LE（逻辑单元） ， 每个LE包括组合逻辑及一个可编程触发器。 一系列LAB构成的逻辑阵列可实现普通逻辑功能， 如计数器、 加法器、 状态机等

典型的FPGA内部结构 :

![image-20200628145955367](https://gitee.com/cpu_code/picture_bed/raw/master//20200628145955.png)

器件内部信号的互连和器件引出端之间的信号互连由快速通道连线提供， 快速通道遍布于整个FPGA器件中， 是一系列水平和垂直走向的连续式布线通道。

实际逻辑电路与查找表的实现 :

![image-20200628150019064](https://gitee.com/cpu_code/picture_bed/raw/master//20200628150019.png)

#### 原理图分析

#### 硬件时序分析

**时序分析的概念**

**典型的硬件时序**

#### 芯片数据手册阅读方法

#### 仪器仪表使用

**万用表**

**示波器**

**逻辑分析仪**

### Linux内核及内核编程

#### Linux内核的发展与演变

**Linux**操作系统是**UNIX**操作系统的一种**克隆系统**

UNIX操作系统

鼻祖

Minix操作系统

Minix操作系统也是UNIX的一种克隆系统

GNU计划

GNU项目已经开发出许多高质量的免费软件， 其中包括 emacs编辑系统、 bash shell程序、 gcc系列编译程序、 GDB调试程序等 . 为Linux操作系统的开发创造了一个合适的环境

POSIX标准

POSIX（Portable Operating System Interface， **可移植的操作系统接口**） 是由IEEE和ISO/IEC开发的一组标准。 基于UNIX， 描述了操作系统的**调用服务接口**， 用于保证编写的应用程序的**移植**

Linux操作系统版本的历史及特点 :

![image-20200629200000931](https://gitee.com/cpu_code/picture_bed/raw/master//20200629200001.png)

android采用Linux内核， 并在内核里加入了一系列补丁

Linux内核开发人员和补丁情况 :

![image-20200629200130885](https://gitee.com/cpu_code/picture_bed/raw/master//20200629200131.png)

#### Linux 2.6后的内核特点

新的调度器

**CFS**（Completely FairScheduler， 完全公平调度） 算法

Linux 3.14 中: 增加 EDF（Earliest Deadline First， 最早截止期限优先） 调度算法

**内核抢占**

一个内核任务可以被抢占， 从而提高系统的**实时性**

Linux 2.4的内核中， 在IRQ1的中断服务程序唤醒RT（实时） 任务后， 必须要等待前面一个Normal（普通） 任务的系统调用完成， 返回用户空间的时候， RT任务才能切入

Linux 2.6内核中， Normal任务的**临界区**（如自旋锁） 结束的时候， RT任务就从内核切入

Linux 2.6以后的内核仍然存在中断、 软中断、 自旋锁等原子上下文进程无法抢占执行

Linux 2.4和2.6以后的内核在抢占上的区别 :

![image-20200629200338341](https://gitee.com/cpu_code/picture_bed/raw/master//20200629200338.png)

改进的线程模型

虚拟内存的变化

文件系统

音频

总线、 设备和驱动模型

电源管理

联网和IPSec

用户界面层

Linux 3.0后ARM架构的变更

#### Linux内核的组成

#### Linux内核源代码的目录结构

#### Linux内核的组成部分

![image-20200629200514816](https://gitee.com/cpu_code/picture_bed/raw/master//20200629200514.png)

#### Linux内核空间与用户空间

#### Linux内核的编译及加载

#### Linux下的C编程特点

#### 工具链

#### 实验室建设

#### 串口工具

### Linux内核模块

#### Linux内核模块简介

#### Linux内核模块程序结构

#### 模块加载函数

#### 模块卸载函数

### Linux文件系统与设备文件

#### Linux文件操作

### 字符设备驱动

### Linux设备驱动中的并发控制

### Linux设备驱动中的阻塞与非阻塞I/O

### Linux设备驱动中的异步通知与异步I/O

### 中断与时钟

### 内存与I/O访问

### Linux设备驱动的软件架构思想

### Linux块设备驱动

### Linux网络设备驱动

### Linux I2C核心、 总线与设备驱动

### USB主机、 设备与Gadget驱动

### I2C、 SPI、 USB驱动架构类比

### ARM Linux设备树

### Linux电源管理的系统架构和驱动

### Linux芯片级移植及底层驱动

### Linux设备驱动的调试

