<!--
 * @由于个人水平有限, 难免有些错误, 还请指点:  
 * @Author: cpu_code
 * @Date: 2020-08-27 12:55:01
 * @LastEditTime: 2020-08-27 13:19:23
 * @FilePath: \notes\ubuntu\use_samba.md
 * @Gitee: [https://gitee.com/cpu_code](https://gitee.com/cpu_code)
 * @Github: [https://github.com/CPU-Code](https://github.com/CPU-Code)
 * @CSDN: [https://blog.csdn.net/qq_44226094](https://blog.csdn.net/qq_44226094)
 * @Gitbook: [https://923992029.gitbook.io/cpucode/](https://923992029.gitbook.io/cpucode/)
-->
# 配置 Linux ubuntu 与 window 10 的共享文件 , 使用 samba


<center>
<img src="https://s1.ax1x.com/2020/06/18/Nnpxmj.png" alt="Nnpxmj.png" title="Nnpxmj.png" />

配置 samba 就进行 win 和Linux 进行文件访问 , 在win的环境下 就访问Linux的代码 , 在win的环境下进行 code 的编码 , 是不是很 nice !!!!

检查下更新

```bash
sudo apt-get upgrade
```



![image-20200827125614558](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9naXRlZS5jb20vY3B1X2NvZGUvcGljdHVyZV9iZWQvcmF3L21hc3Rlci8vMjAyMDA4MjcxMjU2MjEucG5n?x-oss-process=image/format,png)



```bash
 sudo apt-get update
```

![image-20200827125641998](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9naXRlZS5jb20vY3B1X2NvZGUvcGljdHVyZV9iZWQvcmF3L21hc3Rlci8vMjAyMDA4MjcxMjU2NDIucG5n?x-oss-process=image/format,png)



```bash
sudo apt-get dist-upgrade
```



![image-20200827125705303](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9naXRlZS5jb20vY3B1X2NvZGUvcGljdHVyZV9iZWQvcmF3L21hc3Rlci8vMjAyMDA4MjcxMjU3MDUucG5n?x-oss-process=image/format,png)



安装samba服务器

```bash
sudo apt-get install samba samba-common
```



![image-20200827125732914](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9naXRlZS5jb20vY3B1X2NvZGUvcGljdHVyZV9iZWQvcmF3L21hc3Rlci8vMjAyMDA4MjcxMjU3MzIucG5n?x-oss-process=image/format,png)



添加用户(下面的 cpucode 是我的用户名，之后会需要设置samba的密码 )

```bash
sudo smbpasswd -a cpucode
```



![image-20200827125759057](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9naXRlZS5jb20vY3B1X2NvZGUvcGljdHVyZV9iZWQvcmF3L21hc3Rlci8vMjAyMDA4MjcxMjU3NTkucG5n?x-oss-process=image/format,png)

在smb.conf文件最后边加入配置信息

```bash
sudo vim  /etc/samba/smb.conf
```



![image-20200827125838697](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9naXRlZS5jb20vY3B1X2NvZGUvcGljdHVyZV9iZWQvcmF3L21hc3Rlci8vMjAyMDA4MjcxMjU4MzgucG5n?x-oss-process=image/format,png)



配置文件smb.conf的最后添加下面的内容：

只要修改 cpucode 为你的用户名

```bash
[cpucode]
comment = cpucode folder
browseable = yes
path = /home/cpucode
create mask = 0700
directory mask = 0700
valid users = cpucode
force user = cpucode
force group = cpucode
public = yes
available = yes
writable = yes
```



![image-20200827125907818](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9naXRlZS5jb20vY3B1X2NvZGUvcGljdHVyZV9iZWQvcmF3L21hc3Rlci8vMjAyMDA4MjcxMjU5MDcucG5n?x-oss-process=image/format,png)



对配置进行了更改后，需要重启 samba 服务后更改的配置才会生效

```bash
sudo service smbd restart
```



![image-20200827125847295](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9naXRlZS5jb20vY3B1X2NvZGUvcGljdHVyZV9iZWQvcmF3L21hc3Rlci8vMjAyMDA4MjcxMjU4NDcucG5n?x-oss-process=image/format,png)

查看Linux ubuntu 的id地址

```bash
ifconfig
```

![image-20200827130155217](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9naXRlZS5jb20vY3B1X2NvZGUvcGljdHVyZV9iZWQvcmF3L21hc3Rlci8vMjAyMDA4MjcxMzAxNTUucG5n?x-oss-process=image/format,png)



在window系统中输入访问Linux 的 id 地址访问即可 : 

```bash
\\192.168.13.133
```



输入samba用户名及密码访问即可看到共享



```bash
用户 : cpucode

密码 : xxxxx
```



![image-20200827130045682](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9naXRlZS5jb20vY3B1X2NvZGUvcGljdHVyZV9iZWQvcmF3L21hc3Rlci8vMjAyMDA4MjcxMzAwNDUucG5n?x-oss-process=image/format,png)





![image-20200827130120932](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9naXRlZS5jb20vY3B1X2NvZGUvcGljdHVyZV9iZWQvcmF3L21hc3Rlci8vMjAyMDA4MjcxMzAxMjAucG5n?x-oss-process=image/format,png)



右击cpucode ,  点击 映射网络驱动器



![image-20200827131022638](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9naXRlZS5jb20vY3B1X2NvZGUvcGljdHVyZV9iZWQvcmF3L21hc3Rlci8vMjAyMDA4MjcxMzEwMjIucG5n?x-oss-process=image/format,png)



电脑目录下就有 

![image-20200827130753854](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9naXRlZS5jb20vY3B1X2NvZGUvcGljdHVyZV9iZWQvcmF3L21hc3Rlci8vMjAyMDA4MjcxMzA3NTMucG5n?x-oss-process=image/format,png)



 * @由于个人水平有限, 难免有些错误, 还请指点:  
 * @Author: cpu_code
 * @Date: 2020-08-27 12:55:01
 * @LastEditTime: 2020-08-27 13:16:45
 * @FilePath: \notes\ubuntu\use_samba.md
 * @Gitee: [https://gitee.com/cpu_code](https://gitee.com/cpu_code)
 * @Github: [https://github.com/CPU-Code](https://github.com/CPU-Code)
 * @CSDN: [https://blog.csdn.net/qq_44226094](https://blog.csdn.net/qq_44226094)
 * @Gitbook: [https://923992029.gitbook.io/cpucode/](https://923992029.gitbook.io/cpucode/)