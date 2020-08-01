

<center>
<img src="https://s1.ax1x.com/2020/06/18/Nnpxmj.png" alt="Nnpxmj.png" title="Nnpxmj.png" />



## 控制语句





### if 控制语句

格式一：

```bash
if [条件 1]; then
	# 执行第一段程序
else
	# 执行第二段程序
fi  
```



格式二：  



```bash
if [条件 1]; then
	# 执行第一段程序
elif [条件 2]; then
	# 执行第二段程序
else
	# 执行第三段程序
fi
```



栗子 : if_then.sh  

```bash
#!/bin/bash

echo "Press y to continue"
read yn

if [ $yn = "y" ]; then
	echo "script is running..."
else
	echo "stopped!"
fi
```



![image-20200801125823200](https://gitee.com/cpu_code/picture_bed/raw/master//20200801125823.png)



### case 控制语句  



```bash
case $变量名称 in
	" 第一个变量内容 " )
	# 程序段一
	;;
	
	" 第二个变量内容 " )
	# 程序段二
	;;
	
	* )
	# 其它程序段
	exit 1
esac
```



栗子 : case1.sh  

```bash
#!/bin/bash

echo "This script will print your choice"

case "$1" in
    "one" )
    echo "your choice is one"
    ;;
    
    "two" )
    echo "your choice is two"
    ;;
    
    "three" )
    echo "Your choice is three"
    ;;
    
    * )
    echo "Error Please try again!"
    exit 1
    ;;
    
esac
```



![image-20200801125912801](https://gitee.com/cpu_code/picture_bed/raw/master//20200801125912.png)



栗子 : case2.sh  



```bash
#!/bin/bash

echo "Please input your choice:"

read choice
case "$choice" in
    Y | y | yes | Yes | YES )
    echo "It's right"
    ;;
    
    N* | n*)
    echo "It's wrong"
    ;;
    
    *)
    exit 1
esac
```



![image-20200801130318481](https://gitee.com/cpu_code/picture_bed/raw/master//20200801130318.png)



### for 控制语句  



形式一：  



初始值： 变量在循环中的起始值

限制值： 当变量值在这个限制范围内时， 就继续进行循环

执行步阶： 每作一次循环时， 变量的变化量  



```bash
for (( 初始值; 限制值; 执行步阶 ))
do
	# 程序段
done
```



declare 是 bash 的一个内建命令， 可以用来声明 shell 变量、 设置变量的属性。 declare 也可以写作 typeset。

declare -i s 代表强制把 s 变量当做 int 型参数运算。  





栗子 : for1.sh  



```bash
#!/bin/bash

declare -i sum

for (( i=1; i<=100; i=i+1 ))
do
	sum=sum+i
done

echo "The result is $sum"
```



![image-20200801130712549](https://gitee.com/cpu_code/picture_bed/raw/master//20200801130712.png)



形式二：  



第一次循环时， $var 的内容为 con1

第二次循环时， $var 的内容为 con2

第三次循环时， $var 的内容为 con3  

```bash
for var in con1 con2 con3 ...
do
	# 程序段
done
```





栗子 : for2.sh  

```bash
#!/bin/bash

for i in 1 2 3 4 5 6 7 8 9
do
	echo $i
done
```



![image-20200801130901078](https://gitee.com/cpu_code/picture_bed/raw/master//20200801130901.png)



栗子 : for3.sh  



```bash
#!/bin/bash

for name in `ls`
do

if [ -f $name ];then
	echo "$name is file"
elif [ -d $name ];then
	echo "$name is directory"
else
	echo "^_^"
fi

done
```



![image-20200801131046782](https://gitee.com/cpu_code/picture_bed/raw/master//20200801131046.png)





### while 控制语句  



当 condition 成立的时候进入 while 循环， 直到 condition 不成立时才退出循环。

```bash
while [ condition ]
do
	# 程序段
done

```



栗子 : while2.sh  

```bash
#!/bin/bash

declare -i i
declare -i s

while [ "$i" != "6" ]
do
	s+=i;
	i=i+1;
done

echo "The count is $s"
```



![image-20200801140136072](https://gitee.com/cpu_code/picture_bed/raw/master//20200801140136.png)



### unitl 控制语句  



这种方式与 while 恰恰相反， 当 condition 成立的时候退出循环， 否则继续循环。  

```bash
until [ condition ]
do
	#程序段
done
```



栗子 : until2.sh  

```bash
#!/bin/bash

declare -i i
declare -i s

until [ "$i" = "6" ]
do
	s+=i;
	i=i+1;
done

echo "The count is $s"
```



![image-20200801140158254](https://gitee.com/cpu_code/picture_bed/raw/master//20200801140158.png)

### break continue  



```bash
break
```



break 命令允许跳出循环。

break 通常在进行一些处理后退出循环或 case 语句



```bash
continue
```



continue 命令类似于 break 命令

只有一点重要差别， 它不会跳出循环， 只是跳过这个循环步  





## 函数  



有些脚本段间互相重复， 如果能只写一次代码块而在任何地方都能引用那就提高了代码的可重用性。

shell 允许将一组命令集或语句形成一个可用块， 这些块称为 shell 函数。  



定义函数的两种格式：  



格式一：  

```bash
函数名()
{
	# 命令 ...
}
```



格式二：  

```bash
function 函数名()
{
	# 命令 ...
}
```





函数可以放在同一个文件中作为一段代码， 也可以放在只包含函数的单独文件中

所有函数在使用前必须定义， 必须将函数放在脚本开始部分， 直至 shell 解释器首次发现它时， 才可以使用  





调用函数的格式为：  



函数名 param1 param2……  





使用参数同在一般脚本中使用特殊变量  



$1， $2 ...$9 一样  





函数可以使用 return 提前结束并带回返回值  





return 从函数中返回， 用最后状态命令决定返回值。

return 0 无错误返回

return 1 有错误返回  





栗子 : function.sh  



```bash
#!/bin/bash

is_it_directory()
{
	if [ $# -lt 1 ]; then
		echo "I need an argument"
		
		return 1
	fi 
	
	if [ ! -d $1 ]; then
		return 2
	else
		return 3
	fi
} 

echo -n "enter destination directory:"

read direct
is_it_directory $direct

echo "the result is $?"
```



![image-20200801140237198](https://gitee.com/cpu_code/picture_bed/raw/master//20200801140237.png)