# 事件

![Nnpxmj.png](https://s1.ax1x.com/2020/06/18/Nnpxmj.png)

## 事件处理

`GTK+` 所提供的工具库与其应用程序都是基于**事件触发机制**来管理， 所有的应用程序都是基于**事件驱动**。

如果没有事件发生， 应用程序将处于**等待**状态， 不会执行任何操作， 一旦事件发生， 将根据不同的事件做出相应的处理。

在 `GTK+`中,当一个事件发生时， 程序就会通过发送一个**信号**来通知应用程序执行相关的操作， 即调用与这一信号进行连接的**回调函数**, 来完成一次由事件所触发的行动。

### 事件的捕获

```c
/* 设置控件捕获相应的事件
 * events： 事件类型， 它是 GdkEventMask 的枚举常量
 * GDK_BUTTON_PRESS_MASK： 鼠标点击
 * GDK_BUTTON_RELEASE_MASK： 鼠标释放
 * GDK_BUTTON_MOTION_MASK： 鼠标移动
 * GDK_KEY_PRESS_MASK： 键盘按下
 * GDK_ENTER_NOTIFY_MASK： 进入控件区域
 */
void gtk_widget_add_events(GtkWidget *widget, gint events );
```

### 鼠标事件

主窗口默认不接收鼠标事件， 需要手动添加

触发鼠标点击事件的信号： `button-press-event`

触发鼠标释放事件的信号： `button-release-event`

```c
// 回调函数
// event->x， event->y： 得到点击坐标值
// event->button： 鼠标哪个键按下
// event->type: 是否双击
gboolean callback( GtkWidget *widget, GdkEventButton *event, gpointer data );
```

触发鼠标移动事件的信号： `motion-notify-event`

```c
// 回调函数
// event->x， event->y： 得到移动的坐标值
gboolean callback( GtkWidget *widget, GdkEventMotion *event, gpointer data );
```

栗子 : mouse\_event.c

```c
#include <gtk/gtk.h>    // 头文件

// 鼠标点击事件处理函数
gboolean deal_mouse_press(GtkWidget *widget, GdkEventButton *event, gpointer data)
{
    // 判断鼠标点击的类型
    switch(event->button)
    {
        case 1:
            printf("Left Button!!\n");
            break;

        case 2:
            printf("Middle Button!!\n");
            break;

        case 3:
            printf("Right Button!!\n");
            break;

        default:
            printf("Unknown Button!!\n");
    }

    if(event->type == GDK_2BUTTON_PRESS)
    {
        printf("double click\n");
    }

    // 获得点击的坐标值，距离窗口左顶点
    gint i = event->x;
    gint j = event->y;
    printf("press_x = %d, press_y = %d\n", i, j);

    return TRUE;
}

// 鼠标移动事件(点击鼠标任何键)的处理函数
gboolean deal_motion_notify_event(GtkWidget *widget, GdkEventMotion *event, gpointer data)
{
    // 获得移动鼠标的坐标值，距离窗口左顶点
    gint i = event->x;
    gint j = event->y;

    printf("motion_x = %d, motion_y = %d\n", i, j);

    return TRUE;
}

int main( int argc,char *argv[] )
{
    gtk_init(&argc, &argv);        // 初始化

    // 创建顶层窗口
    GtkWidget *window = gtk_window_new(GTK_WINDOW_TOPLEVEL);

    // 设置窗口的标题
    gtk_window_set_title(GTK_WINDOW(window), "cpucode mouse_event");
    // 设置窗口在显示器中的位置为居中
    gtk_window_set_position(GTK_WINDOW(window), GTK_WIN_POS_CENTER);
    // 设置窗口的最小大小
    gtk_widget_set_size_request(window, 400, 300);
    // "destroy" 和 gtk_main_quit 连接
    g_signal_connect(window, "destroy", G_CALLBACK(gtk_main_quit), NULL);

    // 窗口接收鼠标事件
    // GDK_BUTTON_PRESS_MASK ：鼠标点击事件
    // GDK_BUTTON_MOTION_MASK ：按住鼠标移动事件
    gtk_widget_add_events(window, GDK_BUTTON_PRESS_MASK | GDK_BUTTON_MOTION_MASK);

    // "button-press-event" 与 deal_mouse_event 连接，鼠标点击事件
    g_signal_connect(window, "button-press-event", G_CALLBACK(deal_mouse_press), NULL);
    // "motion-notify-event" 与 deal_motion_notify_event 连接，按住鼠标移动事件
    g_signal_connect(window, "motion-notify-event", G_CALLBACK(deal_motion_notify_event), NULL);

    gtk_widget_show_all(window);    // 显示窗口全部控件

    gtk_main();        // 主事件循环

    return 0;
}
```

Makefile :

```text
CC = gcc  
MAINC = mouse_event.c
EXEC = mouse_event

CFLAGS = `pkg-config --cflags --libs gtk+-2.0`

main:  
    $(CC)  $(MAINC)  -o $(EXEC) $(CFLAGS)
clean:
    rm $(EXEC) -rf
```

![](https://gitee.com/cpu_code/picture_bed/raw/master//20200805161952.png)

### 键盘事件

主窗口默认就能接收键盘事件， 其中的键值定义在 `gtk-2.0/gdk/gdkkeysyms-compat.h` 文件里

触发键盘按下事件的信号： `key-press-event`

触发键盘释放事件的信号： `key-release-event`

```c
// 回调函数
// event->keyval： 获取按下(释放)键盘键值
gboolean callback( GtkWidget *widget, GdkEventKey *event, gpointer data );
```

栗子 : key\_event.c

```c
#include <gtk/gtk.h>    // 头文件
#include <gdk/gdkkeysyms.h>    //键盘头文件

// 键盘按下事件处理函数
gboolean deal_key_press(GtkWidget *widget, GdkEventKey  *event, gpointer data)
{
    switch(event->keyval)
    {    
        // 键盘键值类型
        case GDK_Up:
            g_print("Up\n");
            break;

        case GDK_Left:
            g_print("Left\n");
            break;

        case GDK_Right:
            g_print("Right\n");
            break;

        case GDK_Down:
            g_print("Down\n");
            break;

        default:
            printf("Unknown Button!!\n");
    }

    int key = event->keyval; // 获取键盘键值类型
    g_print("keyval = %d\n", key);

    return TRUE;
}

int main( int argc, char *argv[] )
{
    gtk_init(&argc, &argv);        // 初始化

    // 创建顶层窗口
    GtkWidget *window = gtk_window_new(GTK_WINDOW_TOPLEVEL);
    // 设置窗口的标题
    gtk_window_set_title(GTK_WINDOW(window), "cpucode mouse_event");
    // 设置窗口在显示器中的位置为居中
    gtk_window_set_position(GTK_WINDOW(window), GTK_WIN_POS_CENTER);
    // 设置窗口的最小大小
    gtk_widget_set_size_request(window, 400, 300);
    // "destroy" 和 gtk_main_quit 连接
    g_signal_connect(window, "destroy", G_CALLBACK(gtk_main_quit), NULL);

    // "key-press-event" 与 deal_key_press 连接
    g_signal_connect(window, "key-press-event", G_CALLBACK(deal_key_press), NULL);

    gtk_widget_show_all(window);    // 显示窗口全部控件

    gtk_main();        // 主事件循环

    return 0;
}
```

Makefile :

```text
CC = gcc  
MAINC = key_event.c
EXEC = key_event

CFLAGS = `pkg-config --cflags --libs gtk+-2.0`

main:  
    $(CC)  $(MAINC)  -o $(EXEC) $(CFLAGS)
clean:
    rm $(EXEC) -rf
```

![](https://gitee.com/cpu_code/picture_bed/raw/master//20200805163222.png)

### 事件盒子\(GtkEventBox\)

有些控件\( GtkLabel \)， 不响应 GDK 事件。 GTK+通过事件盒子给控件提供一个 GDK 窗口来捕获事件。

```c
//事件盒子的创建
GtkWidget *gtk_event_box_new(void);
```

```c
//添加控件到事件盒子里
void gtk_container_add(GtkContainer *container, GtkWidget *widget );
```

栗子 : event\_box.c

```c
/* 通过使用事件盒子，连接 button-press-event 信号到 GtkLabel。
 * 当标签被双击时，标签中的文本会根据当前的状态改变。
 * 当单击事件发生时，什么都不会发生，尽管在本例中这个信号也被发出了。
 */
 #include <gtk/gtk.h>

static gboolean button_pressed(GtkWidget*, GdkEventButton*, GtkLabel*); // 函数的声明

int main( int argc, char *argv[] )
{
    gtk_init(&argc, &argv);

    GtkWidget *window = gtk_window_new(GTK_WINDOW_TOPLEVEL);    // 主窗口
    gtk_window_set_title(GTK_WINDOW(window), "cpucode Event Box");        // 标题
    gtk_container_set_border_width(GTK_CONTAINER(window), 10);
    gtk_window_set_position(GTK_WINDOW(window), GTK_WIN_POS_CENTER); // 居中显示
    gtk_widget_set_size_request(window, 200, 50);                     // 最小大小
    g_signal_connect(window, "destroy",G_CALLBACK(gtk_main_quit), NULL ); 

    GtkWidget *eventbox = gtk_event_box_new();                // 事件盒子的创建
    gtk_widget_set_events(eventbox, GDK_BUTTON_PRESS_MASK); // 捕获鼠标点击事件
    gtk_container_add( GTK_CONTAINER(window), eventbox );    // 事件盒子放入窗口

    GtkWidget *label = gtk_label_new("Double-Click Me!");    // label
    gtk_container_add( GTK_CONTAINER(eventbox), label );    // label放入事件盒子里

    g_signal_connect(eventbox, "button_press_event", G_CALLBACK(button_pressed), (gpointer)label);

    gtk_widget_show_all(window);    // 显示所有控件

    gtk_main();

    return 0;
}

/* This is called every time a button-press event occurs on the GtkEventBox. */
static gboolean button_pressed( GtkWidget *eventbox, GdkEventButton *event, GtkLabel *label )
{
    if (event->type == GDK_2BUTTON_PRESS)
    {
        const gchar *text = gtk_label_get_text(label);

        if( text[0] == 'D' )
        {
            gtk_label_set_text(label, "I Was Double-Clicked!");
        }
        else
        {
            gtk_label_set_text(label, "Double-Click Me Again!");
        }
    }

    return FALSE;
}
```

Makefile

```text
CC = gcc  
MAINC = event_box.c
EXEC = event_box

CFLAGS = `pkg-config --cflags --libs gtk+-2.0`

main:  
    $(CC)  $(MAINC)  -o $(EXEC) $(CFLAGS)
clean:
    rm $(EXEC) -rf
```

![](https://gitee.com/cpu_code/picture_bed/raw/master//20200805165012.png)

> * @由于个人水平有限, 难免有些错误, 还请指点:  
> * @Author: cpu\_code
> * @Date: 2020-08-06 16:51:19
> * @LastEditTime: 2020-08-06 18:27:18
> * @FilePath: \notes\GTK\Event\_handling.md
> * @Gitee: [https://gitee.com/cpu\_code](https://gitee.com/cpu_code)
> * @Github: [https://github.com/CPU-Code](https://github.com/CPU-Code)
> * @CSDN: [https://blog.csdn.net/qq\_44226094](https://blog.csdn.net/qq_44226094)
> * @Gitbook: [https://923992029.gitbook.io/cpucode/](https://923992029.gitbook.io/cpucode/)

