# Linux使用

## Linux 系统简介

### linux

Linux 就是一个操作系统，与 Windows 和 Mac OS 一样是**操作系统** 。

操作系统在整个计算机系统中的角色 :

![&#x64CD;&#x4F5C;&#x7CFB;&#x7EDF;](https://gitee.com/cpu_code/picture_bed/raw/master//20200728193528.jpeg)

Linux 主要是 **系统调用** 和 **内核** 那两层。使用的操作系统还包含一些在其上运行的应用程序，比如vim、google、vscode等。

### Linus Torvalds

Linux 之父，芬兰赫尔辛基大学

![&#x5BF9;&#xFF0C;&#x8FD9;&#x5C31;&#x662F;&#x73B0;&#x5728;&#x7684;&#x6797;&#x7EB3;&#x65AF;&#xB7;&#x6258;&#x74E6;&#x5179;&#xFF0C;&#x4ED6;&#x559C;&#x6B22;&#x5728;&#x6F14;&#x8BB2;&#x5F53;&#x4E2D;&#x9A82;&#x810F;&#x8BDD;&#xFF0C;&#x8FD8;&#x6709;&#x7AD6;&#x4E2D;&#x6307;](https://gitee.com/cpu_code/picture_bed/raw/master//20200728195438.webp)

### Linux 与 Windows 的不同

#### Linux

* 稳定的系统
* 安全性和漏洞的快速修补
* 多用户
* 用户和用户组的规划
* 相对较少的系统资源占用
* 可定制裁剪，移植到嵌入式平台（如安卓设备）
* 可选择的多种图形用户界面（如 GNOME，KDE）

#### Windows

* 特定的支持厂商
* 足够的游戏娱乐支持度
* 足够的专业软件支持度

## 基本概念及操作

### linux 终端

### 终端的概念

通常我们在使用 `Linux` 时，并不是直接与系统打交道，而是通过一个叫做 `Shell` 的中间程序来完成的，在图形界面下为了实现让我们在一个窗口中完成用户输入和显示输出，Linux 系统还提供了一个叫做**终端模拟器**的程序（`Terminal`）。

![image-20200728195948642](https://gitee.com/cpu_code/picture_bed/raw/master//20200728195948.png)

注意的的**终端**（Terminal）和 **控制台**（Console）是有区别的。

终端本质上是对应着 Linux 上的 `/dev/tty` 设备，Linux 的多用户登录就是通过不同的 `/dev/tty` 设备完成的，

Linux 默认提供了 6 个 界面的 “`terminal`”来让用户登录。

在物理机系统上你可以通过使用`Ctrl+Alt+F1～F6`进行切换

![image-20200728200241440](https://gitee.com/cpu_code/picture_bed/raw/master//20200728200559.png)

![image-20200728200301134](https://gitee.com/cpu_code/picture_bed/raw/master//20200728200701.png)

![image-20200728200348370](https://gitee.com/cpu_code/picture_bed/raw/master//20200728200348.png)

![image-20200728200409866](https://gitee.com/cpu_code/picture_bed/raw/master//20200728200409.png)

![image-20200728200426011](https://gitee.com/cpu_code/picture_bed/raw/master//20200728200426.png)

![image-20200728200441720](https://gitee.com/cpu_code/picture_bed/raw/master//20200728200441.png)

### Shell

不同发行版的各种**终端模拟器**，而是这个 `Shell`（ 壳 ）。

有壳就有核，核就是指 `UNIX/Linux` 内核，Shell\` 是 指“ 提供给使用者使用界面 ”的软件（**命令解析器**）. Shell 隐藏了操作系统底层的细节。

UNIX/Linux 操作系统下的 `Shell` 既是**用户交互的界面**，也是**控制系统的脚本语言**。

Ubuntu 终端默认使用的是 `bash`

### 命令行操作体验

在 linux 中，最最重要的就是**命令**，这就包含了 2 个过程，**输入**和**输出**

* 输入：输入当然就是**打开终端**，然后按**键盘输入**，然后按**回车**
* 输出：输出会返回你想要的**结果**，比如你要看什么文件，就会返回文件的**内容**。执行失败会告诉你哪里错了，执行成功就没有输出

linux 的哲学就是：没有结果就是**最好的结果**

```bash
 # 查看当前目录下的文件
ls

#创建一个名为 test 的文件
touch test

#进入一个目录
cd /etc/

#查看当前所在目录
pwd
```

![image-20200728185847192](https://gitee.com/cpu_code/picture_bed/raw/master//20200728185847.png)

### 重要快捷键：

```text
Tab
```

使用`Tab`键来进行**命令补全**，**补全目录**、**补全命令参数** , 当你忘记某些全称时 , 只要输入它的开头的一部分，然后按下`Tab`键就可以看见提示或者帮助完成

```bash
Ctrl+c
```

当你在 Linux 命令行中无意输入**不知道的命令**，或 使用**错误命令**，导致在终端里出现了你无法预料的情况，比如，屏幕上只有光标在闪烁却无法继续输入命令，或 不停地输出一大堆垃圾结果。

想要立即停止 , 就可以使用`Ctrl+c`键来 发送 `SIGINT` 信号给前台进程组中的所有进程，强制终止程序的执行

```text
tail
```

![image-20200728185953563](https://gitee.com/cpu_code/picture_bed/raw/master//20200728185953.png)

### 其他常用快捷键

| 按键 | 作用 |
| :--- | :--- |
| `Ctrl+d` | 键盘输入结束或退出终端 |
| `Ctrl+s` | 暂停当前程序，暂停后按下任意键恢复运行 |
| `Ctrl+z` | 将当前程序放到后台运行，恢复到前台为命令`fg` |
| `Ctrl+a` | 将光标移至输入行头，相当于`Home`键 |
| `Ctrl+e` | 将光标移至输入行末，相当于`End`键 |
| `Ctrl+k` | 删除从光标所在位置到行末 |
| `Alt+Backspace` | 向前删除一个单词 |
| `Shift+PgUp` | 将终端显示向上滚动 |
| `Shift+PgDn` | 将终端显示向下滚动 |

### 利用历史输入命令

可以使用键盘上的方向上键`↑`，恢复你之前输入过的命令

### 使用通配符

**通配符**是一种特殊语句，主要有星号（`*`）和问号（`?`），用来对字符串进行**模糊匹配**（比如文件名、参数名）。

```bash
# 回到用户家目录
cd /home/cpucode
```

![image-20200728190318106](https://gitee.com/cpu_code/picture_bed/raw/master//20200728190318.png)

```bash
# 进入目录
cd code/

# 查看目录
ls

# 进入目录
cd test/

# 查看目录
ls
```

```bash
# 创建 2 个文件，后缀都为 txt
touch test.txt test2.txt
```

```bash
# 使用通配符找所有的 .txt文件
ls *.txt
```

![image-20200728190434413](https://gitee.com/cpu_code/picture_bed/raw/master//20200728190434.png)

```bash
# 创建多个文件
touch test_{1..6}_cpucode.txt
```

![image-20200728190610224](https://gitee.com/cpu_code/picture_bed/raw/master//20200728190610.png)

Shell 常用通配符：

| 字符 | 含义 |
| :--- | :--- |
| `*` | 匹配 0 或多个字符 |
| `?` | 匹配任意一个字符 |
| `[list]` | 匹配 list 中的任意单一字符 |
| `[^list]` | 匹配 除 list 中的任意单一字符以外的字符 |
| `[c1-c2]` | 匹配 c1-c2 中的任意单一字符 如：\[0-9\]\[a-z\] |
| `{string1,string2,...}` | 匹配 string1 或 string2 \(或更多\)其一字符串 |
| `{c1..c2}` | 匹配 c1-c2 中全部字符 如{1..10} |

### 在命令行中获取帮助

在 Linux 环境中，如果你遇到困难，可以使用`man`命令，它是`Manual pages`的缩写。

Manual pages 是 UNIX 或类 UNIX 操作系统中**在线软件文档**的一种普遍的形式， 内容包括计算机程序（包括 **库** 和**系统调用**）、**正式的标准**和**惯例**，甚至是抽象的概念。用户可以通过执行`man`命令调用手册页。

```bash
# 查看 man 命令本身的使用方式
man man
```

![image-20200728190737552](https://gitee.com/cpu_code/picture_bed/raw/master//20200728190737.png)

![image-20200728190723177](https://gitee.com/cpu_code/picture_bed/raw/master//20200728190723.png)

为了便于查找，man 手册被进行了**分册**（分区段）处理，

| 区段 | 说明 |
| :--- | :--- |
| 1 | 一般命令 |
| 2 | 系统调用 |
| 3 | 库函数，涵盖了 C 标准函数库 |
| 4 | 特殊文件（通常是/dev 中的设备）和驱动程序 |
| 5 | 文件格式和约定 |
| 6 | 游戏和屏保 |
| 7 | 杂项 |
| 8 | 系统管理命令和守护进程 |

在 man 后面加上**相应区段的数字**

```bash
man 1 ls
```

![image-20200728190820767](https://gitee.com/cpu_code/picture_bed/raw/master//20200728190820.png)

显示第一区段中的`ls`命令 man 页面:

![image-20200728190834722](https://gitee.com/cpu_code/picture_bed/raw/master//20200728190834.png)

所有的手册页遵循一个常见的布局，为了通过简单的 ASCII 文本展示而被优化，而这种情况下可能没有任何形式的高亮或字体控制。

一般包括以下部分内容：

NAME（名称）

> 该命令或函数的名称，接着是一行简介。

SYNOPSIS（概要）

> 对于命令，正式的描述它如何运行，以及需要什么样的命令行参数。对于函数，介绍函数所需的参数，以及哪个头文件包含该函数的定义。

DESCRIPTION（说明）

> 命令或函数功能的文本描述。

EXAMPLES（示例）

> 常用的一些示例。

SEE ALSO（参见）

> 相关命令或函数的列表。

常见的例子包括

* OPTIONS（选项）
* EXIT STATUS（退出状态）
* ENVIRONMENT（环境）
* BUGS（程序漏洞）
* FILES（文件）
* AUTHOR（作者）
* REPORTING BUGS（已知漏洞）
* HISTORY（历史）
* COPYRIGHT（版权）

man

```bash
# 搜索
/<关键字>
```

```bash
# 切换到下一个关键字所在处
n
```

```bash
# 为上一个关键字所在处
shift + n
```

```bash
# （空格键）翻页
Space
```

```bash
# （回车键）向下滚动一行
Enter
```

```bash
# （vim 编辑器的移动键）
# 向上滚动一行
k
# 向下滚动一行
j
```

```bash
# 显示使用帮助
h
```

```bash
# 退出
q
```

查看 命令的具体参数的作用 :

```bash
ls --help
```

![image-20200728190925414](https://gitee.com/cpu_code/picture_bed/raw/master//20200728190925.png)

### 输出图形字符

```bash
# 更新一下源
sudo apt-get update
```

![image-20200728191549620](https://gitee.com/cpu_code/picture_bed/raw/master//20200728191549.png)

```bash
# 安装 figlet软件
sudo apt-get install figlet
```

![image-20200728192301137](https://gitee.com/cpu_code/picture_bed/raw/master//20200728192301.png)

```bash
# 显示cpucode
figlet cpucode
```

![image-20200728192237837](https://gitee.com/cpu_code/picture_bed/raw/master//20200728192237.png)

```bash
# 卸载 figlet
sudo apt autoremove figlet
```

![image-20200728192344308](https://gitee.com/cpu_code/picture_bed/raw/master//20200728192344.png)

```bash
# 再次使用无效
figlet cpucode
```

![image-20200728192516058](https://gitee.com/cpu_code/picture_bed/raw/master//20200728192516.png)

## 用户及文件权限管理

### Linux 用户管理

Linux 是一个可以实现**多用户登录**的操作系统

比如“小明”和“小亮”都可以同时登录同一台主机，他们共享一些主机的资源，但他们也分别有自己的**用户空间**，用于存放各自的文件。

但实际上他们的文件都是放在同一个物理磁盘上的甚至同一个逻辑分区或者目录里，但是由于 Linux 的 **用户管理** 和 **权限机制**，不同用户不可以轻易地查看、修改彼此的文件。

#### 查看用户

请打开终端，输入命令：

```bash
who am i
# 或
who mom likes
```

第一 : 打开当前伪终端的用户的**用户名**（要查看当前登录用户的用户名，去掉空格直接使用 `whoami` 即可），

第二 : `pts/0` 中 `pts` 表示**伪终端**，所谓**伪**是相对于 `/dev/tty` 设备而言的，这是“ 真终端 ” , 终端时的那七个使用 `[Ctrl+Alt+F1～F7` 进行切换的 `/dev/tty` 设备

第三 : 当前伪终端的**启动时间**

![image-20200728222026631](https://gitee.com/cpu_code/picture_bed/raw/master//20200728222026.png)

`who` 命令常用参数 :

| 参数 | 说明 |
| :---: | :---: |
| `-a` | 打印能打印的全部 |
| `-d` | 打印死掉的进程 |
| `-m` | 同`am i`，`mom likes` |
| `-q` | 打印当前登录用户数及用户名 |
| `-u` | 打印当前登录用户登录信息 |
| `-r` | 打印运行等级 |

#### 创建用户

在 `Linux` 系统里， `root` 账户拥有整个系统至高无上的**权限**，如 : 删库跑路。

`root` 是 Linux 和 UNIX 系统中的**超级管理员用户**帐户，该帐户拥有整个系统至高无上的权力，可以肆意操作

比如 : 安卓手机（基于 Linux 内核）获得 root 权限之后就相当于获得了手机的最高权限，可以对手机中的任何文件操作。

一般登录ubuntu系统时都是以**普通账户**的身份登录的，要创建用户需要 root 权限，这里就要用到 `sudo` 这个命令了。

**su，su- 与 sudo**

Linux 环境下输入**密码是不会显示**的

`su <user>` 可以切换到用户 user，执行时需要输入目标用户的密码，

`sudo <cmd>` 可以以特权级别运行 cmd 命令，需要当前用户属于 sudo 组，且需要输入当前用户的密码。

`su - <user>` 命令也是切换用户，但是同时用户的环境变量和工作目录也会跟着改变成目标用户所对应的。

创建 test 用户 ：

```bash
sudo adduser test
```

![image-20200729134646084](https://gitee.com/cpu_code/picture_bed/raw/master//20200729134646.png)

重新设置 test的用户密码 :

```bash
sudo passwd test
```

![image-20200729134659023](https://gitee.com/cpu_code/picture_bed/raw/master//20200729134659.png)

查看新用户在 /home 目录下创建一个工作目录：

```bash
ls /home
```

![image-20200729134740919](https://gitee.com/cpu_code/picture_bed/raw/master//20200729134741.png)

切换用户：

```bash
su -l test
```

![image-20200729134838360](https://gitee.com/cpu_code/picture_bed/raw/master//20200729134838.png)

查看用户终端：

```bash
who am i
```

查看用户名

```bash
whoami
```

查看当前路径

```bash
pwd
```

![image-20200729134919367](https://gitee.com/cpu_code/picture_bed/raw/master//20200729134919.png)

退出当前用户跟退出终端一样，可以使用 `exit` 命令 或者 使用快捷键 `Ctrl+D`。

![image-20200729135013163](https://gitee.com/cpu_code/picture_bed/raw/master//20200729135013.png)

![image-20200729134934701](https://gitee.com/cpu_code/picture_bed/raw/master//20200729134934.png)

#### 用户组

在 `Linux` 里面每个用户都有一个归属（**用户组**），用户组简单地理解就是一组用户的集合，它们共享一些资源和权限，同时拥有私有资源 .

比如 家，你的姐姐弟弟（不同的用户）属于同一个家（用户组），你们可以共同拥有这个家（共享资源），爸妈对待你们都一样（共享权限），你的私密的东西 ，别人未经允许不能查看（私有资源和权限）。

一个用户是可以属于多个用户组的，如 : 你既属于家庭，又属于学校 或 公司。

查看 用户的用户组

**groups 命令**

```bash
groups test
```

![groups](https://gitee.com/cpu_code/picture_bed/raw/master//20200826150136.png)

其中冒号之前表示用户，后面表示该用户所属的用户组。

这里可以看到 test用户属于 test用户组，每次新建用户如果不指定用户组的话，默认会自动创建一个与用户名相同的用户组（差不多就相当于家长的意思）。

默认情况下在 sudo 用户组里的可以使用 sudo 命令获得 root 权限。

test 用户也可以使用 sudo 命令，为什么这里没有显示在 sudo 用户组里呢？

可以查看下 `/etc/sudoers.d/test` 文件 :

![etc/sudoers.d](https://gitee.com/cpu_code/picture_bed/raw/master//20200826150724.png)

![etc/sudoers.d](https://gitee.com/cpu_code/picture_bed/raw/master//20200826150707.png)

我们在 `/etc/sudoers.d` 目录下创建了这个文件，

从而给 test 用户赋予了 sudo 权限：

**查看 /etc/group 文件**

```bash
cat /etc/group | sort
```

![image-20200826150625793](https://gitee.com/cpu_code/picture_bed/raw/master//20200826150626.png)

这里 `cat` 命令用于读取指定文件的内容并打印到终端输出，后面会详细讲它的使用。 `| sort` 表示将读取的文本进行一个字典排序再输出，然后你将看到如下一堆输出，你可以在最下面看到 shiyanlou 的用户组信息：

没找到？没关系，你可以使用 `grep` 命令过滤掉一些你不想看到的结果：

```bash
cat /etc/group | grep -E "test"
```

```bash
cat /etc/group | grep -E "cpucode"
```

![image-20200826150524508](https://gitee.com/cpu_code/picture_bed/raw/master//20200826150524.png)

**/etc/group 文件格式说明**

/etc/group 的内容包括用户组（Group）、用户组口令、GID（组 ID） 及该用户组所包含的用户（User），每个用户组一条记录。格式如下：

> group\_name : password : GID : user\_list

password 字段为一个 `x`，表示密码不可见

这里需要注意，如果用户的 GID == 用户组的 GID，那么最后一个字段 `user_list` 就是空的，

这里的 GID 是指用户默认所在组的 GID，可以使用 `id` 命令查看。

![id](https://gitee.com/cpu_code/picture_bed/raw/master//20200826152458.png)

**将其它用户加入 sudo 用户组**

默认情况下新创建的用户是不具有 root 权限的，也不在 sudo 用户组，可以让其加入 sudo 用户组从而获取 root 权限：

```bash
su -l test
```

```bash
sudo ls
```

![image-20200826152836633](https://gitee.com/cpu_code/picture_bed/raw/master//20200826152836.png)

会提示 test 不在 sudoers 文件中，意思就是 test 不在 sudo 用户组中

使用 `usermod` 命令可以为用户添加用户组，同样使用该命令你必需有 root 权限，你可以直接使用 root 用户为其它用户添加用户组，或者用其它已经在 sudo 用户组的用户使用 sudo 命令获取权限来执行该命令。

这里我用 cpucode 用户执行 sudo 命令将 test 添加到 sudo 用户组，让它也可以使用 sudo 命令获得 root 权限

切换回 cpucode 用户 :

```bash
su - cpucode
```

![image-20200826153604518](https://gitee.com/cpu_code/picture_bed/raw/master//20200826153604.png)

或

```bash
exit
```

![image-20200826153640481](https://gitee.com/cpu_code/picture_bed/raw/master//20200826153640.png)

或 直接关闭当前终端打开一个新的终端

```bash
groups test
```

```bash
sudo usermod -G sudo test
```

```bash
groups test
```

![image-20200826153314463](https://gitee.com/cpu_code/picture_bed/raw/master//20200826153314.png)

然后你再切换回 test 用户，现在就可以使用 sudo 获取 root 权限了

```bash
su - test
```

```bash
sudo ls /
```

![image-20200826154039642](https://gitee.com/cpu_code/picture_bed/raw/master//20200826154039.png)

#### 删除用户和用户组

```bash
sudo deluser test --remove-home
```

使用 `--remove-home` 参数在删除用户时候会一并将该用户的工作目录一并删除。

如果不使用那么系统会自动在 /home 目录为该用户保留工作目录

```bash
cat /etc/group | grep "test"
```

```bash
ls /home
```

这个地方有一个错误, 我搜索一下, 并不知道这个原因, 如果你知道, 希望你能告诉我, 谢谢 !

![image-20200826155822302](https://gitee.com/cpu_code/picture_bed/raw/master//20200826155822.png)

找到原因 :

删除这个用户

```bash
sudo userdel -r test
```

杀死这个进程

```bash
sudo kill -9 17490
```

![image-20200826160747068](https://gitee.com/cpu_code/picture_bed/raw/master//20200826160747.png)

![image-20200826160709455](https://gitee.com/cpu_code/picture_bed/raw/master//20200826160709.png)

### linux文件权限

文件权限就是文件的访问控制权限，即哪些用户和组群可以访问文件以及可以执行什么样的操作。

`Unix/Linux` 系统是一个典型的多用户系统，不同的用户处于不同的地位，对文件和目录有不同的访问权限。

\为了保护系统的安全性，Unix/Linux 系统除了对用户权限作了严格的界定外，还在用户身份认证、访问控制、传输安全、文件读写权限等方面作了周密的控制。

在 Unix/Linux 中的每一个文件或目录都包含有访问权限，这些访问权限决定了谁能访问和如何访问这些文件和目录。

#### 查看文件权限

`ls` 命令，用它来列出并显示当前目录下的文件

```bash
ls -l
```

![image-20200826161048127](https://gitee.com/cpu_code/picture_bed/raw/master//20200826161048.png)

![image-20200826162424821](https://gitee.com/cpu_code/picture_bed/raw/master//20200826162425.png)

文件类型和权限 :

![img](https://gitee.com/cpu_code/picture_bed/raw/master//20200826161132.png)

* 文件类型

关于文件类型，就必需时刻牢记 **Linux 里面一切皆文件**，就是因为这一点才有了设备文件（ `/dev` 目录下有各种设备文件，大都跟具体的硬件设备相关）这一说。

`socket`：网络套接字

`pipe` : 管道

`软链接文件`：链接文件是分为两种的，另一种是“ 硬链接 ”（软链接 等同于 Windows 上的快捷方式）。

* 文件权限

读权限，表示你可以使用 `cat <file name>` 之类的命令来读取某个文件的内容；

写权限，表示你可以编辑和修改某个文件的内容；

执行权限，通常指可以运行的二进制程序文件或者脚本文件，如同 Windows 上的 `exe` 后缀的文件，不过 Linux 上不是通过文件后缀名来区分文件的类型。

你需要注意的一点是，**一个目录同时具有读权限和执行权限才可以打开并查看内部文件，而一个目录要有写权限才允许在其中创建其它文件**，这是因为目录文件实际保存着该目录里面的文件的列表等信息。

所有者权限，这一点相信你应该明白了，至于所属用户组权限，是指你所在的用户组中的所有其它用户对于该文件的权限，

比如，你有一个 iPad，那么这个用户组权限就决定了你的兄弟姐妹有没有权限使用它破坏它和占有它。

* 链接数

> 链接到该文件所在的 inode 结点的文件名数目（关于这个概念涉及到 Linux 文件系统的相关概念知识，不在本课程的讨论范围，感兴趣的用户可以查看 [硬链接和软链接的联系与区别](https://www.ibm.com/developerworks/cn/linux/l-cn-hardandsymb-links/index.html#major2)）。

* 文件大小

> 以 inode 结点大小为单位来表示的文件大小，你可以给 ls 加上 `-lh` 参数来更直观的查看文件的大小。

明白了文件权限的一些概念，我们顺带补充一下关于 `ls` 命令的一些其它常用的用法：

* 显示除了 `.`（当前目录）和 `..`（上一级目录）之外的所有文件，包括隐藏文件（Linux 下以 `.` 开头的文件为隐藏文件）。

```bash
$ ls -a
```

![image-20200826161337430](https://gitee.com/cpu_code/picture_bed/raw/master//20200826161337.png)

当然，你可以同时使用 `-a` 和 `-l` 参数：

```bash
ls -al
```

![image-20200826161413484](https://gitee.com/cpu_code/picture_bed/raw/master//20200826161413.png)

查看某一个目录的完整属性，而不是显示目录里面的文件属性：

```bash
ls -dl <目录名>
```

显示所有文件大小，并以普通人类能看懂的方式呈现：

```bash
ls -asSh
```

![image-20200826161435442](https://gitee.com/cpu_code/picture_bed/raw/master//20200826161435.png)

其中小 s 为显示文件大小，大 S 为按文件大小排序，若需要知道如何按其它方式排序，可以使用 `man ls` 命令查询。

![image-20200826161523882](https://gitee.com/cpu_code/picture_bed/raw/master//20200826161524.png)

![image-20200826161536552](https://gitee.com/cpu_code/picture_bed/raw/master//20200826161536.png)

#### 变更文件所有者

#### 修改文件权限

\

