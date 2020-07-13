<!--
 * @Author: cpu_code
 * @Date: 2020-07-12 14:09:20
 * @LastEditTime: 2020-07-13 11:48:29
 * @FilePath: \note\android_bottom\smart_pointer.md
 * @Gitee: https://gitee.com/cpu_code
 * @Github: https://github.com/CPU-Code
 * @CSDN: https://blog.csdn.net/qq_44226094
 * @Gitbook: https://923992029.gitbook.io/cpucode/
--> 


# 智能指针

 * @Author: cpu_code
 * @Date: 2020-07-12 14:09:20
 * @LastEditTime: 2020-07-13 11:48:21
 * @FilePath: \note\android_bottom\smart_pointer.md
 * @Gitee: https://gitee.com/cpu_code
 * @Github: https://github.com/CPU-Code
 * @CSDN: https://blog.csdn.net/qq_44226094
 * @Gitbook: https://923992029.gitbook.io/cpucode/

在`Android`系统的应用程序框架层中， 有一部分代码是使用C++语言开发  .

用`C++`最容易出错的地方就是指针， 一般为忘记释放指针指向的对象所占用的内存， 或者使用了无效指针。   

`C++`指针使用不当， 轻则造成内存泄漏， 重则造成莫名其妙的逻辑错误， 甚至系统崩溃。  

因此， Android系统为我们提供了C++智能指针， 避免出现指针使用不当的问题  

----------

为了解决以上问题 , 通常通过**引用计数技术**来维护对象的生命周期 

每当有一个新的指针**指向**了一个对象时， 这个对象的**引用计数**就**增加1**  ; 每当有一个指针**不再**指向一个对象时， 这个对象的引用计数就**减少1**  ; 当对象的**引用计数为0**时， 它所占用的内存就可以安全地**释放**了  .

智能指针正是一种能够**自动维护对象引用计数**的技术。 这里需要特别强调的是， 智能指针是**一个对象**， 而**不是一个指针**， 但是它引用了一个实际使用的对象。   

在智能指针**构造**时， **增加**它所引用的对象的引用计数； 而在智能指针**析构**时， 就**减少**它所引用的对象的引用计数  .

当 有两个对象A和B， 对象A**引用**了对象B， 而对象B也**引用了**对象A。 一方面， 当对象A不再使用时， 就可以释放它所占用的内存了， 但是由于对象B仍然在引用着它， 因此， 此时对象A就不能被释放； 另一方面， 当对象B不再使用时， 就可以释放它所占用的内存了， 但是由于对象A仍然在引用着它， 因此， 此时对象B也不能被释放  

这个问题也是 **垃圾收集**(Garbage Collection)系统所遇到的经典问题之一， 因为它一次只能收集一个对象所占用的内存。  

--------------------

为了解决以上问题 , 采取另外一种**稍为复杂的引用计数技术**来维护对象的生命周期了。 这种引用计数技术将对象的引用计数分为**强引用计数**和**弱引用计数**两种， 其中， **对象的生命周期只受强引用计数控制**。  

在“父-子”关系中，  “父”对象通过**强引用计数**来引用“子”对象；  “子”对象通过**弱引用计数**来引用“父”对象。   

