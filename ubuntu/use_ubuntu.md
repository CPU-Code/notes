<!--
 * @Author: cpu_code
 * @Date: 2020-07-29 21:15:11
 * @LastEditTime: 2020-07-30 16:52:50
 * @FilePath: \notes\ubuntu\use_ubuntu.md
 * @Gitee: [https://gitee.com/cpu_code](https://gitee.com/cpu_code)
 * @Github: [https://github.com/CPU-Code](https://github.com/CPU-Code)
 * @CSDN: [https://blog.csdn.net/qq_44226094](https://blog.csdn.net/qq_44226094)
 * @Gitbook: [https://923992029.gitbook.io/cpucode/](https://923992029.gitbook.io/cpucode/)
--> 

<center>
<img src="https://s1.ax1x.com/2020/06/18/Nnpxmj.png" alt="Nnpxmj.png" title="Nnpxmj.png" />



## 安装ubuntu



ubuntu的镜像 : 

http://mirrors.aliyun.com/ubuntu-releases/



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200730164412.png" alt="image-20200730164412170" style="zoom:67%;" />



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200730164453.png" alt="image-20200730164453553" style="zoom:67%;" />





进入 vmware 



https://www.vmware.com/products/workstation-player/workstation-player-evaluation.html





<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200730164827.png" alt="image-20200730164733336" style="zoom: 50%;" />



点击 创建 

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200730141335.png" alt="image-20200730141335265" style="zoom:67%;" />





浏览 找到 系统镜像文件, 我把它放在了 vmware文件下



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200730141419.png" alt="image-20200730141419907" style="zoom:67%;" />



设置好信息 , 记住好密码 每次开机都要输入

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200730141530.png" alt="image-20200730141530945" style="zoom:67%;" />



解压在那 设置好位置, 我设在 vmware 下创建了一个文件下

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200730141618.png" alt="image-20200730141618328" style="zoom:67%;" />



默认即可

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200730141658.png" alt="image-20200730141658591" style="zoom:67%;" />



默认即可

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200730141719.png" alt="image-20200730141719673" style="zoom:67%;" />





正在安装中, 时间比较久, 准备好咖啡 ,  谈谈心



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200730141159.png" alt="image-20200730141159390" style="zoom:50%;" />



输入设置好的密码



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200730141815.png" alt="image-20200730141815631" style="zoom:50%;" />



## 换源





进来的第一件事就是**换源,** 我准备换 阿里源  ,  点击那个9点

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200730141958.png" alt="image-20200730141958236" style="zoom:67%;" />





点 server for united states 

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200730142123.png" alt="image-20200730142123770" style="zoom:67%;" />



点击 other 其他

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200730142257.png" alt="image-20200730142257136" style="zoom:67%;" />



找到 china 

![image-20200730142359620](https://gitee.com/cpu_code/picture_bed/raw/master//20200730142359.png)



输入密码



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200730142432.png" alt="image-20200730142432286" style="zoom: 67%;" />



点 X 就可以了

![image-20200730142459685](https://gitee.com/cpu_code/picture_bed/raw/master//20200730142459.png)



点 关闭 close

![image-20200730142537825](https://gitee.com/cpu_code/picture_bed/raw/master//20200730142537.png)





打开终端

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200730142621.png" alt="image-20200730142621049" style="zoom:67%;" />

进行更新源

```bash
sudo apt-get update
```





<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200730142709.png" alt="image-20200730142709783" style="zoom:67%;" />





## ubuntu设置语言



点 manage installed languages 在进入 language support 点击 install

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200730143040.png" alt="image-20200730143040751" style="zoom:67%;" />







点击 install/remove languages 里面找到 chinese





<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200730143200.png" alt="image-20200730143200178" style="zoom:67%;" />



勾选 点击 apply

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200730143217.png" alt="image-20200730143217832" style="zoom:80%;" />





正在下载语言 , 喝口82年咖啡 , 等待



![image-20200730100557569](https://gitee.com/cpu_code/picture_bed/raw/master//20200730100557.png)



下翻, 找到中国 用鼠标拖到最前面

![image-20200730143451380](https://gitee.com/cpu_code/picture_bed/raw/master//20200730143451.png)



点击 apply system-wide

![image-20200730143604074](https://gitee.com/cpu_code/picture_bed/raw/master//20200730143604.png)



点击 中国 然后 点 apply system-wide , 关闭即可

![image-20200730143643401](https://gitee.com/cpu_code/picture_bed/raw/master//20200730143643.png)



进入终端 重启



```bash
sudo reboot
```



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200730143803.png" alt="image-20200730143803567" style="zoom:67%;" />



重新进入系统 , 建议保留旧名 



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200730143903.png" alt="image-20200730143903210" style="zoom:80%;" />



### 下载 NFS 服务





```bash
sudo apt-get update
```



```bash
sudo apt-get install nfs-kernel-server rpcbind
```



![image-20200730101401834](https://gitee.com/cpu_code/picture_bed/raw/master//20200730101401.png)





### 下载SSH服务

```bash
sudo apt-get install openssh-server
```



![image-20200730101500815](https://gitee.com/cpu_code/picture_bed/raw/master//20200730101500.png)



### 下载 vim 编辑软件



```bash
sudo apt-get install vim
```



![image-20200730102008031](https://gitee.com/cpu_code/picture_bed/raw/master//20200730102008.png)



设置 vim 参数

```bash
sudo vim /etc/vim/vimrc
```



![image-20200730104338664](https://gitee.com/cpu_code/picture_bed/raw/master//20200730104338.png)



```shell
" 行号
set number
" 高亮
syntax on
" 底部显示
set showmode
" 编码
set encoding=utf-8
" 鼠标
set mouse=a
" tab = 4
set tabstop=4
```



![image-20200730104411362](https://gitee.com/cpu_code/picture_bed/raw/master//20200730104411.png)







### 下载 网络工具



```bash
sudo apt install net-tools
```



![image-20200730160626629](https://gitee.com/cpu_code/picture_bed/raw/master//20200730160626.png)

查看本机ip

```bash
ifconfig
```





![image-20200730160707963](https://gitee.com/cpu_code/picture_bed/raw/master//20200730160708.png)





## window10 下 操作



### 通过`xshell6`进行连接



软件可自行 google , 这个软件收费, 具体收不收费, 看你的大显神通了 , 点到为止



点击文件 , 点击新建

![image-20200730161120046](https://gitee.com/cpu_code/picture_bed/raw/master//20200730161120.png)

设置名字, 把查到的 ip 填在 主机





<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200730160817.png" alt="image-20200730160817133" style="zoom: 80%;" />



点击 用户身份验证

 输入ubuntu 的用户名

密码

点击连接



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200730160837.png" alt="image-20200730160837784" style="zoom: 67%;" />



成功就会出现这个弹窗, 点击 接受并保存



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200730160903.png" alt="image-20200730160903922" style="zoom:80%;" />







![image-20200730161328042](https://gitee.com/cpu_code/picture_bed/raw/master//20200730161328.png)



### 搭建 文件传输

点击 绿色 图标

![image-20200730161353569](https://gitee.com/cpu_code/picture_bed/raw/master//20200730161353.png)





点击取消, 就可以自动弹出

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200730161448.png" alt="image-20200730161448097" style="zoom:67%;" />



查看 命令解释

```bash
help
```



介绍常用的命令



把 本地数据 上传到 虚拟机

```bash
put
```



把 虚拟机的数据 下载到 本地

```bash
get
```







<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200730161524.png" alt="image-20200730161524257" style="zoom: 80%;" />

​	



### 下载交叉编译



https://releases.linaro.org/components/toolchain/binaries/latest-7/arm-linux-gnueabihf/  



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200730163112.png" alt="image-20200730163112883" style="zoom: 67%;" />



我把它下载到了本地桌面上了



打印本地位置

```bash
lpwd
```



切换本地目录

```bash
lcd
```



打印 虚拟机 文件

```bash
ls
```



![image-20200730161823031](https://gitee.com/cpu_code/picture_bed/raw/master//20200730161823.png)



切换虚拟机目录

```bash
cd cpucode/tools/
```



打印 虚拟机 文件

```bash
ls
```







把本地的 交叉编译器文件 上传到 虚拟机

```bash
put gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf.tar.xz
```





![image-20200730161929285](https://gitee.com/cpu_code/picture_bed/raw/master//20200730161929.png)



查看文件

```bash
ls
```



![image-20200730162747864](https://gitee.com/cpu_code/picture_bed/raw/master//20200730162747.png)





## 添加交叉编译器



创建目录： /usr/local/arm

```bash
sudo mkdir /usr/local/arm
```



把交叉编译器复制到/usr/local/arm 中  

```bash
sudo cp gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf.tar.xz /usr/local/arm/ -f
```



进入到 /usr/local/arm 目录  

```bash
cd /usr/local/arm/
```



对交叉编译工具进行解压  

```bash
sudo tar -vxf gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf.tar.xz
```





查看交叉编译器的路径

```bash
cd gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf/bin/
```



```bash
pwd
```







![image-20200730102135075](https://gitee.com/cpu_code/picture_bed/raw/master//20200730102135.png)









修改环境变量，使用 VIM 打开/etc/profile 文件  



```bash
sudo vim /etc/profile
```



![image-20200730102427600](https://gitee.com/cpu_code/picture_bed/raw/master//20200730102427.png)



添加交叉编译的路径 : 

```shell
export PATH=$PATH:/usr/local/arm/gcc-linaro-7.5.0-2019.12-x86_64_arm-linux-gnueabihf/bin
```



![image-20200730102410909](https://gitee.com/cpu_code/picture_bed/raw/master//20200730102411.png)





下载需要的库:

```bash
sudo apt-get install lsb-core lib32stdc++6
```



![image-20200730102551073](https://gitee.com/cpu_code/picture_bed/raw/master//20200730102551.png)



重启

```bash
sudo reboot
```





![image-20200730102713949](https://gitee.com/cpu_code/picture_bed/raw/master//20200730102713.png)





查看交叉编译器

```bash
arm-linux-gnueabihf-gcc -v
```



![image-20200730102809714](https://gitee.com/cpu_code/picture_bed/raw/master//20200730102809.png)



测试交叉编译器



写 test.c 程序

```bash
vim test.c
```



```bash
arm-linux-gnueabihf-gcc test.c
```





![image-20200730104447743](https://gitee.com/cpu_code/picture_bed/raw/master//20200730104447.png)







查看文件属性

```bash
file a.out
```



![image-20200730104257384](https://gitee.com/cpu_code/picture_bed/raw/master//20200730104257.png)





## 添加 qemu



### 安装KVM

安装kvm加速qemu运行，在终端下执行如下命令：



更新源 : 

```bash
sudo apt-get update
```



安装 kvm : 

```bash
sudo apt-get install qemu qemu-kvm libvirt-bin bridge-utils virt-manager
```







![image-20200730160235343](https://gitee.com/cpu_code/picture_bed/raw/master//20200730160235.png)



### 添加 git 工具



![image-20200730145352905](https://gitee.com/cpu_code/picture_bed/raw/master//20200730145352.png)





添加 韦东山老师的 qemu 的软件 

```bash
git  clone  https://e.coding.net/weidongshan/ubuntu-18.04_imx6ul_qemu_system.git
```



然后等待, 继续喝一口82年咖啡 



![image-20200730145446613](https://gitee.com/cpu_code/picture_bed/raw/master//20200730145446.png)



可以看看使用教程qemu的



   http://wiki.100ask.org/Qemu



```bash
./install_sdl.sh
```





![image-20200730145812647](https://gitee.com/cpu_code/picture_bed/raw/master//20200730145812.png)





```bash
./qemu-imx6ull-gui.sh
```







![image-20200730150843317](https://gitee.com/cpu_code/picture_bed/raw/master//20200730150843.png)



```bash
root
```





![image-20200730151021668](https://gitee.com/cpu_code/picture_bed/raw/master//20200730151021.png)



> * @Author: cpu_code
> * @Date: 2020-07-29 21:15:11
> * @LastEditTime: 2020-07-30 16:52:19
> * @FilePath: \notes\ubuntu\use_ubuntu.md
> * @Gitee: [https://gitee.com/cpu_code](https://gitee.com/cpu_code)
> * @Github: [https://github.com/CPU-Code](https://github.com/CPU-Code)
> * @CSDN: [https://blog.csdn.net/qq_44226094](https://blog.csdn.net/qq_44226094)
> * @Gitbook: [https://923992029.gitbook.io/cpucode/](https://923992029.gitbook.io/cpucode/)

