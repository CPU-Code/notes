



------------------------

## Logcat工具分析  



将日志记录写入到`Logger`日志驱动程序的目的是要将它们记录起来， 等到合适的时候再将它们读取和显示出来， 从而帮助我们分析程序的行为。 

分析日志查看工具Logcat的实现来学习日志记录的读取过程。

`Logcat`是内置在Android系统中的一个实用工具， 可以在主机上执行`adb logcat`命令来查看目标设备上的日志记录。



```shell
# 将Android模拟器启动起来
emulator &

# 激活Android 模拟器中的 Logcat 工具
adb logcat
```

就可以不断地看到`Android`模拟器中的日志记录输出  



在Android模拟器启动之后  :

```shell
# 获得帮助
adb logcat --help
```



`Logcat`工具主要的源代码文件 :

```shell
~/Android/system/core
	include
		cutils
			logprint.h	# 处理日志记录的输出
			event_tag_map.h	# 处理日志记录的输出
			logger.h	# 定义了一些基础数据结构和宏
		android
			log.h # 定义了一些基础数据结构和宏
	liblog	# 处理日志记录的输出
		logprint.c	
		event_tag_map.c
	logcat
		logcat.cpp # Logcat工具的源代码实现
```



使用`Logcat`工具读取和显示`Logger`日志驱动程序中的日志记录的过程， 主要包括三个情景， 分别是**工具初始化过程**、**日志记录的读取**和**输出过程**  



### 基础数据结构  



struct logger_entry  :



```c
// system\core\include\cutils\logger.h

// 从Logger日志驱动程序中提取出来的

// 描述一条日志记录
struct logger_entry {
    uint16_t    len;    /* length of the payload */
    uint16_t    __pad;  /* no matter what, we get 2 bytes of padding */
    int32_t     pid;    /* generating process's pid */
    int32_t     tid;    /* generating process's tid */
    int32_t     sec;    /* seconds since Epoch */
    int32_t     nsec;   /* nanoseconds */
    char        msg[0]; /* the entry's payload */
};

// 描述一条日志记录的最大长度， 包括日志记录头和日志记录有效负载两部分内容的长度
#define LOGGER_ENTRY_MAX_LEN		(4*1024)

// 描述日志记录有效负载的最大长度
#define LOGGER_ENTRY_MAX_PAYLOAD	\
	(LOGGER_ENTRY_MAX_LEN - sizeof(struct logger_entry))
```



struct queued_entry_t  :



```c++
// system\core\logcat\logcat.cpp

// 描述一个日志记录队列
struct queued_entry_t 
{
    // Logcat工具将相同类型的日志记录按照写入时间的先后顺序保存在同一个队列中，
    //这样在输出日志记录时， 沿着这个队列就可以按照时间顺序来依次输出系统中的日志信息了
    
    // 联合体， 用来描述一条日志记录的内容
    union 
    {
        unsigned char buf[LOGGER_ENTRY_MAX_LEN + 1] __attribute__((aligned(4)));
        struct logger_entry entry __attribute__((aligned(4)));
    };
    // 连接下一条日志记录， 从而形成一个队列
    queued_entry_t* next;

    queued_entry_t() 
    {
        next = NULL;
    }
};
```



struct log_device_t  :



```c++

```























### 初始化过程

















### 日志记录的读取过程

















### 日志记录的输出过程  