![image-20200712144527052](https://gitee.com/cpu_code/picture_bed/raw/master//20200712144527.png)

假设对象A为 父 , 对象B为子。 对象A通过**强引用计数**来引用对象B， 而对象B通过**弱智能指针引用**计数来引用对象A。 当对象A不再使用时， 对象A的生命周期**不受对象B**的影响， 此时对象A可以安全地**释放**。 在释放对象A时， 同时也会**释放**它对对象B的**强引用计数**， 因此， 当对象B不再使用时， 对象B也可以安全地释放了。  

Android系统提供了三种类型的C++智能指针， 分别为**轻量级指针**(Light Pointer)、 **强指针**(StrongPointer)和**弱指针**(Weak Pointer)， 其中， 轻量级指针使用了简单的引用计数技术， 而强指针和弱指针使用了强引用计数和弱引用计数技术  

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200712145507.png" alt="image-20200712145507418" style="zoom: 67%;" />



Android系统将**引用计数器**定义为一个**公共类**， 所有支持使用智能指针的对象类都必须要从这个**公共类继承**下来。 这样， Android系统的智能指针就可以通过这个引用计数器来**维护对象的生命周期**了  

-----------------------

## 轻量级指针

**轻量级指针**通过**简单的引用计数**技术来维护对象的生命周期。如果一个类的对象支持使用**轻量级指针**，那么它就必须要从`LightRefBase`类继承下来，因为`LightRefBase`类提供了一个简单的引用计数器。

### 实现原理分析

分析`LightRefBase`类的实现原理

```c++
// frameworks\base\include\utils\RefBase.h

// 模板类
// T 表示对象的实际类型，它必须是继承了LightRefBase类的
template <class T>
class LightRefBase
{
public:
    inline LightRefBase() : mCount(0) { }
    
    // 增加它所引用的对象的引用计数
    inline void incStrong(const void* id) const 
    {
        android_atomic_inc(&mCount);
    }
    
    //减少它所引用的对象的引用计数
    inline void decStrong(const void* id) const 
    {
        if (android_atomic_dec(&mCount) == 1) 
        {
            //释放这个对象所占用的内存
            delete static_cast<const T*>(this);
        }
    }
    //! DEBUGGING ONLY: Get current strong ref count.
    inline int32_t getStrongCount() const 
    {
        return mCount;
    }
    
protected:
    inline ~LightRefBase() { }
    
private:
    mutable volatile int32_t mCount;	// 描述一个对象的引用计数值
};
```



**轻量级指针**的实现类为`sp`，它同时也是强指针的实现类 : 

```c++
// frameworks\base\include\utils\RefBase.h

// 模块类，其中，模板参数T表示对象的实际类型，它也是必须继承了LightRefBase类的
template <typename T>
class sp
{
public:
    // 维护它所引用的对象的强引用计数和弱引用计数
    typedef typename RefBase::weakref_type weakref_type;
    
    // m_ptr 在构造函数里面初始化的，指向实际引用的对象
    inline sp() : m_ptr(0) { }

    sp(T* other);
    sp(const sp<T>& other);
    template<typename U> sp(U* other);
    template<typename U> sp(const sp<U>& other);

    ~sp();
    
    // Assignment

    sp& operator = (T* other);
    sp& operator = (const sp<T>& other);
    
    template<typename U> sp& operator = (const sp<U>& other);
    template<typename U> sp& operator = (U* other);
    
    //! Special optimization for use by ProcessState (and nobody else).
    void force_set(T* other);
    
    // Reset
    
    void clear();
    
    // Accessors

    inline  T&      operator* () const  { return *m_ptr; }
    inline  T*      operator-> () const { return m_ptr;  }
    inline  T*      get() const         { return m_ptr; }

    // Operators
        
    COMPARE(==)
    COMPARE(!=)
    COMPARE(>)
    COMPARE(<)
    COMPARE(<=)
    COMPARE(>=)

private:    
    template<typename Y> friend class sp;
    template<typename Y> friend class wp;

    // Optimization for wp::promote().
    sp(T* p, weakref_type* refs);
    
    T*              m_ptr;
};
```



`sp`类的构造函数有两个版本，一个是普通的构造函数，一个是拷贝构造函数  :

```c++
// frameworks\base\include\utils\RefBase.h

/*
由于成员变量m_ptr所指向的对象是从LightRefBase类继承下来的，
因此，这两个构造函数实际上是调用了LightRefBase类的成员函数incStrong来增加对象的引用计数。
*/

// 首先初始化成员变量m_ptr
template<typename T>
sp<T>::sp(T* other)
    : m_ptr(other)
{
    if (other) 
    {
        // 调用它的成员函数incStrong来增加它的引用计数
        other->incStrong(this);
    }
}

// 首先初始化成员变量m_ptr
template<typename T>
sp<T>::sp(const sp<T>& other)
    : m_ptr(other.m_ptr)
{
    if (m_ptr) 
    {
        // 调用它的成员函数incStrong来增加它的引用计数
        m_ptr->incStrong(this);
    }
}
```



sp类的析构函数的实现  :

```c++
// frameworks\base\include\utils\RefBase.h

/*
析构函数执行的操作刚好与构造函数相反，即调用成员变量m_ptr所指向的对象的成员函数decStrong来减少它的引用计数，
实际上是调用LightRefBase类的成员函数decStrong来减少对象的引用计数
*/

template<typename T>
sp<T>::~sp()
{
    if (m_ptr) 
    {
        // 调用成员函数decStrong来减少它的引用计数
        m_ptr->decStrong(this);
    }
}
```



### 应用实例分析

在`external`目录中建立一个`C++`应用程序`lightpointer`来说明**轻量级指针**的使用方法，它的目录结构如下：  

```shell
~/Android
	external
		lightpointer
			lightpointer.cpp
			Android.mk
```

应用程序`lightpointer`的实现，它只有一个源文件`lightpointer.cpp`和一个编译脚本文件`Android.mk`  

`lightpointer.cpp`  :

```c++
#include <stdio.h>
#include <utils/RefBase.h>

using  namespace android;

// 继承了LightRefBase类的LightClass类 , 能够结合轻量级指针来使用
class LightClass : public LightRefBase<LightClass>
{
public:
    LightClass()
    {
        printf("Construct LightClass Object.\n");
    }
    
    virtual ~LightClass()
    {
        printf("Destory LightClass Object.\n");
    }
    
};

// lightpointer 的入口函数 main
int main(int argc, char** argv)
{
    // 创建了一个 LightClass 对象 pLightClass
    LightClass* pLightClass = new LightClass();
    
    // 创建一个轻量级指针 lpOut 来引用它
    // 创建过程中, 调用 sp 类的构造函数来增加 LightClass 对象 pLightClass 的引用计数，
    // 即此时 LightClass 对象 pLightClass 的引用计数值为 1
    sp<LightClass> lpOut = pLightClass;
    
    // 打印 == 1
    printf("Light Ref Count: %d.\n", pLightClass->getStrongCount());
    
    //内嵌的作用域
    {
        /* 
         * 创建了另外一个轻量级指针 lpInner 来引用 LightClass 对象 pLightClass
		* LightClass 对象 pLightClass 的引用计数值会再增加1
		*/
		sp<LightClass> lpInner = lpOut;

    	// printf语句打印出来的数字就应该等于2
		printf("Light Ref Count: %d.\n", pLightClass->getStrongCount());
	}
    /* 
     * 当应用程序 lightpointer 跳出了作用域之后，轻量级指针 lpInner 就被析构了，
     *这时候 LightClass 对象 pLightClass 的引用计数值就会减少 1
     */
    
    //打印 == 1
    printf("Light Ref Count: %d.\n", pLightClass->getStrongCount());
    
    /*
     *当应用程序 lightpointer 执行完成之后，轻量级指针 lpOut 也会被析构，
     *这时 LightClass 对象 pLightClass 的引用计数值就会再次减少 1，== 0，于是LightClass对象pLightClass就会被释放
     */
    return 0;
}
```

Android.mk  

应用程序`lightpointer`的编译脚本文件，它引用了`libcutils`和 `libutils`两个库  

```makefile
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE_TAGS := optional
LOCAL_MODULE := lightpointer
LOCAL_SRC_FILES := lightpointer.cpp
LOCAL_SHARED_LIBRARIES := \
	libcutils \
	libutils
include $(BUILD_EXECUTABLE)
```

对这个C++工程进行**编译**和**打包**  

```shell
mmm ./external/lightpointer/
make snod
```

编译成功之后，就可以在 `out/target/product/gerneric/system/bin` 目录下看到应用程序文件`lightpointer`了；

打包成功之后，该应用程序就包含在`out/target/product/gerneric`目录下的`Android`系统镜像文件`system.img`中了。  



```shell
# 使用新得到的系统镜像文件 system.img 来启动 Android 模拟器
emulator &

# 使用adb工具连接上它
adb shell

# 进入到/system/bin 目录中
cd system/bin/

# 运行应用程序 lightpointer 来查看它的输出
./lightpointer
```

![image-20200712214524861](https://gitee.com/cpu_code/picture_bed/raw/master//20200712214524.png)

如果能看到上面的输出，就说明我们前面对**轻量级指针**的实现原理的分析是正确的。  



-----------------------------------------

## 强指针和弱指针

强指针和弱指针通过强引用计数和弱引用计数来维护对象的生命周期。  

如果一个类的对象要支持使用**强指针**和**弱指针**，那么它就必须从`RefBase`类继承下来，因为`RefBase`类提供了**强引用计数器**和**弱引用计数器**。  



### 强指针的实现原理分析

分析RefBase类的实现原理  :

```c++
// frameworks\base\include\utils\RefBase.h

class RefBase
{
public:
    // 成员函数 incStrong 和 decStrong 来维护它所引用的对象的引用计数
    void incStrong(const void* id) const;
    void decStrong(const void* id) const;
    
	void forceIncStrong(const void* id) const;

	//! DEBUGGING ONLY: Get current strong ref count.
	int32_t	getStrongCount() const;

    class weakref_type
    {
    public:
        RefBase* refBase() const;
        
        void incWeak(const void* id);
        void decWeak(const void* id);
        
        bool attemptIncStrong(const void* id);
        
        //! This is only safe if you have set OBJECT_LIFETIME_FOREVER.
        bool attemptIncWeak(const void* id);

        //! DEBUGGING ONLY: Get current weak ref count.
        int32_t	getWeakCount() const;

        //! DEBUGGING ONLY: Print references held on object.
        void printRefs() const;

        //! DEBUGGING ONLY: Enable tracking for this object.
        // enable -- enable/disable tracking
        // retain -- when tracking is enable, if true, then we save a stack trace
        //           for each reference and dereference; when retain == false, we
        //           match up references and dereferences and keep only the 
        //           outstanding ones.
        
        void trackMe(bool enable, bool retain);
    };
    
	weakref_type* createWeak(const void* id) const;
            
	weakref_type* getWeakRefs() const;

	//! DEBUGGING ONLY: Print references held on object.
    inline void	printRefs() const { getWeakRefs()->printRefs(); }

	//! DEBUGGING ONLY: Enable tracking of object.
    inline void trackMe(bool enable, bool retain)
    { 
        getWeakRefs()->trackMe(enable, retain); 
    }

protected:
    RefBase();
    virtual ~RefBase();
    
    //! Flags for extendObjectLifetime()
    enum 
	{
        OBJECT_LIFETIME_WEAK    = 0x0001,
        OBJECT_LIFETIME_FOREVER = 0x0003
    };
    
	void extendObjectLifetime(int32_t mode);
            
    //! Flags for onIncStrongAttempted()
    enum 
	{
        FIRST_INC_STRONG = 0x0001
    };
    
    virtual void onFirstRef();
    virtual void onLastStrongRef(const void* id);
    virtual bool onIncStrongAttempted(uint32_t flags, const void* id);
    virtual void onLastWeakRef(const void* id);

private:
    friend class weakref_type;
    class weakref_impl;
    
	RefBase(const RefBase& o);
	RefBase& operator=(const RefBase& o);
    
    //使用一个weakref_impl对象，即成员变量 mRefs 来描述对象的引用计数
	weakref_impl* const mRefs;
};
```



`weakref_impl`类同时为对象提供了强引用计数和弱引用计数  :

```c++
// frameworks\base\libs\utils\RefBase.cpp

/*
weakref_type 类定义在RefBase类的内部，它提供了成员函数 incWeak、decWeak、attemptIncStrong 和 attemptIncWeak 来
维护对象的强引用计数和弱引用计数 . 
weakref_type 类只定义了引用计数维护接口，具体的实现是由 weakref_impl 类提供的
*/
// weakref_impl类继承了weakref_type类
class RefBase::weakref_impl : public RefBase::weakref_type
{
public:
    volatile int32_t    mStrong;	// 对象的强引用计数
    volatile int32_t    mWeak;		// 弱引用计数
    RefBase* const      mBase;		// 指向了它所引用的对象的地址
    /*
     * 一个标志值，用来描述对象的生命周期控制方式
     * 取值范围为0、OBJECT_LIFETIME_WEAK 或者 OBJECT_LIFETIME_FOREVER
     * 0 : 对象的生命周期只受强引用计数影响
     * OBJECT_LIFETIME_WEAK : 对象的生命周期同时受强引用计数和弱引用计数影响；
     * OBJECT_LIFETIME_FOREVER : 对象的生命周期完全不受强引用计数或者弱引用计数影响
     */
    volatile int32_t    mFlags;		

// 类的实现有调试和非调试两个版本
#if !DEBUG_REFS
    // 被编译成非调试版本

    // 成员变量mStrong的值被初始化为 INITIAL_STRONG_VALUE == 1 << 28
    weakref_impl(RefBase* base)
        : mStrong(INITIAL_STRONG_VALUE)
        , mWeak(0)
        , mBase(base)
        , mFlags(0)
    {
    }

    void addStrongRef(const void* /*id*/) { }
    void removeStrongRef(const void* /*id*/) { }
    void addWeakRef(const void* /*id*/) { }
    void removeWeakRef(const void* /*id*/) { }
    void printRefs() const { }
    void trackMe(bool, bool) { }

#else
    // 类被编译成调试版本
	//...
#endif
};
```



RefBase、weakref_type和weakref_impl类的关系  : 

<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200713090936.png" alt="RefBase、weakref_type和weakref_impl类的关系" style="zoom: 67%;" />

每一个RefBase对象都包含了一个weakref_impl对象， 而后者继承了weakref_type类  



--------------------



强指针的实现类为sp，  

sp类的构造函数的实现  : 

```c++
// frameworks\base\include\utils\RefBase.h

// 模块参数 T 是一个继承了 RefBase 类的子类
template<typename T>
sp<T>::sp(T* other)
    : m_ptr(other)
{
    // 调用了 RefBase 类的成员函数 incStrong 来增加对象的强引用计数
    if (other) 
    {
        other->incStrong(this);
    }
}
```



```c++
// frameworks\base\libs\utils\RefBase.cpp

// RefBase 类的成员函数 incStrong实现
void RefBase::incStrong(const void* id) const
{
    weakref_impl* const refs = mRefs;
    refs->addWeakRef(id);
    
    // 增加对象的弱引用计数
    refs->incWeak(id);
    refs->addStrongRef(id);
    
    // 增加对象的强引用计数 , 增加RefBase类的引用计数对象mRefs的成员变量mStrong的值
    // 返回值是对象原来的强引用计数值， 即加1前的值
    const int32_t c = android_atomic_inc(&refs->mStrong);
    LOG_ASSERT(c > 0, "incStrong() called on %p after last strong ref", refs);
#if PRINT_REFS
    LOGD("incStrong of %p from %p: cnt=%d\n", this, id, c);
#endif
    if (c != INITIAL_STRONG_VALUE)  
    {
        return;
    }

    // 将RefBase类的成员函数incStrong需要将它调整为1
    android_atomic_add(-INITIAL_STRONG_VALUE, &refs->mStrong);
    /*
     *调用对象的成员函数 onFirstRef 来通知对象， 它被强指针引用了
     * RefBase 类的成员函数 onFirstRef 是一个空实现，
     * 如果子类想要处理这个事件， 那么就必须要重写成员函数 onFirstRef
     */
    const_cast<RefBase*>(this)->onFirstRef();
}
```



```c++
// frameworks\base\libs\utils\RefBase.cpp

#define INITIAL_STRONG_VALUE (1<<28)
```



```c++
// frameworks\base\libs\utils\RefBase.cpp

// RefBase类的成员变量mRefs是在构造函数中初始化
RefBase::RefBase()
    : mRefs(new weakref_impl(this))
{
//    LOGV("Creating refs %p with RefBase %p\n", mRefs, this);
}
```



```c++
// frameworks\base\libs\utils\RefBase.cpp

// 调用 weakref_type 类的成员函数 incWeak 来增加对象的弱引用计数
void RefBase::weakref_type::incWeak(const void* id)
{
    // this指针指向的实际上是一个weakref_impl对象
    // 将它转换为一个weakref_impl指针 impl
    weakref_impl* const impl = static_cast<weakref_impl*>(this);
    impl->addWeakRef(id);
    
    // 增加它的成员变量mWeak的值， 即增加对象的弱引用计数
    const int32_t c = android_atomic_inc(&impl->mWeak);
    // 调用是与调试相关的
    LOG_ASSERT(c >= 0, "incWeak called on %p after last weak ref", this);
}
```



强指针类`sp`的构造函数的实现 , 主要做的事情就是增加对象的**强引用**计数和**弱引用**计数。 从这里就可以看出， 虽然我们的目的是增加对象的**强引用**计数，但是同时也会增加对象的**弱引用**计数， 即一个对象的**弱引用**计数一定是**大于**或 **等于**它的**强引用**计数的。  



---------------------

sp类的析构函数的实现  :

```c++
// frameworks\base\include\utils\RefBase.h

// sp类的成员变量m_ptr所指向的对象是继承了RefBase类
template<typename T>
sp<T>::~sp()
{
    if (m_ptr) 
    {
        // 调用了RefBase类的成员函数decStrong来减少对象的强引用计数
        m_ptr->decStrong(this);
    }
}
```



```c++
// frameworks\base\libs\utils\RefBase.cpp

// 析构函数 , 减少对象的强引用计数和弱引用计数
void RefBase::decStrong(const void* id) const
{
    weakref_impl* const refs = mRefs;
    
    // 非调试版本中是空函数调用
    refs->removeStrongRef(id);
    
    // 减少对象的强引用计数
    // 返回值是对象原来的强引用计数值，即减1前的值， 保存在变量c中
    const int32_t c = android_atomic_dec(&refs->mStrong);
#if PRINT_REFS
    LOGD("decStrong of %p from %p: cnt=%d\n", this, id, c);
#endif
    LOG_ASSERT(c >= 1, "decStrong() called on %p too many times", refs);
    
    // 如果变量c的值等于1， 就说明此时再也没有强指针引用这个对象了
    if (c == 1) 
    {
        // 调用该对象的成员函数 onLastStrongRef 执行一些业务相关的逻辑
        const_cast<RefBase*>(this)->onLastStrongRef(id);
        
        // 考虑是否需要释放该对象
        // 检查对象的生命周期是否受弱引用计数控制， 
    	//即 RefBase 类的成员变量 mRefs 的标志值 mFlags 的 OBJECT_LIFETIME_WEAK 位是否等于1
        // 不等于1， 就说明对象的生命周期不受弱引用计数影响
        if( (refs->mFlags & OBJECT_LIFETIME_WEAK) != OBJECT_LIFETIME_WEAK) 
        {
            // 释放对象所占用的内存， 这同时会导致RefBase类的析构函数被调用
            delete this;
        }
    }
    
    // 非调试版本中是空函数调用
    refs->removeWeakRef(id);
    
    /*
     * 减少对象的弱引用计数的操作
     * 变量 refs 指向的是 RefBase 类内部的 weakref_impl 对象 mRefs。 
     * weakref_impl 类的成员函数 decWeak 是从父类 weakref_type 继承下来的， 
     * 因此， 接下来实际执行的是 weakref_type 类的成员函数 decWeak
     */
    refs->decWeak(id);
}
```



```c++
// frameworks\base\libs\utils\RefBase.cpp

// RefBase类的析构函数
RefBase::~RefBase()
{
//    LOGV("Destroying RefBase %p (refs %p)\n", this, mRefs);
    
    /*
     * 对象的弱引用计数值 == 0
     * RefBase 类的成员变量 mRefs 指向的是一个 weakref_impl 对象， 它是在 RefBase 类的构造函数中创建的。 
     * 它所属的RefBase对象已经不存在了，而且它所引用的对象的弱引用计数值 == 0， 它也就不需要存在了
    */
    if (mRefs->mWeak == 0) 
    {
//        LOGV("Freeing refs %p of old RefBase %p\n", mRefs, this);
        
        // 把引用计数对象mRefs也一起释放
        delete mRefs;
    }
    /*
     * 在对象的弱引用计数值 > 0的情况下， 我们只能将对应的 RefBase 对象释放掉， 
     * 而不能将该 RefBase 对象内部的 weakref_impl 对象也释放掉
    */ 
}
```



```c++
// frameworks\base\libs\utils\RefBase.cpp

// weakref_type 类的成员函数 decWeak
void RefBase::weakref_type::decWeak(const void* id)
{
    weakref_impl* const impl = static_cast<weakref_impl*>(this);
    
    // 非调试版本中是空函数调用
    impl->removeWeakRef(id);
    
    // 减少对象的弱引用计数， 并且返回减少之前的值， 保存在变量c中
    const int32_t c = android_atomic_dec(&impl->mWeak);
    
    LOG_ASSERT(c >= 1, "decWeak called on %p too many times", this);
    
    if (c != 1) 
    {
        // 说明还有其他的弱指针在引用这个对象，那就不用进一步处理了
        return;
    }
    // c == 1， 那么就说明再也没有弱指针引用这个对象了， 同时也说明没有强指针引用这个对象
    // 当对象的弱引用计数值 == 0 时， 它的强引用计数值也一定 == 0
    
    // 在对象的弱引用计数值 == 0 时， 我们就要考虑是否需要将该对象释放掉。 
    // 这取决于对象的生命周期控制方式， 以及该对象是否被强指针引用过。
    
    if ((impl->mFlags & OBJECT_LIFETIME_WEAK) != OBJECT_LIFETIME_WEAK) 
    {
        // 对象的生命周期只受强引用计数控制
        
        
        if (impl->mStrong == INITIAL_STRONG_VALUE)
        {
            // 对象从来没有被强指针引用过， 那么在该对象的弱引用计数值 == 0时
            // 将该对象释放掉
            delete impl->mBase;
        }
        else 
        {
//            LOGV("Freeing refs %p of old RefBase %p\n", this, impl->mBase);
            // 只释放其内部的引用计数器对象 weakref_impl
            delete impl;
        }
    } 
    else 
    {
        // 对象的生命周期受弱引用计数控制
        
        // 调用对象的成员函数 onLastWeakRef 执行一些业务相关的逻辑
        impl->mBase->onLastWeakRef(id);
        
        /*
         * 检查对象的生命周期是否完全不受强引用计数和弱引用计数控制， 
         * 即 RefBase 类的成员变量 mRefs 的标志值 mFlags 的OBJECT_LIFETIME_FOREVER位是否 == 1
         */
        if ((impl->mFlags & OBJECT_LIFETIME_FOREVER) != OBJECT_LIFETIME_FOREVER) 
        {
            // != 1，释放对象所占用的内存
            delete impl->mBase;
        }
    }
}
```



对对象的生命周期控制方式作一个小结  : 

如果一个对象的**生命周期控制标志值** ==  `0`， 那么只要它的**强引用**计数值 == `0`， 系统就会自动**释放**这个对象  

如果一个对象的**生命周期控制标志值** ==  `OBJECT_LIFETIME_WEAK`， 那么只有当它的**强引用**计数值 和 **弱引用**计数值都 == 0， 系统才会自动**释放**这个对象。

如果一个对象的**生命周期控制标志值**被设置为 `OBJECT_LIFETIME_FOREVER`， 那么系统就**永远不会自动释放**这个对象， 它需要由开发人员来**手动地释放**。  



--------------------------



### 弱指针的实现原理分析

如果一个类的对象支持使用**弱指针**， 那么这个类就必须要从`RefBase`类继承下来， 因为`RefBase`类提供了**弱引用**计数器。   

**弱指针**类`wp`的实现  :

```c++
// frameworks\base\include\utils\RefBase.h

// wp类是一个模块类， 模板参数T表示对象的实际类型， 它必须是从RefBase类继承下来的
template <typename T>
class wp
{
public:
    typedef typename RefBase::weakref_type weakref_type;
    
    inline wp() : m_ptr(0) { }

    wp(T* other);
    wp(const wp<T>& other);
    wp(const sp<T>& other);
    template<typename U> wp(U* other);
    template<typename U> wp(const sp<U>& other);
    template<typename U> wp(const wp<U>& other);

    ~wp();
    
    // Assignment

    wp& operator = (T* other);
    wp& operator = (const wp<T>& other);
    wp& operator = (const sp<T>& other);
    
    template<typename U> wp& operator = (U* other);
    template<typename U> wp& operator = (const wp<U>& other);
    template<typename U> wp& operator = (const sp<U>& other);
    
    void set_object_and_refs(T* other, weakref_type* refs);

    // promotion to sp
    // 将这个弱指针升级为强指针
    // 升级成功，就说明该弱指针所引用的对象还没有被销毁， 可以正常使用
    sp<T> promote() const;

    // Reset
    
    void clear();

    // Accessors
    
    inline  weakref_type* get_refs() const { return m_refs; }
    
    inline  T* unsafe_get() const { return m_ptr; }

    // Operators

    COMPARE_WEAK(==)
    COMPARE_WEAK(!=)
    COMPARE_WEAK(>)
    COMPARE_WEAK(<)
    COMPARE_WEAK(<=)
    COMPARE_WEAK(>=)

    inline bool operator == (const wp<T>& o) const 
    {
        return (m_ptr == o.m_ptr) && (m_refs == o.m_refs);
    }
    template<typename U>
    inline bool operator == (const wp<U>& o) const 
    {
        return m_ptr == o.m_ptr;
    }

    inline bool operator > (const wp<T>& o) const 
    {
        return (m_ptr == o.m_ptr) ? (m_refs > o.m_refs) : (m_ptr > o.m_ptr);
    }
    template<typename U>
    inline bool operator > (const wp<U>& o) const 
    {
        return (m_ptr == o.m_ptr) ? (m_refs > o.m_refs) : (m_ptr > o.m_ptr);
    }

    inline bool operator < (const wp<T>& o) const 
    {
        return (m_ptr == o.m_ptr) ? (m_refs < o.m_refs) : (m_ptr < o.m_ptr);
    }
    template<typename U>
    inline bool operator < (const wp<U>& o) const 
    {
        return (m_ptr == o.m_ptr) ? (m_refs < o.m_refs) : (m_ptr < o.m_ptr);
    }
    
    inline bool operator != (const wp<T>& o) const 
    { 
        return m_refs != o.m_refs; 
    }
    template<typename U> inline bool operator != (const wp<U>& o) const 
    { 
        return !operator == (o); 
    }
    
    inline bool operator <= (const wp<T>& o) const 
    { 
        return !operator > (o); 
    }
    template<typename U> inline bool operator <= (const wp<U>& o) const 
    { 
        return !operator > (o); 
    }
    
    inline bool operator >= (const wp<T>& o) const 
    { 
        return !operator < (o); 
    }
    template<typename U> inline bool operator >= (const wp<U>& o) const 
    { 
        return !operator < (o); 
    }

private:
    template<typename Y> friend class sp;
    template<typename Y> friend class wp;

    // 指向它所引用的对象
    T*              m_ptr;
    // 维护对象的弱引用计数
    weakref_type*   m_refs;
};
```



wp类的构造函数的实现  :

```c++
// frameworks\base\include\utils\RefBase.h

// 模块参数T是一个继承了RefBase类的子类
template<typename T>
wp<T>::wp(T* other)
    : m_ptr(other)
{
    if (other) 
    {
        // 调用 RefBase 类的成员函数 createWeak 来增加对象的弱引用计数
        m_refs = other->createWeak(this);
    }
}
```



```c++
// frameworks\base\libs\utils\RefBase.cpp

// 将它的成员变量mRefs所指向的一个 weakref_impl 对象返回给调用者
RefBase::weakref_type* RefBase::createWeak(const void* id) const
{
    // RefBase类的成员变量 mRefs 指向的是一个 weakref_impl 对象
    // 加实际引用对象的弱引用计数
    mRefs->incWeak(id);
    return mRefs;
}
```



wp类的析构函数的实现  :











### 应用实例分析

在external目录下建立一个C++应用程序weightpointer来说明强指针和弱指针的使用方法  

目录结构  :

```shell
~/Android
	external
		weightpointer
			weightpointer.cpp
			Android.mk
```

















