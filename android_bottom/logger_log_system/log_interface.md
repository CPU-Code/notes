# Logger写接口

## C/C++日志写入接口

`Android`系统就提供了三组常用的C/C++宏来封装日志写入接口， 这些宏有的在程序的**非调试**版本中只是一个空定义， 因此， 可以避免在程序的发布版本中输出日志。

LOGV、 LOGD、 LOGI、 LOGW和LOGE， 它们用来写入类型为`main`的日志记录；

SLOGV、 SLOGD、 SLOGI、 SLOGW和SLOGE， 它们用来写入类型为`system`的日志记录；

LOG\_EVENT\_INT、 LOG\_EVENT\_LONG和LOG\_EVENT\_STRING， 它们用来写入类型为`events`的日志记录

目录 :

```c
~/Android/system/core/include
    cutils
        log.h
```

区分程序是调试版本还是发布版本 :

```c
// system\core\include\cutils\log.h

#ifndef LOG_NDEBUG
#ifdef NDEBUG
#define LOG_NDEBUG 1
#else
// 调试版本中定义为0
#define LOG_NDEBUG 0
#endif
#endif
```

```c
// system\core\include\cutils\log.h

// 当前编译单元的默认日志记录标签
#ifndef LOG_TAG
// 没有日志记录标签
#define LOG_TAG NULL
#endif
```

LOGV 、 LOGD 、 LOGI 、 LOGW 和LOGE

```c
// system\core\include\cutils\log.h

// 用来写入类型为main的日志记录
// 优先级分别为 VERBOSE、 DEBUG、 INFO、 WARN  ERROR
#ifndef LOGV
#if LOG_NDEBUG
#define LOGV(...)   ((void)0)
#else
// 调试版本
#define LOGV(...) ((void)LOG(LOG_VERBOSE, LOG_TAG, __VA_ARGS__))
#endif
#endif

/*
 * Simplified macro to send a debug log message using the current LOG_TAG.
 */
#ifndef LOGD
#define LOGD(...) ((void)LOG(LOG_DEBUG, LOG_TAG, __VA_ARGS__))
#endif

/*
 * Simplified macro to send an info log message using the current LOG_TAG.
 */
#ifndef LOGI
#define LOGI(...) ((void)LOG(LOG_INFO, LOG_TAG, __VA_ARGS__))
#endif

#ifndef LOGW
#define LOGW(...) ((void)LOG(LOG_WARN, LOG_TAG, __VA_ARGS__))
#endif

#ifndef LOGE
#define LOGE(...) ((void)LOG(LOG_ERROR, LOG_TAG, __VA_ARGS__))
#endif
```

使用宏LOG来实现日志写入功能 :

```c
// system\core\include\cutils\log.h

// 实现日志写入功能
#ifndef LOG

// priority加上前缀“ ANDROID_ ”
// ANDROID_＃ ＃ priority的参数都是类型为android_LogPriority的枚举值
#define LOG(priority, tag, ...) \
    LOG_PRI(ANDROID_##priority, tag, __VA_ARGS__)
#endif

#ifndef LOG_PRI

// 向Logger日志驱动程序中写入日志记录
#define LOG_PRI(priority, tag, ...) \
    android_printLog(priority, tag, __VA_ARGS__)
#endif

// 向Logger日志驱动程序中写入日志记录
#define android_printLog(prio, tag, fmt...) \
    __android_log_print(prio, tag, fmt)
```

android\_LogPriority的枚举值 :

```c
// system\core\include\android\log.h

typedef enum android_LogPriority {
    ANDROID_LOG_UNKNOWN = 0,
    ANDROID_LOG_DEFAULT,    /* only for SetMinPriority() */
    ANDROID_LOG_VERBOSE,
    ANDROID_LOG_DEBUG,
    ANDROID_LOG_INFO,
    ANDROID_LOG_WARN,
    ANDROID_LOG_ERROR,
    ANDROID_LOG_FATAL,
    ANDROID_LOG_SILENT,     /* only for SetMinPriority(); must be last */
} android_LogPriority;
```

SLOGV 、 SLOGD 、 SLOGI 、 SLOGW 和SLOGE :

