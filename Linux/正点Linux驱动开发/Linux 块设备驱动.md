



# Linux 块设备驱动  





## 块设备

**块设备**是针对存储设备的，比如 SD 卡、 EMMC、 NAND Flash、 Nor Flash、 SPI Flash、机械硬盘、固态硬盘等  

所以 **块设备驱动**其实就是这些存储设备驱动



与字符设备相比 :

块设备只能以**块为单位**进行读写访问，块是 linux 虚拟文件系统**(VFS**)基本的数据传输单位。

字符设备是以**字节为单位**进行数据传输的，不需要缓冲。  



块设备在结构上是可以进行**随机访问**的，对于这些设备的读写都是**按块**进行的，块设备一般都是使用**缓冲区**来暂时存放数据，等到条件成熟后 , 在一次性将缓冲区中的数据写入块设备中。这么做的目的为了减少了对块设备的擦除次数 , 提高块设备寿命

字符设备是**顺序的数据流**设备，字符设备是按照**字节**进行读写访问的。字符设备不需要缓冲区，对于字符设备的访问都是实时的，而且也不需要按照固定的块大小进行访问。  



------------------------------

## 块设备驱动框架 

block_device 表示 块设备  

```c
// include/linux/fs.h

struct block_device 
{
	dev_t			bd_dev;  /* not a kdev_t - it's a search key */
	int			bd_openers;
	struct inode *		bd_inode;	/* will die */
	struct super_block *	bd_super;
	struct mutex		bd_mutex;	/* open/close mutex */
	void *			bd_claiming;
	void *			bd_holder;
	int			bd_holders;
	bool			bd_write_holder;
#ifdef CONFIG_SYSFS
	struct list_head	bd_holder_disks;
#endif
	struct block_device *	bd_contains;
	unsigned		bd_block_size;
	u8			bd_partno;
	struct hd_struct *	bd_part;
	/* number of times partitions within this device have been opened. */
	unsigned		bd_part_count;
	int			bd_invalidated;
	struct gendisk *	bd_disk;	/* gendisk 结构体指针类型 */
	struct request_queue *  bd_queue;
	struct backing_dev_info *bd_bdi;
	struct list_head	bd_list;
	/*
	 * Private data.  You must have bd_claim'ed the block_device
	 * to use this.  NOTE:  bd_claim allows an owner to claim
	 * the same device multiple times, the owner must take special
	 * care to not mess up bd_private for that case.
	 */
	unsigned long		bd_private;

	/* The counter of freeze processes */
	int			bd_fsfreeze_count;
	/* Mutex for freeze */
	struct mutex		bd_fsfreeze_mutex;
} __randomize_layout;

```



### 注册块设备

```c
// block/genhd.c

/**
 * @function: 注册块设备
 * @parameter: 
 * 		major： 主设备号
 * 		name： 块设备名字
 * @return: 
 *     success: 1~255 之间的话表示自定义主设备号, 返回 0 
 * 			为 0 的话表示由系统自动分配主设备号, 返回值主设备号
 *     error: 负值
 * @note: 
 */
int register_blkdev(unsigned int major, const char *name)
```



### 注销块设备  

```c
// block/genhd.c

/**
 * @function: 注销块设备
 * @parameter: 
 * 		major： 要注销的块设备主设备号
 * 		name： 要注销的块设备名字
 * @return: 
 *     success: 
 *     error: 
 * @note: 
 */
void unregister_blkdev(unsigned int major, const char *name)
```



## gendisk 结构体

gendisk 描述一个磁盘设备

