# 微型计算机组成结构

* @Author: cpu\_code
* @Date: 2020-07-11 20:36:18
* @LastEditTime: 2020-07-12 21:09:37
* @FilePath: \note\linux\_kernel\_0\_12\computer\_composition.md
* @Gitee: [https://gitee.com/cpu\_code](https://gitee.com/cpu_code)
* @Github: [https://github.com/CPU-Code](https://github.com/CPU-Code)
* @CSDN: [https://blog.csdn.net/qq\_44226094](https://blog.csdn.net/qq_44226094)
* @Gitbook: [https://923992029.gitbook.io/cpucode/](https://923992029.gitbook.io/cpucode/)

任何一个系统都可认为由**四个基本部分**组成

**输入**部分用于接收进入系统的信息或数据 , 经过**处理**中心加工后 , 再由**输出**部分送出

**能源**部分为整个系统提供操作运行的能源供给，包括输入和输出部分操作所需要的能量

![&#x7CFB;&#x7EDF;&#x57FA;&#x672C;&#x7EC4;&#x6210;](https://gitee.com/cpu_code/picture_bed/raw/master//20200711203801.png)

计算机系统的处理中心与**输入/输出**部分之间的通道或接口都是**共享**使用的

![&#x7CFB;&#x7EDF;&#x57FA;&#x672C;&#x7EC4;&#x6210;](https://gitee.com/cpu_code/picture_bed/raw/master//20200711203809.png)

计算机系统可分为硬件部分和软件部分，但两者之间互相依存

## 微型计算机组成原理

CPU 通过**地址线**、**数据线**和**控制信号线**组成的本地总线（或称为内部总线）与系统其他部分进行数据通信

**地址线**用于提供内存或 I/O 设备的地址，即指明需要读/写数据的具体位置。

**数据线**用于在 CPU 和内存或 I/O 设备之间提供数据传输的通道 .

**控制线**负责指挥执行的具体读/写操作。

传统 IBM PC 及其兼容计算机的组成框图 :

![image-20200711204253582](https://gitee.com/cpu_code/picture_bed/raw/master//20200711204253.png)

