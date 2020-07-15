# Android底层软硬件结合开发概述

* @Author: cpu\_code
* @Date: 2020-07-11 16:18:27
* @LastEditTime: 2020-07-12 21:08:41
* @FilePath: \note\android\_bottom\summary.md
* @Gitee: [https://gitee.com/cpu\_code](https://gitee.com/cpu_code)
* @Github: [https://github.com/CPU-Code](https://github.com/CPU-Code)
* @CSDN: [https://blog.csdn.net/qq\_44226094](https://blog.csdn.net/qq_44226094)
* @Gitbook: [https://923992029.gitbook.io/cpucode/](https://923992029.gitbook.io/cpucode/)

## Android底层软硬件结合开发概述

Android是 个开源的智能移动平台 一个开源的智能移动平台， 任何 个厂商拿 一个厂商拿到 Android Source 都可以基于自己的硬件架构设计

移动设备。

硬件多样化

厂商自己的利益

软硬件结合开发的弊端？

失去了Android应用的跨平台特性

由于不同厂商研发能力参差不齐，应用程序的稳定性及性能有所不同

分类

NDK开发:

从应用角度来开发软硬结合代码

适用于单个应用程序的底层结合需求

HAL硬件抽象层开发:

从整个系统框架需求角度来开发本地程序

适用于共享功能软硬结合代码

### HAL 概念

HAL（ **Hardware Abstract Layer**）硬件抽象层是Google开发的Android系统里上层应用对底层硬件操作屏蔽的一个软件层次，说白了，就是上层的应用不用关心底层硬件具体是如何工作的，只要向上提供一个统一的接口即可，这种设计思想广泛的存在于当前的**软件架构设计**里。

![image-20200706134655889](https://gitee.com/cpu_code/picture_bed/raw/master//20200706134655.png)

![image-20200706134701523](https://gitee.com/cpu_code/picture_bed/raw/master//20200706134701.png)

![image-20200706134709193](https://gitee.com/cpu_code/picture_bed/raw/master//20200706134709.png)

![image-20200706134718998](https://gitee.com/cpu_code/picture_bed/raw/master//20200706134719.png)

![image-20200706134646845](https://gitee.com/cpu_code/picture_bed/raw/master//20200706134646.png)

HAL存在的原因

1、 并不是所有的硬件设备都有标准的 linux kernel 的接口。

2、 Kernel driver涉及到GPL的版权。某些设备制造商并不愿意公开硬件驱动，所以才通过HAL方式绕过GPL。

3、 针对某些硬件， Android有一些特殊的需求。

HAL简介及现状分析

现有HAL架构由 Patrick Brady \(Google\) 在2008G l oogle I/O演讲中提出， 从上面架构图我们知道，HAL 的目的是为了把 Android framework 与Linux kernel 完整「隔开」。让 Android 不至过度依赖 Linux kernel， 有点「kernel independent」的意思。

AL简介及现状分析

HAL 主要的实现储存于以下目录

libhardware\_legacy/ ‐ 旧的架构、 采取链接库模块的观念进行

libhardware/ ‐ 新架构、调整为 HAL stub 的观念

旧的HAL 架构\(libhardware\_legacy\)

libhardware\_legacy 作法，是传统的「module」方式，也就是将 \*.so 文件当做「shared library」来使用，在runtime（JNI 部份） 以 direct functioncall 使用 HAL module。通过**直接调用函数**的方式，来操作驱动程序。

当然，应用程序也可以不需要通过 JNI 的方式进行， 直接加载 _.so 库（dlopen）的做法调用_.so 里的符号（symbol）也是 一种方式。

总而言之是没有经过封装，上层可以直接操作硬件。

![image-20200706134945797](https://gitee.com/cpu_code/picture_bed/raw/master//20200706134945.png)

![image-20200706135452501](https://gitee.com/cpu_code/picture_bed/raw/master//20200706135452.png)

新的HAL 架构\(libhardware\)

现在的 libhardware 架构，就有「stub」的味道了。 HAL stub 是一种代理人（proxy） 的概念， stub 虽然仍是以 \*.so的形式存在，但 HAL 已经将 \*.so 隐藏起来了。 Stub 向 HAL「提供」 操作函数（operations） ，而 runtime 则是向 HAL 取得特定模块（stub）的 operations，再 callback这些操作函数。 这种以 indirect function call 的架构，让HAL stub 变成是一种「包含」关系，即 HAL 里包含了许许多多的 stub（代理人） 。Runtime 只要说明「类型」，即module ID，就可以取得操作函数。

对于目前的HAL， 可以认为Android定义了HAL层结构框架，通过几个接口访问硬件从而统一了调用方式。

![image-20200706135547846](https://gitee.com/cpu_code/picture_bed/raw/master//20200706135547.png)

![image-20200706135611185](https://gitee.com/cpu_code/picture_bed/raw/master//20200706135611.png)

HAL\_legacy和HAL的对比

* HAL\_legacy：旧式的HAL是一个模块，采用**共享库**形式，在编译时会调用到。 由于采用 function call 形式调用，因此可被多个进程使用，但会被 mapping 到多个进程空间中， 同时需要考虑代码能否安全重入的问题（thread safe）。
* HAL： 新式的HAL采用 **HAL module** 和 **HAL stub** 结合形式，HAL stub不是一个share library，编译时上层只拥有访问HAL stub的函数指针， 并不需要HAL stub。 上层通过HAL module提供的统一接口获取并操作HAL stub，so文件只会被mapping到一个进程， 也不存在重复mapping和重入问题。

HAL\_legacy和HAL的对比

Module

角色是被调用 , 在Android架构里面仅作为程序库\(library\)用途

Stub

Stub又称为代理人\(proxy\) , 扮演操作供应者的概念\(operations provider\)

HAL\_legacy和HAL的对比

HAL stub的框架比较简单： 三个结构体、 两个常量、 一个函数

它的定义在：

```text
 hardware/libhardware/include/hardware/hardware.h
 hardware/libhardware/hardware.c
```

HAL Stub架构分析

```text
 /*
 每个一个HAL Stub都通过 hw_module_t 来描述， 我们称之为一个Stub对象。你可以去“ 继承 ”这个 hw_module_t，然后扩展自己的属性，这个HAL Stub对象必须定义为一个固定的名字：HMI( Hardware Module Information ) ，每一个Stub对象里都封装了一个函数open，通过这个函数来打开这个硬件Stub设备，返回这个硬件对应的Operation interface。
 */
 struct hw_module_t;
```

```text
 /*
 HAL Stub对象的open描述结构体， 它里面只有一个元素： open函数指针
 */
 struct hw_module_methods_t;
 ​
 /*
 由结构体hw_module_t可知，它的open函数返回一个硬件对应的Operation interface， 这个硬件的operation interface的描述由hw_device_t 结构体来描述
 */
 struct hw_device_t;
```

```text
 // 这个就是HAL Stub对象固定的名字
 #define HAL_MODULE_INFO_SYM HMI
 // 这个是以字符串的名字
 #define HAL_MODULE_INFO_SYM_AS_STR "HMI"
 //这个函数是通过硬件名来获得硬件HAL Stub对象
 int hw_get_module(const char *id, const struct hw_module_t **module);
```

```text
 //注册一个Stub对象
 const struct led_modulet_HAL_MODULE_INFO_SYM = {
     common: {
     tag: HARDWARE_MODULE_TAG,
     version_major: 1,
     version_minor: 0,
     id: LED_HARDWARE_MODULE_ID,
     name: "led HAL Stub",
     author: "farsight",
     methods: &led_module_methods,
     },
     // 扩展属性放在这儿
 }
```

![image-20200706135929366](https://gitee.com/cpu_code/picture_bed/raw/master//20200706135929.png)

![image-20200706135949036](https://gitee.com/cpu_code/picture_bed/raw/master//20200706135949.png)

### Android里的Jni调用机制

JVM 很好的解决java的可移植性问题，使得java软件可以在不同的硬件平台下运行，但是JVM不是万能的，毕竟它只是一个模拟环境，对于一些外围的硬件就很难模拟， JVM只是模拟了一部分的寄存器， 栈结构， JVM储存器，这些都是软件运行的基础硬件结构。

对于像android手机这样的平台， 有很多的外围设备在软件层是需要操作的，如wifi，蓝牙，触摸屏，重力感应，键盘等，如果直接通过java语言操作是很难实现的， JVM无法模拟这些硬件。

JNI\( Java Native Interface \)， 中文为JAVA本地调用。 从Java1.1开始，Java Native Interface\( JNI \)标准成为java平台的一部分，它允许Java代码 和 其他语言写的代码进行交互。 JNI一开始是为了本地已编译语言， 尤其是 C 和 C++ 而设计的，但是它并不妨碍你使用其他语言，只要调用约定受支持就可以了。使用 java 与本地已编译的代码交互，通常会丧失平台可移植性。

但是， 有些些情况下这样做是可以接受的， 甚至是必须的，比如， 使用一些旧的库，与硬件、操作系统进行交互，或者为了提高程序的性能。 JNI标准至少保证本地代码能工作在任何Java虚拟机实现下。

![image-20200706140038704](https://gitee.com/cpu_code/picture_bed/raw/master//20200706140038.png)

Jni在 Java和C、 C++等语言之间建立了一个**桥梁作用**，因此， JNI首先要做的，就是统一两者间的数据类型。

所有其它引用类型，在JNI中都被定义为 jobject 类型，在C中都定义为 void\*。

| Java Language Type | JNI Type |
| :--- | :--- |
| boolean | jboolean |
| byte | jbyte |
| char | jchar |
| short | jshort |
| int | jint |
| long | jlong |
| float | jfloat |
| double | jdouble |

Java中可以直接调用底层语言的函数或方法， Jni规定了Java调用底层语言的方法签名规范。

| 类型签名 | Java 类型 | 类型签名 | Java 类型 |
| :--- | :--- | :--- | :--- |
| Z | boolean | \[ | \[\] |
| B | byte | \[I | int\[\] |
| C | char | \[F | float\[\] |
| S | short | \[B | byte\[\] |
| I | int | \[C | char\[\] |
| J | long | \[S | short\[\] |
| F | float | \[D | double\[\] |
| D | double | \[J | long\[\] |
| L | fully-qualified-class（全限定的类） | \[Z | boolean\[\] |

Android里的Jni调用机制

* 函数签名通常是以下结构： 返回值 fun\(参数1，参数2，参数3） ;
* 其对应的Jni方法签名格式为： \(参数1参数2参数3\)返回值
* 注意：

函数名，在Jni中没有体现出来

参数列表相挨着， 中间没有逗号， 没有空格

返回值出现在（）后面

如果参数是引用类型， 那么参数应该为： L类型;

HAL Led调用流程

```text
/LedDemo/Led Service(java)/Led Service(Native)/Led HAL Stub1:创建服务对象<<create>>2:通过JNI初始化本地服务()3: 查找并得到HAL操作接口()4:返回device操作接口5:操作service提供的API()6: 通过JNI调用本地服务操作接口()7: 调用保存的devicer操作接口()8: 返回HAL操作结果9: 返回本地服务操作结果10: 返回框架服务操作结果/LedDemo/Led Service(java)/Led Service(Native)/Led HAL Stub
```

```text
/Led APP/Led Service(java)/Led Service(Native)/Led HAL Stub1:new LedService()2:system.loadLibrary()3: JNI_OnLoad()注册java方法与本地函数的映射关系4: _init()5: hw_get_module()6: Return module_t object7: callback module_t open interface()8: Return device_t operations9: LedService.set_on/set_off()10: _set_on, _set_off()11: devict_t->set_on/set_off()/Led APP/Led Service(java)/Led Service(Native)/Led HAL Stub
```