```c
// include/linux/genhd.h
//描述一个磁盘设备
struct gendisk 
{
	/* major, first_minor and minors are input parameters only,
	 * don't use directly.  Use disk_devt() and disk_max_parts().
	 */
	int major;			/* 磁盘设备的主设备号 */ /* major number of driver */
	int first_minor;	/* 磁盘的第一个次设备号 */
	int minors;   	 	/* 磁盘的分区数量 */  /* maximum number of minors, =1 for
                                         * disks that can't be partitioned. */

	char disk_name[DISK_NAME_LEN];	/* name of major driver */
	char *(*devnode)(struct gendisk *gd, umode_t *mode);

	unsigned short events;		/* supported events */
	unsigned short event_flags;	/* flags related to event processing */

	/* Array of pointers to partitions indexed by partno.
	 * Protected with matching bdev lock but stat and other
	 * non-critical accesses use RCU.  Always access through
	 * helpers.
	 */
	struct disk_part_tbl __rcu *part_tbl;	/* 磁盘对应的分区表 */
	struct hd_struct part0;

	const struct block_device_operations *fops;	/* 块设备操作集 */
	struct request_queue *queue;	/* 请求队列 */
	void *private_data;

	int flags;
	struct rw_semaphore lookup_sem;
	struct kobject *slave_dir;

	struct timer_rand_state *random;
	atomic_t sync_io;		/* RAID */
	struct disk_events *ev;
#ifdef  CONFIG_BLK_DEV_INTEGRITY
	struct kobject integrity_kobj;
#endif	/* CONFIG_BLK_DEV_INTEGRITY */
	int node_id;
	struct badblocks *bb;
	struct lockdep_map lockdep_map;
};
```



### 申请 gendisk  

```c
// include/linux/genhd.h

/**
 * @function: 申请 gendisk
 * @parameter: 
 * 		minors： 次设备号数量
 * @return: 
 *     success: 申请到的 gendisk
 *     error: NULL
 * @note: 
 */
#define alloc_disk(minors) alloc_disk_node(minors, NUMA_NO_NODE)
```



### 删除 gendisk  

```c
// block/genhd.c
/**
 * @function: 删除 gendisk
 * @parameter: 
 * 		disk : 要删除的 gendisk
 * @return: 
 *     success: 
 *     error: 
 * @note: 
 */
void del_gendisk(struct gendisk *disk)
```



### 将 gendisk 添加到内核  

```c
// include/linux/genhd.h

/**
 * @function: 将 gendisk 添加到内核
 * @parameter: 
 * 		disk： 要添加到内核的 gendisk
 * @return: 
 *     success: 
 *     error: 
 * @note: 
 */
static inline void add_disk(struct gendisk *disk)
```



### 设置 gendisk 容量  

```c
// include/linux/genhd.h

/**
 * @function: 设置 gendisk 容量
 * @parameter: 
 * 		disk： 要设置容量的 gendisk
 * 		size： 磁盘容量大小
 * @return: 
 *     success: 
 *     error: 
 * @note: 
 *		2MB 的磁盘，扇区数量 = (2*1024*1024)/512 = 4096
 */
static inline void set_capacity(struct gendisk *disk, sector_t size)
```



### 调整 gendisk 引用计数  

```c
//block/genhd.c

// 增加 gendisk 的引用计数 
struct kobject *get_disk(struct gendisk *disk)

/**
 * @function: 减少 gendisk 的引用计数
 * @parameter: 
 * @return: 
 *     success: 
 *     error: 
 * @note: 
 */
void put_disk(struct gendisk *disk)
```



## block_device_operations 结构体  



```c
// include/linux/blkdev.h

//块设备的操作集
struct block_device_operations 
{
	int (*open) (struct block_device *, fmode_t);	/* 打开指定的块设备 */
	void (*release) (struct gendisk *, fmode_t);	/* 关闭(释放)指定的块设备 */
	int (*rw_page)(struct block_device *, sector_t, struct page *, unsigned int);	/* 读写指定的页 */
	int (*ioctl) (struct block_device *, fmode_t, unsigned, unsigned long);		/* 块设备的 I/O 控制 */
	int (*compat_ioctl) (struct block_device *, fmode_t, unsigned, unsigned long);	/* 块设备的 I/O 控制 */
	unsigned int (*check_events) (struct gendisk *disk, 
				      unsigned int clearing);
	/* ->media_changed() is DEPRECATED, use ->check_events() instead */
	int (*media_changed) (struct gendisk *);
	void (*unlock_native_capacity) (struct gendisk *);
	int (*revalidate_disk) (struct gendisk *);
	int (*getgeo)(struct block_device *, struct hd_geometry *);	/* 获取磁盘信息 */
	/* this callback is with swap_lock and sometimes page table lock held */
	void (*swap_slot_free_notify) (struct block_device *, unsigned long);
	int (*report_zones)(struct gendisk *, sector_t sector,
			unsigned int nr_zones, report_zones_cb cb, void *data);
	struct module *owner;	/* 此结构体属于哪个模块 */
	const struct pr_ops *pr_ops;
};
```



