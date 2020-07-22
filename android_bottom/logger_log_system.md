

# Logger日志系统

`Android`提供的`Logger`日志系统是基于内核中的`Logger`日志驱动程序实现的， 它会将日志记录保存在**内核空间**中。  

`Logger`日志驱动程序在内部使用一个**环形缓冲区**来保存日志。 所以 当`Logger`日志驱动程序中的环形缓冲区满了之后， 新的日志就会覆盖旧的日志。  

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200719180652.png" alt="image-20200719180652875" style="zoom:67%;" />

Logger日志驱动程序根据**日志的类型**及**日志的输出量**来进行分类， 日志的类型一共有四种， 它们分别是`main`、 `system`、 `radio`和`events`。

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200719181103.png" alt="image-20200719181103329" style="zoom: 67%;" />

在Logger日志驱动程序中， 这四种类型的日志分别通过`/dev/log/main`、 `/dev/log/system`、 `/dev/log/radio`和 `/dev/log/events`四个设备文件来进行访问 的

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200719181420.png" alt="image-20200719181420876" style="zoom: 67%;" />

类型为`main`的日志是**应用程序**级别的， 而类型为`system`的日志是**系统级别**的。 

类型为`radio`的日志是与**无线设备**相关的， 因为数据量很大， 所以  把它们单独记录在一起， 也可以避免它们覆盖其他类型的日志。

类型为`events`的日志是专门用来**诊断系统问题**的， 应用程序开发者一般不使用该种类型的日志。  

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200719181857.png" alt="image-20200719181857429" style="zoom:67%;" />



-------------

`Android`系统在**应用程序框架层**中提供了`android.util.Log`、 `android.util.Slog`和`android.util.EventLog`三个Java接口来往Logger日志驱动程序中写入日志， 它们写入的日志类型分别为`main`、 `system`和`events`。 

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200719190216.png" alt="image-20200719190216722" style="zoom:67%;" />

**注意**:  如果使用`android.util.Log`和`android.util.Slog`接口写入的日志的标签值是以“`RIL`”开头 或 等于“`HTC_RIL`”、 “`AT`”、 “`GSM`”、 “`STK`”、 “`CDMA`”、 “`PHONE`”和“`SMS`”时， 它们就会被转换为`radio`类型的日志写入到`Logger`日志驱动程序中。 

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200719190743.png" alt="image-20200719190743669" style="zoom:67%;" />

-------------------

对应者有 `Android`系统在**运行时库层**也提供了三组C/C++宏来往`Logger`日志驱动程序中写入日志。 

其中， 宏`LOGV`、`LOGD`、 `LOGI`、 `LOGW`和`LOGE`用来写入`main`类型的日志， 

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200719191203.png" alt="image-20200719191203480" style="zoom:67%;" />

宏`SLOGV`、 `SLOGD`、`SLOGI`、 `SLOGW`和`SLOGE`用来写入`system`类型的日志， 

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200719191307.png" alt="image-20200719191307622" style="zoom:67%;" />

宏`LOG_EVENT_INT`、`LOG_EVENT_LONG`和`LOG_EVENT_STRING`用来写入`events`类型的日志。 

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200719191427.png" alt="image-20200719191427203" style="zoom:67%;" />



----------

不管是Java日志写入接口还是C/C++日志写入接口， 它们都是通过运行时库层的日志库`liblog`来往`Logger`日志驱动程序中写入日志的。 另外，系统还提供了一个**Logcat工具**来读取和显示`Logger`日志驱动程序中的日志。  



Logger日志系统框架  : 

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200718120435.png" style="zoom: 50%;" />



-------------

## Logger日志格式  



`Logger`日志一共划分为`main`、 `system`、 `radio`和`events`四种类型  



main、 system 和 radio 的日志格式  :

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200719192334.png" alt="image-20200719192334248" style="zoom: 67%;" />



`priority` : 日志优先级， 整数 ; `tag` : 日志标签， 字符串； `msg` : 日志内容， 字符串  

日志优先级和日志标签可以作为在显示日志中的**过滤字段**。 日志优先级按照重要程度一般划分为`VERBOSE`、 `DEBUG`、 `INFO`、`WARN`、 `ERROR`和`FATAL`六种。  

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200719192556.png" alt="image-20200719192556290" style="zoom:67%;" />



------------------------------------

`events`的日志格式  :



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200719193309.png" alt="image-20200719193309682" style="zoom:67%;" />



tag : 日志标签， 整数； msg : 日志内容， 二进制数据，它的内容格式是由**日志写入者来决定**的。

 一般，日志内容是由一个或 多个值组成的， 每个值的前面都有一个字段来描述它的类型



---------------

`events`的日志内容的格式  :

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200719194813.png" alt="image-20200719194813521" style="zoom: 80%;" />



值的类型为**整数**（int）对应 1  、 **长整数**（long）2 、 **字符串**（string）3 或 **列表**（list） 4



`events`类型的日志标签是一个整数值， 在显示时不具有可读性，所以 `Android`系统用设备上的日志标签文件`/system/etc/event-log-tags`来描述这些标签值的含义。 这样，`Logcat`工具在显示**events**类型的日志时， 就可以把日志中的标签值转换为**字符串**  



--------

日志标签文件还用来描述**events**类型的日志内容的格式  

日志标签文件格式  :

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200719195258.png" alt="image-20200719195257831" style="zoom: 80%;" />



tag number : 日志标签值， 它的取值范围为0～2147483648 = 2 * 2^30； 

tag name :  日志标签值对应的字符串描述， 它是由字母[A-Z][a-z]、 数字[0-9] 或 下画线“_”组成的；

format for tag value : 描述组成日志内容的值格式  



------

events的日志内容的值格式  :