```c
// system\core\include\cutils\log.h

// 用来写入类型为system的日志记录
// 写入的日志记录的优先级分别为VERBOSE、 DEBUG、 INFO、 WARN和ERROR
#ifndef SLOGV
#if LOG_NDEBUG
#define SLOGV(...)   ((void)0)
#else
// 调试版本
// 向Logger日志驱动程序中写入日志记录
#define SLOGV(...) ((void)__android_log_buf_print(LOG_ID_SYSTEM, ANDROID_LOG_VERBOSE, LOG_TAG, __VA_ARGS__))
#endif
#endif

#ifndef SLOGD
#define SLOGD(...) ((void)__android_log_buf_print(LOG_ID_SYSTEM, ANDROID_LOG_DEBUG, LOG_TAG, __VA_ARGS__))
#endif

#ifndef SLOGI
#define SLOGI(...) ((void)__android_log_buf_print(LOG_ID_SYSTEM, ANDROID_LOG_INFO, LOG_TAG, __VA_ARGS__))
#endif

#ifndef SLOGW
#define SLOGW(...) ((void)__android_log_buf_print(LOG_ID_SYSTEM, ANDROID_LOG_WARN, LOG_TAG, __VA_ARGS__))
#endif

#ifndef SLOGE
#define SLOGE(...) ((void)__android_log_buf_print(LOG_ID_SYSTEM, ANDROID_LOG_ERROR, LOG_TAG, __VA_ARGS__))
#endif
```

LOG\_EVENT\_INT 、 LOG\_EVENT\_LONG 和LOG\_EVENT\_STRING :

```c
// system\core\include\cutils\log.h

typedef enum 
{
    // 整数
    EVENT_TYPE_INT      = 0,
    // 长整数
    EVENT_TYPE_LONG     = 1,
    // 字符串
    EVENT_TYPE_STRING   = 2,
    // 列表
    EVENT_TYPE_LIST     = 3,
} AndroidEventLogType;

// 写入类型为events的日志记录
#define LOG_EVENT_INT(_tag, _value) {                                       \
        int intBuf = _value;                                                \
        // 往Logger日志驱动程序中写入日志记录
        (void) android_btWriteLog(_tag, EVENT_TYPE_INT, &intBuf,            \
            sizeof(intBuf));                                                \
    }

#define LOG_EVENT_LONG(_tag, _value) {                                      \
        long long longBuf = _value;                                         \
        // 往Logger日志驱动程序中写入日志记录
        (void) android_btWriteLog(_tag, EVENT_TYPE_LONG, &longBuf,          \
            sizeof(longBuf));                                               \
    }

// 往Logger日志驱动程序中写入一条内容为字符串值的日志记录
#define LOG_EVENT_STRING(_tag, _value)                                      \
// 空定义
    ((void) 0)  /* not implemented -- must combine len with string */

// 向Logger日志驱动程序中写入日志记录
#define android_printLog(prio, tag, fmt...) \
    __android_log_print(prio, tag, fmt)
```

## Java日志写入接口

Android系统在应用程序框架层中定义了三个Java日志写入接口，

android.util.Log`、`android.util.Slog`和`android.util.EventLog， 写入的日志记录类型分别为main、 system和events。

这三个Java日志写入接口是通过`JNI`方法来调用日志库liblog提供的函数来实现日志记录的写入功能的。

android.util.Log :

