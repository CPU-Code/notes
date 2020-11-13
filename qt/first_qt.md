# 第一个QT

![Nnpxmj.png](https://s1.ax1x.com/2020/06/18/Nnpxmj.png)

下载地址：[https://www.qt.io/download-open-source](https://www.qt.io/download-open-source)

![UTOOLS1592481283123.png](http://yanxuan.nosdn.127.net/c80fd0a922a0b6e76fdcf92cef6117f7.png)

打开 Qt Creator 界面选择 New Project 或者选择菜单栏 【文件】 -【新建文件或项目】 菜单项

![NmNNgf.png](https://s1.ax1x.com/2020/06/18/NmNNgf.png)

弹出 New Project 对话框，选择 Application，选择 Qt Widgets Application ，选择 Choose 按钮， 弹出如下对话框

![NmUAsS.png](https://s1.ax1x.com/2020/06/18/NmUAsS.png)

设置项目名称和路径，按照向导进行下一步，

![NmO8C4.png](https://s1.ax1x.com/2020/06/18/NmO8C4.png)

选择编译套件

![NmH60O.png](https://s1.ax1x.com/2020/06/18/NmH60O.png)

向导会默认添加一个继承自 CMainWindow 的类，可以在此修改类的名字和基类。 默认的基类有 QMainWindow、 QWidget 以及 QDialog 三个，我们可以选择 QWidget（类似于空窗口）

![NmUvlV.png](https://s1.ax1x.com/2020/06/18/NmUvlV.png)

系统会默认给我们添加 文件 和一个.pro项目文件，点击完成， 即可创建出一个 Qt 桌面程序。

![NmaKTH.png](https://s1.ax1x.com/2020/06/18/NmaKTH.png)

点击运行，马上就要看见成功了

![Nma0ts.png](https://s1.ax1x.com/2020/06/18/Nma0ts.png)

为绿色就成功了，这也是成功男士看破红尘的时候，既然绿了自然就有结果了

![NmdENj.png](https://s1.ax1x.com/2020/06/18/NmdENj.png)

这就是成功的结果，现在就是看看结果的准确性了（做一下亲子鉴定）

![NmdlbF.png](https://s1.ax1x.com/2020/06/18/NmdlbF.png)

## 分析一下代码：

### `myqt.pro` 工程文件\(project\)，它是 qmake 自动生成的用于生产 makefile 的配置文件

```text
#-------------------------------------------------
#
# Project created by QtCreator 2020-06-18T17:23:42
#
#-------------------------------------------------

# 包含的模块
QT       += core gui

# 大于 Qt4 版本才包含 widget 模块
greaterThan(QT_MAJOR_VERSION, 4): QT += widgets

# 应用程序名 生成的.exe 程序名称
TARGET = myqt

# 模板类型 应用程序模板
TEMPLATE = app

# The following define makes your compiler emit warnings if you use
# any feature of Qt which as been marked as deprecated (the exact warnings
# depend on your compiler). Please consult the documentation of the
# deprecated API in order to know how to port your code away from it.
DEFINES += QT_DEPRECATED_WARNINGS

# You can also make your code fail to compile if you use deprecated APIs.
# In order to do so, uncomment the following line.
# You can also select to disable deprecated APIs only up to a certain version of Qt.
#DEFINES += QT_DISABLE_DEPRECATED_BEFORE=0x060000    # disables all the APIs deprecated before Qt 6.0.0


# 源文件
SOURCES += \
        main.cpp \
        widget.cpp

# 头文件
HEADERS += \
        widget.h
# .ui 设计文件
FORMS += \
        widget.ui
```

### `main.cpp`

```cpp
#include "widget.h"

//标准类名声明头文件没有.h 后缀
#include <QApplication>

int main(int argc, char *argv[])
{
    // 应用程序类
    QApplication a(argc, argv);

    //MyWidget对象
    Widget w;


    w.show();

    //程序进入消息循环
    return a.exec();
}
```

`widget.cpp`

```cpp
#include "widget.h"
#include "ui_widget.h"

Widget::Widget(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::Widget)
{
    ui->setupUi(this);
}

Widget::~Widget()
{
    delete ui;
}
```

### `widget.h` 头文件

```cpp
#ifndef WIDGET_H
#define WIDGET_H

#include <QWidget>

namespace Ui {
class Widget;
}

class Widget : public QWidget
{
    Q_OBJECT

public:
    explicit Widget(QWidget *parent = 0);
    ~Widget();

private:
    Ui::Widget *ui;
};

#endif // WIDGET_H
```

QT的所有代码都已经同步在[gitee](https://gitee.com/cpu_code/QT)中，并持续更新。

> 由于个人水平有限, 难免有些错误, 希望各位点评
>
> @Author: cpu\_code
>
> @Date: 2020-06-11 20:24:20
>
> @LastEditTime: 2020-06-11 20:26:09
>
> @FilePath: \QT\firstQt\main.cpp
>
> @Gitee: [https://gitee.com/cpu\_code](https://gitee.com/cpu_code)
>
> @Github: [https://github.com/CPU-Code](https://github.com/CPU-Code)
>
> @CSDN: [https://blog.csdn.net/qq\_44226094](https://blog.csdn.net/qq_44226094)
>
> @Gitbook: [https://923992029.gitbook.io/cpucode/](https://923992029.gitbook.io/cpucode/)

