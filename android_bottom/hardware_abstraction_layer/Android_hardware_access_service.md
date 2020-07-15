
## 开发Android硬件访问服务

 * @Author: cpu_code
 * @Date: 2020-07-15 12:31:07
 * @LastEditTime: 2020-07-15 13:25:47
 * @FilePath: \notes\android_bottom\hardware_abstraction_layer\Android_hardware_access_service.md
 * @Gitee: https://gitee.com/cpu_code
 * @Github: https://github.com/CPU-Code
 * @CSDN: https://blog.csdn.net/qq_44226094
 * @Gitbook: https://923992029.gitbook.io/cpucode/

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

```shell
# 重新编译services模块
mmm ./frameworks/base/services/java/
```

```shell
# 重新打包Android系统镜像文件system.img
make snod
```
