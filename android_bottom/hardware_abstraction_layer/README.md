# 硬件抽象层

* @Author: cpu\_code
* @Date: 2020-07-12 22:20:34
* @LastEditTime: 2020-07-13 22:52:02
* @FilePath: \notes\android\_bottom\hardware\_abstraction\_layer.md
* @Gitee: [https://gitee.com/cpu\_code](https://gitee.com/cpu_code)
* @Github: [https://github.com/CPU-Code](https://github.com/CPU-Code)
* @CSDN: [https://blog.csdn.net/qq\_44226094](https://blog.csdn.net/qq_44226094)
* @Gitbook: [https://923992029.gitbook.io/cpucode/](https://923992029.gitbook.io/cpucode/)

Android系统的**硬件抽象层**（Hardware Abstract Layer， HAL） 运行在用户空间中， 它向下屏蔽硬件驱动模块的实现细节， 向上提供硬件访问服务。

Android系统的体系结构 :

![Android&#x7CFB;&#x7EDF;&#x7684;&#x4F53;&#x7CFB;&#x7ED3;&#x6784;](https://gitee.com/cpu_code/picture_bed/raw/master//20200713152328.png)

## 开发Android硬件驱动程序

### 实现内核驱动程序模块

驱动程序freg的目录结构 :

```text
~/Android/kernel/goldfish
    drivers
        freg
            freg.h    # 源代码文件
            freg.c    # 源代码文件
            Kconfig    # 编译选项配置文件
            Makefile    # 编译脚本文件
```

freg.h :

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

freg.c

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

/* devfs文件系统的设备属性操作方法 */
static ssize_t freg_val_show(struct device* dev, struct device_attribute* attr,  char* buf);
static ssize_t freg_val_store(struct device* dev, struct device_attribute* attr, const char* buf, size_t count);

/* devfs文件系统的设备属性 */
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
    err = cdev_add(&(dev->dev),devno, 1);
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

Kconfig :

```text
# kernel\goldfish\drivers\freg\Kconfig
config FREG
    tristate "Fake Register Driver"
    default n
    help
    This is the freg driver for android system.
```

Makefile :

```text
# kernel\goldfish\drivers\freg\Makefile

#  $（CONFIG_FREG） 是一个变量， 它的值与驱动程序freg的编译选项有关

#如果选择将驱动程序freg内建到内核中， 那么变量$（CONFIG_FREG） 的值为y； 
# 如果选择以模块的方式来编译驱动程序freg， 那么变量$（CONFIG_FREG） 的值为m； 
# 如果变量$（CONFIG_FREG） 的值既不为y， 也不为m，那么驱动程序freg就不会被编译

obj-$(CONFIG_FREG) += freg.o
```

### 修改内核Kconfig文件

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

```text
# drivers/Kconfig

menu "Device Drivers"

# 将驱动程序 freg 的 Kconfig 文件包含进来
source "drivers/freg/Kconfig"

source "drivers/base/Kconfig"

# ...

endmenu
```

### 修改内核Makefile文件

```text
# drivers/Makefile

# 当 make 编译内核时，编译系统就会对驱动程序 freg 进行编译
obj-$(CONFIG_FREG)        += freg/
obj-y                += gpio/
obj-$(CONFIG_PCI)        += pci/
# ...
```

### 编译内核驱动程序模块

在编译驱动程序freg之前， 我们需要执行make menuconfig命令来配置它的编译方式

```text
make menuconfig
```

执行make命令来编译驱动程序freg

```text
make
```

驱动程序freg编译成功 :

![image-20200713165801391](https://gitee.com/cpu_code/picture_bed/raw/master//20200713165801.png)

### 验证内核驱动程序模块

```text
# 使用 得到的内核镜像文件zImage来启动Android模拟器
emulator -kernel kernel/goldfish/arch/arm/boot/zImage &

# # 用adb工具连接上
adb shell

# 进入 /dev目录下
cd dev

# 查看 一个设备文件freg
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
```

```text
# 进入到/sys/class/freg/freg
cd sys/class/freg/freg

# 读取val文件的内容
cat val

# 往文件val中写入一个新的内容
echo '' > freg

# 将文件val中的内容读取出
cat freg
```

## 开发C可执行程序验证Android硬件驱动程序

在Android源代码工程环境中开发的C可执行程序源文件一般保存在external目录中，因此， 我们进入到external目录中， 并且创建一个freg目录， 用来保存我们将要开发的C可执行程序源文件。

目录结构 :

```text
~/Android
    exiternal
        freg
            freg.c
            Android.mk
```

源文件freg.c :

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

编译脚本文件Android.mk :

```text
# 将编译结果保存在 out/target/product/gerneric/system/bin 目录中
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE_TAGS := optional
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

```text
# 将得到的 system.img 文件启动 Android模拟器
emulator -kernal kernel/goldfish/arch/arm/boot/zImage &

# adb工具连接上它
adb shell

# 进入到/system/bin目录中
cd system/bin

# 执行里面的freg文件
./freg
```

## 开发Android硬件抽象层模块

### 硬件抽象层模块编写规范

#### 硬件抽象层模块文件命名规范

硬件抽象层模块文件的命名规范 :

```c
// hardware/libhardware/hardware.c

/**
 * There are a set of variant filename for modules. The form of the filename
 * is "<MODULE_ID>.variant.so" so for the led module the Dream variants 
 * of base "ro.product.board", "ro.board.platform" and "ro.arch" would be:
 *
 * MODULE_ID : 模块的ID
 * led.trout.so
 * led.msm7k.so
 * led.ARMV6.so
 * led.default.so
 */

// variant 表示四个系统属性 ro.hardware、 ro.product.board、 ro.board.platform 和 ro.arch 之一
static const char *variant_keys[] = {
    "ro.hardware",     /* 由 init 进程负责设置 */ /* This goes first so that it can pick up a different
                       file on the emulator. */
    "ro.product.board",
    "ro.board.platform",
    "ro.arch"
};
```

#### 硬件抽象层模块结构体定义规范

结构体hw\_module\_t :

```c
// hardware\libhardware\include\hardware\hardware.h

/*
 * Value for the hw_module_t.tag field
 */

#define MAKE_TAG_CONSTANT(A,B,C,D) (((A) << 24) | ((B) << 16) | ((C) << 8) | (D))

#define HARDWARE_MODULE_TAG MAKE_TAG_CONSTANT('H', 'W', 'M', 'T')
#define HARDWARE_DEVICE_TAG MAKE_TAG_CONSTANT('H', 'W', 'D', 'T')

struct hw_module_t;
struct hw_module_methods_t;
struct hw_device_t;

/**
 * Every hardware module must have a data structure named HAL_MODULE_INFO_SYM
 * and the fields of this data structure must begin with hw_module_t
 * followed by module specific information.
 */
/* 
硬件抽象层中的每一个模块都必须自定义一个硬件抽象层模块结构体， 
而且它的第一个成员变量的类型必须为 hw_module_t
*/
typedef struct hw_module_t 
{
    /** tag must be initialized to HARDWARE_MODULE_TAG */
    /* 成员变量 tag 的值必须设置为 HARDWARE_MODULE_TAG， 即设置为一个常量值（'H'＜＜24|'W'＜＜16|'M'＜＜8|'T'）,
       用来标志这是一个硬件抽象层模块结构体 
     */
    uint32_t tag;

    /** major version number for the module */
    uint16_t version_major;

    /** minor version number of the module */
    uint16_t version_minor;

    /** Identifier of module */
    const char *id;

    /** Name of this module */
    const char *name;

    /** Author/owner/implementor of the module */
    const char *author;

    /** Modules methods */
    /* 定义了一个硬件抽象层模块的操作方法列表 */
    struct hw_module_methods_t* methods;

    /** module's dso */
    /* 保存加载硬件抽象层模块后得到的句柄值 */
    /*
     * 加载硬件抽象层模块的过程实际上就是调用 dlopen 函数来加载与其对应的动态链接库文件的过程。 
     * 在调用 dlclose 函数来卸载这个硬件抽象层模块时， 要用到这个句柄值
     */
    void* dso;

    /** padding to 128 bytes, reserved for future use */
    uint32_t reserved[32-7];

} hw_module_t;

/**
 * Name of the hal_module_info
 */
/*
硬件抽象层中的每一个模块都必须存在一个导出符号 HAL_MODULE_IFNO_SYM， 即“HMI”， 
它指向一个自定义的硬件抽象层模块结构体
*/
#define HAL_MODULE_INFO_SYM         HMI
```

struct hw\_module\_methods\_t :

```c
// hardware\libhardware\include\hardware\hardware.h

typedef struct hw_module_methods_t 
{
    /** Open a specific device */
    /**
     * @function: 打开硬件抽象层模块中的硬件设备
     * @parameter: 
     *        module : 要打开的硬件设备所在的模块
     *        id : 要打开的硬件设备的ID
     *        device : 一个输出参数，描述一个已经打开的硬件设备
     * @return: 
     *     success: 
     *     error: 
     * @note: 
     */
    int (*open)(const struct hw_module_t* module, 
                const char* id, 
                struct hw_device_t** device);

} hw_module_methods_t;
```

struct hw\_device\_t :

```c
// hardware\libhardware\include\hardware\hardware.h

#define HARDWARE_DEVICE_TAG MAKE_TAG_CONSTANT('H', 'W', 'D', 'T')

/**
 * Every device data structure must begin with hw_device_t
 * followed by module specific public methods and attributes.
 */
/*
 * 硬件抽象层模块中的每一个硬件设备都必须自定义一个硬件设备结构体，
 * 而且它的第一个成员变量的类型必须为hw_device_t
 */
typedef struct hw_device_t 
{
    /** tag must be initialized to HARDWARE_DEVICE_TAG */
    /*
     * tag 必须 == HARDWARE_DEVICE_TAG，即设置为一个常量值（'H'＜＜24|'W'＜＜16|'D'＜＜8|'T'）,
     * 用来标志这是一个硬件抽象层中的硬件设备结构体
     */
    uint32_t tag;

    /** version number for hw_device_t */
    uint32_t version;

    /** reference to the module this device belongs to */
    struct hw_module_t* module;

    /** padding reserved for future use */
    uint32_t reserved[12];

    /** Close this device */
    /* 关闭一个硬件设备 */
    int (*close)(struct hw_device_t* device);

} hw_device_t;
```

### 编写硬件抽象层模块接口

将虚拟硬件设备freg在硬件抽象层中的模块名称定义为freg

目录结构：

```text
~/Android/hardware/libhardware
    include
        hardware
            freg.h
    Modules
        freg
            freg.cpp
            Android.mk
```

`freg.h` 源代码文件

```c
// Android/hardware/libhardware/include/hardware/freg.h

#ifndef ANDROID_FREG_INTERFACE_H
#define ANDROID_FREG_INTERFACE_H

__BEGIN_DECLS

/**
 * The id of this module
 */
// 定义模块ID
#define FREG_HARDWARE_MODULE_ID "freg"

/**
 * The id of this device
 */
// 定义设备ID
#define FREG_HARDWARE_DEVICE_ID "freg"

// 自定义模块结构体
struct freg_module_t 
{
    // 第一个成员变量的类型为 hw_module_t
    struct hw_module_t common;
};

// 自定义设备结构体 , 描述虚拟硬件设备 freg
struct freg_device_t 
{
    // 第一个成员变量的类型为 hw_device_t
    struct hw_device_t common;
    // 一个文件描述符 , 用来描述打开的设备文件/dev/freg
    int fd;

    // 写虚拟硬件设备 freg 的寄存器 val 的内容
    int (*set_val)(struct freg_device_t* dev, int val);
    // 读虚拟硬件设备 freg 的寄存器 val 的内容
    int (*get_val)(struct freg_device_t* dev, int* val);
};

__END_DECLS

#endif
```

freg.cpp 硬件抽象层模块 freg 的实现文件 :

```c
// Android/hardware/libhardware/Modules/freg/freg.cpp

#define LOG_TAG "FregHALStub"

#include <hardware/hardware.h>
#include <hardware/freg.h>

#include <fcntl.h>
#include <errno.h>

#include <cutils/log.h>
#include <cutils/atomic.h>

#define DEVICE_NAME "/dev/freg"
#define MODULE_NAME "Freg"
#define MODULE_AUTHOR "cpucode"

/* 设备打开接口 */
static int freg_device_open(const struct hw_module_t* module, const char* id, struct hw_device_t** device);
/* 设备关闭接口 */
static int freg_device_close(struct hw_device_t* device);
/* 设备寄存器写接口 */
static int freg_set_val(struct freg_device_t* dev, int val);
/* 设备寄存器读接口 */
static int freg_get_val(struct freg_device_t* dev, int* val);

/* 定义模块操作方法结构体变量 */
static struct hw_module_methods_t freg_module_methods = {
    open: freg_device_open
};

/* 定义模块结构体变量 */
// 每一个硬件抽象层模块必须导出一个名称为 HAL_MODULE_INFO_SYM 的符号
struct freg_module_t HAL_MODULE_INFO_SYM = {
    common: {
        // tag 必须 == HARDWARE_MODULE_TAG
        tag: HARDWARE_MODULE_TAG,    
        version_major: 1,
        version_minor: 0,
        id: FREG_HARDWARE_MODULE_ID,
        name: MODULE_NAME,
        author: MODULE_AUTHOR,
        methods: &freg_module_methods,
    }
};

// 打开操作
static int freg_device_open(const struct hw_module_t* module, 
                            const char* id, 
                            struct hw_device_t** device) 
{
    // 判断 id 与虚拟硬件设备freg的ID值是否 匹配
    if(!strcmp(id, FREG_HARDWARE_DEVICE_ID))
    {
        struct freg_device_t* dev;

        dev = (struct freg_device_t*)malloc(sizeof(struct freg_device_t));
        if(!dev) 
        {
            LOGE("Failed to alloc space for freg_device_t.");
            return -EFAULT;    
        }

        memset(dev, 0, sizeof(struct freg_device_t));

        // 硬件设备标签（dev-＞common.tag） 必须 == HARDWARE_DEVICE_TAG
        dev->common.tag = HARDWARE_DEVICE_TAG;
        dev->common.version = 0;
        dev->common.module = (hw_module_t*)module;
        // 关闭函数设置为 freg_device_close
        dev->common.close = freg_device_close;
        // 写函数
        dev->set_val = freg_set_val;
        // 读函数
        dev->get_val = freg_get_val;

        // 打开虚拟硬件设备文件/dev/freg ， 且将得到的文件描述符保存在结构体 freg_device_t 的成员变量 fd 中
        if((dev->fd = open(DEVICE_NAME, O_RDWR)) == -1)
        {
            LOGE("Failed to open device file /dev/freg -- %s.", strerror(errno));
            free(dev);

            return -EFAULT;
        }

        *device = &(dev->common);

        LOGI("Open device file /dev/freg successfully.");

        return 0;
    }

    return -EFAULT;
}

// 虚拟硬件设备freg的关闭函数
static int freg_device_close(struct hw_device_t* device) 
{
    struct freg_device_t* freg_device = (struct freg_device_t*)device;

    if(freg_device) 
    {
        // 关闭设备文件/dev/freg
        close(freg_device->fd);
        // 释放设备
        free(freg_device);
    }

    return 0;
}

static int freg_set_val(struct freg_device_t* dev, int val) 
{
    if(!dev) 
    {
        LOGE("Null dev pointer.");
        return -EFAULT;
    }

    LOGI("Set value %d to device file /dev/freg.", val);
    // 写虚拟硬件设备freg的寄存器val的内容
    write(dev->fd, &val, sizeof(val));

    return 0;
}

static int freg_get_val(struct freg_device_t* dev, int* val) 
{
    if(!dev) 
    {
        LOGE("Null dev pointer.");
        return -EFAULT;
    }

    if(!val) 
    {
        LOGE("Null val pointer.");
        return -EFAULT;
    }

    // 读虚拟硬件设备freg的寄存器val的内容
    read(dev->fd, val, sizeof(*val));

    LOGI("Get value %d from device file /dev/freg.", *val);

    return 0;
}
```

Android.mk 编译脚本文件 :

```text
# Android/hardware/libhardware/Modules/freg/Android.mk
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE_TAGS := optional
LOCAL_PRELINK_MODULE := false
# 保存在 out/target/product/generic/system/lib/hw
LOCAL_MODULE_PATH := $(TARGET_OUT_SHARED_LIBRARIES)/hw
LOCAL_SHARED_LIBRARIES := liblog
LOCAL_SRC_FILES := freg.cpp
LOCAL_MODULE := freg.default
# 将该硬件抽象层模块编译成一个动态链接库文件， 名称为 freg.default
include $(BUILD_SHARED_LIBRARY)
```

```text
# 编译
mmm ./hardware/libhardware/moduels/freg/

# 打包
make snod

# 在 out/target/product/generic/system/lib/hw 目录下得到一个 freg.default.so 文件
```

### 硬件抽象层模块的加载过程

负责加载硬件抽象层模块的函数 hw\_get\_module :

```c
// hardware\libhardware\include\hardware\hardware.h
/**
 * Get the module info associated with a module by id.
 * @return: 0 == success, <0 == error and *pHmi == NULL
 */
// id: 输入参数， 表示要加载的硬件抽象层模块ID；
// module : 输出参数， 如果加载成功， 那么它指向一个自定义的硬件抽象层模块结构体
// 加载硬件抽象层模块
int hw_get_module(const char *id, const struct hw_module_t **module);
```

分析hw\_get\_module函数的实现 :

```c
// hardware\libhardware\hardware.c

/** Base path of the hal modules */
// 定义要加载的硬件抽象层模块文件所在的目录
// 编译好的模块文件位于 out/target/product/generic/system/lib/hw 目录中， 
//而这个目录经过打包后， 就对应于设备上的 /system/lib/hw 目录
#define HAL_LIBRARY_PATH1 "/system/lib/hw"
// 定义的目录为 /vendor/lib/hw ，用来保存设备厂商所提供的硬件抽象层模块接口文件
#define HAL_LIBRARY_PATH2 "/vendor/lib/hw"

/**
 * There are a set of variant filename for modules. The form of the filename
 * is "<MODULE_ID>.variant.so" so for the led module the Dream variants 
 * of base "ro.product.board", "ro.board.platform" and "ro.arch" would be:
 *
 * led.trout.so
 * led.msm7k.so
 * led.ARMV6.so
 * led.default.so
 */

// 组装要加载的硬件抽象层模块的文件名称
static const char *variant_keys[] = {
    "ro.hardware",  /* This goes first so that it can pick up a different
                       file on the emulator. */
    "ro.product.board",
    "ro.board.platform",
    "ro.arch"
};

// 数组 variant_keys的大小
static const int HAL_VARIANT_KEYS_COUNT = (sizeof(variant_keys)/ sizeof(variant_keys[0]));

int hw_get_module(const char *id, const struct hw_module_t **module) 
{
    int status;
    int i;
    const struct hw_module_t *hmi = NULL;
    char prop[PATH_MAX];
    char path[PATH_MAX];

    /*
     * Here we rely on the fact that calling dlopen multiple times on
     * the same .so will simply increment a refcount (and not load
     * a new copy of the library).
     * We also assume that dlopen() is thread-safe.
     */

    /* Loop through the configuration variants looking for a module */
    for (i = 0 ; i < HAL_VARIANT_KEYS_COUNT + 1 ; i++) 
    {
        // 根据数组 variant_keys 在 HAL_LIBRARY_PATH1 和 HAL_LIBRARY_PATH2 目录中检查对应的硬件抽象层模块文件是否存在， 
        //如果存在， 则结束for循环； 


        if (i < HAL_VARIANT_KEYS_COUNT) 
        {
            // 获得的系统属性“ro.hardware”的值
            if (property_get(variant_keys[i], prop, NULL) == 0) 
            {
                continue;
            }

            snprintf(path, sizeof(path), "%s/%s.%s.so", HAL_LIBRARY_PATH1, id, prop);
            if (access(path, R_OK) == 0) 
            {
                break;
            }

            snprintf(path, sizeof(path), "%s/%s.%s.so", HAL_LIBRARY_PATH2, id, prop);
            if (access(path, R_OK) == 0) 
            {
                break;
            }

        } 
        else 
        {
            snprintf(path, sizeof(path), "%s/%s.default.so", HAL_LIBRARY_PATH1, id);
            if (access(path, R_OK) == 0) 
            {
                break;
            }

        }
    }

    status = -ENOENT;
    if (i < HAL_VARIANT_KEYS_COUNT + 1) 
    {
        /* load the module, if this fails, we're doomed, and we should not try
         * to load a different variant. */
        // 加载硬件抽象层模块的操作
        status = load(id, path, module);
    }

    return status;
}
```

load 函数来执行硬件抽象层模块的加载操作 :

```c
// hardware\libhardware\hardware.c

/**
 * Load the file defined by the variant and if successful
 * return the dlopen handle and the hmi.
 * @return 0 = success, !0 = failure.
 */
static int load(const char *id,
                const char *path,
                const struct hw_module_t **pHmi)
{
    int status;
    void *handle;
    struct hw_module_t *hmi;

    /*
     * load the symbols resolving undefined symbols before
     * dlopen returns. Since RTLD_GLOBAL is not or'd in with
     * RTLD_NOW the external symbols will not be global
     */
    /* 将它加载到内存中 */
    handle = dlopen(path, RTLD_NOW);
    if (handle == NULL) 
    {
        char const *err_str = dlerror();
        LOGE("load: module=%s\n%s", path, err_str?err_str:"unknown");
        status = -EINVAL;

        goto done;
    }

    /* Get the address of the struct hal_module_info. */
    const char *sym = HAL_MODULE_INFO_SYM_AS_STR;

    // 获得里面名称为 HAL_MODULE_INFO_SYM_AS_STR 的符号
    // 符号指向的是一个自定义的硬件抽象层模块结构体， 
    // 它包含了对应的硬件抽象层模块的所有信息
    // 将模块中的 HMI 符号转换为一个 hw_module_t 结构体指针
    hmi = (struct hw_module_t *)dlsym(handle, sym);
    if (hmi == NULL) 
    {
        LOGE("load: couldn't find symbol %s", sym);
        status = -EINVAL;

        goto done;
    }

    /* Check that the id matches */
    /* 验证加载得到的硬件抽象层模块ID 是否 与所要求加载的硬件抽象层模块ID一致 */
    if (strcmp(id, hmi->id) != 0) 
    {
        LOGE("load: id=%s != hmi->id=%s", id, hmi->id);
        status = -EINVAL;

        goto done;
    }

    //将成功加载后得到的模块句柄值 handle 保存在 hw_module_t 结构体指针 hmi 的成员变量 dso 中
    hmi->dso = handle;

    /* success */
    status = 0;

    done:
    if (status != 0) 
    {
        hmi = NULL;
        if (handle != NULL) 
        {
            dlclose(handle);
            handle = NULL;
        }
    } 
    else 
    {
        LOGV("loaded HAL id=%s path=%s hmi=%p handle=%p", id, path, *pHmi, handle);
    }

    *pHmi = hmi;

    return status;
}
```

### 处理硬件设备访问权限问题

```c
// Android/hardware/libhardware/Modules/freg/freg.cpp

// 打开操作
static int freg_device_open(const struct hw_module_t* module, 
                            const char* id, 
                            struct hw_device_t** device) 
{
    // 判断 id 与虚拟硬件设备freg的ID值是否 匹配
    if(!strcmp(id, FREG_HARDWARE_DEVICE_ID))
    {
        struct freg_device_t* dev;

        dev = (struct freg_device_t*)malloc(sizeof(struct freg_device_t));
        if(!dev) 
        {
            LOGE("Failed to alloc space for freg_device_t.");
            return -EFAULT;    
        }

        memset(dev, 0, sizeof(struct freg_device_t));

        // 硬件设备标签（dev-＞common.tag） 必须 == HARDWARE_DEVICE_TAG
        dev->common.tag = HARDWARE_DEVICE_TAG;
        dev->common.version = 0;
        dev->common.module = (hw_module_t*)module;
        // 关闭函数设置为 freg_device_close
        dev->common.close = freg_device_close;
        // 写函数
        dev->set_val = freg_set_val;
        // 读函数
        dev->get_val = freg_get_val;

        // 打开虚拟硬件设备文件/dev/freg ， 且将得到的文件描述符保存在结构体 freg_device_t 的成员变量 fd 中
        // 不修改设备文件/dev/freg的访问权限 , 打不开
        if((dev->fd = open(DEVICE_NAME, O_RDWR)) == -1)
        {
            LOGE("Failed to open device file /dev/freg -- %s.", strerror(errno));
            free(dev);

            return -EFAULT;
        }

        *device = &(dev->common);

        LOGI("Open device file /dev/freg successfully.");

        return 0;
    }

    return -EFAULT;
}
```

`Android`提供了另外的一个`uevent`机制， 可以在系统启动时修改设备文件的访问权限

```text
# system\core\rootdir\ueventd.rc
# ...
/dev/binder               0666   root       root
/dev/freg                 0666   root       root

# logger should be world writable (for logging) but not readable
/dev/log/*                0662   root       log

# the msm hw3d client device node is world writable/readable.
/dev/msm_hw3dc            0666   root       root

# gpu driver for adreno200 is globally accessible
/dev/kgsl                 0666   root       root

#...
```

#### 解压ramdisk.img镜像文件

```text

```

```text
# 将ramdisk.img改名为ramdisk.img.gz
mv ./out/target/product/generic/reamdisk.img ./reamdisk.img.gz

# 解压
gunzip ./ramdisk.img.gz
```

#### 还原ramdisk.img镜像文件

```text
# 创建目录
mkdir ramdisk

# 进入目录
cd ./ramdisk/

# 解除归档
cpio -i -F ../ramdisk.img
```

#### 修改ueventd.rc文件

```text
# 进入目录
cd /ramdisk

# 修改 文件
vim ueventd.rc
```

```text
# system\core\rootdir\ueventd.rc
# ...
/dev/binder               0666   root       root
# 赋予了系统中的所有用户访问设备文件/dev/freg的权限
/dev/freg                 0666   root       root

# logger should be world writable (for logging) but not readable
/dev/log/*                0662   root       log

# the msm hw3d client device node is world writable/readable.
/dev/msm_hw3dc            0666   root       root

# gpu driver for adreno200 is globally accessible
/dev/kgsl                 0666   root       root

#...
```

#### 重新打包ramdisk.img镜像文件

```text
# 删除
rm -f ../ramdisk.img

# 把 ramdisk 目录归档成 cpio 文件
find . | cpio -o -H newc > ../ramdisk.img.unzip

# 切换到上目录
cd ..

# 压缩成gzip文件
gzip -c ./ramdisk.img.unzip > .ramdisk.img.gz

# 删除
rm -f ./ramdisk.img.unzip
rm -R ./ramdisk

# 转移 并改名
mv ./ramdisk.img.gz ./out/target/product/generic/ramdisk.img
```

## 开发Android硬件访问服务

### 定义硬件访问服务接口

```java
// frameworks\base\core\java\android\os\IFregService.aidl

package android.os;

interface IFregService 
{
    // 往虚拟硬件设备freg的寄存器val中写入一个整数
    void setVal(int val);
    // 从虚拟硬件设备freg的寄存器val中读出一个整数
    int getVal();
}
```

```text
# frameworks/base/Android.mk

## READ ME: ########################################################
##
## When updating this list of aidl files, consider if that aidl is
## part of the SDK API.  If it is, also add it to the list below that
## is preprocessed and distributed with the SDK.  This list should
## not contain any aidl files for parcelables, but the one below should
## if you intend for 3rd parties to be able to send those objects
## across process boundaries.
##
## READ ME: ########################################################
LOCAL_SRC_FILES += \
    core/java/android/accessibilityservice/IAccessibilityServiceConnection.aidl \
    #...
    core/java/android/net/IThrottleManager.aidl \
    core/java/android/nfc/IP2pTarget.aidl \
    core/java/android/os/IVibratorService.aidl \
    # 将需要的添加到编译脚本文件中
    core/java/android/os/IFregService.aidl \
    core/java/android/service/urlrenderer/IUrlRendererService.aidl \
    #...
    voip/java/android/net/sip/ISipService.aidl
#
```

```text
# 对硬件访问服务接口IFregService进行编译
mmm ./frameworks/base/
```

### 实现硬件访问服务

```java
// frameworks\base\services\java\com\android\server\FregService.java

package com.android.server;

import android.content.Context;
import android.os.IFregService;
import android.util.Slog;

// 硬件访问服务FregService继承了IFregService.Stub类
public class FregService extends IFregService.Stub 
{
    private static final String TAG = "FregService";

    private int mPtr = 0;

    FregService() 
    {
        // 调用 JNI 方法 init_native 来打开虚拟硬件设备 freg ，
        // 并且获得它的一个句柄值， 保存在成员变量 mPtr 中
        mPtr = init_native();

        if(mPtr == 0) 
        {
            Slog.e(TAG, "Failed to initialize freg service.");
        }
    }

    public void setVal(int val) 
    {
        if(mPtr == 0) 
        {
            Slog.e(TAG, "Freg service is not initialized.");
            return;
        }
        // 调用 JNI 方法 setVal_native 来写虚拟硬件设备 freg 的寄存器 val
        setVal_native(mPtr, val);
    }    

    public int getVal() 
    {
        if(mPtr == 0) 
        {
            Slog.e(TAG, "Freg service is not initialized.");
            return 0;
        }

        //调用 JNI 方法 getVal_native 来读虚拟硬件设备 freg 的寄存器 val
        return getVal_native(mPtr);
    }

    private static native int init_native();
    private static native void setVal_native(int ptr, int val);
    private static native int getVal_native(int ptr);
};
```

```text
# 重新编译 Android 系统的 services 模块
mmm ./frameworks/base/services/java/
```

### 实现硬件访问服务的JNI方法

```cpp
// frameworks\base\services\jni\com_android_server_FregService.cpp

#define LOG_TAG "FregServiceJNI"

#include "jni.h"
#include "JNIHelp.h"
#include "android_runtime/AndroidRuntime.h"

#include <utils/misc.h>
#include <utils/Log.h>
#include <hardware/hardware.h>
#include <hardware/freg.h>

#include <stdio.h>

namespace android
{
    //设置虚拟硬件设备 freg 的寄存器的值
    static void freg_setVal(JNIEnv* env, jobject clazz, jint ptr, jint value) 
    {
        int val = value;

        //将参数 ptr 转换为 freg_device_t 结构体变量
        freg_device_t* device = (freg_device_t*)ptr;
        if(!device)
        {
            LOGE("Device freg is not open.");
            return;
        }

        LOGI("Set value %d to device freg.", val);

        device->set_val(device, val);
    }

    //读取虚拟硬件设备freg的寄存器的值
    static jint freg_getVal(JNIEnv* env, jobject clazz, jint ptr)
    {
        int val = 0;

        //将传输ptr转换为 freg_device_t 结构体变量
        freg_device_t* device = (freg_device_t*)ptr;
        if(!device) 
        {
            LOGE("Device freg is not open.");
            return 0;
        }

        device->get_val(device, &val);

        LOGI("Get value %d from device freg.", val);

        return val;
    }

    //打开虚拟硬件设备freg
    static inline int freg_device_open(const hw_module_t* module, 
                                       struct freg_device_t** device) 
    {
        return module->methods->open(module, 
                                     FREG_HARDWARE_DEVICE_ID, 
                                     (struct hw_device_t**)device);
    }

    //初始化虚拟硬件设备freg
    static jint freg_init(JNIEnv* env, jclass clazz)
    {
        freg_module_t* module;
        freg_device_t* device;

        LOGI("Initializing HAL stub freg......");

        //加载硬件抽象层模块freg
        // 根据 FREG_HARDWARE_MODULE_ID 来加载 Android 硬件抽象层模块 freg
        if(hw_get_module(FREG_HARDWARE_MODULE_ID, 
                         (const struct hw_module_t**)&module) == 0) 
        {
            LOGI("Device freg found.");
            //打开虚拟硬件设freg
            if(freg_device_open(&(module->common), &device) == 0) 
            {
                LOGI("Device freg is open.");
                //将freg_device_t 接口转换为整型句柄值值返回
                return (jint)device;
            }

            LOGE("Failed to open device freg.");
            return 0;
        }

        LOGE("Failed to get HAL stub freg.");

        return 0;
    }

    //java本地接口方法表
    // 把JNI方法表method_table注册到Java虚拟机
    static const JNINativeMethod method_table[] = {
        {"init_native", "()I", (void*)freg_init},
        {"setVal_native", "(II)V", (void*)freg_setVal},
        {"getVal_native", "(I)I", (void*)freg_getVal},
    };

    //注册java本地接口方法
    int register_android_server_FregService(JNIEnv *env) 
    {
        return jniRegisterNativeMethods(env, 
                                        "com/android/server/FregService", 
                                        method_table, 
                                        NELEM(method_table));
    }

};
```

```cpp
// frameworks/base/services/jni/onload.cpp

#include "JNIHelp.h"
#include "jni.h"
#include "utils/Log.h"
#include "utils/misc.h"

namespace android 
{
// ...
int register_android_server_location_GpsLocationProvider(JNIEnv* env);
    // 声明
int register_android_server_FregService(JNIEnv* env);
};

using namespace android;

extern "C" jint JNI_OnLoad(JavaVM* vm, void* reserved)
{
    JNIEnv* env = NULL;
    jint result = -1;

    if (vm->GetEnv((void**) &env, JNI_VERSION_1_4) != JNI_OK) {
        LOGE("GetEnv failed!");
        return result;
    }
    LOG_ASSERT(env, "Could not retrieve the env!");

    //...
    register_android_server_location_GpsLocationProvider(env);
    //调用
    register_android_server_FregService(env);

    return JNI_VERSION_1_4;
}
```

```text
# frameworks/base/services/jni/Android.mk

LOCAL_PATH:= $(call my-dir)
include $(CLEAR_VARS)

# 修改变量LOCAL_SRC_FILES的值
LOCAL_SRC_FILES:= \
    #...
    com_android_server_location_GpsLocationProvider.cpp \
    com_android_server_FregService.cpp \
    onload.cpp

LOCAL_C_INCLUDES += \
    $(JNI_H_INCLUDE)

LOCAL_SHARED_LIBRARIES := \
    libandroid_runtime \
    libcutils \
    libhardware \
    libhardware_legacy \
    libnativehelper \
    libsystem_server \
    libutils \
    libui \
    libsurfaceflinger_client

ifeq ($(TARGET_SIMULATOR),true)
ifeq ($(TARGET_OS),linux)
ifeq ($(TARGET_ARCH),x86)
LOCAL_LDLIBS += -lpthread -ldl -lrt
endif
endif
endif

ifeq ($(WITH_MALLOC_LEAK_CHECK),true)
    LOCAL_CFLAGS += -DMALLOC_LEAK_CHECK
endif

LOCAL_MODULE:= libandroid_servers

include $(BUILD_SHARED_LIBRARY)
```

```text
# 重新编译libandroid_servers模块 , 得到的libandroid_servers.so文件
mmm ./frameworks/base/services/jni/
```

### 启动硬件访问服务

```java
//...
class ServerThread extends Thread 
{
    //...
    // 修改 ServerThread 类的成员函数run的实现
    @Override
    public void run() 
    {
        //...
        if (factoryTest != SystemServer.FACTORY_TEST_LOW_LEVEL) 
        {
            //...
            try {
                Slog.i(TAG, "DiskStats Service");
                ServiceManager.addService("diskstats", new DiskStatsService(context));
            } catch (Throwable e) {
                Slog.e(TAG, "Failure starting DiskStats Service", e);
            }

            try {
                Slog.i(TAG, "Freg Service");
                ServiceManager.addService("freg", new FregService());
            } catch (Throwable e) {
                Slog.e(TAG, "Failure starting Freg Service", e);
            }
        }
        //...
    }
}

//...
```

```text
# 重新编译services模块
mmm ./frameworks/base/services/java/
```

```text
# 重新打包Android系统镜像文件system.img
make snod
```

## 开发Android应用程序来使用硬件访问服务

```text
~/Android/packages/experimental/Freg
    # 配置文件
    AndroidManifest.xml
    Android.mk
    # 源代码目录
    src
        shy/luo/freg
            Freg.java
    # 资源目录res
    res
        layout
            main.xml
            values
                string.xml
            drawable
                icon.png
```

Freg.java :

```java
// Android\packages\experimental\Freg\src\shy\luo\freg

public class Freg extends Activity implements OnClickListener 
{
    private final static String LOG_TAG = "shy.luo.freg.FregActivity";

    private IFregService fregService = null;

    private EditText valueText = null;
    private Button readButton = null;
    private Button writeButton = null;
    private Button clearButton = null;

    /** Called when the activity is first created. */
    @Override
    public void onCreate(Bundle savedInstanceState) 
    {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);

        fregService = IFregService.Stub.asInterface(ServiceManager.getService("freg"));

        valueText = (EditText)findViewById(R.id.edit_value);
        readButton = (Button)findViewById(R.id.button_read);
        writeButton = (Button)findViewById(R.id.button_write);
        clearButton = (Button)findViewById(R.id.button_clear);

        readButton.setOnClickListener(this);
        writeButton.setOnClickListener(this);
        clearButton.setOnClickListener(this);

        Log.i(LOG_TAG, "Freg Activity Created");
    }

    @Override
    public void onClick(View v) 
    {
        if(v.equals(readButton)) 
        {
            try {
                int val = fregService.getVal();
                String text = String.valueOf(val);
                valueText.setText(text);
            } catch (RemoteException e) {
                Log.e(LOG_TAG, "Remote Exception while reading value from freg service.");
            }        
        }
        else if(v.equals(writeButton)) 
        {
            try {
                String text = valueText.getText().toString();
                int val = Integer.parseInt(text);
                fregService.setVal(val);
            } catch (RemoteException e) {
                Log.e(LOG_TAG, "Remote Exception while writing value to freg service.");
            }
        }
        else if(v.equals(clearButton)) 
        {
            String text = "";
            valueText.setText(text);
        }
    }
}
```

