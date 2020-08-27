<!--
 * @Author: cpu_code
 * @Date: 2020-06-25 20:45:48
 * @LastEditTime: 2020-06-27 23:02:59
 * @FilePath: \md\音视频\audio音频\linux音视频框架.md
 * @Gitee: https://gitee.com/cpu_code
 * @CSDN: https://blog.csdn.net/qq_44226094
--> 
 * @Author: cpu_code
 * @Date: 2020-06-25 20:45:48
 * @LastEditTime: 2020-06-27 23:02:59
 * @FilePath: \md\音视频\audio音频\linux音视频框架.md
 * @Gitee: https://gitee.com/cpu_code
 * @CSDN: https://blog.csdn.net/qq_44226094





<img src="https://s1.ax1x.com/2020/06/18/Nnpxmj.png" alt="Nnpxmj.png" title="Nnpxmj.png" />

# 方式不过W



![image-20200625204833407](https://gitee.com/cpu_code/picture_bed/raw/master//20200625204833.png)



![image-20200625204912573](https://gitee.com/cpu_code/picture_bed/raw/master//20200625204912.png)



1、信息接口( /proc/asound);

2 , 控制接口( /dev/snd/ controlCX);

3、混音器接口(/dev/snd/ mixerCXDX)

4、PCM接口(/dev/snd/ pcmCXDX);

5、迷笛接口( raw MIDI ) ( /dev/snd/midiCXDX)

6、音序器接口(/dev/snd/seq);

7、定时器接口( /dev/snd/timer);

和OSS类似上述接口也是以文件的形式存在的,  但这些接口是提供给 alsa-lib 库使用的。



## PCM接口

该接口由 时钟脉冲( BCLK )、帧同步信号( FS ) 及 接收数据( DR ) 和 发送数据( DX ) 组成

在 FS 信号的上升沿,  数据传输从 MSB( Most Significant Bit ) 开始,  FS频率 = 采样率

FS 信号之后开始数据字的传输,  单个的数据位按顺序进行传输,  一个时钟周期传输一个数据字

发送 MSB 时, 信号的等级首先降到最低 ,  以避兔在不同终端的接口使用不的数据方案时造成 MSB 的丢失

## IIS接口

IIS ( Inter - IC Sound ) 由飞利浦公司开发,  是一种常用的音频设备接口,  主要用CD、MD、MP3等设备

S3C2440一共有5个引脚用于IIS: IISDO、IISDI、 IISSCLK、 IISLRCK 和 CDCLK。

IISDO用于数字音频信号的输出 

IISDI用于数字音频信号的输入

另外三个引脚都与音频信号的频率有关 , 可见要用好IIS, 就要把信号频率设置正确。 

IISSCLK为串行时钟  ,  每一个时钟信号传送一位音频信号 ,  因此 IISSCLK的频率 = 声道数 x 采样频率 x 采样位数

如 采样频率 fs 为44.1kHz ,  采样的位数为16位 , 声道数2个 ( 左、右两个声道 ) , 则IISSCLK的频率 = 32 * fs = 1411.2 kHz

## AC'97 ( Audio Codec1997 )

以 Intel为首的五个PC厂商Intel , Creativeabs、NS、 Analog Device与 Yamaha共同提出的规格标准。与 PCM 和 IIS 不同 ,  AC'97 不只是一种数据格式  ,  用于音频编码的内部架构规格  ,  它还具有控制功能。

AC'97采用 AC-Link 与外部的编解码器相连  ,  AC-Link接口包括 **位时钟**( BITCLK)、**同步信号校正**( SYNC ) 和 从编码到处理器 及 从处理器中解码( SDATDIN 与 SDATAOUT ) 的数据队列( AC97_BITCLK / AC_SYNC / AC97_SDI / AC97_SDO / AC97_RSTn )

OSS( 开放声音系统 )是 linux 下的商业音频驱动 , 因其未开放源代码 , 已经逐步走向淘汰行列 . 在2.6内核之后OSS已经被取消 ,  出现了新的内核音频设备驱动 --- **ALSA** ( Advanced Linux Sound Architecture ) ,  该类驱动向下兼容 , 支持OSS . DSS支持2个基本音频设备: mixer( 混音器 ) 和 dsp ( 数字信号处理器 )