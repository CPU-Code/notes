# Linux运行时格式

> \r 错误

![image-20200729191352647](https://gitee.com/cpu_code/picture_bed/raw/master//20200729191352.png)

用`vim`打开那个执行错误的 sh脚本文件

![image-20200729190905345](https://gitee.com/cpu_code/picture_bed/raw/master//20200729190912.png)

进入最后一行模式下

```bash
:set ff
```

![image-20200729191126646](https://gitee.com/cpu_code/picture_bed/raw/master//20200729191126.png)

显示 fileformat=dos

![image-20200729191327335](https://gitee.com/cpu_code/picture_bed/raw/master//20200729191327.png)

解决方法 :

```bash
:set ff=unix
```

![image-20200729191417383](https://gitee.com/cpu_code/picture_bed/raw/master//20200729191417.png)

查看是否更改 :

```bash
:set ff
```

![image-20200729191432619](https://gitee.com/cpu_code/picture_bed/raw/master//20200729191432.png)

结果 :

![image-20200729191444906](https://gitee.com/cpu_code/picture_bed/raw/master//20200729191444.png)

保存退出即可

```bash
:x
```

![image-20200729191504339](https://gitee.com/cpu_code/picture_bed/raw/master//20200729191504.png)

运行, 没有出错

![image-20200729191526420](https://gitee.com/cpu_code/picture_bed/raw/master//20200729191526.png)

> \* @Author: cpu\_code
>
> \* @Date: 2020-07-29 19:07:52
>
> \* @LastEditTime: 2020-07-29 19:20:17
>
> \* @FilePath: \notes\ubuntu\Format\_problem.md
>
> \* @Gitee: [https://gitee.com/cpu\_code](https://gitee.com/cpu_code)
>
> \* @Github: [https://github.com/CPU-Code](https://github.com/CPU-Code)
>
> \* @CSDN: [https://blog.csdn.net/qq\_44226094](https://blog.csdn.net/qq_44226094)
>
> \* @Gitbook: [https://923992029.gitbook.io/cpucode/](https://923992029.gitbook.io/cpucode/)

