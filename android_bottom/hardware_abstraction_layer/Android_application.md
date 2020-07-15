
## 开发Android应用程序来使用硬件访问服务

 * @Author: cpu_code
 * @Date: 2020-07-15 12:32:24
 * @LastEditTime: 2020-07-15 13:24:50
 * @FilePath: \notes\android_bottom\hardware_abstraction_layer\Android_application.md
 * @Gitee: https://gitee.com/cpu_code
 * @Github: https://github.com/CPU-Code
 * @CSDN: https://blog.csdn.net/qq_44226094
 * @Gitbook: https://923992029.gitbook.io/cpucode/


```shell
~/Android/packages/experimental/Freg
    # 配置文件
    AndroidManifest.xml
    Android.mk
    # 源代码目录
    src
        shy/luo/freg
            Freg.java
    # 资源目录res
    res
        layout
            main.xml
            values
                string.xml
            drawable
                icon.png
```

Freg.java :

```java
// Android\packages\experimental\Freg\src\shy\luo\freg\Freg.java

public class Freg extends Activity implements OnClickListener 
{
    private final static String LOG_TAG = "shy.luo.freg.FregActivity";

    private IFregService fregService = null;

    private EditText valueText = null;
    private Button readButton = null;
    private Button writeButton = null;
    private Button clearButton = null;

    /** Called when the activity is first created. */
    @Override
    public void onCreate(Bundle savedInstanceState) 
    {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);

       // 将这个 Binder 代理对象接口转换为一个 FregService 代理对象接口，
        //并且保存在 Activity 组件 Freg 的成员变量 fregService 中
        fregService = IFregService.Stub.asInterface(
            // 获得一个名称为“ freg ”的服务的 Binder 代理对象接口
            ServiceManager.getService("freg"));

        // 编辑框, 用来显示或者输入虚拟硬件设备freg的寄存器val的值
        valueText = (EditText)findViewById(R.id.edit_value);
        // 三个按钮Read、 Write和Clear
        // 读虚拟硬件设备freg的寄存器val
        readButton = (Button)findViewById(R.id.button_read);
        // 写虚拟硬件设备freg的寄存器val
        writeButton = (Button)findViewById(R.id.button_write);
        // 清空编辑框
        clearButton = (Button)findViewById(R.id.button_clear);

        readButton.setOnClickListener(this);
        writeButton.setOnClickListener(this);
        clearButton.setOnClickListener(this);

        Log.i(LOG_TAG, "Freg Activity Created");
    }

    @Override
    public void onClick(View v) 
    {
        if(v.equals(readButton)) 
        {
            try {
                // 获取虚拟硬件设备freg的寄存器val的值
                int val = fregService.getVal();
                String text = String.valueOf(val);
                valueText.setText(text);
            } catch (RemoteException e) {
                Log.e(LOG_TAG, "Remote Exception while reading value from freg service.");
            }        
        }
        else if(v.equals(writeButton)) 
        {
            try {
                String text = valueText.getText().toString();
                int val = Integer.parseInt(text);
                // 设置虚拟硬件设备freg的寄存器val的值
                fregService.setVal(val);
            } catch (RemoteException e) {
                Log.e(LOG_TAG, "Remote Exception while writing value to freg service.");
            }
        }
        else if(v.equals(clearButton)) 
        {
            String text = "";
            valueText.setText(text);
        }
    }
}
```



main.xml  主界面配置文件  

```xml
<-- packages\experimental\Freg\res\layout\main.xml -->
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    >
    <-- 显示一个TextView控件 -->
    <LinearLayout
    	android:layout_width="fill_parent"
    	android:layout_height="wrap_content"
    	android:orientation="vertical" 
    	android:gravity="center">
    	<TextView 
			android:layout_width="wrap_content"
       		android:layout_height="wrap_content" 
        	android:text="@string/value">
    	</TextView>
    	<EditText 
     		android:layout_width="fill_parent"
        	android:layout_height="wrap_content" 
        	android:id="@+id/edit_value"
        	android:hint="@string/hint">
    	</EditText>
    </LinearLayout>
     <LinearLayout
    	android:layout_width="fill_parent"
    	android:layout_height="wrap_content"
    	android:orientation="horizontal" 
    	android:gravity="center">
         
         <-- 三个Button控件 -->
    	<Button 
    		android:id="@+id/button_read"
    		android:layout_width="wrap_content"
    		android:layout_height="wrap_content"
    		android:text="@string/read">
    	</Button>
    	<Button 
    		android:id="@+id/button_write"
    		android:layout_width="wrap_content"
    		android:layout_height="wrap_content"
    		android:text="@string/write">
    	</Button>
    	<Button 
    		android:id="@+id/button_clear"
    		android:layout_width="wrap_content"
    		android:layout_height="wrap_content"
    		android:text="@string/clear">
    	</Button>
    </LinearLayout>
</LinearLayout>

```



strings.xml  字符串资源文件  

```xml
<-- packages\experimental\Freg\res\values\string.xml -->
<?xml version="1.0" encoding="utf-8"?>
<-- 应用程序中使用到的各个字符串 -->
<resources>
    <string name="app_name">Freg</string>
    <string name="value">Value</string>
    <string name="hint">Please input a value...</string>
    <string name="read">Read</string>
    <string name="write">Write</string>
    <string name="clear">Clear</string>
</resources>
```



icon.png   图标文件  



AndroidManifest.xml   配置文件  

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
      package="shy.luo.freg"
      android:versionCode="1"
      android:versionName="1.0">
    <application android:icon="@drawable/icon" android:label="@string/app_name">
        <-- 定义了一个Activity组件 -->
        <activity android:name=".Freg"
                  android:label="@string/app_name">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
</manifest> 
```



Android.mk  编译脚本文件

```makefile
LOCAL_PATH:= $(call my-dir)
include $(CLEAR_VARS)

LOCAL_MODULE_TAGS := optional

LOCAL_SRC_FILES := $(call all-subdir-java-files)

LOCAL_PACKAGE_NAME := Freg

include $(BUILD_PACKAGE)
```



编译和打包  

```shell
mmm ./packages/experimental/Freg/

make snod
```



启动  

```shell
emulator -kernel kernel/goldfish/arch/arm/boot/zImage
```







