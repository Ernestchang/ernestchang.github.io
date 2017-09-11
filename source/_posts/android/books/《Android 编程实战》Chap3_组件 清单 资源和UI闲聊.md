title: 《Android 编程实战》Chap3_组件 清单 资源和UI闲聊
date: 

categories: 
- android 
- 《Android 编程实战》
tags:  
- android
---


> 阅读《Android 编程实战》一书的随记笔记
> 注：本文主要参考[http://szysky.com](http://szysky.com)

## 应用程序清单

> `AndroidManifest`是一个定义各种组件以及应用程序各个方向的XML文件. 该清单是所有`Android`的核心.

### manifest元素

`AndroidManifest.xml`文件的根节点元素是`manifest`. 应用程序的**包名**和**唯一识别符**都定义在该节点. 还可以在该节点中定义`Linux用户ID`和`名称(应用程序将运行在该ID下)`. 应用的版本, APK的安装位置.

```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.test.printertestdemo"
    android:sharedUserId="com.szysky.userid"
    android:sharedUserLabel="@string/user_name"
    android:installLocation="auto"
    android:versionCode="6"
    android:versionName="2.0" >
    
</manifest>
```

这是一个比较完整的配置. `sharedUserId`和`sharedUserLabel`是Linux用户ID和名称. 应用程序将会运行在上面. 默认情况下, 这些都是系统分配的. 如果为使用证书发布的多个应用设置同样的值时, 开发者就可以访问相同的数据, 并有可能可以共享这些应用的进程(下面说道).

多个应用共享同一个进程可以节省应用程序使用的`RAM`总量. 但当其中一个应用出现运行时错误, 那所有共享相同进程的其他应用也会崩溃.

使用`installLocation`属性来控制应用程序的安装位置. 即可以在设备的内部存储器(data分区)安装应用程序, 也可以在外部存储装置(SD卡)安装. 需要注意: 只能通过这个属性控制APK的**安装位置**. 应用程序数据仍然安全地存储在内部存储器中.

### user-feature

这个属性元素`<user-feature>`可以让`Google Play`去过滤应用在哪台设备可见. 比如应用需要一个打电话的功能, 但是当前设备只支持wifi的平板电脑. 或者说一个Android电视是不支持触屏控制, 但是应用需要这个硬件特性的支持. 就可以通过这个属性设置.

例如:

```
<user-feature android:name="android.hardware.microphone">
<!--需要电话功能支持-->
<user-feature android:name="android.hardware.telephony"> 
<!--支持多点触控的触摸屏-->
<user-feature android:name="android.hardware.touchscreen.multitouch.jazzhand">
```

### application

使用自定义的`Application`必须要做的就是在清单文件中指定`<application>`中的`name`属性到自定义的application.

列举一下属性:

- `label`, `description`: 可以让用户在设置页面中看到关于应用的更多详细的描述.
- `android:backupAgent`: 这是`Google`提供的一个备份服务, 需要实现自己的**备份代理**的类,并使用此属性指向此类.
- `android:largeHeap`: 可以让系统知道需要更多的内存. 但是, 除非特别需要, 否则不要这么做. 因为这样做会浪费系统资源,而且**系统会更快的终止应用程序**.
- `android:process`: 当多应用程序共享一个用户ID, 并且`process`属性设置为一个值的时候, 可以**强制这些应用共享同一个进程**. (所以如果需要使多应用使用一个进程, 那么所有的应用必须共享同一个用户ID并同一个证书签名和相同的process)
- `android:theme`: 属相设置整个应用的主题.

### 组件元素和属性

每个标准组件(`activity`, `service`, `broadcastReceiver`, `contentProvider`)在清单文件都有自己的节点元素, `AS`创建的默认属性通常能满足大部分情况. 下面看一下其他的属相把

------

**android:enabled = "false"**

我们所定义的所有组件默认都是**可用的**. 当把enable设置为false的时候, 可以修改默认行为, 这将阻止组件接收`Intent`.

- 禁用的`Activity`不会显示在应用程序启动器中;
- 禁用的`Service`不会响应`startService()`调用;
- 禁用的`BroadcastReceiver`不会监听`BroadcastIntent`;
- 禁用的`ContentProvider`不会响应`ContentResolver`.

代码中同样可以设置:

```
PackageManager pm = getPackageManager();
		
// 启用一个activity
pm.setComponentEnabledSetting(new ComponentName(this, MainActivity.class),
		PackageManager.COMPONENT_ENABLED_STATE_ENABLED,
		PackageManager.DONT_KILL_APP);
	
// 禁用一个activity
pm.setComponentEnabledSetting(new ComponentName(this, MainActivity.class),
		PackageManager.COMPONENT_ENABLED_STATE_DISABLED,
		PackageManager.DONT_KILL_APP);
```

------

有时候出于安全的原因, 我们不希望把某个组件暴露给系统的其他应用使用, 这时可以通过`android:exported="false"`, 这会有效地把对外可被调用属性关闭.

------

如果你想设置一个组件供其他应用使用, 但又想提供某个安全级别, 则可以**定义权限**, 要做到这一点, 那么调用者应该在清单文件中添加使用权限(user-permission), 然后在提供者应用注册一个权限(permission).

### Intent过滤

在`Android`中所有的组件都是通过`Intent`访问的, `Intent`抽象地描述了要执行的动作. `Intent`会被发送给`Activity`, `Service`, `BroadcastReceiver`. 一旦发送, `Android系统`的Intent将决定将Intent分发给哪个组件.

1. `Intent`要确认的第一件事情就是分辨出**隐式**还是**显示**. `显示Intent`包含有关包和组件的名称信息, 他们可以立即分发, 因为只能匹配到一个组件. 此方法通常应用于程序内部的通信. `隐式Intent`的取决于三个因素: **action**, **数据的URI和类型**, **以及类别**. 注意: **extras和flags对Intent区分没有影响**
2. `action`是最主要的, 通常也是唯一应该重点关注的. Android有许多预定义的action, 开发者也可以定义自己的action. 开发者定义自己的action时, 通常在前面加上自己的包名, 这样不同应用之间action定义的冲突也就基本不存在了.
3. `data`并没有实际的数据, 只是包含了`URI`和`MIME`类型, 使用`Activity`打开只包含特定类型文件会非常有用.
4. `category和Activity最相关`. 所有通过`startActivity()`发送的隐式Intent都至少定义了一个`category(android.intent.category,DEFAULT)`, 所以除非Activity的`Intent-filter`也包含这个`category`. 否则就不会通过`Intent`的匹配. 比较例外的是图标启动器的`intent-filter`

```
<activity
  android:name=".PrinterTestDemoAct"
  android:label="@string/app_name">
  <intent-filter>
      <action android:name="android.intent.action.MAIN" />
      <category android:name="android.intent.category.LAUNCHER" />
  </intent-filter>
  
  <intent-filter>
      <action android:name="android.intent.action.VIEW"/>
      <category android:name="android.intent.category.DEFAULT"/>
      <data android:mimeType="video/*"/>
  </intent-filter>
</activity>
```

如上清单配置: 有两个`intent-filter`, 第一个是用于桌面图标启动应用; 第二个是用于打开video开头的`MIME`类型文件.

第一个`intent-filter`不需要`category.DEFAULT`. 因为它将由启动器程序管理.
第二个`intent-filter`是我们经常使用`startActivity()`方法会匹配到的, 所以其中的`category`应该设置`default`, 无论被打开视频文件是本地还是网络, 只要`MIME`类型以`Video`开头就行.

## resources 和 assets

开发中除了实现业务逻辑的代码, 还有就是一些像`XML布局`,`图标`,`文本值`等这样的内容, 这些静态内容存储在APK文件的`resources`或者`assets`目录中. 所有的组员都属于一个特定的类型(比如layout, drawable等), 而`assets`是简单地存储在应用内的通用文件. 大多数非代码的内容都以资源的形式呈现, 因为他们紧密集成在Android API中.

从开发的角度来看, Android中的资源特性是平台最强大的功能之一, 它允许开发者根据**屏幕尺寸**, **定位参数**, **设备能力**等来提供多个版本的资源. 最常见的解决方案是通过在不同的文件中提供多个版本的文本字符串来支持多语言, 或者通过提供不同的布局文件和不同大小的图标来支持各种屏幕的尺寸.

------

**在应用中定义资源要遵循的几个规则**

1. 一定要在应用中为每种类型的资源都提供一个默认值, 就是在没有任何限定符的资源目录中提供默认的图片, 图标, 文本字符串和布局. 确保当所有特定资源文件都无法匹配设备, 可以加载默认的资源而不会出现崩溃.
2. 避免硬编码, 例如字符串和值. 虽然可能会需要做一些额外工作也不算太累, 相比与之后当需要改变整个应用中的所有字体, 外边距, 内边距, 增加字符串本地化的时候. 修改起来这简直就是轻而易举的简单.
3. 通过提供**限定符**来过滤资源. 例如, 如果想为高密度像素的设备提供一套高分辨率的图标, 可以通过`drawable-xxxhdpi`目录内存储

------

### 高级String资源

通常, 应用中的所有字符串都应该放在`string`资源中. 这样比在java代码中相比, 可以消除一下问题, 特别是当涉及格式化字符串和本地化时.

比如: 在`string.xml`声明

```
<string name="personal_message">Hi %s !</string>

<string-array name="default_categories">
   <item>Work!</item>
   <item>personal</item>
</string-array>
```

在java中使用

```
String oneStr = getString(R.string.personal_message, "suzeyu");
		String[] stringArray = getResources().getStringArray(R.array.default_categories);
```

第一个在获取string资源的时候, 传入了一个字符串参数, 并在`%s`处进行插入处理.

### 本地化

要做到本地化的支持, 把默认字符串资源复制到一个适当语言和地区限定符的新目录中, 然后开始翻译文本.

如果要翻译一种语言的资源文件到另一种语言, 你可以通过使用**谷歌翻译工具Google Translator ToolKit**, 这是谷歌的一个服务, 允许上传一个文件, 然后将字符串自动翻译为指定的语言.

### 使用assets

如果有一些较大的资源文件可以考虑放到`assets`和`raw`目录下.

考虑这样一种场景, 如果有一个游戏有许多的音效, 那么就可以在`assets`文件夹内创建一个`soundfx`目录, 接下来把所有的音频文件都放入此子目录中. (`assets`目录支持子文件夹)

如下代码显示了如何使用`SoundPool`和`AssetManager`来加载`assets/soundfx`目录内的所有音频文件. 并且最后关闭打开的文件, 防止内存泄漏.

```
public HashMap<String, Integer> loadSoundEffects(SoundPool soundPool) throws IOException{
		// 获取assets资源管理者
		AssetManager assets = getAssets();
		String[] soundEffectFiles = assets.list("soundfx");
		HashMap<String, Integer> soundEffectMap = new HashMap<String, Integer>();

		for (String soundEffectFile: soundEffectFiles) {
			AssetFileDescriptor fileDescriptor = assets.openFd("soundfx/" + soundEffectFile);
			int id = soundPool.load(fileDescriptor, 1);
			soundEffectMap.put(soundEffectFile, id);
			
			fileDescriptor.close();
		}
		return soundEffectMap;
	}
```

## UI的一些小知识

### 文本

**字体:**人脑利用图像识别来分辨不同的字母汉字, 所以逻辑上来说, 应该使用很容易辨认的字体, 而高度装饰字体只适用于标识(logo)等其他场合.

------

**文本布局:**例如报纸中的布局分成若干个专栏, 人们更偏向于稍微适中, 比较短的文本阅读. 所以切记不要让你的文本布局显示一行很长很长的文字. 尽量可以保持一行在`45~72`个字符.

------

**android中的大小单位**

- `dp` (density-independent) 控制控件大小
- `sp` 基于dp产生, 主要用于文字大小
- `pt` (points, 磅)
- `px` (pixel, 像素)
- `mm` (millimeter, 毫米)
- `in` (inch, 英寸)

------

**推荐尺寸:**一个按钮一个比较好的大小值为`48dp`转换之后相当于9毫米, 这个尺寸可以让用户手指在触摸屏上轻松的交互.

------

**图标大小:**针对之前说的`48dp`,当不同分辨密度下相对应的为 48像素(MDPI), 72像素(HDPI), 96像素(XHDPI), 以及144像素(XXHDPI). 这就是所谓的2:3:4:6. 对应的DPI分别指~160, ~240, ~320, ~480.

如果将位图作为图标使用, 那么就应该使用dp而不是像素为单位. 一个正常的图标, 如主屏幕上的启动图标, 应该为`48dp*48dp`, 而操作栏(action bar)的图标为`32dp*32dp`, 通知栏的图标`24dp*24dp`

------

**字体的大小:**谷歌创建的`Roboto`字体被用于所有标准的Android程序. 不同的字体的尺寸即使一样, 实际看起来可能也会不同, 主要是因为他们的`x-height`不同, `x-height`即为当前字体小写字母x的高度. 经常使用的`x-height`较大的字体有`Arial`和`Verdana`

谷歌为Android字体定义了四个标准大小:

- `micro` 极小, 12sp
- `small` 小, 14sp
- `medium` 中, 18sp
- `large` 大, 22sp

------

**典型透视:**当我们需要凭借记忆画一个物体时, 大多数人会从一个略高于它的角度来描述它, 这就叫做**典型透视**.

**人脸识别:**人类擅长人脸识别, 许多人可以轻松地从一堆人中挑选出自己的朋友. 实际上相对于识别物体, 我们能更快识别人脸. 此外, 我们对文字的理解远远慢于识别人脸.

------

不要让用户思考! 用户不应该花大量时间来研究如何使用应用程序. 开发者应该让第一次看到应用程序UI的用户无需置疑下一步该如何操作;

**视觉线索**是指一个物体向用户发出的如何与之交互的信息. 人们对文字的理解慢于对简单图标的理解.