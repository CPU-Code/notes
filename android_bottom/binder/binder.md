# binder

Android应用程序是由`Activity`、 `Service`、 `Broadcast Receiver`和 `Content Provider`四种类型的组件构成的， 它们有可能运行在同一个进程中， 也有可能运行在不同的进程中。 此外， 各种系统组件也运行在独立的进程中， 例如， Activity管理服务`ActivityManagerService`和Package管理服务`PackageManagerService`都运行在**系统进程**`System`中。

![image-20200725142202987](https://gitee.com/cpu_code/picture_bed/raw/master//20200725142203.png)

Android系统是基于`Linux`内核开发的。 Linux内核提供了丰富的进程间通信机制， 如管道（Pipe） 、 信号（Signal） 、 消息队列（Message） 、 共享内存（Share Memory） 和 插口（Socket） 等。

![image-20200725142429073](https://gitee.com/cpu_code/picture_bed/raw/master//20200725142429.png)

而 Android系统开发了一套新的**进程间通信机制**——Binder。

`Binder`进程间通信机制在**进程间传输数据**时， 只需要执行一次**复制操作**， 所以，提高了效率，节省了内存空间。`Binder`进程间通信机制是在`OpenBinder`的基础上实现的， 它采用`CS`通信方式， 其中， **提供服务**的进程称为`Server`进程， 而**访问服务**的进程称为`Client`进程。

同一个Server进程可以同时运行多个组件来向`Client`进程提供服务， 这些组件称为Service组件。同一个Client进程也可以同时向多个`Service`组件请求服务， 每一个请求都对应有一个Client组件\( `Service`代理对象\) 。

Binder进程间通信机制的每一个Server进程和Client进程都维护一个Binder线程池来处理进程间的通信请求， 因此， Server进程和Client进程可以**并发**地提供和访问服务。 Server进程和Client进程的通信要依靠运行在**内核空间**的`Binder`驱动程序来进行。

Binder驱动程序向**用户空间**暴露了一个设备文件`/dev/binder`， 使得应用程序进程可以间接地通过它来建立通信通道。

`Service`组件在启动时， 会将自己注册到一个`Service Manager`组件中， 以便`Client`组件可以通过`Service Manager`组件找到它。

因此， 我们将`Service Manager`组件称为`Binder`进程间通信机制的**上下文管理者**， 同时由于它也需要与普通的`Server`进程和`Client`进程通信， 我们也将它看作是一个`Service`组件， 只不过它是一个特殊的`Service`组件。

Binder进程间通信机制 :

![Binder&#x8FDB;&#x7A0B;&#x95F4;&#x901A;&#x4FE1;&#x673A;&#x5236; ](https://gitee.com/cpu_code/picture_bed/raw/master//20200725130011.png)

`Client`、 `Service` 和 `Service Manager` 运行在**用户空间**， 而`Binder`驱动程序运行在**内核空间**， 其中， `Service Manager` 和 `Binder`驱动程序由系统负责提供， 而 `Client` 和 `Service` 组件由应用程序来实现。 `Client`、 `Service`和`Service Manager`均是通过系统调用 `open`、 `mmap` 和 `ioctl` 来访问设备文件 `/dev/binder`， 从而实现与 `Binder` 驱动程序的交互， 而交互的目的就是为了能够间接地执行进程间通信。

分析 Binder 进程间通信机制的四个使用情景， 分别是：

（1） Service Manager的启动过程。

（2） Service Manager代理对象的获取过程。

（3） Service组件的启动过程。

（4） Service代理对象的获取过程。

上述四个使用情景都是基于Binder进程间通信机制的C/C++实现来分析

## Binder驱动程序

## Binder进程间通信库

