<!--
 * @Author: cpu_code
 * @Date: 2020-07-31 12:47:29
 * @LastEditTime: 2020-08-01 14:27:24
 * @FilePath: \notes\shell\condition.md
 * @Gitee: [https://gitee.com/cpu_code](https://gitee.com/cpu_code)
 * @Github: [https://github.com/CPU-Code](https://github.com/CPU-Code)
 * @CSDN: [https://blog.csdn.net/qq_44226094](https://blog.csdn.net/qq_44226094)
 * @Gitbook: [https://923992029.gitbook.io/cpucode/](https://923992029.gitbook.io/cpucode/)
--> 


<center>
<img src="https://s1.ax1x.com/2020/06/18/Nnpxmj.png" alt="Nnpxmj.png" title="Nnpxmj.png" />

## 条件测试语句



判断字符串是否相等， 可能还要检查文件状态或进行数字测试， 只有这些测试完成才能做下一步动作

test 命令： 用于测试**字符串**、 **文件状态**和**数字**

test 命令有两种格式:



```bash
test condition
```

使用方括号时， 要注意在条件两边加上空格

```bash
[ condition ]
```



shell 脚本中的条件测试如下：**文件测试**、 **字符串测试**、 **数字测试**、 **复合测试**



### 文件



文件测试： 测试文件状态的条件表达式  



| -e 是否存在 |    -d 是目录    |   -f 是文件   |
| :---------: | :-------------: | :-----------: |
|   -r 可读   |     -w 可写     |   -x 可执行   |
| -L 符号连接 | -c 是否字符设备 | -b 是否块设备 |
| -s 文件非空 |                 |               |



栗子 : test_file.sh



```bash
#!/bin/bash

test -e /dev/qaz
echo $?

test -e /home
echo $?

test -d /home
echo $?

test -f /home
echo $?

mkdir test_sh

chmod 500 test_sh
[ -r test_sh ]
echo $?

[ -w test_sh ]
echo $?

[ -x test_sh ]
echo $?

[ -s test_sh ]
echo $?

[ -c /dev/console ]
echo $?

[ -b /dev/sda ]
echo $?

[ -L /dev/stdin ]
echo $?
```



![image-20200801140435708](https://gitee.com/cpu_code/picture_bed/raw/master//20200801140435.png)



### 字符串  



字符串测试



```bash
test str_operator “str”

test “str1” str_operator “str2”

[ str_operator “str” ]
[ “str1” str_operator “str2” ]
```



其中 str_operator 可以是:

| = 两个字符串相等 | != 两个字符串不相等 |
| :--------------: | :-----------------: |
|     -z 空串      |      -n 非空串      |



栗子 : test_string.sh



```bash
#!/bin/bash

test -z $yn
echo $?

echo "please input a y/n"

read yn

[ -z "$yn" ]
echo 1:$?

[ $yn = "y" ]
echo 2:$?
```



![image-20200801140332504](https://gitee.com/cpu_code/picture_bed/raw/master//20200801140332.png)



### 数字  



测试数值格式如下:



```bash
test num1 num_operator num2
[ num1 num_operator num2 ]
```



num_operator 可以是 :



 数值相等

```bash
-eq
```



数值不相等

```bash
-ne
```



 数 1 大于数 2

```bash
-gt
```

 

数 1 大于等于数 2

```bash
-ge
```



数 1 小于等于数 2

```bash
-le 
```



 数 1 小于数 2  

```bash
-lt
```





![image-20200731125644070](https://gitee.com/cpu_code/picture_bed/raw/master//20200731125644.png)







栗子 : test_num.sh  



```bash
#!/bin/bash

echo "please input a num(1-9)"
read num

[ $num -eq 5 ]
echo $?

[ $num -ne 5 ]
echo $?

[ $num -gt 5 ]
echo $?

[ $num -ge 5 ]
echo $?

[ $num -le 5 ]
echo $?

[ $num -lt 5 ]
echo $?
```



![image-20200801140308980](https://gitee.com/cpu_code/picture_bed/raw/master//20200801140309.png)



### 复合测试



命令执行控制:  



`&&`



```bash
command1 && command2
```



&&左边命令（command1） 执行成功( 即返回 0 ）,  shell 才执行 && 右边的命令（command2）



 `||` 

```bash
command1 || command2
```



||左边的命令（command1） 执行失败 ( 即返回非 0 ）,  shell 才执行 || 右边的命令（command2）  





栗子 : 

```bash
test -e /home && test -d /home && echo "true"

test 2 -lt 3 && test 5 -gt 3 && echo "equal"

test "aaa" = "aaa" || echo "not equal" && echo "equal"
```





多重条件判定  



`-a`   



( and )两状况同时成立！ 同时具有 r 与 x 权限时， 才为 true.



```bash
test -r file -a -x file file 
```



`-o` 

(or)两状况任何一个成立！ 具有 r 或 x 权限时， 就传回 true.

```bash
test -r file -o -x file file 
```



! 

相反状态  ,  当 file 不具有 x 时， 回传 true

```bash
test ! -x file
```

> * @Author: cpu_code
> * @Date: 2020-07-31 09:46:09
> * @LastEditTime: 2020-08-01 14:25:26
> * @FilePath: \notes\shell\condition.md
> * @Gitee: [https://gitee.com/cpu_code](https://gitee.com/cpu_code)
> * @Github: [https://github.com/CPU-Code](https://github.com/CPU-Code)
> * @CSDN: [https://blog.csdn.net/qq_44226094](https://blog.csdn.net/qq_44226094)
> * @Gitbook: [https://923992029.gitbook.io/cpucode/](https://923992029.gitbook.io/cpucode/)





