# C++对C的扩展

##  **::作用域运算符**

通常情况下，如果有两个同名变量，一个是全局变量，另一个是局部变量，那么局部变量在其作用域内具有较高的优先权，它将屏蔽全局变量。

```c++
//全局变量
int a = 10;
void test()
{
    /* 变量a是test函数内定义的局部变量，因此输出的结果为局部变量a的值 */
	//局部变量
	int a = 20;
	//全局a被隐藏
	cout << "a:" << a << endl;
}
```

作用域运算符可以用来解决局部变量与全局变量的重名问题 

```c++
//全局变量
int a = 10;
//1. 局部变量和全局变量同名
void test()
{
	int a = 20;
	//打印局部变量a
	cout << "局部变量a:" << a << endl;
	//打印全局变量a
	cout << "全局变量a:" << ::a << endl;
}
```

**作用域运算符**可以用来解决**局部变量**与**全局变量**的重名问题，即在局部变量的作用域内，可用 **::** 对被屏蔽的同名的全局变量进行访问

##  名字控制

创建名字是程序设计过程中一项最基本的活动，当一个项目很大时，它会不可避免地包含大量名字。c++允许我们对名字的产生和名字的可见性进行控制。

我们之前在学习c语言可以通过**static**关键字来使得名字只得在本编译单元内可见，在c++中我们将通过一种通过命名空间来控制对名字的访问。

###  C++命名空间(namespace)

在c++中，名称（name）可以是**符号常量**、**变量**、**函数**、**结构**、**枚举**、**类**和**对象**等等。工程越大，名称互相冲突性的可能性越大。另外使用多个厂商的类库时，也可能导致名称冲突。为了避免，在大规模程序的设计中，以及在程序员使用各种各样的C++库时，这些标识符的命名发生冲突，标准C++引入关键字namespace（**命名空间** / **名字空间** / **名称空间**），可以更好地控制标识符的作用域。

### 命名空间使用语法

创建一个命名空间:

```c++
namespace A
{
	int a = 10;
}
namespace B
{
	int a = 20;
}
void test()
{
	cout << "A::a : " << A::a << endl;
	cout << "B::a : " << B::a << endl;
}
```

命名空间只能全局范围内定义（**以下错误写法**）

```c++
void test()
{
	namespace A
    {
		int a = 10;
	}
	namespace B
    {
		int a = 20;
	}
	cout << "A::a : " << A::a << endl;
	cout << "B::a : " << B::a << endl;
}
```

命名空间可嵌套命名空间

```c++
namespace A
{
	int a = 10;
	namespace B
    {
		int a = 20;
	}
}
void test()
{
	cout << "A::a : " << A::a << endl;
	cout << "A::B::a : " << A::B::a << endl;
}
```

命名空间是开放的，即可以随时把新的成员加入已有的命名空间中

```c++
namespace A
{
	int a = 10;
}

namespace A
{
	void func()
    {
		cout << "hello namespace!" << endl;
	}
}

void test()
{
	cout << "A::a : " << A::a << endl;
	A::func();
}
```

声明和实现可分离

```c++
#pragma once

namespace MySpace
{
	void func1();
	void func2(int param);
}
void MySpace::func1()
{
	cout << "MySpace::func1" << endl;
}
void MySpace::func2(int param)
{
	cout << "MySpace::func2 : " << param << endl;
}
```

无名命名空间，意味着命名空间中的标识符只能在**本文件内访问**，相当于给这个标识符加上了static，使得其可以作为**内部连接**

```c++
namespace
{
	
	int a = 10;
	void func()
    { 
        cout << "hello namespace" << endl; 
    }
}
void test()
{
	cout << "a : " << a << endl;
	func();
}
```

命名空间别名

```c++
namespace veryLongName
{
	int a = 10;
    
	void func()
    { 
        cout << "hello namespace" << endl; 
    }
}

void test()
{
	namespace shortName = veryLongName;
	cout << "veryLongName::a : " << shortName::a << endl;
	veryLongName::func();
	shortName::func();
}
```



###  using声明

using声明可使得指定的标识符可用

```c++
namespace A
{
	int paramA = 20;
	int paramB = 30;
    
	void funcA()
    { 
        cout << "hello funcA" << endl; 
    }
	void funcB()
    { 
        cout << "hello funcA" << endl; 
    }
}

void test()
{
	//1. 通过命名空间域运算符
	cout << A::paramA << endl;
	A::funcA();
    
	//2. using声明
	using A::paramA;
	using A::funcA;
    
	cout << paramA << endl;
	//cout << paramB << endl; //不可直接访问
	funcA();
	//3. 同名冲突
	//int paramA = 20; //相同作用域注意同名冲突
}
```

