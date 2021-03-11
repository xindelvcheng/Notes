# Android教程

[TOC]



注意：本教程主要来自谷歌官方文档，有些是总结，如果谷歌中文文档该部分非常简明，则可能直接复制过来。

注意：本教程非零起点，而是注重原理与实践，所以需要扎实的JavaSE、操作系统、GUI编程基础。

## 一.基础知识

安卓概念从大到小依次为：

Android系统

→Java虚拟机

→App

→组件：Activity、Fragment、Service、内容提供程序和广播接收器→Fragment（子组件）

→组件生命周期。

编写安卓应用，就是处理App之下的四层。需要注意的是，组件与组件构成的App并不是一个严丝合缝的整体（如普通的桌面应用那样），而是相对独立、系统可以选择性地唤起需要的部分的，类比一种很少见的桌面应用（如Mac下的QtCreator，打开之后它不是一个整体而是分散成很多个小窗口，包括界面设计器、详情菜单等等，你可以选择性地打开自己需要的窗口。

#### 1.应用架构

谷歌文档中给出安卓应用的建议架构可分为两层：

（1）数据层：此层负责与数据库和文件交互，并向Service层提供`ViewModel`。

（2）表示层：此层负责使用Activity或Fragment展示数据。

虽然设计的架构不一定要使用谷歌建议的这个，但至少**数据显示与数据获取不应放在同一个线程**，否则1.当OS销毁应用时数据也会丢失；2.当网络不佳时，整个应用会阻塞。



![img](assets/final-architecture.png)

#### 2.打包发布

如下图所示，典型 Android 应用模块的构建流程通常按照以下步骤执行：