```java
// frameworks\base\core\java\android\util\Log.java

public final class Log 
{

    // 优先级分别为VERBOSE、 DEBUG、 INFO、 WARN  ERROR
    /**
     * Priority constant for the println method; use Log.v.
     */
    public static final int VERBOSE = 2;

    /**
     * Priority constant for the println method; use Log.d.
     */
    public static final int DEBUG = 3;

    /**
     * Priority constant for the println method; use Log.i.
     */
    public static final int INFO = 4;

    /**
     * Priority constant for the println method; use Log.w.
     */
    public static final int WARN = 5;

    /**
     * Priority constant for the println method; use Log.e.
     */
    public static final int ERROR = 6;

    /**
     * Priority constant for the println method.
     */
    public static final int ASSERT = 7;

    //...

    public static int v(String tag, String msg) 
    {
        // 实现日志记录写入功能
        return println_native(LOG_ID_MAIN, VERBOSE, tag, msg);
    }

    public static int v(String tag, String msg, Throwable tr) {
        return println_native(LOG_ID_MAIN, VERBOSE, tag, msg + '\n' + getStackTraceString(tr));
    }

    public static int d(String tag, String msg) {
        return println_native(LOG_ID_MAIN, DEBUG, tag, msg);
    }

    public static int d(String tag, String msg, Throwable tr) {
        return println_native(LOG_ID_MAIN, DEBUG, tag, msg + '\n' + getStackTraceString(tr));
    }

    public static int i(String tag, String msg) {
        return println_native(LOG_ID_MAIN, INFO, tag, msg);
    }

    public static int i(String tag, String msg, Throwable tr) {
        return println_native(LOG_ID_MAIN, INFO, tag, msg + '\n' + getStackTraceString(tr));
    }

    public static int w(String tag, String msg) {
        return println_native(LOG_ID_MAIN, WARN, tag, msg);
    }

    public static int w(String tag, String msg, Throwable tr) {
        return println_native(LOG_ID_MAIN, WARN, tag, msg + '\n' + getStackTraceString(tr));
    }

    public static int w(String tag, Throwable tr) {
        return println_native(LOG_ID_MAIN, WARN, tag, getStackTraceString(tr));
    }

    public static int e(String tag, String msg) {
        return println_native(LOG_ID_MAIN, ERROR, tag, msg);
    }

    public static int e(String tag, String msg, Throwable tr) {
        return println_native(LOG_ID_MAIN, ERROR, tag, msg + '\n' + getStackTraceString(tr));
    }

    //...

    /** @hide */ public static final int LOG_ID_MAIN = 0;
    /** @hide */ public static final int LOG_ID_RADIO = 1;
    /** @hide */ public static final int LOG_ID_EVENTS = 2;
    /** @hide */ public static final int LOG_ID_SYSTEM = 3;

    // 实现日志记录写入功能
    /** @hide */ public static native int println_native(int bufID,
            int priority, String tag, String msg);
}
```

```cpp
// frameworks\base\core\jni\android_util_Log.cpp

static jint android_util_Log_println_native(JNIEnv* env, jobject clazz,
        jint bufID, jint priority, jstring tagObj, jstring msgObj)
{
    const char* tag = NULL;
    const char* msg = NULL;

    // 检查写入的日志记录的内容msgObj是否为null
    if (msgObj == NULL) 
    {
        jclass npeClazz;

        npeClazz = env->FindClass("java/lang/NullPointerException");
        assert(npeClazz != NULL);

        env->ThrowNew(npeClazz, "println needs a message");
        return -1;
    }

    // 0、 1、 2和3四个值表示的日志记录的类型分别为main、radio、 events和system

    // 检查写入的日志记录的类型值是否位于0和LOG_ID_MAX之间
    if (bufID < 0 || bufID >= LOG_ID_MAX) 
    {
        jclass npeClazz;

        npeClazz = env->FindClass("java/lang/NullPointerException");
        assert(npeClazz != NULL);

        env->ThrowNew(npeClazz, "bad bufID");
        return -1;
    }

    if (tagObj != NULL)
        tag = env->GetStringUTFChars(tagObj, NULL);
    msg = env->GetStringUTFChars(msgObj, NULL);

    // 往Logger日志驱动程序中写入日志记录
    int res = __android_log_buf_write(bufID, (android_LogPriority)priority, tag, msg);

    if (tag != NULL)
        env->ReleaseStringUTFChars(tagObj, tag);
    env->ReleaseStringUTFChars(msgObj, msg);

    return res;
}
```

android.util.Slog :

