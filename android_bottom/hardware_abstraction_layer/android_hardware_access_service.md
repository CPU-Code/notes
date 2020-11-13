# 安卓硬件访问服务

![](https://gitee.com/cpu_code/picture_bed/raw/master//20200715211412.png)

硬件访问服务通过**硬件抽象层模块**来为应用程序**提供硬件读写**操作。 由于**硬件抽象层模块**是使用`C++`语言开发的， 而应用程序框架层中的**硬件访问服务**是使用`Java`语言开发的， 因此， 硬件访问服务必须通过`Java`本地接口（`Java Native Interface`， `JNI`） 来调用硬件抽象层模块的接口。

![image-20200716105744277](https://gitee.com/cpu_code/picture_bed/raw/master//20200716105744.png)

`Android`系统的**硬件访问服务**通常运行在系统进程`System`中， 而使用这些硬件访问服务的**应用程序**运行在另外的进程中， 即应用程序需要通过**进程间通信机制**来访问这些硬件访问服务。

`Android`系统提供了一种高效的进程间通信机制——`Binder`进程间通信机制， 应用程序就是通过它来访问运行在系统进程`System`中的硬件访问服务的。 `Binder`进程间通信机制要求提供服务的一方**必须实现**一个具有**跨进程访问**能力的服务接口， 以便使用服务的一方可以通过这个**服务接口**来访问它。 因此， 在实现硬件访问服务之前， 我们首先要定义它的服务接口。

## 定义硬件访问服务接口

`Android`系统提供了一种描述语言来定义具有**跨进程访问**能力的服务接口， 这种描述语言称为`Android`接口描述语言（`Android Interface Definition Language`， `AIDL`） 。 以`AIDL`定义的服务接口文件是以`aidl`为后缀名的， 在编译时， 编译系统会将它们转换成`Java`文件， 然后再对它们进行编译。

硬件访问服务接口定义 :

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

由于服务接口`IFregService`是使用`AIDL`语言描述的， 所以 需要将其添加到**编译脚本**文件中， 这样编译系统才能将其转换为`Java`文件， 然后再对它进行编译

编译脚本文件 :

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

编译 :

```text
# 对硬件访问服务接口IFregService进行编译
mmm ./frameworks/base/
```

编译后得到的`framework.jar`文件就包含有`IFregService`接口， 它继承了`android.os.IInterface`接口。

在`IFregService`接口内部， 定义了一个`Binder`本地对象类`Stub`，它实现了`IFregService`接口， 并且继承了`android.os.Binder`类。 此外， 在`IFregService.Stub`类内部， 还定义了一个`Binder`代理对象类`Proxy`， 它同样也实现了`IFregService`接口。

用`AIDL`定义的服务接口是用来进行**进程间通信**的， 其中， **提供服务**的进程称为`Server`进程， 而**使用服务**的进程称为`Client`进程。

在`Server`进程中， 每一个服务都对应有一个`Binder`本地对象， 它通过一个桩（`Stub`） 来等待`Client`进程发送进程间通信请求。

`Client`进程在访问运行`Server`进程中的服务之前， 首先要获得它的一个`Binder`代理对象接口（`Proxy`） ， 然后通过这个`Binder`代理对象接口向它发送进程间通信请求。

![image-20200716110435819](https://gitee.com/cpu_code/picture_bed/raw/master//20200716110436.png)

## 实现硬件访问服务

硬件访问服务实现 :

```java
// frameworks\base\services\java\com\android\server\FregService.java

package com.android.server;

import android.content.Context;
import android.os.IFregService;
import android.util.Slog;

// 硬件访问服务 FregService 继承了 IFregService.Stub 类
public class FregService extends IFregService.Stub 
{
    private static final String TAG = "FregService";

    private int mPtr = 0;

    FregService() 
    {
        // 调用 JNI 方法 init_native 来打开虚拟硬件设备 freg ，
        // 并且获得它的一个句柄值， 保存在成员变量 mPtr 中
        // 这个句柄值实际上是指向虚拟硬件设备freg在硬件抽象层中的一个设备对象
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

编译后得到的`services.jar`文件就包含有`FregService`类

## 实现硬件访问服务的JNI方法

实现了硬件访问服务`FregService`的`JNI`方法:

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
            //打开虚拟硬件设freg , 打开设备ID为 FREG_HARDWARE_DEVICE_ID 的硬件设备
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
    // 将函数freg_init、 freg_setVal和 freg_getVal的JNI方法注册
    //为init_native、 setVal_native和 getVal_native
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

增加 `register_android_server_FregService`函数的**声明**和**调用** :

`onload.cpp`文件实现在`libandroid_servers`模块中。 当系统加载`libandroid_servers`模块时， 就会调用实现在`onload.cpp`文件中的`JNI_OnLoad`函数。 这样， 就可以将前面定义的三个`JNI`方法`init_native`、 `setVal_native`和`getVal_native`注册到`Java`虚拟机中

![image-20200716121659492](https://gitee.com/cpu_code/picture_bed/raw/master//20200716121659.png)

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

进入到`frameworks/base/services/jni`目录中， 打开里面的`Android.mk`文件， 修改变量`LOCAL_SRC_FILES`的值

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

编译后得到的`libandroid_servers.so`文件就包含有`init_native`、 `setVal_native`和`getVal_native`这三个`JNI`方法

## 启动硬件访问服务

`Android`系统的**硬件访问服务**通常是在**系统进程**`System`中启动的， 而系统进程`System`是由**应用程序孵化器**进程`Zygote`负责启动的。 由于应用程序孵化器进程`Zygote`是在系统启动时启动的， 所以要把**硬件访问服务**运行在系统进程`System`中， 就实现了开机时自动启动

![image-20200716102646715](https://gitee.com/cpu_code/picture_bed/raw/master//20200716102646.png)

打开里面的`SystemServer.java`文件， 修改`ServerThread`类的成员函数`run`的实现

```java
// frameworks/base/services/java/com/android/server/SystemServer.java
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

系统进程`System`在启动时， 会创建一个`ServerThread`线程来启动系统中的关键服务，其中就包括一些硬件访问服务。

在`ServerThread`类的成员函数`run`中， 首先创建一个`FregService`实例， 然后把它注册到`Service Manager`中。 `Service Manager`是`Android`系统的`Binder`进程间通信机制的一个重要角色， 它负责**管理系统中的服务对象**。

注册到`Service Manager`中的服务对象都有一个对应的名称， 使用这些服务的`Client`进程就是通过这些名称来向`Service Manager`请求它们的`Binder`代理对象接口的， 以便可以访问它们所提供的服务。 硬件访问服务`FregService`注册到`Service Manager`之后， 它的启动过程就完成了

![image-20200716103437599](https://gitee.com/cpu_code/picture_bed/raw/master//20200716103437.png)

```text
# 重新编译services模块
mmm ./frameworks/base/services/java/
```

编译后得到的`services.jar`文件就包含有硬件访问服务`FregService`， 并且在系统启动时， 将它运行在系统进程`System`中

```text
# 重新打包Android系统镜像文件system.img
make snod
```

> 由于个人水平有限, 难免有些错误, 希望各位点评
>
> * @Author: cpu\_code
> * @Date: 2020-07-15 12:31:07
> * @LastEditTime: 2020-07-16 12:19:42
> * @FilePath: \notes\android\_bottom\hardware\_abstraction\_layer\Android\_hardware\_access\_service.md
> * @Gitee: [https://gitee.com/cpu\_code](https://gitee.com/cpu_code)
> * @Github: [https://github.com/CPU-Code](https://github.com/CPU-Code)
> * @CSDN: [https://blog.csdn.net/qq\_44226094](https://blog.csdn.net/qq_44226094)
> * @Gitbook: [https://923992029.gitbook.io/cpucode/](https://923992029.gitbook.io/cpucode/)