using声明碰到函数重载

```c++
namespace A
{
	void func(){}
	void func(int x){}
	int  func(int x,int y){}
}
void test()
{
	using A::func;
    
	func();
	func(10);
	func(10, 20);
}
```

如 命名空间包含一组用相同名字重载的函数，using声明就声明了这个重载函数的所有集合。

###  using编译指令

using编译指令使整个命名空间标识符可用:

```c++
namespace A
{
	int paramA = 20;
	int paramB = 30;
    
	void funcA()
    { 
        cout << "hello funcA" << endl; 
    }
    
	void funcB()
    { 
        cout << "hello funcB" << endl; 
    }
}

void test01()
{
	using namespace A;
	cout << paramA << endl;
	cout << paramB << endl;
	funcA();
	funcB();

	//不会产生二义性
	int paramA = 30;
	cout << paramA << endl;
}

namespace B
{
	int paramA = 20;
	int paramB = 30;
    
	void funcA()
    { 
        cout << "hello funcA" << endl; 
    }
	void funcB()
    { 
        cout << "hello funcB" << endl; 
    }
}

void test02()
{
	using namespace A;
	using namespace B;
	//二义性产生，不知道调用A还是B的paramA
	//cout << paramA << endl;
}
```

注意：使用using声明或using编译指令会增加命名冲突的可能性。也就是说，如果有名称空间，并在代码中使用作用域解析运算符，则不会出现二义性

### 命名空间使用

需要记住的关键问题是当引入一个全局的using编译指令时，就为该文件打开了该命名空间，它不会影响任何其他的文件，所以可以在每一个实现文件中调整对命名空间的控制。比如，如果发现某一个实现文件中有太多的using指令而产生的命名冲突，就要对该文件做个简单的改变，通过明确的限定或者using声明来消除名字冲突，这样不需要修改其他的实现文件。

##  全局变量检测增强

c语言代码：

```c++
int a = 10; //赋值，当做定义
int a; 		//没有赋值，当做声明

int main()
{
	printf("a:%d\n",a);
	return EXIT_SUCCESS;
}
```

此代码在c++下编译失败,在c下编译通过.

##  C++中所有的变量和函数都必须有类型

c语言代码：

```c++
//i没有写类型，可以是任意类型
int fun1(i)
{
	printf("%d\n", i);
	return 0;
}
//i没有写类型，可以是任意类型
int fun2(i)
{
	printf("%s\n", i);
	return 0;
}
//没有写参数，代表可以传任何类型的实参
int fun3()
{ 
	printf("fun33333333333333333\n");
	return 0;
}

//C语言，如果函数没有参数，建议写void，代表没有参数
int fun4(void)
{
	printf("fun4444444444444\n");
	return 0;
}

g()
{
	return 10;
}

int main()
{

	fun1(10);
	fun2("abc");
	fun3(1, 2, "abc");
	printf("g = %d\n", g());

	return 0;
}
```

以上c代码c编译器编译可通过，c++编译器无法编译通过。

在C语言中，int fun() 表示返回值为int，接受任意参数的函数，int fun(void) 表示返回值为int的无参函数。

在C++ 中，int fun() 和 int fun(void) 具有相同的意义，都表示返回值为int的无参函数。

## 更严格的类型转换

在C++，不同类型的变量一般是不能直接赋值的，需要相应的强转

c语言代码：

```c++
typedef enum COLOR
{ 
    GREEN, 
    RED, 
    YELLOW 
} color;

int main()
{

	color mycolor = GREEN;
	mycolor = 10;
    
	printf("mycolor:%d\n", mycolor);
    
	char* p = malloc(10);
    
	return EXIT_SUCCESS;
}
```

以上c代码c编译器编译可通过，c++编译器无法编译通过

## struct类型加强

c中定义结构体变量需要加上struct关键字，c++不需要。

c中的结构体只能定义成员变量，不能定义成员函数。c++即可以定义成员变量，也可以定义成员函数