```java
// frameworks\base\core\java\android\util\Slog.java

// 注释中有一个“@hide”关键字， 表示这是一个隐藏接口，
//只在系统内部使用， 应用程序一般不应该使用该接口来写入日志记录
/**
 * @hide
 */
public final class Slog 
{
    // 写入的日志记录的类型为system

    private Slog() {
    }

    // 成员函数有v、 d、 i、w和e
    // 优先级分别为VERBOSE、 DEBUG、 INFO、 WARN和ERROR

    public static int v(String tag, String msg) {
        return Log.println_native(Log.LOG_ID_SYSTEM, Log.VERBOSE, tag, msg);
    }

    public static int v(String tag, String msg, Throwable tr) {
        return Log.println_native(Log.LOG_ID_SYSTEM, Log.VERBOSE, tag,
                msg + '\n' + Log.getStackTraceString(tr));
    }

    public static int d(String tag, String msg) {
        return Log.println_native(Log.LOG_ID_SYSTEM, Log.DEBUG, tag, msg);
    }

    public static int d(String tag, String msg, Throwable tr) {
        return Log.println_native(Log.LOG_ID_SYSTEM, Log.DEBUG, tag,
                msg + '\n' + Log.getStackTraceString(tr));
    }

    public static int i(String tag, String msg) {
        return Log.println_native(Log.LOG_ID_SYSTEM, Log.INFO, tag, msg);
    }

    public static int i(String tag, String msg, Throwable tr) {
        return Log.println_native(Log.LOG_ID_SYSTEM, Log.INFO, tag,
                msg + '\n' + Log.getStackTraceString(tr));
    }

    public static int w(String tag, String msg) {
        return Log.println_native(Log.LOG_ID_SYSTEM, Log.WARN, tag, msg);
    }

    public static int w(String tag, String msg, Throwable tr) {
        return Log.println_native(Log.LOG_ID_SYSTEM, Log.WARN, tag,
                msg + '\n' + Log.getStackTraceString(tr));
    }

    public static int w(String tag, Throwable tr) {
        return Log.println_native(Log.LOG_ID_SYSTEM, Log.WARN, tag, Log.getStackTraceString(tr));
    }

    public static int e(String tag, String msg) {
        return Log.println_native(Log.LOG_ID_SYSTEM, Log.ERROR, tag, msg);
    }

    public static int e(String tag, String msg, Throwable tr) {
        return Log.println_native(Log.LOG_ID_SYSTEM, Log.ERROR, tag,
                msg + '\n' + Log.getStackTraceString(tr));
    }

    public static int println(int priority, String tag, String msg) {
        return Log.println_native(Log.LOG_ID_SYSTEM, priority, tag, msg);
    }
}
```

android.util.EventLog :

```java
// frameworks\base\core\java\android\util\EventLog.java

public class EventLog 
{
    //...
    // 向Logger日志驱动程序中写入类型为events的日志记录， 
    // 这些日志记录的内容分别为整数、 长整数、 字符串和列表。

    // 整数
    public static native int writeEvent(int tag, int value);

    // 长整数
    public static native int writeEvent(int tag, long value);

    // 字符串
    public static native int writeEvent(int tag, String str);

    // 列表
    public static native int writeEvent(int tag, Object... list);

    //...
}
```

写入**整数**和**长整数**类型日志记录的`JNI`方法`writeEvent`的实现 :

```cpp
//frameworks\base\core\jni\android_util_EventLog.cpp

static jint android_util_EventLog_writeEvent_Integer(JNIEnv* env, jobject clazz,
                                                     jint tag, jint value)
{
    // 向Logger日志驱动程序中写入日志记录
    // EVENT_TYPE_INT : 要写入的日志记录的内容为一个整数
    return android_btWriteLog(tag, EVENT_TYPE_INT, &value, sizeof(value));
}

static jint android_util_EventLog_writeEvent_Long(JNIEnv* env, jobject clazz,
                                                  jint tag, jlong value)
{
    // 向Logger日志驱动程序中写入日志记录
    // EVENT_TYPE_LONG :  要写入的日志记录的内容为一个长整数
    return android_btWriteLog(tag, EVENT_TYPE_LONG, &value, sizeof(value));
}
```

整数的日志记录的内存布局 :

