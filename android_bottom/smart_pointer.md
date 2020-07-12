

# 智能指针

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

轻量级指针通过简单的引用计数技术来维护对象的生命周期。如果一个类的对象支持使用轻量级指针，那么它就必须要从`LightRefBase`类继承下来，因为`LightRefBase`类提供了一个简单的引用计数器。

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



轻量级指针的实现类为`sp`，它同时也是强指针的实现类 : 

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

```









-----------------------------------------

## 强指针和弱指针



### 强指针的实现原理分析







### 弱指针的实现原理分析







### 应用实例分析



















