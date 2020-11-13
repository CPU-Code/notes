# 使用vscode

## 下载并介绍软件

进入 ubantu : **条件 ubantu有网络, 换源成功,**

点击 ubantu software

![image-20200728163409818](https://gitee.com/cpu_code/picture_bed/raw/master//20200728163416.png)

进入后就可以看见这个

![image-20200728162716877](https://gitee.com/cpu_code/picture_bed/raw/master//20200728162724.png)

点击搜索图标 输入`code` 就可以查到了

![image-20200728163040787](https://gitee.com/cpu_code/picture_bed/raw/master//20200728163040.png)

进入到 `vs code`中 点击插件的图标, 下载`c/c++`的插件, `chinese`语言插件

![image-20200728170435307](https://gitee.com/cpu_code/picture_bed/raw/master//20200728170435.png)

点击终端就可以会弹出 `bash` 的终端

![image-20200728170622489](https://gitee.com/cpu_code/picture_bed/raw/master//20200728170622.png)

就可以使用Linux一下操作命令了

```bash
ls     #显示 

cd     #进入目录

mkdir     #插件目录
```

![image-20200728170755461](https://gitee.com/cpu_code/picture_bed/raw/master//20200728170755.png)

点击文件 , 打开文件夹 , 找到自己的目录

![image-20200728170844480](https://gitee.com/cpu_code/picture_bed/raw/master//20200728170844.png)

我拿test目录做实验 然后点击 OK

![image-20200728171105468](https://gitee.com/cpu_code/picture_bed/raw/master//20200728171105.png)

## 从一个简单的 C/C++ 示例开始  :

代码文件浏览区中通过右键 新建文件 \( New File \) 创建一个名为 `cpucode.c` 的 C 语言文件。

![image-20200728171155236](https://gitee.com/cpu_code/picture_bed/raw/master//20200728171155.png)

![image-20200728171442429](https://gitee.com/cpu_code/picture_bed/raw/master//20200728171442.png)

然后，在编辑区域键入以下 C 代码，按 快捷键 `Ctrl+s`进行保存

```c
#include<stdio.h>

int main()
{
    printf("cpucdoe\n");

    return 0;
}
```

看见有冒点, 说明没有保存 ，按 快捷键 `Ctrl+s`进行保存

![image-20200728171932959](https://gitee.com/cpu_code/picture_bed/raw/master//20200728171933.png)

执行上方的 C 语言代码，需要先编译代码。编译代码需要用到 Linux 终端，接下来在终端中输入以下命令。

```bash
ls    # 看见c文件

ls -l # 文件的详细信息
```

![image-20200728171706359](https://gitee.com/cpu_code/picture_bed/raw/master//20200728171706.png)

```bash
gcc -o cpucode cpucode.c    # 编译成可执行文件

./cpucode    # 执行文件 看见输出结果
```

**注意参数**是小写字母 o，不是数字 0。

单击回车，这时目录下会生成了一个名为 `cpucode` 的文件，这是 C 语言程序编译后得到的可执行程序。

![image-20200728172037842](https://gitee.com/cpu_code/picture_bed/raw/master//20200728172037.png)

## 前端示例

首先，在代码文件浏览区中通过右键 新建文件 \( New File \) 创建一个名为 `cpucode.html` 的 HTML 文件，然后在编辑区域输入以下 HTML 代码：

![image-20200728172626711](https://gitee.com/cpu_code/picture_bed/raw/master//20200728172626.png)

然后，我们键入以下 HTML 内容：

```markup
<!DOCTYPE html>
<html>
    <head>
        <title>欢迎来到 HTML 的世界</title>
    </head>
    <body>
        <p>cpucode 示例教学.</p>
    </body>
</html>
```

代码会自动保存。此时，如果希望预览刚刚编写的 HTML 页面效果，可以单击编辑器页面右上角的**预览图标**打开。

除了预览按钮，你还可以：**选择代码文件** → **右键** → **Open With** → **Preview** 打开预览：

你可以看到，右侧的 HTML 页面已经可以展示出来了。如果后续编辑和修改代码，预览页面也会动态更新。

## Java 示例

除了前端开发， WebIDE 界面也非常适合进行 Java Web 开发。在 WebIDE 中，Java 主要使用命令行和 maven 来开发项目。

首先，我们在 WebIDE 提供的终端中输入命令，使用 Maven 建立项目：

```text
mvn archetype:generate -DgroupId=com.shiyanlou -DartifactId=demo -DarchetypeArtifactId=maven-archetype-webapp copy
```

如果你的终端被不慎关闭了，可以点击顶部菜单栏 Terminal → New Terminal 打开终端。

稍等片刻，就会创建好一个名为 demo 的 Maven 项目。创建过程中，可能需要单击回车进行步骤确认。

接下来，我们尝试启动 Maven 的示例项目。在 Java Web 开发过程中，需要运行 Web 服务进行调试，这个时候就需要 Jetty 或者 Tomcat。所以，需要先修改刚刚新建的项目配置，添加 Jetty maven 插件。

你需要打开 `demo` 文件夹下方的 `pom.xml` 配置文件，并使用下方配置替换默认内容，复制并粘贴即可。

```markup
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.shiyanlou</groupId>
  <artifactId>demo</artifactId>
  <packaging>war</packaging>
  <version>1.0-SNAPSHOT</version>
  <name>demo Maven Webapp</name>
  <url>http://maven.apache.org</url>
    <build>
        <plugins>
            <plugin>
                <groupId>org.eclipse.jetty</groupId>
                <artifactId>jetty-maven-plugin</artifactId>
                <version>9.4.12.v20180830</version>
                <configuration>
                    <scanIntervalSeconds>10</scanIntervalSeconds>
                    <webApp>
                        <contextPath>/</contextPath>
                    </webApp>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

上方的配置中，我们新增了 `jetty-maven-plugin` 插件支持。接下来，继续在终端操作：

```bash
cd demo
mvn jetty:run
```

上方的命令中，`cd demo` 是切换路径到 `demo` 文件夹中，`mvn jetty:run` 则是启动 Web 服务。由于需要下载的依赖较多，执行后需要稍等一会。项目启动完成之后，你可以看到服务运行在公网 `8080` 端口。

此时，我们就可以通过蓝桥线上环境提供的 Web 服务，打开运行在 `8080` 端口的 Maven 示例项目。

在环境右侧工具栏中找到「Web 服务」按钮，然后单击打开。

线上环境会自动分配一个临时域名给当前的 Web 服务，你可以在浏览器新的标签页面看到预览效果。

看到 Hello，World 默认页面，意味着刚刚 Maven 项目运行正常。

**特别提醒**：为了保证环境运行安全，线上环境提供的 Web 服务仅监听 `8080` 端口。Maven 项目默认运行在 `8080` 端口，其他一些服务运行时则可能需要手动指定端口号。

