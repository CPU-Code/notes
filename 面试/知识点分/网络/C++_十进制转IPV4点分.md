<!--
 * @Author: cpu_code
 * @Date: 2020-06-24 13:53:34
 * @LastEditTime: 2020-06-24 16:57:59
 * @FilePath: \md\面试\知识点分\网络\C++ 十进制 转IPV4点分.md
 * @Gitee: https://gitee.com/cpu_code
 * @CSDN: https://blog.csdn.net/qq_44226094
--> 
 * @Author: cpu_code
 * @Date: 2020-06-24 13:53:34
 * @LastEditTime: 2020-06-24 16:57:51
 * @FilePath: \md\面试\知识点分\网络\C++ 十进制 转IPV4点分.md
 * @Gitee: https://gitee.com/cpu_code
 * @CSDN: https://blog.csdn.net/qq_44226094

# 粉丝不过W

## 4字节十进制数 转为 IPV4点分十进制 -- C++语言实现

```c++
//IPv4表示，通过4字节整数，比如1 6777 2418(0x0A 00 01 02)，表示10.0.1.2
// 4字节 = 4 * 1字节 = 4 * 8bit = 32 bit
// 2 ^32 = 1024 * 1024 *1024 * 4 = 42 9496 7296 = 0xff ff ff ff + 1
//写出一个函数，实现输入一个四字节整数，使用字符模式输出ip地址
/****note: linux 走不通 , 求建议*****/
#include <iostream>
#include <string.h>

using namespace std;

char* fun(unsigned int num)
{
    int ip_int_Split[4] = {0};
    char ip_str_Split[4][4] = {0};
    char * ip = NULL; 
    int i = 0;
    int len = 0;

     //ip_int_Split[0] = (num >> 24)& 0x000000FF;
     //ip_int_Split[1] = (num >> 16) & 0x000000FF;
     //ip_int_Split[2] = (num >> 8) & 0x000000FF;
     //ip_int_Split[3] = num & 0x000000FF;
    for(i = 0; i < 4; i++)
    {
    	ip_int_Split[i] = (num >> ((3 - i) * 8)) & 0x000000FF;
    }
 
    for(i = 0; i < 4; i++)
    {
        //把整数转换成字符串，并返回指向转换后的字符串的指针
    	itoa(ip_int_Split[i], ip_str_Split[i], 10);
        //字符串追加
    	strcat(ip_str_Split[i], ".");
    	strcat(ip, ip_str_Split[i]);
    }

    len = strlen(ip);
    *(ip + len - 1) = '/0';

    return ip;
}

int main()
{ 
    char rip[16] = {0};
    char* ip = NULL;

    ip = fun(167772418);
    strcpy(rip, ip);
    cout << rip << endl;

    return 0;
}
```