![image-20200720151030266](https://gitee.com/cpu_code/picture_bed/raw/master//20200720151030.png)

长整数的日志记录的内存布局 :

![image-20200720151039785](https://gitee.com/cpu_code/picture_bed/raw/master//20200720151039.png)

宏android\_btWriteLog的定义 :

```cpp
// system\core\include\cutils\log.h

// 往Logger日志驱动程序中写入日志记录
#define android_btWriteLog(tag, type, payload, len) \
    __android_log_btwrite(tag, type, payload, len)
```

写入字符串类型日志记录的JNI方法writeEvent的实现 :

```cpp
static jint android_util_EventLog_writeEvent_String(JNIEnv* env, jobject clazz,
                                   jint tag, jstring value) 
{
    uint8_t buf[MAX_EVENT_PAYLOAD];

    // Don't throw NPE -- I feel like it's sort of mean for a logging function
    // to be all crashy if you pass in NULL -- but make the NULL value explicit.
    const char *str = value != NULL ? env->GetStringUTFChars(value, NULL) : "NULL";
    jint len = strlen(str);
    const int max = sizeof(buf) - sizeof(len) - 2;  // Type byte, final newline
    if (len > max) 
    {
        len = max;
    }

    // 日志记录内容的类型为字符串
    buf[0] = EVENT_TYPE_STRING;
    // 字符串的长度
    memcpy(&buf[1], &len, sizeof(len));
    // 字符串内容
    memcpy(&buf[1 + sizeof(len)], str, len);
    // ‘\n'来结束该日志记录
    buf[1 + sizeof(len) + len] = '\n';

    if (value != NULL) env->ReleaseStringUTFChars(value, str);

    // 将日志记录写入到Logger日志驱动程序
    return android_bWriteLog(tag, buf, 2 + sizeof(len) + len);
}
```

内容为字符串的日志记录的内存布局 :

![image-20200720151051111](https://gitee.com/cpu_code/picture_bed/raw/master//20200720151051.png)

第一个字段记录日志记录内容的类型为字符串，

第二个字段描述该字符串的长度，

第三个字段保存的是字符串内容，

第四个字段使用特殊字符‘\n'来结束该日志记录

写入列表类型日志记录的JNI方法`writeEvent`的实现 :

```cpp
static jint android_util_EventLog_writeEvent_Array(JNIEnv* env, jobject clazz,
                                  jint tag, jobjectArray value) 
{
    if (value == NULL) 
    {
        return android_util_EventLog_writeEvent_String(env, clazz, tag, NULL);
    }

    uint8_t buf[MAX_EVENT_PAYLOAD];
    const size_t max = sizeof(buf) - 1;  // leave room for final newline
    size_t pos = 2;  // Save room for type tag & array count

    // 一个列表中最多可以包含255个元素
    jsize copied = 0, num = env->GetArrayLength(value);
    for (; copied < num && copied < 255; ++copied) 
    {
        // 依次取出列表中的元素， 且根据它们的值类型来组织缓冲区buf的内存布局
        jobject item = env->GetObjectArrayElement(value, copied);

        if (item == NULL || env->IsInstanceOf(item, gStringClass)) 
        {
            // 值类型为字符串
            if (pos + 1 + sizeof(jint) > max) 
                break;
            const char *str = item != NULL ? env->GetStringUTFChars((jstring) item, NULL) : "NULL";
            jint len = strlen(str);
            if (pos + 1 + sizeof(len) + len > max) 
                len = max - pos - 1 - sizeof(len);

            // EVENT_TYPE_STRING值
            buf[pos++] = EVENT_TYPE_STRING;
            // 字符串长度值
            memcpy(&buf[pos], &len, sizeof(len));
            // 字符串的内容
            memcpy(&buf[pos + sizeof(len)], str, len);

            pos += sizeof(len) + len;
            if (item != NULL) 
                env->ReleaseStringUTFChars((jstring) item, str);
        } 
        else if (env->IsInstanceOf(item, gIntegerClass)) 
        {
            // 值类型为整数

            jint intVal = env->GetIntField(item, gIntegerValueID);
            if (pos + 1 + sizeof(intVal) > max) 
                break;

            // 第一个字节设置为一个EVENT_TYPE_INT值
            buf[pos++] = EVENT_TYPE_INT;
            // 整数值
            memcpy(&buf[pos], &intVal, sizeof(intVal));
            pos += sizeof(intVal);
        } 
        else if (env->IsInstanceOf(item, gLongClass)) 
        {
            // 值类型为长整数

            jlong longVal = env->GetLongField(item, gLongValueID);
            if (pos + 1 + sizeof(longVal) > max) 
                break;

            // buf的内容就为一个EVENT_TYPE_LONG值
            buf[pos++] = EVENT_TYPE_LONG;
            // 长整数值
            memcpy(&buf[pos], &longVal, sizeof(longVal));
            pos += sizeof(longVal);
        } 
        else 
        {
            // 抛出一个异常

            jniThrowException(env,
                    "java/lang/IllegalArgumentException",
                    "Invalid payload item type");
            return -1;
        }
        env->DeleteLocalRef(item);
    }

    // 列表类型的日志记录
    buf[0] = EVENT_TYPE_LIST;
    // 列表元素个数
    buf[1] = copied;
    // 日志记录的结束标志
    buf[pos++] = '\n';
    // 将日志记录写入到Logger日志驱动程序中
    return android_bWriteLog(tag, buf, pos);
}
```

内容为列表的日志记录的内存布局 :

![image-20200720151100719](https://gitee.com/cpu_code/picture_bed/raw/master//20200720151100.png)

