title: 《Android 编程实战》Chap10_隐藏的Android API
date: 

categories: 
- android 
- 《Android 编程实战》
tags:  
- android
---
> 阅读《Android 编程实战》一书的随记笔记
> 注：本文主要参考[http://szysky.com](http://szysky.com)

## 官方API和隐藏API

SDK文档中的所有类, 接口, 方法以及常量都属于官方API. 虽然这些API通常能满足大多数应用的需求, 但开发者有时候需要访问更多的东西, 但却不知道如何在官方API中找到它们.

`Android SDK`中包含了一个`JAR`文件(android.jar), 在编译代码的时候会引用它, 该文件位于`<sdk root>/platforms/android-<API Level>/目录`. 不过这里面全是空类, 方法中所有的代码都被移除了, 只声明了`public`和`protected`的类. 构建Android平台时, SDK会包含该JAR文件.

通过检查每一个源文件, 并移除所有被`@hide`注解的域(常量), 方法和类, 在构建SDK时会生成方法体为空的`android.jar`文件. 这意味着仍然可以在运行的设备上方法这些符号, 但是在编译时却找不到.

Android会自动隐藏某些API, 而不需要使用`@hide`注解. 这些API位于`com.android.internal`包中, 不属于`android.jar`文件, 但却包含大量供android平台使用的内部代码. android系统应用还包含一些其他隐藏API, 这些API通常提供没有包含在官方SDK中的系统`ContentProvider`信息.

## 发现隐藏API

寻找API最简单的方法是在`Android`源码中搜索他们. 但是`Android`源码非常多, 还好有几个在线网站已经对这些代码进行了索引, 并提供了搜索功能. [AndroidXRef](http://androidxref.com/)就是其中的一个.

大部分隐藏的API都位于`frameworks`项目, 所有`android`包中的API都可以在frameworks项目中找到, 该项目还包含大部分`com.android.internal`包中的API.

## 安全地调用隐藏API

对于需要编译时链接的API, 也就是接口, 类, 或者方法, 开发者有两个选择. 第一种修改SDK的JAR文件, 使之包含所有需要的类和接口, 并使用该SDK来编译应用程序. 另一种解决方案是使用`Java`反射API来动态查找要调用的类和方法. 两种方法都可以利弊.

### 从设备中提取隐藏API

要做到编译时链接隐藏API, 开发者首先要提取和处理设备中的库文件. 即可以从模拟器提取库文件, 也可以从设备中提取这些文件, 因为它们只是用来编译代码. 由于这个过程需要提取出大量文件, 建议单独创建一个空的工作目录. 另外可能要提取多个版本的库文件, 所以开发者还应为每个API级别创建一个工作目录.

![img](http://szysky.com/2016/10/04/%E3%80%8AAndroid-%E7%BC%96%E7%A8%8B%E5%AE%9E%E6%88%98%E3%80%8B10-%E9%9A%90%E8%97%8F%E7%9A%84Android-API/devicespull.png)

这里会把设备的`/system/framework`目录全部拉取出来, 这些文件都是Android设备上基于Java的系统库, 他们是由`Dalvik`虚拟机加载的Dex优化文件. 下一步决定哪些文件包含隐藏API, 以便把它们转成可以在编译时使用的Java类文件. 大部分隐藏API都位于`framework.odex`文件, `bouncycastle.odex`文件包含了加密的库.

> 从Android 4.2开始, 原来位于`framework.odex`的几个隐藏API都放在了其他文件中. 例如`Telephony`类现在是可选的了(因为并不是所有的Android设备都支持电话), 可以在`telephony-common.odex`文件中找到它.

一旦知道需要转换的文件, 就可以下载`Smali`工具, 它能把优化后的Dex文件(.odex)转换为中间格式(.smali). 接下来使用`dex2Jar`工具再把这种中间格式转换回Java类文件.

------

**修改SDK的错误处理**

当使用前面介绍的隐藏API方法时, 很难确定抽取类的方法签名是否和用户设备中相应的方法签名匹配. 虽然修改后的SDK可能在开发用的设备上正常工作, 但是用户的设备制造商可能修改了这些隐藏API. 当这种情况发生时, 应用程序会抛出`NoSuchMethodException`或者`ClassNotFoundException`异常.

有几种方法可以处理种种情况. 可以结合使用反射来检测是否存在隐藏API. 推荐使用这种方式, 因为它结合了两种方法的优点. 另一种方法是简单地捕获异常, 防止应用程序崩溃.

不管使用哪一种方法, 都是要确保调用隐藏API的时候发生错误的处理. 最起码可以确保应用程序在测试过的设备上能正常工作.

### 使用反射调用隐藏API

使用Java中的反射API比修改`Android SDK`更安全, 因为它可以在调用`隐藏API`前检测它们是否存在. 但是, 由于所有隐藏API的绑定和调用都发生在运行时, 反射会比前面介绍的方法更慢.

使用反射调用隐藏API需要两步. 首先, 需要查找要调用的类和方法, 并把他们的引用存到`Method`对象中. 当持有了引用后, 接下来就可以调用对象的方法.

后面会演示了查找Wi-Fi网络共享的例子.

## 隐藏API的使用

### 接收和阅读SMS

`Android`中使用隐藏API最常见的例子是接收和阅读`SMS`, 虽然官方API包含了`RECEIVE_SMS`和`READ_SMS`这两个权限, 但实际执行的API却是隐藏的.

应用程序要想接收`SMS`必须声明使用`RECEIVE_SMS`权限, 并且实现`BroadcastReceiver`, 已处理收到的短信.

如下:

```
// 清单文件
<uses-permission android:name="android.permission.RECEIVE_SMS"/>

<receiver android:name=".hideapi.SmsReceiver">
  <intent-filter>
      <action android:name="android.provider.Telephony.SMS_RECEIVED"/>
  </intent-filter>
</receiver>
```

```
public class SmsReceiver extends BroadcastReceiver {
    // Telephony.java 中隐藏的常量
    public static final String SMS_RECEIVED_ACTION
            = "android.provider.Telephony.SMS_RECEIVED";

    public static final String MESSAGE_SERVICE_NUMBER = "+461234567890";
    private static final String MESSAGE_SERVICE_PREFIX = "MYSERVICE";

    public void onReceive(Context context, Intent intent) {
        String action = intent.getAction();
        if (SMS_RECEIVED_ACTION.equals(action)) {
            // 通过 pdus 获取SMS数据的隐藏键
            Object[] messages =
                    (Object[]) intent.getSerializableExtra("pdus");
            for (Object message : messages) {
                byte[] messageData = (byte[]) message;
                SmsMessage smsMessage =
                        SmsMessage.createFromPdu(messageData);
                Log.e("haha", "收到消息来自: "+smsMessage.getOriginatingAddress()+ "   内容:"+smsMessage.getMessageBody());

                processSms(smsMessage);
            }
        }
    }

    // 只关心指定的电话号码
    private void processSms(SmsMessage smsMessage) {
        String from = smsMessage.getOriginatingAddress();
        if (MESSAGE_SERVICE_NUMBER.equals(from)) {
            String messageBody = smsMessage.getMessageBody();
            if (messageBody.startsWith(MESSAGE_SERVICE_PREFIX)) {
                // TODO: 数据验证通过开始处理
                Log.e("haha", "processSms: "+messageBody);
            }
        }
    }
}
```

上面利用广播监听`Intent`操作`android.provider.Telephony.SMS_RECEIVED`. 这个例子中唯一隐藏的部分就是`Intent`的action. 以及用来从`Intent("pdus")`检索SMS数据的字符串.

要读取已经收到的`SMS`, 需要查询一个隐藏的`ContentProvider`, 并声明使用`READ_SMS`权限. `android.provider`包中的`Telephony`类提供了所有需要的信息. 使用该类最佳的方式是把它复制到自己的项目中, 并修改类的包结构. 由于`Telephony`类还包含其他隐藏类和方法的调用, 所以还必须删除或者重构这些调用, 以便能够编译代码. 取决于使用`隐藏API`的数量, 有时候简单复制一些常量声明而不是整个类就足够了.

阅读的部分就省略了, 除了某些特定方向的应用, 基本上应用是不会去读入用户的短信.

### 隐藏设置

`Android`设备有数百种不同的设置, 都可以通过`Settings`类访问, 除了为每个设置提供访问的值, `Android`还提供给了一些类`Intent`操作, 使他们可以打开特定的设置UI. 例如, 要启动飞行模式设置, 在创建Intent时可以使用`Settings.ACTION_AIRPLANE_MODE_SETTINGS`

`Settings`类包含了一些隐藏的设置键和Intent操作, 当应用程序需要弄清楚设备的细节或者呈现一个特定系统设置的快捷方式时, 启动的一些常量值是非常方便的.