## 块设备 I/O 请求过程  



### 请求队列 request_queue  

```c
//include/linux/blkdev.h

struct request_queue 
{
	//...
	/* sw queues */
	struct blk_mq_ctx __percpu	*queue_ctx;

	unsigned int		queue_depth;

	/* hw dispatch queues */
	struct blk_mq_hw_ctx	**queue_hw_ctx;
	unsigned int		nr_hw_queues;

	struct backing_dev_info	*backing_dev_info;

	/*
	 * The queue owner gets to use this for whatever they like.
	 * ll_rw_blk doesn't touch it.
	 */
	void			*queuedata;

	/*
	 * various queue flags, see QUEUE_* below
	 */
	unsigned long		queue_flags;
	/*
	 * Number of contexts that have called blk_set_pm_only(). If this
	 * counter is above zero then only RQF_PM and RQF_PREEMPT requests are
	 * processed.
	 */
	atomic_t		pm_only;

	/*
	 * ida allocated id for this queue.  Used to index queues from
	 * ioctx.
	 */
	int			id;

	/*
	 * queue needs bounce pages for pages above this limit
	 */
	gfp_t			bounce_gfp;

	spinlock_t		queue_lock;

	/*
	 * queue kobject
	 */
	struct kobject kobj;

	/*
	 * mq queue kobject
	 */
	struct kobject *mq_kobj;

#ifdef  CONFIG_BLK_DEV_INTEGRITY
	struct blk_integrity integrity;
#endif	/* CONFIG_BLK_DEV_INTEGRITY */

#ifdef CONFIG_PM
	struct device		*dev;
	int			rpm_status;
	unsigned int		nr_pending;
#endif

	/*
	 * queue settings
	 */
	unsigned long		nr_requests;	/* Max # of requests */


    

#ifdef CONFIG_BLK_DEV_ZONED
	/*
	 * Zoned block device information for request dispatch control.
	 * nr_zones is the total number of zones of the device. This is always
	 * 0 for regular block devices. conv_zones_bitmap is a bitmap of nr_zones
	 * bits which indicates if a zone is conventional (bit set) or
	 * sequential (bit clear). seq_zones_wlock is a bitmap of nr_zones
	 * bits which indicates if a zone is write locked, that is, if a write
	 * request targeting the zone was dispatched. All three fields are
	 * initialized by the low level device driver (e.g. scsi/sd.c).
	 * Stacking drivers (device mappers) may or may not initialize
	 * these fields.
	 *
	 * Reads of this information must be protected with blk_queue_enter() /
	 * blk_queue_exit(). Modifying this information is only allowed while
	 * no requests are being processed. See also blk_mq_freeze_queue() and
	 * blk_mq_unfreeze_queue().
	 */
	unsigned int		nr_zones;
	unsigned long		*conv_zones_bitmap;
	unsigned long		*seq_zones_wlock;
#endif /* CONFIG_BLK_DEV_ZONED */

	/*
	 * sg stuff
	 */
	unsigned int		sg_timeout;
	unsigned int		sg_reserved_size;
	int			node;
#ifdef CONFIG_BLK_DEV_IO_TRACE
	struct blk_trace	*blk_trace;
	struct mutex		blk_trace_mutex;
#endif
	/*
	 * for flush operations
	 */
	struct blk_flush_queue	*fq;

	struct list_head	requeue_list;
	spinlock_t		requeue_lock;
	struct delayed_work	requeue_work;

	struct mutex		sysfs_lock;
	struct mutex		sysfs_dir_lock;

	/*
	 * for reusing dead hctx instance in case of updating
	 * nr_hw_queues
	 */
	struct list_head	unused_hctx_list;
	spinlock_t		unused_hctx_lock;

	int			mq_freeze_depth;

#if defined(CONFIG_BLK_DEV_BSG)
	struct bsg_class_device bsg_dev;
#endif

#ifdef CONFIG_BLK_DEV_THROTTLING
	/* Throttle data */
	struct throtl_data *td;
#endif
	struct rcu_head		rcu_head;
	wait_queue_head_t	mq_freeze_wq;
	/*
	 * Protect concurrent access to q_usage_counter by
	 * percpu_ref_kill() and percpu_ref_reinit().
	 */
	struct mutex		mq_freeze_lock;
	struct percpu_ref	q_usage_counter;

	struct blk_mq_tag_set	*tag_set;
	struct list_head	tag_set_list;
	struct bio_set		bio_split;

#ifdef CONFIG_BLK_DEBUG_FS
	struct dentry		*debugfs_dir;
	struct dentry		*sched_debugfs_dir;
	struct dentry		*rqos_debugfs_dir;
#endif

	bool			mq_sysfs_init_done;

	size_t			cmd_size;

	struct work_struct	release_work;

#define BLK_MAX_WRITE_HINTS	5
	u64			write_hints[BLK_MAX_WRITE_HINTS];
};
```



