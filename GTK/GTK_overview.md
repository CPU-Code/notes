



# Gtk+概述



## GUI  



GUI 含义： （Graphics User Interface）  

图形用户界面， 是计算机与使用者之间的对话接口， 是计算机重要的组成部分， 比如说咱们使用电脑或手机看到的 Windows 的桌面或 wps 软件显示的窗口界面等都是 GUI， 都是图形界面开发出来的图形界面的软件。  



GUI 组成   : 桌面、 视窗、 菜单、 按钮、 图标等等  



GUI 特点  :



可以使操作更加简单， 更加快捷、 更加人性化。 八十岁的老奶奶也会使用智能手机

早期的操作系统比如 DOS， 属于 CUI（Command line User Interface）

命令行模式的人机接口。  





GTK+  

GTK+是一套在 GIMP 的基础上发展而来的高级的、 可伸缩的现代化、 跨平台图形工具包， 它可以很方便地制作图形交互界面( GUI )。  



GTK+特点：  

稳定、 跨平台、 多种语言绑定、 接口丰富、 与时俱进、 算法丰富、 移动嵌入式应用广泛  



[https://www.gtk.org/](https://www.gtk.org/)







## 环境搭建  







## 空白窗口  



栗子 : first_window.c



```c
#include <gtk/gtk.h>

int main(int argc, char *argv[])
{
    // 初始化，必须在控件定义之前使用
    gtk_init(&argc, &argv&);

    /* 创建窗口(参数有两种形式)
	 * GTK_WINDOW_TOPLEVEL：顶层窗口，包含窗口的标题栏和边框，有最大化最小化
	 * GTK_WINDOW_POPUP：弹出窗口, 没有窗口的标题栏和边框，不能改变大小和移动
	 */
    GtkWidget *window = gtk_window_new(GTK_WINDOW_TOPLEVEL);

    // 显示窗口控件
    gtk_widget_show(window);	

    // 主事件循环，程序也必须要这个	
    gtk_main();

    return 0;
}
```



Makefile

```makefile
CC = gcc
MAINC = first_window.c
EXEC = first_window

CFLAGS = `pkg-config --cflags --libs gtk+-2.0`

main:
	$(CC) $(MAINC) -o $(EXEC) $(CFLAGS)

clean:
	rm $(EXEC) -rf
```





GTK 程序的基本框架  :



```c
#include <gtk/gtk.h>

int main(int argc, char *argv[])
{
    // 初始化，必须在控件定义之前使用
    gtk_init(&argc, &argv&);

    //代码

    // 主事件循环，程序也必须要这个	
    gtk_main();

    return 0;
}
```





GtkWidget 是 GTK+控件类型， GtkWidget * 能指向任何控件的指针类型。  

```c
GtkWidget *window = gtk_window_new(GTK_WINDOW_TOPLEVEL);
```



创建一个窗口并返回这个窗口的控件指针  

```c
gtk_window_new()
```





窗口的创建  :

```c
/*
GtkWindowType 是一个枚举， 有两种情况：
GTK_WINDOW_TOPLEVEL： 有边框
GTK_WINDOW_POPUP： 没边框
*/
GtkWidget *gtk_window_new(GtkWindowType type);
```



标题的设置  :

```c
void gtk_window_set_title(GtkWindow *window, const gchar *title);
```



窗口最小大小的设置  

```c
void gtk_widget_set_size_request(GtkWidget *widget,gint width,gint height);
```



窗口伸缩设置( FALSE 为不可伸缩 )  

```c
void gtk_window_set_resizable(GtkWindow *window, gboolean resizable);
```



显示或隐藏所有控件  :

```c
void gtk_widget_show_all(GtkWidget *widget);

void gtk_widget_hide_all(GtkWidget *widget);
```



窗口在显示器位置的设置  



```c
/*
position 常用有 4 种情况：
GTK_WIN_POS_NONE： 不固定
GTK_WIN_POS_CENTER: 居中
GTK_WIN_POS_MOUSE: 出现在鼠标位置
GTK_WIN_POS_CENTER_ALWAYS: 窗口总是居中
*/
void gtk_window_set_position(GtkWindow *window, GtkWindowPosition position);
```



栗子　：　Window.c  



```c
#include <gtk/gtk.h>

int main(int argc, char *argv[])
{
    //初始化
    gtk_init(&argc, &argv);

    //创建顶部窗口
    GtkWidget *Window = gtk_window_new(GTK_WINDOW_TOPLEVEL);

    //设置窗口标题
    gtk_window_set_title(GTK_WINDOW(Window), "cpucode Window");

    /* 
    GTK_WIN_POS_NONE： 不固定
    GTK_WIN_POS_CENTER: 居中
    GTK_WIN_POS_MOUSE: 出现在鼠标位置
    GTK_WIN_POS_CENTER_ALWAYS: 窗口总是居中 
    */
    gtk_window_set_position(GTK_WINDOW(Window), GTK_WIN_POS_CENTER);

    //设置窗口大小
    gtk_widget_set_size_request(Window, 400, 300);

    //设置窗口固定
    gtk_window_set_resizable(GTK_WINDOW(Window), FALSE);

    //"destroy" 和 gtk_main_quit 连接
    g_signal_connect(Window, "destroy", G_CALLBACK(gtk_main_quit), NULL);

    // 显示窗口全部控件
    gtk_widget_show_all(Window);
    //gtk_widget_hide_all(window);	// 隐藏窗口

    //主事件循环
    gtk_main();

    return 0;
}

```



 <img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200801165629.png"/>







## 带按钮的窗口



控件的介绍  :

控件是对数据和方法的封装。 控件有自己的属性和方法。 属性是指控件的特征。 方法是指控件的一些简单而可见的功能。  



控件的分类： **容器**控件， **非容器**控件。

容器控件： 它可以容纳别的控件。 容器控件分为两类， 一类只能容纳一个控件， 如窗口， 按钮； 另一类能容纳多个控件， 如布局控件。

非容器控件： 它不可以容纳别的控件， 如标签、 行编辑。  



```c
// 创建一个带内容的按钮
GtkWidget *gtk_button_new_with_label(const gchar *label );
```



```c
//获得按钮上面的文本内容
const gchar *gtk_button_get_label(GtkButton *button );
```



```c
// 把控件添加到窗口容器里
// container： 容纳控件的容器
// widget： 要添加的控件
void gtk_container_add(GtkContainer *container, GtkWidget *widget);
```



栗子 : window_with_button.c



```c
#include <gtk/gtk.h> // 头文件

int main( int argc,char *argv[] )
{
	gtk_init(&argc, &argv); // 初始化
    
	GtkWidget *window = gtk_window_new(GTK_WINDOW_TOPLEVEL); // 创建顶层窗口
    
	// 设置窗口边框的宽度(窗口里的控件与窗口边框间隔为 15)
	gtk_container_set_border_width(GTK_CONTAINER(window), 15);
    
    // 创建按钮， 文本信息
    GtkWidget *button = gtk_button_new_with_label("cpuocde");
    const char *str = gtk_button_get_label( GTK_BUTTON(button) ); // 获得按钮的内容
    
    printf("str = %s\n", str);
    
    // 把按钮放入窗口(窗口也是一种容器)
	 gtk_container_add(GTK_CONTAINER(window), button);
    
    // 显示控件有两种方法： 逐个显示， 全部显示
    // gtk_widget_show(button);
    // gtk_widget_show(window);
    
    gtk_widget_show_all(window); // 显示窗口全部控件
    
    gtk_main(); // 主事件循环
    return 0;
}
```



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200802215312.png"/>





## 信号与回调函数机制  



GTK+采用了**信号**与**回调函数**来处理窗口外部传来的事件、 消息或信号。 当信号发生时， 程序自动调用为信号连接的回调函数。  



窗口关闭时触发的常用信号： `destroy` , `delete-event`

操作按钮触发的常用信号： `clicked`, `pressed`， `released`  



```c
/*
 * 信号与回调函数的连接
 * instance： 信号的发出者
 * detailed_signal： 要连接信号的名称
 * c_handler： 回调函数的名称， 需要用 G_CALLBACK()进行转换
 * data： 传递给回调函数的参数
 */
gulong g_signal_connect(gpointer instance, const gchar *detailed_signal, GCallback c_handler, gpointer data );
```



回调函数的定义  



```c
//信号连接函数
g_signal_connect(button, "pressed", G_CALLBACK(callback), NULL);
```



```c
// 回调函数
// button： 信号的发出者
// user_data： 传给回调函数的数据
void callback( GtkButton *button, gpointer user_data );
```



栗子 : signal.c  

```c
#include <gtk/gtk.h>	// 头文件

// 按钮按下的处理函数, gpointer 相当于 void *
void deal_pressed(GtkButton *button, gpointer user_data)
{
    // 获得按钮的文本信息
	const gchar *text = gtk_button_get_label( button );

	// g_print() 相当于C语言的 printf(), gchar相当于char
	g_print("button_text = %s; user_data = %s\n", text, (gchar *)user_data);
}

int main( int argc,char *argv[] )
{
    gtk_init(&argc, &argv);		// 初始化

    // 创建顶层窗口
    GtkWidget *window = gtk_window_new(GTK_WINDOW_TOPLEVEL); 

    // 设置窗口边框的宽度(窗口里的控件与窗口边框间隔为15)
    gtk_container_set_border_width(GTK_CONTAINER(window), 15);

    /* 当窗口关闭时，窗口会触发destroy信号，
	 * 自动调用gtk_main_quit()结束程序运行。
	 */
    g_signal_connect(window, "destroy", G_CALLBACK(gtk_main_quit), NULL);

    //创建按钮
    GtkWidget *button = gtk_button_new_with_label("cpucode");

    // 把按钮放入窗口(窗口也是一种容器)
    gtk_container_add(GTK_CONTAINER(window), button);	

    /* 按钮按下(pressed)后会自动调用 deal_pressed()
	 * "is pressed"是传给deal_pressed()的数据
	 */
    g_signal_connect(button, "pressed", G_CALLBACK(deal_pressed), "is pressed");

    gtk_widget_show_all(window);	// 显示窗口全部控件

    gtk_main();		// 主事件循环

    return 0;
}
```



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200803164000.png"/>







## 常用布局  



设定控件在整个窗口中的位置和尺寸  



常用的布局方式

*   水平布局 `GtkHBox`

*   垂直布局 `GtkVBox`

*   表格布局 `GtkTable`
*   固定布局 `GtkFixed`  



### 水平布局  



```c
// 创建水平布局容器
// homogeneous： 容器内控件是否均衡排放(大小一致)
// spacing： 控件之间的间隔
GtkWidget *gtk_hbox_new(gboolean homogeneous, gint spacing);
```



```c
// 添加控件到布局容器中
gtk_container_add(GtkContainer *container, GtkWidget *widget );
```



栗子 : hbox.c 横向布局  



```c
#include <gtk/gtk.h>

int main(int argc, char *argv[])
{
    gtk_init(&argc, &argv);

    //创建窗口
    GtkWidget *window = gtk_window_new(GTK_WINDOW_TOPLEVEL);

    //设置标题
    gtk_window_set_title(GTK_WINDOW(window), "cpucode layout");

    //设置窗口边框的宽度
    gtk_container_set_border_width(GTK_CONTAINER(window), 15);

    g_signal_connect(window, "destroy", G_CALLBACK(gtk_main_quit), NULL);

    /*
     * 创建横向盒状容器
     * GtkWidget *gtk_vbox_new(gboolean homogeneous, gint spacing);
     * homogeneous：容器内控件是否均衡排放(TRUE or FALSE)
     * spacing: 控件之间的间隔
     */
    GtkWidget *hbox = gkt_hbox_new(TRUE, 10);

    //把横向容器放到窗口
    gtk_container_add(GTK_CONTAINER(window), hbox);
    //button1
    GtkWidget *button = gtk_button_new_with_label("button1");
    // 按钮添加到布局容器里
    gtk_container_add(GTK_CONTAINER(hbox), button);

    // button2
	button = gtk_button_new_with_label("button2");
    // 按钮添加到布局容器里
	gtk_container_add(GTK_CONTAINER(hbox), button);

    // button3
	button = gtk_button_new_with_label("button3");
    gtk_container_add(GTK_CONTAINER(hbox), button); 	// 按钮添加到布局容器里

    gtk_widget_show_all(window);	// 显示窗口控件

    gtk_main(); 	// 主事件循环

	return 0; 
}

```



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200803172641.png"/>





### 垂直布局  



垂直布局容器的操作和水平布局的操作几乎是一样  



```c
//创建垂直布局容器
gtk_vbox_new()
```

```c
//标签的创建
GtkWidget *gtk_label_new(const gchar *str);
```



```c
// 设置标签的内容
void gtk_label_set_text(GtkLabel *label, const gchar *str);
```



```c
//获得标签的内容
const gchar *gtk_label_get_label(GtkLabel *label );
```



栗子 : vbox.c 纵向布局  



```c
#include <gtk/gtk.h>	// 头文件

/*
 * 设置label字体的大小
 * label: 需要操作的label
 * size:  字体的大小
 */
void set_label_font_size(GtkWidget *label, int size)
{
    // 字体指针
    PangoFontDescription *font;

    //参数为字体名字
    font = pango_font_description_from_string("cpucode");

	// #define PANGO_SCALE 1024
	// 设置字体大小   
    pango_font_description_set_size(font, size * PANGO_SCALE);
    // 改变label的字体大小
    gtk_widget_modify_font(label, font);
    // 释放字体指针分配的空间
    pango_font_description_free(font);
}

int main(int argc, char *argv[])
{
    gtk_init(&argc, &argv);   // 初始化

    // 创建窗口
    GtkWidget *window = gtk_window_new(GTK_WINDOW_TOPLEVEL);
    // 设置标题 
    gtk_window_set_title(GTK_WINDOW(window), "cpucode vertical layout");

    g_signal_connect(window, "destroy", G_CALLBACK(gtk_main_quit), NULL);
     // 设置窗口边框的宽度
    gtk_container_set_border_width(GTK_CONTAINER(window), 15);
    // 创建纵向盒状容器
    GtkWidget *vbox = gtk_vbox_new(TRUE, 15);
    // 把纵向盒状容器放入窗口
    gtk_container_add(GTK_CONTAINER(window), vbox);

    //label one
    // 创建标签
    GtkWidget *label = gtk_label_new("label one");
    // 设置label字体大小，这个为自定义的函数
    set_label_font_size(label, 30);
    // 将按钮放在布局容器里
    gtk_container_add(GTK_CONTAINER(vbox), label);

    // label two
	label = gtk_label_new("text");

    gtk_label_set_text(GTK_LABEL(label), "label two");   	   // 设置label的内容
    const char *str = gtk_label_get_label( GTK_LABEL(label) ); // 获得标签的内容
    g_print("str = %s\n", str);
    
	// 设置label字体大小，这个为自定义的函数
    set_label_font_size(label, 30);
    gtk_container_add(GTK_CONTAINER(vbox), label); 		// 将按钮放在布局容器里

    // label three
	label = gtk_label_new("label three");
	set_label_font_size(label, 30);	// 设置label字体大小，这个为自定义的函数
	gtk_container_add(GTK_CONTAINER(vbox), label); 		// 将按钮放在布局容器里
	
	gtk_widget_show_all(window);	// 显示窗口控件

    // 主事件循环
    gtk_main();

    return 0;
}
```



Makefile

```makefile
CC = gcc  
MAINC = vbox.c
EXEC = vbox

CFLAGS = `pkg-config --cflags --libs gtk+-2.0`

main:  
	$(CC) $(MAINC) -o $(EXEC) $(CFLAGS)
clean:
	rm $(EXEC) -rf
```





<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200803181120.png"/>





### 表格布局  



```c
// 创建表格布局容器
// rows:行数
// coumns:列数
// homogeneous： 容器内表格的大小是否一致
GtkWidget*gtk_table_new(guint rows, guint columns, gboolean homogeneous );
```



```c
// 添加控件到布局容器中
// table: 要容纳控件的容器
// widget: 被容纳控件
// 后四个参数 : 控件摆放的坐标
void gtk_table_attach_defaults(GtkTable *table, GtkWidget *widget, 
                     guint left_attach, guint right_attach,
                     guint top_attach, guint bottom_attach );
```



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200806132259.png" alt="image-20200806132252551"  />



栗子 : table.c 表格布局  

```c
#include <gtk/gtk.h>	// 头文件

// button 1, button 2处理函数
void callback(GtkButton *button, gpointer data)
{
	g_print("%s is clicked!\n", (gchar *)data);
}

// close button处理函数
void close_window(GtkWidget *widget, GdkEvent *event, gpointer data)
{
	gtk_main_quit();
}

int main(int argc, char *argv[]) 
{
    gtk_init(&argc, &argv); 	// 初始化
    // 创建窗口
    GtkWidget *window = gtk_window_new(GTK_WINDOW_TOPLEVEL);
    // 设置标题 
    gtk_window_set_title(GTK_WINDOW(window), "cpucode table");

    g_signal_connect(window, "destroy", G_CALLBACK(gtk_main_quit), NULL);
    // 设置窗口边框的宽度
    gtk_container_set_border_width(GTK_CONTAINER(window), 15);

    /*
	 * 创建表格布局容器
	 * GtkWidget * gtk_table_new(guint rows, guint columns, gboolean homogeneous);
	 * rows:         行数
	 * coumns:       列数
	 * homogeneous： 容器内控件是否均衡排放(TRUE or FALSE)
	 */
    GtkWidget *table = gtk_table_new(2, 2, TRUE);
    gtk_container_add(GTK_CONTAINER(window), table); // 容器加入窗口

    // button 1
	GtkWidget *button = gtk_button_new_with_label("buttton 1");
    g_signal_connect(button, "pressed", G_CALLBACK(callback), "button 1");
	gtk_table_attach_defaults(GTK_TABLE(table), button, 0, 1, 0, 1);// 把按钮加入布局

    // button 2
	button = gtk_button_new_with_label("button 2");
    g_signal_connect(button, "pressed", G_CALLBACK(callback), "button 2");
    gtk_table_attach_defaults(GTK_TABLE(table), button, 1, 2, 0, 1);

    // close button
	button = gtk_button_new_with_label("close");
    g_signal_connect(button, "pressed", G_CALLBACK(close_window), NULL);
	gtk_table_attach_defaults(GTK_TABLE(table), button, 0, 2, 1, 2);

    gtk_widget_show_all(window);	// 显示窗口控件

    gtk_main(); 	// 主事件循环

    return 0;
}
```



Makefile

```makefile
CC = gcc
MAINC = table.c
EXEC = table

CFLAGS = `pkg-config --cflags --libs gtk+-2.0`

main:
	$(CC) $(MAINC) -o $(EXEC) $(CFLAGS)
clean:
	rm $(EXEC) -rf
```



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200804132656.png"/>



### 固定布局  



```c
//创建固定布局容器
GtkWidget *gtk_fixed_new(void);
```



```c
// 添加控件到容器中
// fixed： 要容纳控件的容器
// widget： 被容纳控件
// x， y： 控件摆放位置的起点坐标
void gtk_fixed_put( GtkFixed *fixed, GtkWidget *widget, gint x, gint y );
```



栗子 : fixed.c 固定布局  

```c
#include <gtk/gtk.h> 

int main( int argc, char *argv[]) 
{
    gtk_init(&argc, &argv); 	// 初始化

    GtkWidget *window = gtk_window_new (GTK_WINDOW_TOPLEVEL); 	 // 创建窗口 
    gtk_window_set_title(GTK_WINDOW(window), "cpucode"); // 设置标题

    // 为窗口的 "destroy" 事件设置一个信号处理函数  
	g_signal_connect (window, "destroy", G_CALLBACK (gtk_main_quit), NULL);

    GtkWidget *fixed = gtk_fixed_new(); 	//创建一个固定容器
    gtk_container_add(GTK_CONTAINER (window), fixed); // 固定放进窗口

    GtkWidget *button; 
	int i; 

    for(i = 1 ; i <= 3 ; i++)
    {
        button = gtk_button_new_with_label ("Press me");  // 创建按钮
        // 将按钮组装到一个固定容器的窗口中
		gtk_fixed_put(GTK_FIXED(fixed), button, i * 50, i * 50); 
    }

    gtk_widget_show_all(window);  // 显示所有控件
    
    gtk_main();  //进入事件循环 
	 
	return 0; 
}
```



Makefile

```makefile
CC = gcc
MAINC = fixed.c
EXEC = fixed

CFLAGS = `pkg-config --cflags --libs gtk+-2.0`

main:
	$(CC) $(MAINC) -o $(EXEC) $(CFLAGS)
clead:
	rm $(EXEC) -rf
```



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200804132758.png"/>





## 行编辑  



```c
//行编辑的创建
GtkWidget *gtk_entry_new(void);
```



```c
//显示模式(FALSE 为密码模式)
void gtk_entry_set_visibility(GtkEntry *entry, gboolean visible );
```



```c
// 获得文本内容
const gchar *gtk_entry_get_text(GtkEntry *entry );
```



```c
//设置行编辑的内容
void gtk_entry_set_text(GtkEntry *entry, const gchar *text);
```



```c
//常用信号：按回车键触发
activate
```





栗子 : entry.c 行编辑  

```c
#include <gtk/gtk.h> 

// 按Enter，获取行编辑的内容
void enter_callback( GtkWidget *widget, gpointer entry ) 
{ 
	const gchar *entry_text; 

	// 获得文本内容
	entry_text = gtk_entry_get_text(GTK_ENTRY(entry)); 
	printf("Entry contents: %s\n", entry_text); 
}

int main( int argc, char *argv[] )
{
    gtk_init (&argc, &argv); 	// 初始化

    GtkWidget *window = gtk_window_new(GTK_WINDOW_TOPLEVEL); 	// 创建窗口
    gtk_window_set_title(GTK_WINDOW(window), "cpucode Entry"); 		// 设置标题
    g_signal_connect(window, "destroy", G_CALLBACK(gtk_main_quit), NULL);

    GtkWidget *vbox = gtk_vbox_new(TRUE, 5); 	// 垂直布局，控件均衡摆放，控件间距为0
    gtk_container_add(GTK_CONTAINER(window), vbox); 	// 容器放入窗口

    GtkWidget *entry = gtk_entry_new();  // 创建行编辑 	
	gtk_container_add(GTK_CONTAINER(vbox), entry); 	// 加入垂直布局里
    gtk_entry_set_max_length(GTK_ENTRY(entry), 100);     // 设置行编辑显示最大字符的长度
	gtk_entry_set_text(GTK_ENTRY(entry), "hello word");  // 设置内容
	gtk_entry_set_visibility(GTK_ENTRY(entry), FALSE); 	 // 密码模式

    /* 如果我们想在用户输入文本时进行响应，可以为 activate 设置回调函数。
	 * 当用户在文本输入构件内部按回车键时引发Activate信号；
	 */
    g_signal_connect(entry, "activate", G_CALLBACK(enter_callback), entry);

    GtkWidget *hbox = gtk_hbox_new(TRUE, 0); 	  // 水平布局容器
    gtk_container_add(GTK_CONTAINER(vbox), hbox); // 水平布局容器加入垂直布局容器里

    // "button 1"
    GtkWidget *button1 = gtk_button_new_with_label("button 1");
    gtk_container_add(GTK_CONTAINER(hbox), button1); // 按钮1添加到水平布局容器里

    // "button 2"
    GtkWidget *button2 = gtk_button_new_with_label("button 2"); 
    gtk_container_add(GTK_CONTAINER(hbox), button2); // 按钮2添加到水平布局容器里

    // close button
    GtkWidget *button_close = gtk_button_new_with_label("close"); 
    g_signal_connect(button_close, "clicked", G_CALLBACK (gtk_main_quit), NULL); 
    gtk_container_add(GTK_CONTAINER(vbox), button_close); // 关闭按钮添加到垂直布局容器里

    gtk_widget_show_all(window); // 显示窗口所有控件
 
    gtk_main(); 		// 主事件循环
 
    return 0;
}
```



Makefile

```makefile
CC = gcc
MAINC = entry.c
EXEC = entry

CFLAGS = `pkg-config -cflags --libs gtk+-2.0`

main:
	$(CC) $(MAINC) -o $(EXEC) $(CFLAGS)
clean:
	rm $(EXEC) -rf
```



<img src="https://gitee.com/cpu_code/picture_bed/raw/master//20200804185425.png"/>























