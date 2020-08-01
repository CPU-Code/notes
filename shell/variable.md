

<center>
<img src="https://s1.ax1x.com/2020/06/18/Nnpxmj.png" alt="Nnpxmj.png" title="Nnpxmj.png" />



## 变量





### 自定义变量  



定义变量

变量名=变量值

```bash
num=10
```



引用变量

$变量名



  把变量 `num` 的值付给变量 i

```bash
 i=$num 
```



显示变量



使用 `echo` 命令可以显示单个变量取值



```bash
echo $num
```



清除变量

使用 `unset` 命令清除变量

```bash
unset varname
```



变量的其它用法：



从键盘输入一个字符串付给变量 string

```bash
read string
```



定义一个只读变量, 只能在定义时初始化,  以后不能改变,  不能被清除。

```bash
readonly var=100
```



使用 export 说明的变量， 会被导出为环境变量， 其它 shell 均可使用

```bash
export var=300
```



**注意**： 此时必须使用下面命令 才可以生效

```bash
source xxx.sh 
```





注意事项：



变量名只能包含英文字母下划线， 不能以数字开头



```bash
# 错误
1_num=10 

# 正确
num_1=20 
```



等号两边不能直接接空格符， 若变量中本身就包含了空格， 则整个字符串都要用双引号、 或单引号括起来；

 双引号内的特殊字符可以保有**变量特性**， 但是单引号内的特殊字符则仅为**一般字符**。



```bash
# 错误
name=aa bb  

# 正确
name="aa bb"  

# 输出： aa bb is me 
echo "$name is me"  

# 输出： $name is me 
echo '$name is me'
```



栗子 : var.sh



```bash
#!/bin/bash

echo "this is the var test shell script "
name="cpucode"

echo "$name is me"
echo '$name is me'
echo "please input a string"

read string
echo "the string of your input is $string"

readonly var=1000

#var=200

export public_var=300
```



![image-20200801125332281](https://gitee.com/cpu_code/picture_bed/raw/master//20200801125332.png)



### 环境变量  



`shell` 在开始执行时就已经定义了一些和系统的工作环境有关的变量， 我们在 shell 中可以直接使用`$name` 引用

定义：

一般在`~/.bashrc` 或`/etc/profile` 文件中（系统自动调用的脚本） 使用 `export` 设置， 

允许用户后来更改 , 一般， 所有环境变量均为大写  

```bash
 VARNAME=value;
 
 export VARNAME
```



显示所有的环境变量

```bash
env
```



清除环境变量

```bash
unset
```



常见环境变量：



保存注册目录的完全路径名

```bash
HOME
```



保存用冒号分隔的目录路径名， shell 将按 PATH 变量中给出的顺序搜索这些目录， 找到的第一个与命令名称一致的可执行文件将被执行。

```bash
PATH：
```



```bash
PATH=$HOME/bin:/bin:/usr/bin;

export PATH
```

主机名

```bash
HOSTNAME
```

默认的 shell 命令解析器

```bash
SHELL
```

此变量保存登录名

```bash
LOGNAME
```



当前工作目录的绝对路径名  

```bash
PWD
```





栗子 : export2.sh  

```bash
#!/bin/bash

echo "You are welcome to use bash"

echo "Current work dirctory is $PWD"
echo "the host name is $HOSTNAME"
echo "your home dir $HOME"
echo "Your shell is $SHELL"
```



![image-20200801125413452](https://gitee.com/cpu_code/picture_bed/raw/master//20200801125413.png)



### 预设变量  



传给 shell 脚本参数的数量

```bash
$#
```

传给 shell 脚本参数的内容

```bash
$*
```



运行脚本时传递给其的参数， 用空格隔开

```bash
$1、 $2、 $3、 ...、 $9
```



 命令执行后返回的状态 

用于检查上一个命令执行是否正确 ( 在 Linux 中， 命令退出状态为 `0` 表示该命令**正确执行**， 任何非 0 值表示**命令出错**)。

```bash
$?
```



当前执行的进程名

```bash
`$0`
```



当前进程的进程号

变量最常见的用途是用作**临时文件**的名字以保证临时文件不会重复  

```bash
`$$`
```





栗子 : $.sh

```bash
#!/bin/bash

echo "your shell script name is $0"
echo "the params of your input is $*"

echo "the num of your input params is $#"
echo "the params is $1 $2 $3 $4"

ls
echo "the cmd state is $?"

cd /root

echo "the cmd state is $?"
echo "process id is $$"
```



![image-20200801125502921](https://gitee.com/cpu_code/picture_bed/raw/master//20200801125503.png)





### 脚本变量的特殊用法： "" `` ' \ () {}  





（双引号）： 包含的变量会被解释

```bash
""
```



（单引号）： 包含的变量会当做字符串解释

```bash
''
```

反引号)： 反引号中的内容作为系统命令， 并执行其内容， 可以替换输出为一个变量

```bash
``
```



```bash
$ echo "today is `date` "
```



 转义字符：

同 c 语言 \n \t \r \a 等 echo 命令需加-e 转义

```bash
\
```



( 命令序列 )：

由子 shell 来完成,  不影响当前 shell 中的变量

```bash
( )
```



{ 命令序列 }：

在当前 shell 中执行， 会影响当前变量  

```bash
{ }
```





栗子 : var_spe.sh  

```bash
#!/bin/bash

name=teacher

string1="good moring $name"
string2='good moring $name'

echo $string1
echo $string2

echo "today is `date` "
echo 'today is `date` '
echo -e "this \n is\ta\ntest"

( 
	name=student;
	echo "1 $name" 
)

echo 1:$name

{
	name=student; 
	echo "2 $name"; 
}

echo 2:$name
```



![image-20200801125540280](https://gitee.com/cpu_code/picture_bed/raw/master//20200801125540.png)