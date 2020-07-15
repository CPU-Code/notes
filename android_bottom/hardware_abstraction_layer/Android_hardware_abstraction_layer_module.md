## 开发Android硬件抽象层模块

--------------------

### 硬件抽象层模块编写规范



-------------------------------

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



--------------------------------------

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



---------------------------------------

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



-----------------------------------------

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



-------------------------------------

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



-----------------------------------

#### 解压ramdisk.img镜像文件

```text

```

```text
# 将ramdisk.img改名为ramdisk.img.gz
mv ./out/target/product/generic/reamdisk.img ./reamdisk.img.gz

# 解压
gunzip ./ramdisk.img.gz
```



-------------------------

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



--------------------

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