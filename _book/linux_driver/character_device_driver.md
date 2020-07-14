<!--
 * @Author: cpu_code
 * @Date: 2020-07-11 18:58:16
 * @LastEditTime: 2020-07-12 21:09:16
 * @FilePath: \note\linux_driver\character_device_driver.md
 * @Gitee: https://gitee.com/cpu_code
 * @Github: https://github.com/CPU-Code
 * @CSDN: https://blog.csdn.net/qq_44226094
 * @Gitbook: https://923992029.gitbook.io/cpucode/
--> 

# 字符设备驱动开发

 * @Author: cpu_code
 * @Date: 2020-07-12 14:09:20
 * @LastEditTime: 2020-07-12 21:08:35
 * @FilePath: \note\android_bottom\smart_pointer.md
 * @Gitee: https://gitee.com/cpu_code
 * @Github: https://github.com/CPU-Code
 * @CSDN: https://blog.csdn.net/qq_44226094
 * @Gitbook: https://923992029.gitbook.io/cpucode/

## 字符设备驱动简介

**字符设备**是 Linux 驱动中最基本的设备驱动，字符设备就是一个一个字节，按照**字节流**进行读写操作的设备，读写数据是分**先后顺序**的。比如我们最常见的点灯、按键、 IIC、 SPI，LCD 等等都是字符设备，这些设备的驱动就叫做字符设备驱动  



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200711190323.png" alt="Linux 应用程序对驱动程序的调用流程" style="zoom: 67%;" />







