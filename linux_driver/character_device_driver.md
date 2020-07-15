# 字符设备驱动开发

* @Author: cpu\_code
* @Date: 2020-07-12 14:09:20
* @LastEditTime: 2020-07-12 21:08:35
* @FilePath: \note\android\_bottom\smart\_pointer.md
* @Gitee: [https://gitee.com/cpu\_code](https://gitee.com/cpu_code)
* @Github: [https://github.com/CPU-Code](https://github.com/CPU-Code)
* @CSDN: [https://blog.csdn.net/qq\_44226094](https://blog.csdn.net/qq_44226094)
* @Gitbook: [https://923992029.gitbook.io/cpucode/](https://923992029.gitbook.io/cpucode/)

## 字符设备驱动简介

**字符设备**是 Linux 驱动中最基本的设备驱动，字符设备就是一个一个字节，按照**字节流**进行读写操作的设备，读写数据是分**先后顺序**的。比如我们最常见的点灯、按键、 IIC、 SPI，LCD 等等都是字符设备，这些设备的驱动就叫做字符设备驱动

![Linux &#x5E94;&#x7528;&#x7A0B;&#x5E8F;&#x5BF9;&#x9A71;&#x52A8;&#x7A0B;&#x5E8F;&#x7684;&#x8C03;&#x7528;&#x6D41;&#x7A0B;](https://gitee.com/cpu_code/picture_bed/raw/master//20200711190323.png)