#### 初始化请求队列

```c
/**
 * @function: 初始化请求队列
 * @parameter: 
 * 		rfn： 请求处理函数指针
 * 		lock： 自旋锁指针
 * @return: 
 *     success: 申请到的 request_queue 地址
 *     error: NULL
 * @note: 给请求队列分配一个 I/O 调度器，用于机械存储设备，比如机械硬盘等
 */
request_queue *blk_init_queue(request_fn_proc *rfn, spinlock_t *lock)
```



#### 删除请求队列

```c
// block\blk-core.c
/**
 * @function: 删除请求队列
 * @parameter: 
 * 		q： 需要删除的请求队列
 * @return: 
 *     success: 
 *     error: 
 * @note: 
 */
void blk_cleanup_queue(struct request_queue *q)
```



#### 分配请求队列并绑定制造请求函数

```c
// block\blk-core.c
/**
 * @function:  request_queue 申请
 * @parameter: 
 * 		gfp_mask： 内存分配掩码	, 参考 include/linux/gfp.h 中的相关宏定义
 * @return: 
 *     success: 无 I/O 调度的 request_queue
 *     error: 
 * @note: 用于那么非机械的存储设备、无需 I/O 调度器，比如 EMMC、 SD 卡等
 */
struct request_queue *blk_alloc_queue(gfp_t gfp_mask)
```

```c
// block\blk-settings.c

/**
 * @function: 请求队列绑定一个“制造请求”
 * @parameter: 
 * 		q： 需要绑定的请求队列
 * 		mfn：需要绑定的“制造”请求
 * @return: 
 *     success: 
 *     error: 
 * @note: 用于那么非机械的存储设备、无需 I/O 调度器，比如 EMMC、 SD 卡等
 */
void blk_queue_make_request(struct request_queue *q, make_request_fn *mfn)
```



### 请求 request  

