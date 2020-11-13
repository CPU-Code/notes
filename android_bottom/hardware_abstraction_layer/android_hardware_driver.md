# 安卓硬件驱动

* @Author: cpu\_code
* @Date: 2020-07-15 11:51:27
* @LastEditTime: 2020-07-15 19:50:05
* @FilePath: \notes\android\_bottom\hardware\_abstraction\_layer\Android\_hardware\_driver.md
* @Gitee: [https://gitee.com/cpu\_code](https://gitee.com/cpu_code)
* @Github: [https://github.com/CPU-Code](https://github.com/CPU-Code)
* @CSDN: [https://blog.csdn.net/qq\_44226094](https://blog.csdn.net/qq_44226094)
* @Gitbook: [https://923992029.gitbook.io/cpucode/](https://923992029.gitbook.io/cpucode/)

Android系统的**硬件抽象层**（Hardware Abstract Layer， HAL） 运行在用户空间中， 它向下屏蔽硬件驱动模块的实现细节， 向上提供硬件访问服务。

Android系统的体系结构 :

![Android&#x7CFB;&#x7EDF;&#x7684;&#x4F53;&#x7CFB;&#x7ED3;&#x6784;](https://gitee.com/cpu_code/picture_bed/raw/master//20200713152328.png)

依次涉及Android系统的**硬件驱动模块**、 **硬件抽象层**、 **外部库** 和 **运行时库层**、 **应用程序框架层** 和 **应用程序层**等

开发一个应用访问硬件的流程为 : 首先在Android系统的**内核空间**中为一个**硬件开发驱动**程序， 接着在用户空间中为该硬件添加一个**硬件抽象层模块**， 并且在应用程序框架层中添加一个**硬件访问服务**， 最后开发一个应用程序来访问该硬件服务

![image-20200715144129859](https://gitee.com/cpu_code/picture_bed/raw/master//20200715144130.png)

## 开发Android硬件驱动程序

### 实现内核驱动程序模块

驱动程序`freg`的目录结构 :

```text
~/Android/kernel/goldfish
    drivers
        freg
            freg.h    # 源代码文件
            freg.c    # 源代码文件
            Kconfig    # 编译选项配置文件
            Makefile    # 编译脚本文件
```

`freg.h` 源代码文件 :

```c
// kernel\goldfish\drivers\freg\freg.h

#ifndef _FAKE_REG_H_
#define _FAKE_REG_H_

#include <linux/cdev.h>
#include <linux/semaphore.h>

//定义了四个字符串常量，分别用来描述虚拟硬件设备 freg 在设备文件系统中的名称
#define FREG_DEVICE_NODE_NAME  "freg"
#define FREG_DEVICE_FILE_NAME  "freg"
#define FREG_DEVICE_PROC_NAME  "freg"
#define FREG_DEVICE_CLASS_NAME "freg"

// 描述虚拟硬件设备freg
struct fake_reg_dev 
{
    // 描述一个虚拟寄存器
    int val;
    // 一个信号量, 用来同步访问虚拟寄存器 val
    struct semaphore sem;
    // 一个标准的Linux字符设备结构体变量， 用来标志该虚拟硬件设备 freg 的类型为字符设备
    struct cdev dev;
};

#endif
```

`freg.c` 实现文件 :

```c
// kernel\goldfish\drivers\freg\freg.c

#include <linux/init.h>
#include <linux/module.h>
#include <linux/types.h>
#include <linux/fs.h>
#include <linux/proc_fs.h>
#include <linux/device.h>
#include <asm/uaccess.h>

#include "freg.h"

/* 主设备号 */
static int freg_major = 0;
/* 从设备号变量 */
static int freg_minor = 0;

/* 设备类别 */
static struct class* freg_class = NULL;
/* 设备变量 */
static struct fake_reg_dev* freg_dev = NULL;

/* 传统的设备文件操作方法 */
static int freg_open(struct inode* inode, struct file* filp);
static int freg_release(struct inode* inode, struct file* filp);
static ssize_t freg_read(struct file* filp, char __user *buf, size_t count, loff_t* f_pos);
static ssize_t freg_write(struct file* filp, const char __user *buf, size_t count, loff_t* f_pos);

/* 传统的设备文件操作方法表 */
static struct file_operations freg_fops = {
        .owner = THIS_MODULE,
        .open = freg_open,
        .release = freg_release,
        .read = freg_read,
        .write = freg_write,
};

/* devfs 文件系统的设备属性操作方法 */
static ssize_t freg_val_show(struct device* dev, struct device_attribute* attr,  char* buf);
static ssize_t freg_val_store(struct device* dev, struct device_attribute* attr, const char* buf, size_t count);

/* devfs 文件系统的设备属性 */
static DEVICE_ATTR(val, S_IRUGO | S_IWUSR, freg_val_show, freg_val_store);

/* 打开设备方法 */
static int freg_open(struct inode* inode, struct file* filp) 
{
    struct fake_reg_dev* dev;

    //将自定义设备结构体保存在文件指针的私有数据域中, 以便访问设备时可以直接拿来用
    dev = container_of(inode->i_cdev, struct fake_reg_dev, dev);
    filp->private_data = dev;

    return 0;
}

/* 设备文件释放时调用, 空实现 */
static int freg_release(struct inode* inode, struct file* filp) 
{
    return 0;
}

/* 读取设备的寄存器val的值 */
static ssize_t freg_read(struct file* filp, char __user *buf, size_t count, loff_t* f_pos) 
{
    ssize_t err = 0;
    struct fake_reg_dev* dev = filp->private_data;

    /* 同步访问 */
    if(down_interruptible(&(dev->sem))) 
    {    
        return -ERESTARTSYS;
    }

    //检查类型是否一致
    if(count < sizeof(dev->val)) 
    {
        goto out;
    }

    /* 将寄存器val的值复制到用户提供的缓冲区中 */
    if(copy_to_user(buf, &(dev->val), sizeof(dev->val))) 
    {
        err = -EFAULT;
        goto out;
    }

    err = sizeof(dev->val);

out:
    // 撤销同步
    up(&(dev->sem));

    return err;
}

/* 写设备的寄存器val的值 */
static ssize_t freg_write(struct file* filp, const char __user *buf, size_t count, loff_t* f_pos) 
{
    struct fake_reg_dev* dev = filp->private_data;
    ssize_t err = 0;

    /* 同步访问 */
    if(down_interruptible(&(dev->sem))) 
    {
        return -ERESTARTSYS;
    }

    //检查类型是否一致
    if(count != sizeof(dev->val)) 
    {
        goto out;
    }

    /* 将用户提供的缓冲区的值写到设备寄存器中 */
    if(copy_from_user(&(dev->val), buf, count)) 
    {
        err = -EFAULT;
        goto out;
    }

    err = sizeof(dev->val);

out:
    up(&(dev->sem));
    return err;
}

/* 将寄存器val的值读取到缓冲区buf中, 内部使用 */
static ssize_t __freg_get_val(struct fake_reg_dev* dev, char* buf) 
{
    int val = 0;

    /* 同步访问 */
    if(down_interruptible(&(dev->sem))) 
    {
        return -ERESTARTSYS;
    }

    val = dev->val;

    up(&(dev->sem));

    return snprintf(buf, PAGE_SIZE, "%d\n", val);
}

/* 把缓冲区buf的值写到设备寄存器val中, 内部使用 */
static ssize_t __freg_set_val(struct fake_reg_dev* dev, const char* buf, size_t count) 
{
    int val = 0;

    /* 将字符串转换为数字 */
    val = simple_strtol(buf, NULL, 10);

    /* 同步访问 */
    if(down_interruptible(&(dev->sem))) 
    {
        return -ERESTARTSYS;
    }

    dev->val = val;
    up(&(dev->sem));

    return count;
}

/* 读设备属性val的值 */
static ssize_t freg_val_show(struct device* dev, struct device_attribute* attr, char* buf) 
{
    struct fake_reg_dev* hdev = (struct fake_reg_dev*)dev_get_drvdata(dev);

    return __freg_get_val(hdev, buf);
}

/* 写设备属性val的值 */
static ssize_t freg_val_store(struct device* dev, 
                              struct device_attribute* attr, 
                              const char* buf, 
                              size_t count) 
{
    struct fake_reg_dev* hdev = (struct fake_reg_dev*)dev_get_drvdata(dev);

    return __freg_set_val(hdev, buf, count);
}

/* 读取设备寄存器 val 的值 , 保存到page 缓冲区中*/
static ssize_t freg_proc_read(char* page, char** start, 
                              off_t off, 
                              int count, 
                              int* eof, 
                              void* data) 
{
    if(off > 0) 
    {
        *eof = 1;
        return 0;
    }

    return __freg_get_val(freg_dev, page);    
}

/* 把缓冲区的值 buff 保存到设备寄存器 val 中 */
static ssize_t freg_proc_write(struct file* filp, 
                               const char __user *buff, 
                               unsigned long len, 
                               void* data) 
{    
    int err = 0;
    char* page = NULL;

    if(len > PAGE_SIZE) 
    {
        printk(KERN_ALERT"The buff is too large: %lu.\n", len);
        return -EFAULT;
    }

    page = (char*)__get_free_page(GFP_KERNEL);
    if(!page) 
    {
        printk(KERN_ALERT"Failed to alloc page.\n");
        return -ENOMEM;
    }

    /* 先把用户提供的缓冲区的值复制到内核缓冲区中 */
    if(copy_from_user(page, buff, len)) 
    {
        printk(KERN_ALERT "Failed to copy buff from user.\n");
        err = -EFAULT;

        goto out;
    }

    err = __freg_set_val(freg_dev, page, len);

out:
    free_page((unsigned long)page);
    return err;    
}

/* 创建 /proc/freg 文件 */
static void freg_create_proc(void) 
{
    struct proc_dir_entry* entry;

    entry = create_proc_entry(FREG_DEVICE_PROC_NAME, 0, NULL);
    if(entry) 
    {
        entry->owner = THIS_MODULE;
        entry->read_proc = freg_proc_read;
        entry->write_proc = freg_proc_write;
    }
}

/* 删除 /proc/freg 文件 */
static void freg_remove_proc(void) 
{
    remove_proc_entry(FREG_DEVICE_PROC_NAME, NULL);
}

/* 初始化设备 */
static int  __freg_setup_dev(struct fake_reg_dev* dev) 
{
    int err;
    dev_t devno = MKDEV(freg_major, freg_minor);

    memset(dev, 0, sizeof(struct fake_reg_dev));

    /* 初始化字符设备 */
    cdev_init(&(dev->dev), &freg_fops);
    dev->dev.owner = THIS_MODULE;
    dev->dev.ops = &freg_fops;

    /* 注册字符设备 */
    err = cdev_add(&(dev->dev), devno, 1);
    if(err) 
    {
        return err;
    }    

    /* 初始化信号量 */
    init_MUTEX(&(dev->sem));
    /* 初始化寄存器val的值 */
    dev->val = 0;

    return 0;
}

/* 模块加载方法 */
static int __init freg_init(void) 
{ 
    int err = -1;
    dev_t dev = 0;
    struct device* temp = NULL;

    printk(KERN_ALERT"Initializing freg device.\n");

    /* 动态分配主设备号 和 从设备号 */
    err = alloc_chrdev_region(&dev, 0, 1, FREG_DEVICE_NODE_NAME);
    if(err < 0) 
    {
        printk(KERN_ALERT"Failed to alloc char dev region.\n");
        goto fail;
    }

    freg_major = MAJOR(dev);
    freg_minor = MINOR(dev);

    /* 分配freg设备结构体 */
    freg_dev = kmalloc(sizeof(struct fake_reg_dev), GFP_KERNEL);
    if(!freg_dev) 
    {
        err = -ENOMEM;
        printk(KERN_ALERT"Failed to alloc freg device.\n");
        goto unregister;
    }

    /* 初始化设备 */
    err = __freg_setup_dev(freg_dev);
    if(err) 
    {
        printk(KERN_ALERT"Failed to setup freg device: %d.\n", err);
        goto cleanup;
    }

    /* 在/sys/class/ 目录下创建设备类别目录freg */
    freg_class = class_create(THIS_MODULE, FREG_DEVICE_CLASS_NAME);
    if(IS_ERR(freg_class)) 
    {
        err = PTR_ERR(freg_class);
        printk(KERN_ALERT"Failed to create freg device class.\n");
        goto destroy_cdev;
    }

    /* 在 /dev/ 目录 和 /sys/class/freg目录下分别创建设备文件frag */
    temp = device_create(freg_class, NULL, dev, "%s", FREG_DEVICE_FILE_NAME);
    if(IS_ERR(temp)) 
    {
        err = PTR_ERR(temp);
        printk(KERN_ALERT"Failed to create freg device.\n");
        goto destroy_class;
    }

    /* 在 /sys/class/freg/freg 目录下创建属性文件val */
    err = device_create_file(temp, &dev_attr_val);
    if(err < 0) 
    {
        printk(KERN_ALERT"Failed to create attribute val of freg device.\n");
        goto destroy_device;
    }

    dev_set_drvdata(temp, freg_dev);

    /* 创建 /proc/freg文件 */
    freg_create_proc();

    printk(KERN_ALERT"Succedded to initialize freg device.\n");

    return 0;

destroy_device:
    device_destroy(freg_class, dev);
destroy_class:
    class_destroy(freg_class);
destroy_cdev:
    cdev_del(&(freg_dev->dev));    
cleanup:
    kfree(freg_dev);
unregister:
    unregister_chrdev_region(MKDEV(freg_major, freg_minor), 1);    
fail:
    return err;
}

/* 模块卸载方法 */
static void __exit freg_exit(void) 
{
    dev_t devno = MKDEV(freg_major, freg_minor);

    printk(KERN_ALERT"Destroy freg device.\n");

    /* 删除 /proc/freg 文件 */
    freg_remove_proc();

    /* 注销设备类别 和 设备 */
    if(freg_class) 
    {
        device_destroy(freg_class, MKDEV(freg_major, freg_minor));
        class_destroy(freg_class);
    }

    /* 删除字符设备 和 释放设备内存 */
    if(freg_dev) 
    {
        cdev_del(&(freg_dev->dev));
        kfree(freg_dev);
    }

    /* 释放设备号资源 */
    unregister_chrdev_region(devno, 1);
}

MODULE_LICENSE("GPL");
MODULE_DESCRIPTION("Fake Register Driver");

module_init(freg_init);
module_exit(freg_exit);
```

### 修改驱动编译文件

Kconfig 编译选项配置文件 :

```text
# kernel\goldfish\drivers\freg\Kconfig
# 执行 make menuconfig 命令来设置这些编译选项

config FREG
    tristate "Fake Register Driver"
    default n
    help
    This is the freg driver for android system.
```

驱动程序 一般有三种方式来编译。

第一种方式是**直接内建**在内核中； 第二种方式是编译成**内核模块**； 第三种方式是**不编译**到内核中。

默认的编译方式为 n， 就是 **不编译到内核**中， 因此， 在编译驱动程序 之前， 我们需要执行 `make menuconfig` 命令来修改它的**编译选项**， 以便可以将 驱动程序 **内建到内核**中 或 以**模块的方式**来编译。

Makefile 编译脚本文件 :

```text
# kernel\goldfish\drivers\freg\Makefile

#  $（CONFIG_FREG） 是一个变量， 它的值与驱动程序freg的编译选项有关

#如果选择将驱动程序freg内建到内核中， 那么变量$（CONFIG_FREG） 的值为y； 
# 如果选择以模块的方式来编译驱动程序freg， 那么变量$（CONFIG_FREG） 的值为m； 
# 如果变量$（CONFIG_FREG） 的值既不为y， 也不为m，那么驱动程序freg就不会被编译

obj-$(CONFIG_FREG) += freg.o
```

### 修改内核编译文件

修改内核Kconfig文件 :

在默认情况下，在执行`make menuconfig`命令配置内核编译选项时， 编译系统是无法找到 对应的驱动`Kconfig`文件的。 所以需要修改内核的根`Kconfig`文件， 使得编译系统能够找到驱动程序`freg`的`Kconfig`文件

当执行`make menuconfig`命令时， 编译系统会读取`arch/$(ARCH)` 目录下的`Kconfig`文件， 其中， `$(ARCH)` 指向**编译的目标CPU体系架构**

一般我们都会将`$(ARCH)` 的值设置为`arm`， 因此， 就需要修改`arch/arm`目录下的`Kconfig`文件， 使得编译系统可以找到**驱动程序**`freg`的`Kconfig`文件。

Arm架构下:

```text
# arch/arm/Kconfig

menu "Device Drivers"

# 将驱动程序freg的Kconfig文件包含进来
source "drivers/freg/Kconfig"
source "drivers/base/Kconfig"
source "drivers/connector/Kconfig"

# ...

endmenu
```

x86体系架构下的Kconfig文件 :

```text
# drivers/Kconfig

# 把drivers目录下的Kconfig文件包含进去
source "drivers/Kconfig"

endmenu
```

修改内核`Makefile`文件 :

在默认情况下， 在执行`make`命令编译内核时， 编译系统是无法找到这个`Makefile`文件的。 这时候， 就需要修改`drivers`目录下的`Makefile`文件， 使得编译系统能够找到驱动程序`freg`的`Makefile`文件

```text
# drivers/Makefile

# 当 make 编译内核时，编译系统就会对驱动程序 freg 进行编译
obj-$(CONFIG_FREG)        += freg/
obj-y                += gpio/
obj-$(CONFIG_PCI)        += pci/
# ...
```

### 编译内核驱动程序模块

在编译驱动程序`freg`之前， 我们需要执行`make menuconfig`命令来配置它的编译方式

```text
make menuconfig
```

第一个配置界面中用上下箭头键选择“`Device Drivers`”项， 按`Enter`键

第二个配置界面中继续用上下箭头键选择“`Fake Register Driver`”项， 按`Y`键或 `M`键， 就可以看到选项前面方括号中的字符变成“ `*` ”或者“`M`”符号， 它们分别表示将驱动程序`freg`编译到**内核**中 或 以**模块**的方式来编译

如果我们要以**模块**的方式来编译驱动程序freg， 那就必须在第一个配置界面中选择`“Enable loadable module support`”选项， 并且按 `Y` 键将它的值设置为`true`， 就是让内核可以支持**动态加载模块** .

如果要使得内核支持**动态卸载**模块， 那么就要在第一个配置界面中选择“`Enable loadable module support`”选项中的子选项“`Module unloading` ”， 并且按`Y`键将它的值设置为`true`。

第二个配置界面中按`M` 键来配置“`Fake RegisterDriver`”选项配置完成后， 保存编译配置选项， 退出`make menuconfig`命令

执行`make`命令来编译驱动程序`freg`

```text
make
```

驱动程序freg编译成功 :

![image-20200713165801391](https://gitee.com/cpu_code/picture_bed/raw/master//20200713165801.png)

编译得到的内核镜像文件`zImage`保存在`arch/arm/boot目录下`

### 验证内核驱动程序模块

通过`proc`文件系统和`devfs`文件系统来验证它的功能是否正确

```text
# 使用 得到的内核镜像文件zImage来启动Android模拟器
emulator -kernel kernel/goldfish/arch/arm/boot/zImage &

# # 用adb工具连接上
adb shell

# 进入 /dev目录下
cd dev

# 查看 一个设备文件freg 
# 存在，说明成功的注册到设备文件系统中
ls freg
```

```text
# 进入到/proc
cd proc

# 读取文件freg的内容
cat freg

# 往文件freg中写入一个新的内容
echo '5' > freg

# 将文件freg的内容读取出来
cat freg

# 值一样 , 说明 proc文件系统接口成功
```

```text
# 进入到/sys/class/freg/freg
cd sys/class/freg/freg

# 读取val文件的内容
cat val

# 往文件val中写入一个新的内容
echo '0' > freg

# 将文件val中的内容读取出
cat freg

# 值一样 , 说明 devfs文件系统接口成功
```

## 开发C可执行程序验证Android硬件驱动程序

通过编写一个**C可执行程序**来验证驱动程序`freg`所提供的**dev文件系统接口**的正确性， 这是通过调用`read`和`write`函数读写设备文件`/dev/freg`的内容来实现

在`Android`源代码工程环境中， 不仅可以用`C/C++`语言来开发**可执行程序**，还可以开发**动态链接库**， 也就是 `so`文件。

使用`adb`工具命令连接上`Android`模拟器之后， 进入到`/system/bin`或者`/system/lib`目录中， 就可以看到很多**可执行程序**或者**动态链接库**文件。

在`Android`源代码工程环境中开发的**C可执行程序**源文件一般保存在`external`目录中，因此， 需要进入到`external`目录中， 并且创建一个`freg`目录， 用来保存我们将要开发的**C可执行程序源文件**。

目录结构 :

```text
~/Android
    exiternal
        freg
            freg.c    # 源文件
            Android.mk # 编译脚本文件
```

源文件 freg.c :

```c
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>

#define FREG_DEVICE_NAME "/dev/freg"

int main(int argc, char** argv)
{
    int fd = -1;
    int val = 0;

    // 以读写方式打开设备文件/dev/freg
    fd = open(FREG_DEVICE_NAME, O_RDWR);
    if(fd == -1)
    {
        printf("Failed to open device %s.\n", FREG_DEVICE_NAME);
        return -1;
    }

    printf("Read original value:\n");

    // 读取它的内容， 即读取虚拟硬件设备 freg 的寄存器 val 的内容
    read(fd, &val, sizeof(val));
    // 打印出来
    printf("%d.\n\n", val);

    val = 5;
    printf("Write value %d to %s.\n\n", val, FREG_DEVICE_NAME);
    // 将一个整数 5 写入到虚拟硬件设备 freg 的寄存器 val 中
    write(fd, &val, sizeof(val));

    printf("Read the value again:\n");

    // 读取它的内容， 即读取虚拟硬件设备 freg 的寄存器 val 的内容
    read(fd, &val, sizeof(val));
    // 打印
    printf("%d.\n\n", val);

    close(fd);
    return 0;
}
```

编译脚本文件`Android.mk` :

```text
# 将编译结果保存在 out/target/product/gerneric/system/bin 目录中
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE_TAGS := optional
# 源文件
LOCAL_MODULE := freg
LOCAL_SRC_FILES := $(call all-subdir-c-files)

# 当前要编译的是一个可执行应用程序模块
include $(BUILD_EXECUTABLE)
```

```text
# 编译
mmm ./external/freg/
# 打包这个C可执行程序
make snod
```

**编译成功**后， 就可以在`out/target/product/gerneric/system/bin`目录下看到一个`freg`文件；

**打包成功**后， **编译好的文件**就会包含在`out/target/product/gerneric`目录下的`Android`系统镜像文件`system.img`里

```text
# 将得到的 system.img 文件启动 Android模拟器
emulator -kernal kernel/goldfish/arch/arm/boot/zImage &

# adb工具连接上它
adb shell

# 进入到/system/bin目录中
cd system/bin

# 执行里面的freg文件
./freg

# 验证驱动程序freg的dev文件系统访问接口的正确性
```