![events的日志内容的值格式  ](https://gitee.com/cpu_code/picture_bed/raw/master//20200718122136.png)



name : 日志内容值的名称； 

data type :  日志内容值的数据类型， 它的取值范围为1～4， 分别表示**整数**（int） 、 **长整数**（long） 、 **字符串**（string） 和**列表**（list） ；

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200719200800.png" alt="image-20200719200800560" style="zoom:67%;" />

data unit :  日志内容值的数据单位， 它的取值范围是1～6， 分别表示**对象数量**（numberof objects） 、 **字节**（Number of bytes） 、 **毫秒数**（Number of milliseconds） 、 **分配额**（Number of allocations） 、 **标志**（ID） 和 **百分比**（Percent）   

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200719201122.png" alt="image-20200719201122683" style="zoom:67%;" />



从Android模拟器上的日志标签文件`/system/etc/event-log-tags`中取出一行内容来说明类型为`events`的日志格式  

![image-20200718123314841](https://gitee.com/cpu_code/picture_bed/raw/master//20200718123314.png)

2722 : 日志标签值， battery_level :  描述日志标签值2722的含义。

日志标签值 == 2722的日志内容由三个值组成， 它们分别是level、 voltage和temperature，

level :  数据类型 :  整数 ;  单位 :  百分比

voltage :  数据类型 :  整数 ;  单位 :  对象数量

temperature :  数据类型 :  整数 ;  单位 :  对象数量



--------------------

## Logger日志驱动程序  



Logger日志驱动程序实现在系统的内核空间中  

目录结构  :

```shell
~/Android/kernel/goldfish
	drivers
		staging
			android
				logger.h # 源文件
				logger.c
```



-------------------------

### 基础数据结构  



Logger日志驱动程序主要使用到了`logger_entry`、 `logger_log` ,  `logger_reader`三个结构体  



struct logger_entry  :

```c
// kernel\goldfish\drivers\staging\android\logger.h

// 描述一个日志记录
struct logger_entry 
{
/* 实际的日志记录的有效负载长度 */
	__u16		len;	/* length of the payload */
	/* 将成员变量pid的地址值对齐到4字节边界的 */
	__u16		__pad;	/* no matter what, we get 2 bytes of padding */
	/* 写入该日志记录的进程的TGID  线程组ID */
	__s32		pid;	/* generating process's pid */
	/* 写入该日志记录的进程的PID 进程ID */
	__s32		tid;	/* generating process's tid */
	/* 写入该日志记录的时间 用标准基准时间（Epoch）来描述*/
	__s32		sec;	/* seconds since Epoch */
	/* 写入该日志记录的时间 */
	__s32		nsec;	/* nanoseconds */
	/* 实际写入的日志记录内容，长度是可变的，len决定 */
	char		msg[0];	/* the entry's payload */
};

//每一个日志记录的最大长度 == 4K
#define LOGGER_ENTRY_MAX_LEN		(4*1024)

// 有效负载长度最大 == 4K - 结构体logger_entry的大小
#define LOGGER_ENTRY_MAX_PAYLOAD	\
	(LOGGER_ENTRY_MAX_LEN - sizeof(struct logger_entry))
```



struct logger_log  :

```c
// kernel\goldfish\drivers\staging\android\logger.c
// 描述一个日志缓冲区
struct logger_log 
{
	//指向一个内核缓冲区， 用来保存日志记录， 它是循环使用的
	unsigned char *		buffer;	/* the ring buffer itself */
	// 描述一个日志设备
	struct miscdevice	misc;	/* misc device representing the log */
	// 等待队列，用来记录那些正在等待读取新的日志记录的进程, 每一个进程都使用一个结构体logger_reader来描述
	wait_queue_head_t	wq;	/* wait queue for readers */
	// 列表， 用来记录那些正在读取日志记录的进程
	struct list_head	readers; /* this log's readers */
	// 互斥量， 用来保护日志缓冲区的并发访问
	struct mutex		mutex;	/* mutex protecting buffer */
	// 下一条要写入的日志记录在日志缓冲区中的位置
	size_t			w_off;	/* current write head offset */
	// 当一个新的进程读取日志时，会从日志缓冲区中的什么位置开始读取
	size_t			head;	/* new readers start here */
	// 大小
	size_t			size;	/* size of the log */
};
```



struct logger_reader  :



```c
// 描述一个正在读取某一个日志缓冲区的日志记录的进程
struct logger_reader 
{
	// 指向要读取的日志缓冲区结构体
	struct logger_log *	log;	/* associated log */
	// 一个列表项， 用来连接所有读取同一种类型日志记录的进程
	struct list_head	list;	/* entry in logger_log's list */
	// 当前进程要读取的下一条日志记录在日 志缓冲区中的位置
	size_t			r_off;	/* current read head offset */
};
```



-------------------

### 日志设备的初始化过程  



日志设备的初始化过程是在`Logger`日志驱动程序的入口函数`logger_init`中进行的。



```c
// kernel\goldfish\drivers\staging\android\logger.c

#define DEFINE_LOGGER_DEVICE(VAR, NAME, SIZE) \
// 将各自的日志记录保存在一个静态分配的unsigned char数组
// 大小由SIZE决定
static unsigned char _buf_ ## VAR[SIZE]; \
static struct logger_log VAR = { \
	.buffer = _buf_ ## VAR, \
	.misc = { \
		.minor = MISC_DYNAMIC_MINOR, \
		.name = NAME, \
		.fops = &logger_fops, \
		.parent = NULL, \
	}, \
	// 等待队列
	.wq = __WAIT_QUEUE_HEAD_INITIALIZER(VAR .wq), \
	// 队列
	.readers = LIST_HEAD_INIT(VAR .readers), \
	// 互斥量
	.mutex = __MUTEX_INITIALIZER(VAR .mutex), \
	.w_off = 0, \
	.head = 0, \
	.size = SIZE, \
};

// 宏DEFINE_LOGGER_DEVICE来定义的
// 传入参数分别为变量名、 设备文件名，分配的日志缓冲区的大小
// system和 main 的日志记录保存在同一个日志缓冲区 log_main 中 大小为64K
// events的日志记录保存在日志缓冲区 log_events 大小为256K
// radio的日志记录保存在日志缓冲区 log_radio 大小为64K
DEFINE_LOGGER_DEVICE(log_main, LOGGER_LOG_MAIN, 64*1024)
DEFINE_LOGGER_DEVICE(log_events, LOGGER_LOG_EVENTS, 256*1024)
DEFINE_LOGGER_DEVICE(log_radio, LOGGER_LOG_RADIO, 64*1024)
```



```c
// kernel\goldfish\drivers\staging\android\logger.h

#define LOGGER_LOG_RADIO	"log_radio"	/* radio-related messages */
#define LOGGER_LOG_EVENTS	"log_events"	/* system/hardware events */
#define LOGGER_LOG_MAIN		"log_main"	/* everything else */
```



`Logger`日志驱动程序初始化完成之后， 我们就可以在`/dev/log`目录下看到三个日志设备文件`main`、 `events`和`radio`  



三个日志设备对应的文件操作函数表均为`logger_fops`  :

```c
// kernel\goldfish\drivers\staging\android\logger.c

static struct file_operations logger_fops = {
	.owner = THIS_MODULE,
	.read = logger_read,
	.aio_write = logger_aio_write,
	.poll = logger_poll,
	.unlocked_ioctl = logger_ioctl,
	.compat_ioctl = logger_ioctl,
	.open = logger_open,
	.release = logger_release,
};
```

日志设备打开、 读取和写入函数`open`、 `read`和`aio_write`的定义， 它们分别被设置为`logger_open`、 `logger_read`和`logger_aio_write`函数  



每一个硬件设备都有自己的**主设备号**和**从设备号**。 因为日志设备的类型为misc， 所以这类型的设备都是一样的主设备号， 因此， 在定义这些日志设备时， 就不需要指定它们的主设备号了， 等到注册这些日志设备时， 会统一指定。 

从宏`DEFINE_LOGGER_DEVICE`的定义可以看出， 这些日志设备的从设备号被设置为`MISC_DYNAMIC_MINOR`， 表示由系统动态分配  



```c
// kernel\goldfish\include\linux\miscdevice.h
#define MISC_DYNAMIC_MINOR	255
```



`logger_init`日志设备的初始化过程  :

```c
// kernel\goldfish\drivers\staging\android\logger.c

static int __init logger_init(void)
{
	int ret;
	// 将三个日志设备注册到系统

	ret = init_log(&log_main);
	if (unlikely(ret))
		goto out;

	ret = init_log(&log_events);
	if (unlikely(ret))
		goto out;

	ret = init_log(&log_radio);
	if (unlikely(ret))
		goto out;

out:
	return ret;
}
```



`init_log`来实现  :

```c
// kernel\goldfish\drivers\staging\android\logger.c

static int __init init_log(struct logger_log *log)
{
	int ret;

	//将相应的日志设备注册到系统
	ret = misc_register(&log->misc);
	if (unlikely(ret)) {
		printk(KERN_ERR "logger: failed to register misc "
		       "device for log '%s'!\n", log->misc.name);
		return ret;
	}

	printk(KERN_INFO "logger: created %luK log '%s'\n",
	       (unsigned long) log->size >> 10, log->misc.name);

	return 0;
}
```



misc_register实现 : 

```c
// kernel\goldfish\drivers\char\misc.c

int misc_register(struct miscdevice * misc)
{
	struct miscdevice *c;
	dev_t dev;
	int err = 0;
	// 系统中所有的misc设备都保存在一个misc_list列表
	
	INIT_LIST_HEAD(&misc->list);

	mutex_lock(&misc_mtx);

	// 检查所要注册的misc设备的从设备号是否已经被注册
	list_for_each_entry(c, &misc_list, list) 
	{
		if (c->minor == misc->minor) 
		{
			// 注册失败，因为同一类型设备的从设备号是不能相同的
			
			mutex_unlock(&misc_mtx);
			return -EBUSY;
		}
	}

	// 从设备号是动态分配的 == MISC_DYNAMIC_MINOR
	if (misc->minor == MISC_DYNAMIC_MINOR) 
	{
		// 分配的misc从设备号一共有64个

		int i = DYNAMIC_MINORS;
		
		while (--i >= 0)
		{
			// 从设备号的使用情况记录在数组misc_minors
			// 从设备号已经被分配了，那么它在数组misc_minors中所对应的位== 1，否则 == 0
			// i >> 3， 然后取出从设备号i的低三位 == i & 7
			if ( (misc_minors[i>>3] & (1 << (i & 7))) == 0)
			{
				// 没有被分配
				break;
			}
		}

		if (i < 0) 
		{
			// 注册失败

			mutex_unlock(&misc_mtx);
			return -EBUSY;
		}
		// 将它分配给当前正在注册的日志设备
		misc->minor = i;
	}

	if (misc->minor < DYNAMIC_MINORS)
	{
		// 在数组misc_minors中记录该从设备号已经被分配
		misc_minors[misc->minor >> 3] |= 1 << (misc->minor & 7);
	}

	// 使用 主设备号MISC_MAJOR和 得到的从设备号 , 创建一个设备号对象dev
	dev = MKDEV(MISC_MAJOR, misc->minor);

	// 将日志设备misc注册到系统
	// misc->name 指定日志设备的名称，就是 它在设备系统目录/dev中对应的文件名称
	misc->this_device = device_create(misc_class, misc->parent, dev, NULL,
					  "%s", misc->name);
	
	if (IS_ERR(misc->this_device)) {
		err = PTR_ERR(misc->this_device);
		goto out;
	}

	/*
	 * Add it to the front, so that later devices can "override"
	 * earlier defaults
	 */
	// 将它添加到misc设备列表misc_list中， 表示日志设备注册成功
	list_add(&misc->list, &misc_list);
 out:
	mutex_unlock(&misc_mtx);
	
	return err;
}
```



```c
// kernel\goldfish\drivers\char\misc.c

#define DYNAMIC_MINORS 64 /* like dynamic majors */
// 类型为unsigned char , 每一个元素可以管理8个从设备号，
// 64个从设备号就需要8个元素，所以 数组misc_minors的大小为8
static unsigned char misc_minors[DYNAMIC_MINORS / 8];
```



```c
// kernel\goldfish\include\linux\major.h
// misc设备的主设备号
#define MISC_MAJOR		10
```



`Android`系统使用一种`uevent`机制来管理系统的设备文件。当Logger日志驱动程序注册一个**日志设备**时， 内核就会发出一个`uevent`事件， 这个`uevent`最终由`init`进程中的`handle_device_event`函数来处理  



```c
// system\core\init\devices.c

static void handle_device_event(struct uevent *uevent)
{
	//...
   /* are we block or char? where should we live? */
    if(!strncmp(uevent->subsystem, "block", 5)) 
    {
		//...
    } 
    else 
    {
		//...
			else if (!strncmp(uevent->subsystem, "graphics", 8)) {
            base = "/dev/graphics/";
            mkdir(base, 0755);
        } 
        else if (!strncmp(uevent->subsystem, "oncrpc", 6)) {
			//...
        else if(!strncmp(uevent->subsystem, "misc", 4) &&
                    !strncmp(name, "log_", 4)) 
        {
            //新注册的设备类型为misc，并且设备名称是以“log_”开头的，就会在/dev目录中创建一个log目录
            base = "/dev/log/";
            mkdir(base, 0755);
            // // 将设备名称的前四个字符去掉， 即去掉前面的“log_”子字符串
            name += 4;
        } else
            base = "/dev/";
    }

	//...
}
```



------------------

### 日志设备文件的打开过程  

从`Logger`日志驱动程序读取或者写入日志记录之前， 首先要调用函数`open`来打开对应的日志设备文件  

在Logger日志驱动程序中， 日志设备文件的打开函数定义为`logger_open`  :

```c
// kernel\goldfish\drivers\staging\android\logger.c

static int logger_open(struct inode *inode, struct file *file)
{
	struct logger_log *log;
	int ret;

	// 将相应的设备文件设置为不可随机访问， 就是 不可以用lseek来设置设备文件的当前读写位置， 
	//因为日志记录的读取或者写入都必须按顺序来进行。
	ret = nonseekable_open(inode, file);
	if (ret)
		return ret;

	// 根据从设备号获得要操作的日志缓冲区结构体。
	log = get_log_from_minor(MINOR(inode->i_rdev));
	if (!log)
		return -ENODEV;

	if (file->f_mode & FMODE_READ) 
	{
		// 以读模式打开日志设备的情况

		// 为当前进程创建一个logger_reader结构体reader
		struct logger_reader *reader;

		// 初始化
		reader = kmalloc(sizeof(struct logger_reader), GFP_KERNEL);
		if (!reader)
			return -ENOMEM;

		// 获得的日志缓冲区结构体log保存在结构体reader的成员变量log中
		reader->log = log;
		INIT_LIST_HEAD(&reader->list);

		mutex_lock(&log->mutex);
		
		// 该进程从log-＞head位置开始读取日志记录
		reader->r_off = log->head;
		list_add_tail(&reader->list, &log->readers);

		mutex_unlock(&log->mutex);

		// 把结构体reader保存在参数file的成员变量private_data中
		file->private_data = reader;
	} 
	else
	{
		// 以写模式打开日志设备的情况

		// 把前面获得的日志缓冲区结构体log保存在参数file的成员变量private_data中
		file->private_data = log;
	}

	return 0;
}
```



```c
// kernel\goldfish\drivers\staging\android\logger.c

static struct logger_log * get_log_from_minor(int minor)
{
	if (log_main.misc.minor == minor)
		return &log_main;
	if (log_events.misc.minor == minor)
		return &log_events;
	if (log_radio.misc.minor == minor)
		return &log_radio;
	return NULL;
}
```



-----------------

### 日志记录的读取过程  



当进程调用`read`函数从日志设备读取日志记录时， `Logger`日志驱动程序中的函数 `logger_read`就会被调用从相应的日志缓冲区中**读取日志记录**  



```c
// kernel\goldfish\drivers\staging\android\logger.c
static ssize_t logger_read(struct file *file, 
                  char __user *buf,
                  size_t count, 
                  loff_t *pos)
{
	// 得到日志记录读取进程结构体reader
	struct logger_reader *reader = file->private_data;
	// 得到日志缓冲区结构体 log
	struct logger_log *log = reader->log;
	ssize_t ret;
	DEFINE_WAIT(wait);

start:
	while (1) 
	{
		// 循环 检查日志缓冲区结构体 log 中是否有日志记录可读取

		// 将等待队列项wait加入日志记录读取进程的等待队列log-＞wq中
		prepare_to_wait(&log->wq, &wait, TASK_INTERRUPTIBLE);

		mutex_lock(&log->mutex);

		// w_off用来描述下一条新写入的日志记录在日志缓冲区中的位置
		// r_off用来描述当前进程下一条要读取的日志记录在日志缓冲区中的位置

		// 两者相等时，说明日志缓冲区结构体log中没有新的日志记录可读
		ret = (log->w_off == reader->r_off);

		mutex_unlock(&log->mutex);
		
		// ret == false，日志缓冲区结构体log中有新的日志记录可读
		if (!ret)
		{
			// 跳出循环
			break;
		}
			
		// 当前进程是否 以非阻塞模式来打开日志设备文件
		if (file->f_flags & O_NONBLOCK) 
		{
			// 当前进程就不会因为日志缓冲区结构体log中没有日志记录可读而进入睡眠状态，
			//它直接返回到用户空间中
			ret = -EAGAIN;
			
			break;
		}

		// 检查当前进程是否有信号正在等待处理
		if (signal_pending(current)) 
		{
			// 不能使当前进程进入睡眠状态， 而必须要让它结束当前系统调用， 
			//以便它可以立即返回去处理那些待处理信号
			ret = -EINTR;

			break;
		}

		// 请求内核进行一次进程调度， 从而进入睡眠状态的
		schedule();
	}

	// 将等待队列项wait从日志记录读取进程的等待队列log-＞wq中移除
	finish_wait(&log->wq, &wait);
	if (ret)
		return ret;

	// 获取互斥量log-＞mutex时， 可能会失败。 
	// 在这种情况下， 当前进程就会进入睡眠状态， 直到成功获取互斥量log-＞mutex为止
	// 在当前进程的睡眠过程中， 日志缓冲区结构体log中的日志记录可能已经被其他进程访问过了，
	// 所以 log-＞w_off的值可能已经发生了变化
	mutex_lock(&log->mutex);

	/* is there still something to read or did we race? */
	// 重新判断日志缓冲区结构体 w_off 和日志读取进程结构体 r_off 是否相等
	if (unlikely(log->w_off == reader->r_off)) 
	{
		mutex_unlock(&log->mutex);
		goto start;
	}

	/* get the size of the next entry */
	// 得到下一条要读取的日志记录的长度
	ret = get_entry_len(log, reader->r_off);
	if (count < ret) 
	{
		ret = -EINVAL;
		goto out;
	}

	/* get exactly one entry from the log */
	// 执行真正的日志记录读取操作
	ret = do_read_log_to_user(log, reader, buf, ret);

out:
	mutex_unlock(&log->mutex);

	return ret;
}
```



`get_entry_len` :

日志记录结构体logger_entry :  每一条日志记录都是由两部分内容组成的， 其中一部分内容用来**描述日志记录**， 即结构体logger_entry本身所占据的内容； 另一部分内容是**日志记录的有效负载**， 即真正的日志记录内容。 

由于结构体logger_entry的大小是固定的， 因此只要知道它的**有效负载长度**， 就可以得到一条日志记录的长度。 

日志记录结构体logger_entry的**有效负载长度记录**在它的成员变量len中。 因为 成员变量`len`是日志记录结构体`logger_entry`的第一个成员变量， 因此， 它们的起始地址是相同的。 

`len`的类型为__u16， 只要读取该结构体变量地址的**前两个字节**的内容就是日志记录的长度。

因为 日志缓冲区是**循环使用**的， 所以 一条日志记录的前两个字节的存储 分两种情况来考虑 : 

第一种情况是该日志记录的**前两个字节是连在一起**的； 第二种情况就是前两个字节分别保存在日志缓冲区的**首字节**和**尾字节**中  

```c
// kernel\goldfish\drivers\staging\android\logger.c

static __u32 get_entry_len(struct logger_log *log, size_t off)
{
	__u16 val;

	// 将日志缓冲区结构体log的总长度size - 下一条要读取的日志记录的位置off
	switch (log->size - off) 
	{
		case 1:
			// 该日志记录的长度值分别保存在日志缓冲区的首尾字节中

			// 将它的前1个字节从日志缓冲区的首字节中读取出来，并将它的内容保存在变量vals
			memcpy(&val, log->buffer + off, 1);
			// 将它的后1个字节从日志缓冲区的尾字节中读取出来，并将它的内容保存在变量vals + 1
			memcpy(((char *) &val) + 1, log->buffer, 1);
			
			break;

		default:
			// 该日志记录的长度值保存在两个连在一起的字节中， 即保存在地址log-＞buffer+off中

			// 将日志记录的长度值读取出来， 并且保存在变量val中
			memcpy(&val, log->buffer + off, 2);
	}

	// 将变量val的值 + 结构体logger_entry的大小， 就得到下一条要读取的日志记录的总长度	
	return sizeof(struct logger_entry) + val;
}
```



`do_read_log_to_user` : 



```c
// kernel\goldfish\drivers\staging\android\logger.c
static ssize_t do_read_log_to_user(struct logger_log *log,
				   struct logger_reader *reader,
				   char __user *buf,
				   size_t count)
{
	size_t len;

	/*
	 * We read from the log in two disjoint operations. First, we read from
	 * the current read head offset up to 'count' bytes or to the end of
	 * the log, whichever comes first.
	 */
	// 计算第一次要读取的日志记录的长度
	len = min(count, log->size - reader->r_off);
	// 将这一部分的日志记录复制到用户空间缓冲区buf中
	if (copy_to_user(buf, log->buffer + reader->r_off, len))
		return -EFAULT;

	/*
	 * Second, we read any remaining bytes, starting back at the head of
	 * the log.
	 */
	// 检查上一次是否已经将日志记录的内容读取完毕
	if (count != len)
	{
		// 未读取完毕

		// 继续将第二部分的日志记录复制到用户空间缓冲区buf中，就完成了一条日志记录的读取
		if (copy_to_user(buf + len, log->buffer, count - len))
			return -EFAULT;
	}

	// 当前进程从日志缓冲区结构体log中读取了一条日志记录之后，就要修改它的成员变量r_off的值，
	// 表示下次要读取的是日志缓冲区结构体log中的下一条日志记录
	
	// 将日志读取进程结构体reader的成员变量r_off的值 + 前面已经读取的日志记录的长度count ，
	// 然后再使用宏logger_offset来对计算结果进行调整
	reader->r_off = logger_offset(reader->r_off + count);

	return count;
}
```



由于日志缓冲区是**循环使用**的， 如果计算得到的下一条日志记录的位置**大于**日志缓冲区的长度， 那么就需要将它绕回到日志缓冲区的前面  

```c
// kernel\goldfish\drivers\staging\android\logger.c
#define logger_offset(n)	((n) & (log->size - 1))
```





-----------------------------------

### 日志记录的写入过程  



当进程调用函数write、 writev或者aio_write往日志设备写入日志记录时， Logger日志驱动程序中的函数`logger_aio_write`就会被调用来执行日志记录的写入操作  



对于类型为`main`、 `system`和`radio`的日志记录来说， 它们的内容是由优先级（priority） 、 标签（tag） 和内容（msg） 三个字段组成的， 分别保存在iov[0]、 iov[1]和iov[2]中，  nr_segs == 3。 



对于类型为`events`的日志记录来说， 它们的内容只有**标签**和**内容**两个字段， 分别保存在iov[0]和iov[1]中， nr_segs == 2。 



在特殊情况下， 当类型为events的日志记录的内容只有一个值时， iov的大小 == 3， iov[0] :  日志标签， iov[1] :  日志记录的内容值类型， iov[2] :  日志记录的内容值。 



无论要写入的是什么类型的日志记录， 它的总长度值都是保存在参数iocb的成员变量ki_left中的， 单位是字节。  



```c
/*
 * logger_aio_write - our write method, implementing support for write(),
 * writev(), and aio_write(). Writes are our fast path, and we try to optimize
 * them above all else.
 */
ssize_t logger_aio_write(struct kiocb *iocb, const struct iovec *iov,
			 unsigned long nr_segs, loff_t ppos)
{
	struct logger_log *log = file_get_log(iocb->ki_filp);
	size_t orig = log->w_off;
	struct logger_entry header;
	struct timespec now;
	ssize_t ret = 0;

	now = current_kernel_time();

	header.pid = current->tgid;
	header.tid = current->pid;
	header.sec = now.tv_sec;
	header.nsec = now.tv_nsec;
	header.len = min_t(size_t, iocb->ki_left, LOGGER_ENTRY_MAX_PAYLOAD);

	/* null writes succeed, return zero */
	if (unlikely(!header.len))
		return 0;

	mutex_lock(&log->mutex);

	/*
	 * Fix up any readers, pulling them forward to the first readable
	 * entry after (what will be) the new write offset. We do this now
	 * because if we partially fail, we can end up with clobbered log
	 * entries that encroach on readable buffer.
	 */
	fix_up_readers(log, sizeof(struct logger_entry) + header.len);

	do_write_log(log, &header, sizeof(struct logger_entry));

	while (nr_segs-- > 0) {
		size_t len;
		ssize_t nr;

		/* figure out how much of this vector we can keep */
		len = min_t(size_t, iov->iov_len, header.len - ret);

		/* write out this segment's payload */
		nr = do_write_log_from_user(log, iov->iov_base, len);
		if (unlikely(nr < 0)) {
			log->w_off = orig;
			mutex_unlock(&log->mutex);
			return nr;
		}

		iov++;
		ret += nr;
	}

	mutex_unlock(&log->mutex);

	/* wake up any blocked readers */
	wake_up_interruptible(&log->wq);

	return ret;
}
```





当进程以**写模式**打开日志设备时， `Logger`日志驱动程序会把对应的日志缓冲区结构体log保存在一个打开文件结构体file的成员变量private_data中。 参数iocb的成员变量ki_filp正好指向该打开文件结构体

















-------------------

## 运行时库层日志库  



`Android`系统在运行时库层提供了一个用来和Logger日志驱动程序进行交互的日志库`liblog`。 

通过日志库`liblog`提供的接口， 应用程序就可以方便地往Logger日志驱动程序中写入日志记录。 

位于**运行时库层**的C/C++日志写入接口和位于**应用程序框架层**的Java日志写入接口都是通过`liblog`库提供的日志写入接口来往Logger日志驱动程序中写入日志记录的.

liblog库的日志记录写入接口  :



liblog日志库的函数调用关系  :

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200719141308.png" alt="liblog日志库的函数调用关系  " style="zoom: 50%;" />



根据写入的日志记录的类型不同， 这些函数可以划分为三个类别。 

函数`__android_log_assert`、 `__android_log_vprint`和`__android_log_print`用来写入类型为`main`的日志记录； 

函数`__android_log_btwrite`和`__android_log_bwrite`用来写入类型为`events`的日志记录； 

函数`__android_log_buf_print`可以写入任意一种类型的日志记录。 

**注意**， 在函数`__android_log_write`和`__android_log_buf_write`中， 如果要写入的日志记录的标签以“ RIL ”开头或者等于“ HTC_RIL ”、 “ AT ”、 “ GSM ”、 “ STK ”、 “ CDMA ”、 “ PHONE ”或 “ SMS ”， 那么它们就会被认为是`radio`类型的日志记录。  



无论写入的是什么类型的日志记录， 它们最终都是通过调用函数`write_to_log`写入到Logger日志驱动程序中的。 

当函数`write_to_log`第一次被调用时， 实际上执行的是函数`__write_to_log_init`。 

函数`__write_to_log_init`执行的是一些日志库的初始化操作， 

接着将函数指针`write_to_log`重定向到函数`__write_to_log_kernel`或者`__write_to_log_null`中， 这取决于是否成功地将日志设备文件打开  



描述日志库liblog提供的日志记录写入函数的实现  : 

write_to_log  :

```c
// system\core\liblog\logd_write.c

// 初始化日志库liblog
static int __write_to_log_init(log_id_t, struct iovec *vec, size_t nr);

// write_to_log在开始的时候被设置为函数__write_to_log_init
static int (*write_to_log)(log_id_t, struct iovec *vec, size_t nr) = __write_to_log_init;
```



```c
// system\core\liblog\logd_write.c

static int log_fds[(int)LOG_ID_MAX] = { -1, -1, -1, -1 };

// 初始化日志库liblog
static int __write_to_log_init(log_id_t log_id, struct iovec *vec, size_t nr)
{
	//...
    // write_to_log指向的是自己
    if (write_to_log == __write_to_log_init) {

        // 打开系统中的日志设备文件，并把得到的文件描述符保存在全局数组log_fds中
        // 实际打开/dev/log/main /dev/log/radio  /dev/log/events  /dev/log/system 四个日志设备文件
        log_fds[LOG_ID_MAIN] = log_open("/dev/"LOGGER_LOG_MAIN, O_WRONLY);
        log_fds[LOG_ID_RADIO] = log_open("/dev/"LOGGER_LOG_RADIO, O_WRONLY);
        log_fds[LOG_ID_EVENTS] = log_open("/dev/"LOGGER_LOG_EVENTS, O_WRONLY);
        log_fds[LOG_ID_SYSTEM] = log_open("/dev/"LOGGER_LOG_SYSTEM, O_WRONLY);

        // 将函数指针write_to_log指向函数__write_to_log_kernel
        write_to_log = __write_to_log_kernel;

        // 判断/dev/log/main  /dev/log/radio  /dev/log/events三个日志设备文件是否都打开成功
        if (log_fds[LOG_ID_MAIN] < 0 || log_fds[LOG_ID_RADIO] < 0 ||
                log_fds[LOG_ID_EVENTS] < 0) 
        {
            log_close(log_fds[LOG_ID_MAIN]);
            log_close(log_fds[LOG_ID_RADIO]);
            log_close(log_fds[LOG_ID_EVENTS]);
            log_fds[LOG_ID_MAIN] = -1;
            log_fds[LOG_ID_RADIO] = -1;
            log_fds[LOG_ID_EVENTS] = -1;

            // 将函数指针write_to_log 指向 函数 __write_to_log_null
            write_to_log = __write_to_log_null;
        }

        // 判断日志设备文件/dev/log/system是否打开成功
        if (log_fds[LOG_ID_SYSTEM] < 0) 
        {
            // 不成功

            // 将类型为system和main的日志记录都写入到日志设备文件/dev/log/main中
            log_fds[LOG_ID_SYSTEM] = log_fds[LOG_ID_MAIN];
        }
    }
	//...
    return write_to_log(log_id, vec, nr);
}
```



```c
// system\core\include\cutils\log.h

typedef enum {
    LOG_ID_MAIN = 0,
    LOG_ID_RADIO = 1,
    LOG_ID_EVENTS = 2,
    LOG_ID_SYSTEM = 3,

    LOG_ID_MAX
} log_id_t;
```



```c
// system\core\include\cutils\logger.h

#define LOGGER_LOG_MAIN		"log/main"
#define LOGGER_LOG_RADIO	"log/radio"
#define LOGGER_LOG_EVENTS	"log/events"
#define LOGGER_LOG_SYSTEM	"log/system"
```



```c
// system\core\liblog\logd_write.c

// 正式环境中编译日志库liblog时， 宏FAKE_LOG_DEVICE的值定义为0

#if FAKE_LOG_DEVICE
// This will be defined when building for the host.
#define log_open(pathname, flags) fakeLogOpen(pathname, flags)
#define log_writev(filedes, vector, count) fakeLogWritev(filedes, vector, count)
#define log_close(filedes) fakeLogClose(filedes)
#else
// log_open实际上指向的是打开文件操作函数open
#define log_open(pathname, flags) open(pathname, flags)

// 写文件操作函数writev
#define log_writev(filedes, vector, count) writev(filedes, vector, count)

// 关闭文件操作函数close
#define log_close(filedes) close(filedes)
#endif
```



__write_to_log_ker nel    :

```c
// system\core\liblog\logd_write.c

// 把日志记录写入到Logger日志驱动程序中
static int __write_to_log_kernel(log_id_t log_id, struct iovec *vec, size_t nr)
{
    ssize_t ret;
    int log_fd;

    if (/*(int)log_id >= 0 &&*/ (int)log_id < (int)LOG_ID_MAX) 
    {
        // 根据log_id在全局数组log_fds中找到对应的日志设备文件描述符
        log_fd = log_fds[(int)log_id];
    } 
    else 
    {
        return EBADF;
    }

    do 
    {
        // 把日志记录写入到Logger日志驱动程序中
        ret = log_writev(log_fd, vec, nr);

        // 返回值小于0， 并且错误码等于EINTR， 那就重新执行写入日志记录的操作
        
        // 这种情况一般出现在当前进程等待写入日志记录的过程中，刚好有新的信号需要处理，
        //内核就会返回一个EINTR错误码给调用者， 表示调用者需要再次执行相同的操作
    } while (ret < 0 && errno == EINTR);

    return ret;
}
```



__write_to_log_null  :



```c
// system\core\liblog\logd_write.c

static int __write_to_log_null(log_id_t log_fd, struct iovec *vec, size_t nr)
{
    // 一个空实现 
    //日志设备文件打开失败的情况下， 函数指针write_to_log才会指向该函数
    return -1;
}
```





__android_log_write  :



```c
// system\core\liblog\logd_write.c

int __android_log_write(int prio, const char *tag, const char *msg)
{
    struct iovec vec[3];

    // 在默认情况下，写入的日志记录的类型是 main
    log_id_t log_id = LOG_ID_MAIN;

    if (!tag)
        tag = "";

    /* XXX: This needs to go! */
    if (!strcmp(tag, "HTC_RIL") ||
        !strncmp(tag, "RIL", 3) || /* Any log tag with "RIL" as the prefix */
        !strcmp(tag, "AT") ||
        !strcmp(tag, "GSM") ||
        !strcmp(tag, "STK") ||
        !strcmp(tag, "CDMA") ||
        !strcmp(tag, "PHONE") ||
        !strcmp(tag, "SMS"))
            // 类型为radio的日志记录
            log_id = LOG_ID_RADIO;

    // 日志记录的优先级
    vec[0].iov_base   = (unsigned char *) &prio;
    vec[0].iov_len    = 1;

    // 标签
    vec[1].iov_base   = (void *) tag;
    vec[1].iov_len    = strlen(tag) + 1;

    //内容
    vec[2].iov_base   = (void *) msg;
    vec[2].iov_len    = strlen(msg) + 1;

    // 把紧跟在日志记录标签和内容后面的字符串结束符号‘\0'也写入到Logger日志驱动程序中， 
    //目的是为了以后从Logger日志驱动程序读取日志时，可以将日志记录的标签字段和内容字段解析出来
    
    // 写入到Logger日志驱动程序中
    return write_to_log(log_id, vec, 3);
}
```



`__android_log_buf_write`  :



```c
// system\core\liblog\logd_write.c

int __android_log_buf_write(int bufID, int prio, const char *tag, const char *msg)
{
    // 可以指定写入的日志记录的类型

    struct iovec vec[3];

    if (!tag)
        tag = "";

    /* XXX: This needs to go! */
    if (!strcmp(tag, "HTC_RIL") ||
        !strncmp(tag, "RIL", 3) || /* Any log tag with "RIL" as the prefix */
        !strcmp(tag, "AT") ||
        !strcmp(tag, "GSM") ||
        !strcmp(tag, "STK") ||
        !strcmp(tag, "CDMA") ||
        !strcmp(tag, "PHONE") ||
        !strcmp(tag, "SMS"))

            // radio的日志记录
            bufID = LOG_ID_RADIO;

    vec[0].iov_base   = (unsigned char *) &prio;
    vec[0].iov_len    = 1;
    vec[1].iov_base   = (void *) tag;
    vec[1].iov_len    = strlen(tag) + 1;
    vec[2].iov_base   = (void *) msg;
    vec[2].iov_len    = strlen(msg) + 1;

    // 把紧跟在日志记录标签和内容后面的字符串结束符号‘\0'也写入到Logger日志驱动程序中， 
    //目的是为了以后从Logger日志驱动程序读取日志时，可以将日志记录的标签字段和内容字段解析出来

    return write_to_log(bufID, vec, 3);
}
```





`__android_log_vprint` 、 `__android_log_print` 、 `__android_log_assert`  :



```c
// system\core\liblog\logd_write.c

int __android_log_vprint(int prio, const char *tag, const char *fmt, va_list ap)
{
    char buf[LOG_BUF_SIZE];

    // 使用格式化字符串来描述要写入的日志记录内容
    vsnprintf(buf, LOG_BUF_SIZE, fmt, ap);

    // 向Logger日志驱动程序中写入日志记录
    return __android_log_write(prio, tag, buf);
}

int __android_log_print(int prio, const char *tag, const char *fmt, ...)
{
    va_list ap;

    // // 使用格式化字符串来描述要写入的日志记录内容
    char buf[LOG_BUF_SIZE];

    va_start(ap, fmt);
    vsnprintf(buf, LOG_BUF_SIZE, fmt, ap);
    va_end(ap);

    // 向Logger日志驱动程序中写入日志记录
    return __android_log_write(prio, tag, buf);
}

void __android_log_assert(const char *cond, const char *tag,
			  const char *fmt, ...)
{
    // 使用格式化字符串来描述要写入的日志记录内容
    char buf[LOG_BUF_SIZE];

    if (fmt) {
        va_list ap;
        va_start(ap, fmt);
        vsnprintf(buf, LOG_BUF_SIZE, fmt, ap);
        va_end(ap);
    } else {
        /* Msg not provided, log condition.  N.B. Do not use cond directly as
         * format string as it could contain spurious '%' syntax (e.g.
         * "%d" in "blocks%devs == 0").
         */
        if (cond)
            snprintf(buf, LOG_BUF_SIZE, "Assertion failed: %s", cond);
        else
            strcpy(buf, "Unspecified assertion failed");
    }

    // 向Logger日志驱动程序中写入日志记录
    __android_log_write(ANDROID_LOG_FATAL, tag, buf);

    __builtin_trap(); /* trap so we have a chance to debug the situation */
}
```





__android_log_buf_print  :



```c
// system\core\liblog\logd_write.c

int __android_log_buf_print(int bufID, int prio, const char *tag, const char *fmt, ...)
{
    va_list ap;
    char buf[LOG_BUF_SIZE];

    va_start(ap, fmt);

    // 格式化字符串来描述要写入的日志记录内容
    vsnprintf(buf, LOG_BUF_SIZE, fmt, ap);
    
    va_end(ap);

    // 向Logger日志驱动程序中写入日志记录 , 指定要写入的日志记录的类型
    return __android_log_buf_write(bufID, prio, tag, buf);
}
```





`__android_log_bwrite` 、 `__android_log_btwrite`  :



```c
// system\core\liblog\logd_write.c

int __android_log_bwrite(int32_t tag, const void *payload, size_t len)
{
    struct iovec vec[2];
    
    // 名称
    vec[0].iov_base = &tag;
    vec[0].iov_len = sizeof(tag);
    
    // 单位
    vec[1].iov_base = (void*)payload;
    vec[1].iov_len = len;
    
    // 写入的日志记录的类型为events
    return write_to_log(LOG_ID_EVENTS, vec, 2);
}

int __android_log_btwrite(int32_t tag, char type, 
                  const void *payload, size_t len)
{
    struct iovec vec[3];

    // 名称
    vec[0].iov_base = &tag;
    vec[0].iov_len = sizeof(tag);

    // 定要写入的日志记录内容的值类型
    vec[1].iov_base = &type;
    vec[1].iov_len = sizeof(type);

    // 单位
    vec[2].iov_base = (void*)payload;
    vec[2].iov_len = len;

    // 写入的日志记录的类型为events
    return write_to_log(LOG_ID_EVENTS, vec, 3);
}
```





-----------------------

## C/C++日志写入接口  



`Android`系统就提供了三组常用的C/C++宏来封装日志写入接口， 这些宏有的在程序的**非调试**版本中只是一个空定义， 因此， 可以避免在程序的发布版本中输出日志。

LOGV、 LOGD、 LOGI、 LOGW和LOGE， 它们用来写入类型为`main`的日志记录； 

SLOGV、 SLOGD、 SLOGI、 SLOGW和SLOGE， 它们用来写入类型为`system`的日志记录； 

LOG_EVENT_INT、 LOG_EVENT_LONG和LOG_EVENT_STRING， 它们用来写入类型为`events`的日志记录  



目录 : 

```c
~/Android/system/core/include
    cutils
    	log.h
```



区分程序是调试版本还是发布版本  : 

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



使用宏LOG来实现日志写入功能  :



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



android_LogPriority的枚举值  : 

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





SLOGV 、 SLOGD 、 SLOGI 、 SLOGW 和SLOGE  :

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



LOG_EVENT_INT 、 LOG_EVENT_LONG 和LOG_EVENT_STRING  :



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





---------------

## Java日志写入接口

Android系统在应用程序框架层中定义了三个Java日志写入接口， 

android.util.Log`、 `android.util.Slog`和`android.util.EventLog， 写入的日志记录类型分别为main、 system和events。 

这三个Java日志写入接口是通过`JNI`方法来调用日志库liblog提供的函数来实现日志记录的写入功能的。



android.util.Log  :



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





```c++
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



android.util.Slog  :



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



android.util.EventLog  :



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



写入**整数**和**长整数**类型日志记录的`JNI`方法`writeEvent`的实现  : 

```c++
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



整数的日志记录的内存布局  : 

![image-20200720151030266](https://gitee.com/cpu_code/picture_bed/raw/master//20200720151030.png)



长整数的日志记录的内存布局  : 

![image-20200720151039785](https://gitee.com/cpu_code/picture_bed/raw/master//20200720151039.png)



宏android_btWriteLog的定义  :

```c++
// system\core\include\cutils\log.h

// 往Logger日志驱动程序中写入日志记录
#define android_btWriteLog(tag, type, payload, len) \
    __android_log_btwrite(tag, type, payload, len)
```



写入字符串类型日志记录的JNI方法writeEvent的实现  :



```c++
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



内容为字符串的日志记录的内存布局  :

![image-20200720151051111](https://gitee.com/cpu_code/picture_bed/raw/master//20200720151051.png)



第一个字段记录日志记录内容的类型为字符串， 

第二个字段描述该字符串的长度， 

第三个字段保存的是字符串内容， 

第四个字段使用特殊字符‘\n'来结束该日志记录  



写入列表类型日志记录的JNI方法`writeEvent`的实现  :



```c++
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



内容为列表的日志记录的内存布局  :

![image-20200720151100719](https://gitee.com/cpu_code/picture_bed/raw/master//20200720151100.png)





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















