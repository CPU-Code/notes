# shell 概述

![Nnpxmj.png](https://s1.ax1x.com/2020/06/18/Nnpxmj.png)

## shell 概述

shell 的两层含义： 既是一种**应用程序**, 又是一种**程序设计语言**

#### 作为应用程序：

交互式地解释、 执行用户输入的命令， 将用户的操作翻译成机器可以识别的语言， 完成相应功能 , 称之为 shell 命令解析器

`shell` 是 **用户** 和 **Linux 内核** 之间的接口程序

用户在提示符下输入的命令都由 `shell` 先解释然后传给 Linux 核心

它调用了系统核心的大部分功能来执行程序、 并以**并行**的方式协调各个程序的运行

Linux 系统中提供了好几种不同的 `shell` 命令解释器， 如 sh、 ash、 bash 等。 一般默认使用 `bash` 作为默认的解释器。

![image-20200731094726569](https://gitee.com/cpu_code/picture_bed/raw/master//20200731094726.png)

`shell` 是用户跟内核通信几种方式的一种

![image-20200731094738405](https://gitee.com/cpu_code/picture_bed/raw/master//20200731094738.png)

#### 作为程序设计语言：

它定义了各种变量和参数， 并提供了许多在高级语言中才具有的控制结构， 包括**循环**和**分支**完成类似于 windows 下批处理操作， 简化我们对系统的管理与应用程序的部署 , 称之为 `shell` 脚本

c/c++等语言， 属于**编译性语言**（编写完成后需要使用编译器完成编译、 汇编、 链接等过程变为二进制代码方可执行）

shell 脚本是一种**脚本语言**， 我们只需使用任意文本编辑器， 按照语法编写相应程序， 增加可执行权限， 即可在安装 shell 命令解释器的环境下执行

shell 脚本主要用于：

帮助开发人员或系统管理员将复杂而又反复的操作放在一个文件中， 通过简单的一步执行操作完成相应任务， 从而解放他们的负担

增加可执行权限 :

```bash
chmod +x xxx.sh
```

直接执行即可

```bash
./xxx.sh
```

shell 脚本大体可以分为两类：

**系统调用** :

这类脚本**无需用户调用**， 系统会在合适的时候调用

`/etc/profile`: 此文件为系统的每个用户设置环境信息, 当用户第一次登录时, 该文件被执行. 并从`/etc/profile.d`目录的配置文件中搜集`shell`的设置.

`/etc/bash.bashrc`: 为每一个运行`bash shell`的用户执行此文件. 当bash shell被打开时, 该文件被读取.

`~/.bashrc`: 该文件包含专用于你的bash shell的bash信息, 当登录时以及每次打开新的shell时, 该文件被读取.

`~/.bash_logout`: 当每次**退出**系统\( 退出bash shell \)时, 执行该文件.

另外, `/etc/profile` 中设定的变量\(**全局**\)的可以作用于任何用户, 而 `~/.bashrc` 等中设定的变量\(**局部**\) 只能继承 `/etc/profile` 中的变量, 他们是"父子"关系.

用户编写， 需要**手动调用**的 :

无论是系统调用的还是需要我们自己调用的， 其语法规则都一样

### shell 脚本的定义与执行

定义以开头：

```bash
#!/bin/bash
```

`\#!` 用来声明脚本由什么 `shell` 解释， 否则使用默认 `shell`

单个"`#`"号代表注释当前行

执行：

增加可执行权限后执行

```bash
chmod + x xxx.sh ./xxx.sh
```

直接指定使用 bash 解释 xxx.sh

```bash
bash xxx.sh
```

使用当前 shell 读取解释 xxx.sh

```bash
.xxx.sh

# 或 

source test.sh
```

三种执行脚本的方式不同点:

`./` 和 `bash` 执行过程基本一致，

前者首先检测 `#!`， 使用 `#!` 指定的 `shell`， 如果没有使用**默认**的 `shell`

后者明确指定 `bash` 解释器去执行脚本， 脚本中 `#!` 指定的解释器不起作用

用 `./` 和 `bash` 去执行会在后台启动一个新的 `shell` 去执行脚本

用 `.` 去执行脚本不会启动新的 `shell` , 直接由当前的 `shell` 去解释执行脚本

栗子 clear.sh :

```bash
#!/bin/bash

# 清屏
clear

# 打印
echo "this is the first shell script"
```

![image-20200731101132522](https://gitee.com/cpu_code/picture_bed/raw/master//20200731101132.png)

> * @Author: cpu\_code
> * @Date: 2020-07-31 09:46:09
> * @LastEditTime: 2020-08-01 14:25:26
> * @FilePath: \notes\shell\shell\_overview.md
> * @Gitee: [https://gitee.com/cpu\_code](https://gitee.com/cpu_code)
> * @Github: [https://github.com/CPU-Code](https://github.com/CPU-Code)
> * @CSDN: [https://blog.csdn.net/qq\_44226094](https://blog.csdn.net/qq_44226094)
> * @Gitbook: [https://923992029.gitbook.io/cpucode/](https://923992029.gitbook.io/cpucode/)