```c++
//1. 结构体中即可以定义成员变量，也可以定义成员函数
struct Student
{
	string mName;
	int mAge;
    
	void setName(string name)
    { 
        mName = name; 
    }
	void setAge(int age)
    { 
        mAge = age; 
    }
	void showStudent()
    {
		cout << "Name:" << mName << " Age:" << mAge << endl;
	}
};

//2. c++中定义结构体变量不需要加struct关键字
void test01()
{
	Student student;
	student.setName("John");
	student.setAge(20);
	student.showStudent();
}
```

## 新增bool类型关键字

标准c++的bool类型有两种内建的常量 true(转换为整数1) 和 false(转换为整数0) 表示状态。这三个名字都是关键字。

*   bool类型只有两个值，true(1值)，false(0值)
*   bool类型占1个字节大小
*   给bool类型赋值时，非0值会自动转换为true(1)，0值会自动转换false(0)

```c++
void test()
{	
    cout << sizeof(false) << endl; //为1，//bool类型占一个字节大小
	bool flag = true;			 // c语言中没有这种类型
	flag = 100;					 //给bool类型赋值时，非0值会自动转换为true(1),0值会自动转换false(0)
}
```

[c语言中的bool类型]

c语言中也有bool类型，在c99标准之前是没有bool关键字，c99标准已经有bool类型，包含头文件stdbool.h，就可以使用和c++ 一样的bool类型。

##  三目运算符功能增强

n c语言三目运算表达式返回值为数据值，为右值，不能赋值：

```c
int a = 10;
int b = 20;

printf("ret:%d\n", a > b ? a : b);
//思考一个问题，(a > b ? a : b) 三目运算表达式返回的是什么？
	
//(a > b ? a : b) = 100;
//返回的是右值
```

c++语言三目运算表达式返回值为变量本身(引用)，为左值，可以赋值

```c++
int a = 10;
int b = 20;

printf("ret:%d\n", a > b ? a : b);
//思考一个问题，(a > b ? a : b) 三目运算表达式返回的是什么？

cout << "b:" << b << endl;
//返回的是左值，变量的引用
(a > b ? a : b) = 100;//返回的是左值，变量的引用
cout << "b:" << b << endl;
```

[左值和右值概念]

  在c++中可以放在赋值操作符左边的是左值，可以放到赋值操作符右面的是右值。

  有些变量即可以当左值，也可以当右值。

  左值为Lvalue，L代表Location，表示内存可以寻址，可以赋值。

  右值为Rvalue，R代表Read,就是可以知道它的值。

  比如:int temp = 10; temp在内存中有地址，10没有，但是可以Read到它的值



## C/C++中的const

###  const概述

const 单词字面意思为**常数**，不变的。它是c/c++中的一个关键字，是一个限定符，它用来限定一个变量不允许改变，它将一个对象转换成一个常量。

```c++
const int a = 10;
A = 100; //编译错误,const是一个常量，不可修改
```

###  C/C++中const的区别

####  C中的const

常量的引进是在c++早期版本中，当时标准C规范正在制定。那时，尽管C委员会决定在C中引入const，但是，他们c中的const理解为” 一个不能改变的普通变量 ”，也就是认为const应该是一个只读变量，既然是变量那么就会给const分配内存，并且在c中const是一个全局只读变量，c语言中const修饰的只读变量是外部连接的。

```c++
const int arrSize = 10;
int arr[arrSize];
```

因为arrSize占用某块内存，所以C编译器不知道它在编译时的值是多少

#### C++中的const