1. 编译器将您的源代码转换成 DEX 文件（Dalvik 可执行文件，其中包括在 Android 设备上运行的字节码），并将其他所有内容转换成编译后的资源。
2. APK 打包器将 DEX 文件和编译后的资源合并到一个 APK 中。不过，在将应用安装并部署到 Android 设备之前，必须先为 APK 签名。
3. APK 打包器使用调试或发布密钥库为 APK 签名：
   1. 如果您构建的是调试版应用（即专用于测试和分析的应用），则打包器会使用调试密钥库为应用签名。Android Studio 会自动使用调试密钥库配置新项目。
   2. 如果您构建的是打算对外发布的发布版应用，则打包器会使用发布密钥库为应用签名。如需创建发布密钥库，请参阅[在 Android Studio 中为应用签名](https://developer.android.google.cn/studio/publish/app-signing?hl=zh_cn#studio)。
4. 在生成最终 APK 之前，打包器会使用 [zipalign](https://developer.android.google.cn/studio/command-line/zipalign?hl=zh_cn) 工具对应用进行优化，以减少其在设备上运行时所占用的内存。

构建流程结束时，您将获得应用的调试版 APK 或发布版 APK，以用于部署、测试或发布给外部用户。

![img](assets/build-process_2x.png)

#### 3.组件

应用组件是 Android 应用的基本构建块，是我们主要编写代码的地方。每个组件都是一个入口点，系统或用户可通过该入口点进入您的应用。有些组件会依赖于其他组件。

共有四种不同的应用组件类型：

- Activity：页面
- 服务：后台应用
- 广播接收器：事件处理
- 内容提供程序：系统数据库

这四种组件都是Android提供的类，当继承了其中一个，就是那种组件了。主要的业务逻辑就在这些组件的生命周期回调函数中。

#### 4.清单文件

AndroidManifest.xml，是应用交给Android系统的花名册，大部分的组件都需要在其中注册后才能使用。

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest ... >
    <application android:icon="@drawable/app_icon.png" ... >
        <activity android:name="com.example.project.ExampleActivity"
                  android:label="@string/example_label" ... >
        </activity>
        ...
    </application>
</manifest>
```

其中四种组件的名称为activity、service、receiver、provider

#### 5.Intent

An intent is an abstract description of an operation to be performed.

Intent是上面提到的四大组件之间沟通的信使。它相对全局变量的特别之处在于它是Android系统提供的、可以跨进程、跨应用通信——最常用的方式是跳转到一个Activity。

Intent分显式和隐式，取决于你知不知道或者关不关心用户到底用什么实现某个操作（如到底是分享到qq还是分享到微信）。

一般而言：拉起本应用的页面使用显式Intent，拉起别的应用的页面使用隐式Intent

##### （1）显式Intent

```java
Intent intent = new Intent(this, EmptyActivity2.class);
EditText editText = (EditText) findViewById(R.id.editText);
String message = editText.getText().toString();
intent.putExtra("Test", message);
startActivity(intent);
```

##### （2）隐式Intent

使用隐式 Intent 时，Android 系统通过将 Intent 的内容与在设备上其他应用的[清单文件](https://developer.android.google.cn/guide/topics/manifest/manifest-intro?hl=zh_cn)中声明的 IntentFilter进行比较，从而找到要启动的相应组件。

①AndroidManifest.xml

```
<manifest ... >
    ...
    <application ... >
        <activity android:name="com.example.project.ComposeEmailActivity">
            <intent-filter>
                <action android:name="android.intent.action.SEND" />
                <data android:type="*/*" />
                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
        </activity>
    </application>
</manifest>
```

②MainActivity.java

```
// Create the text message with a string
Intent sendIntent = new Intent();
sendIntent.setAction(Intent.ACTION_SEND);
sendIntent.putExtra(Intent.EXTRA_TEXT, textMessage);
sendIntent.setType("text/plain");

// Verify that the intent will resolve to an activity
if (sendIntent.resolveActivity(getPackageManager()) != null) {
    startActivity(sendIntent);
}
```

#### 6.权限

每款 Android 应用都在访问受限的沙盒中运行。如果应用需要使用其自己的沙盒外的资源或信息，则必须请求相应权限。

对于所有 Android 版本，要声明应用需要某项权限，请在应用清单中**添加 \<uses-permission> 元素**，作为顶级元素\<manifest>的子项。

如果应用需要一项危险权限，那么**每次**执行需要该权限的操作时，您都必须检查自己是否具有该权限。

```java
if (ContextCompat.checkSelfPermission(thisActivity, Manifest.permission.WRITE_CALENDAR)
    != PackageManager.PERMISSION_GRANTED) {
    // Permission is not granted
}
```

（1）如果您的应用需要相机功能，并使用 Android 2.1（[API 级别](https://developer.android.google.cn/guide/topics/manifest/uses-sdk-element?hl=zh_cn#ApiLevels) 7）中引入的 API，您必须在清单文件中声明以下要求，如下方示例所示：

```xml
<manifest ... >
    <uses-feature android:name="android.hardware.camera.any"
                  android:required="true" />
    <uses-sdk android:minSdkVersion="7" android:targetSdkVersion="19" />
    ...
</manifest>
```

（2）

```xml
    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
            package="com.example.snazzyapp">

        <uses-permission android:name="android.permission.INTERNET"/>
        <!-- other permissions go here -->

        <application ...>
            ...
        </application>
    </manifest>
    
```

#### 7.资源

文字（不要硬编码而应该单独放一个文件中用id索引）、图标、图片、音视频等不需要进行编译的部分，都在res目录下。

例如，如果您的应用包含名为 `logo.png` 的图像文件（保存在 `res/drawable/` 目录中）

## 二.四大组件

一代版本一代标准，代代版本都有四大组件。

四大组件的使用方法均为扩展 Android提供的对应的类，然后重写关键生命周期方法以插入应用逻辑。

#### 1.Activity

类似Windows中的窗口。编写安卓应用首先就是编写Activity的生命周期中的各个回调函数。

![img](assets/activity_lifecycle.png)

#### 1.1.Fragment

类似Windows窗口中的Tab标签页。它必须依附于Activity。

![img](assets/fragment_lifecycle.png)

之所以把Fragment拿出来一个标题单独讲，是因为它非常常用。但并不是说它和Activity是平级的，实际上它更像是一个控件——和`Button`、`EditText`一样。源码如下：

```java
public class Fragment implements ComponentCallbacks, OnCreateContextMenuListener, LifecycleOwner,
        ViewModelStoreOwner 
```

并非继承View，当然也没有`setContentView`和`findViewById`方法，要想显示内容必须获得一个使用inflate得到一个View返回回去。

inflate是一个高层API，可能叫`createView`更符合直觉，但那是一个通过名称创建View的更底层的API。

```java
import android.os.Bundle;
import android.support.v4.app.Fragment;
import android.view.LayoutInflater;
import android.view.ViewGroup;

public class ArticleFragment extends Fragment {
    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate the layout for this fragment
        return inflater.inflate(R.layout.article_view, container, false);
    }
}
    
```