```c
// include/linux/blkdev.h
struct request 
{
	//...
	unsigned int cmd_flags;		/* op and common flags */
	req_flags_t rq_flags;

	int tag;
	int internal_tag;

	/* the following two fields are internal, NEVER access directly */
	unsigned int __data_len;	/* total data len */
	sector_t __sector;		/* sector cursor */

	struct bio *bio;
	struct bio *biotail;

	struct list_head queuelist;

	/*
	 * The hash is used inside the scheduler, and killed once the
	 * request reaches the dispatch list. The ipi_list is only used
	 * to queue the request for softirq completion, which is long
	 * after the request has been unhashed (and even removed from
	 * the dispatch list).
	 */
	union {
		struct hlist_node hash;	/* merge hash */
		struct list_head ipi_list;
	};

	/*
	 * The rb_node is only used inside the io scheduler, requests
	 * are pruned when moved to the dispatch queue. So let the
	 * completion_data share space with the rb_node.
	 */
	union {
		struct rb_node rb_node;	/* sort/lookup */
		struct bio_vec special_vec;
		void *completion_data;
		int error_count; /* for legacy drivers, don't use */
	};

	/*
	 * Three pointers are available for the IO schedulers, if they need
	 * more they have to dynamically allocate it.  Flush requests are
	 * never put on the IO scheduler. So let the flush fields share
	 * space with the elevator data.
	 */
	union {
		struct {
			struct io_cq		*icq;
			void			*priv[2];
		} elv;

		struct {
			unsigned int		seq;
			struct list_head	list;
			rq_end_io_fn		*saved_end_io;
		} flush;
	};

	struct gendisk *rq_disk;
	struct hd_struct *part;
#ifdef CONFIG_BLK_RQ_ALLOC_TIME
	/* Time that the first bio started allocating this request. */
	u64 alloc_time_ns;
#endif
	/* Time that this request was allocated for this IO. */
	u64 start_time_ns;
	/* Time that I/O was submitted to the device. */
	u64 io_start_time_ns;

	/*
	 * rq sectors used for blk stats. It has the same value
	 * with blk_rq_sectors(rq), except that it never be zeroed
	 * by completion.
	 */
	unsigned short stats_sectors;

	/*
	 * Number of scatter-gather DMA addr+len pairs after
	 * physical address coalescing is performed.
	 */
	unsigned short nr_phys_segments;
	//...

	unsigned int extra_len;	/* length of alignment and padding */

	//...

	/*
	 * completion callback.
	 */
	rq_end_io_fn *end_io;
	void *end_io_data;
};
```



#### 获取请求

```c
/**
 * @function: 从request_queue中依次获取每个request
 * @parameter: 
 * 		q： 指定 request_queue
 * @return: 
 *     success: request_queue 中下一个要处理的请求(request)
 *     error: NULL
 * @note: 
 */
request *blk_peek_request(struct request_queue *q)
```



#### 开启请求

```c
/**
 * @function: 获取到下一个要处理的请求以后就要开始处理这个请求
 * @parameter: 
 * 		req： 要开始处理的请求
 * @return: 
 *     success: 
 *     error:
 * @note: 
 */
void blk_start_request(struct request *req)
```



#### 一步到位处理请求

```c
//一次性完成请求的获取和开启
struct request *blk_fetch_request(struct request_queue *q)
{
	struct request *rq;

	rq = blk_peek_request(q);
    
	if (rq)
    {
        blk_start_request(rq);
    }
	
	return rq;
}
```



#### 其他和请求有关的函数  

```c
//请求中指定字节数据被处理完成
blk_end_request();
```

```c
//请求中所有数据全部处理完成
blk_end_request_all();
```

```c
//当前请求中的 chunk
blk_end_request_cur();
```

```c
//处理完请求，直到下一个错误产生
blk_end_request_err();
```

```c
//和 blk_end_request 函数一样，但是需要持有队列锁
__blk_end_request();
```

```c
//和 blk_end_request_all 函数一样，但是需要持有队列锁
__blk_end_request_all();
```

```c
//和 blk_end_request_cur 函数一样，但是需要持有队列锁
__blk_end_request_cur()
```

```c
// 和 blk_end_request_err 函数一样，但是需要持有队列锁
__blk_end_request_err()
```



### bio 结构

```c
// include/linux/blk_types.h


```







request_queue、 request 和 bio 之间的关系  : 

![image-20200711124502371](C:/Users/cpucode/AppData/Roaming/Typora/typora-user-images/image-20200711124502371.png)