在c++中，一个const不必创建内存空间，而在c中，一个const总是需要一块内存空间。在c++中，是否为const常量分配内存空间依赖于如何使用。一般说来，如果一个const仅仅用来把一个名字用一个值代替(就像使用#define一样)，那么该存储局空间就不必创建。

如果存储空间没有分配内存的话，在进行完数据类型检查后，为了代码更加有效，值也许会折叠到代码中。

不过，取一个const地址, 或者把它定义为extern,则会为该const创建内存空间。

 在c++中，出现在所有函数之外的const作用于整个文件(也就是说它在该文件外不可见)，默认为内部连接，c++中其他的标识符一般默认为外部连接。

####  C/C++中`const`异同总结

**c**语言全局`const`会被存储到只读数据段。c++中全局`const`当声明`extern`或者对变量取地址时，编译器会分配存储地址，变量存储在只读数据段。两个都受到了只读数据段的保护，不可修改。

```c
const int constA = 10;

int main()
{
	int* p = (int*)&constA;
	*p = 200;
}
```

以上代码在c/c++中编译通过，在运行期，修改`constA`的值时，发生写入错误。原因是修改只读数据段的数据。

c语言中局部`const`存储在堆栈区，只是不能通过变量直接修改`const`只读变量的值，但是可以跳过编译器的检查，通过指针间接修改`const`值。

```c
const int constA = 10;
int* p = (int*)&constA;

*p = 300;
printf("constA:%d\n",constA);
printf("*p:%d\n", *p);
```

c语言中，通过指针间接赋值修改了`constA`的值。

**c++**中对于局部的`const`变量要区别对待：

1、对于基础数据类型，也就是`const int a = 10`这种，编译器会把它放到符号表中，不分配内存，当对其取地址时，会分配内存

```c++
const int constA = 10;
int* p = (int*)&constA;
*p = 300;

cout << "constA:" << constA << endl;
cout << "*p:" << *p << endl;
```

`constA`在符号表中，当我们对`constA`取地址，这个时候为`constA`分配了新的空间，`*p`操作的是分配的空间，而`constA`是从符号表获得的值。

2、对于基础数据类型，如果用一个变量初始化`const`变量，如果`const int a = b`，那么也是会给`a`分配内存

```c++
int b = 10;
const int constA = b;
int* p = (int*)&constA;
*p = 300;

cout << "constA:" << constA << endl;
cout << "*p:" << *p << endl;
```

`constA` 分配了内存，所以我们可以修改`constA`内存中的值。

3、对于自定数据类型，比如类对象，那么也会分配内存

```c++
const Person person; //未初始化age
//person.age = 50; //不可修改
Person* pPerson = (Person*)&person;
//指针间接修改
pPerson->age = 100;
cout << "pPerson->age:" << pPerson->age << endl;

pPerson->age = 200;
cout << "pPerson->age:" << pPerson->age << endl;
```

为`person`分配了内存，所以我们可以通过指针的间接赋值修改`person`对象。

c中`const`默认为外部连接，c++中`const`默认为内部连接。当c语言两个文件中都有`const int a`的时候，编译器会报重定义的错误。而在c++中，则不会，因为c++中的`const`默认是内部连接的。如果想让c++中的`const`具有外部连接，必须显示声明为: `extern const int a = 10;`

`const`由c++采用，并加进标准c中，尽管他们很不一样。在c中，编译器对待`const`如同对待变量一样，只不过带有一个特殊的标记，意思是” 你不能改变我 ”。在c++中定义`const`时，编译器为它创建空间，所以如果在两个不同文件定义多个同名的`const`，链接器将发生链接错误。简而言之，`const`在c++中用的更好。

### 尽量以`const`替换`#define`

在旧版本C中，如果想建立一个常量，必须使用预处理器

`#define MAX 1024;`

我们定义的宏`MAX`从未被编译器看到过，因为在预处理阶段，所有的`MAX`已经被替换为了`1024`，于是`MAX`并没有将其加入到符号表中。但我们使用这个常量获得一个编译错误信息时，可能会带来一些困惑，因为这个信息可能会提到`1024`，但是并没有提到`MAX`。如果`MAX`被定义在一个不是你写的头文件中，你可能并不知道`1024`代表什么，也许解决这个问题要花费很长时间。

解决办法就是用一个常量替换上面的宏。

`const int max= 1024;`

`const`和`#define`区别总结：

`const`有类型，可进行编译器类型安全检查。`#define`无类型，不可进行类型检查。

`const`有作用域，而`#define`不重视作用域，默认定义处到文件结尾。如果定义在指定作用域下有效的常量，那么`#define`就不能用。

宏常量没有类型，所以调用了`int`类型重载的函数。`const`有类型，所以调用希望的short类型函数？

```c++
#define PARAM 128
const short param = 128;

void func(short a)
{
	cout << "short!" << endl;
}

void func(int a)
{
	cout << "int" << endl;
}
```

宏常量不重视作用域

```c++
void func1()
{
	const int a = 10;
	#define A 20 
    //#undef A  //卸载宏常量A
}
void func2()
{
	//cout << "a:" << a << endl; //不可访问，超出了const int a作用域
	cout << "A:" << A << endl; //#define作用域从定义到文件结束或者到#undef，可访问
}
int main()
{
	func2();
	return EXIT_SUCCESS;
}
```

问题: 宏常量可以有命名空间吗？

```c++
namespace MySpace
{
	#define num 1024
}
void test()
{
	//cout << MySpace::NUM << endl; //错误
	//int num = 100; //命名冲突
	cout << num << endl;
}
```



## 引用(reference)





