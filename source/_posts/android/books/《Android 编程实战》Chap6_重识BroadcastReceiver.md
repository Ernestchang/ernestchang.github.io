title: 《Android 编程实战》Chap6_重识BroadcastReceiver
date: 

categories: 
- android 
- 《Android 编程实战》
tags:  
- android
---
> 阅读《Android 编程实战》一书的随记笔记
> 注：本文主要参考[http://szysky.com](http://szysky.com)

## BroadcastReceiver

`Android`中发送广播事件最常用的方式是通过`Content.sendBroadcast()`方法给`BroadcastReceiver`发送`Intent`对象. 许多标准系统事件都被定义成操作字符串, 并可以在`Intent`类的API文档中查看. 例如, 如果需要在用户连接或者断开充电器的时候收到通知, 可以使用`Intent`中定义的两个广播操作: `ACTION_POWER_DISCONNECTED`和`ACTION_POWER_CONNECTED`.

------

**举例: 例如监听手机充电状态改变的广播**

首先派生出一个`BroadcastReceiver`的子类.复写`onReceiver()`方法. 如下

```
public class ChargerConnectedReceiver extends BroadcastReceiver {
    public ChargerConnectedReceiver() {
    }

    @Override
    public void onReceive(Context context, Intent intent) {
        String action = intent.getAction();

        if (Intent.ACTION_POWER_CONNECTED.equals(action)){
            Toast.makeText(context, "手机充电啦", Toast.LENGTH_SHORT).show();
        }else if (Intent.ACTION_POWER_DISCONNECTED.equals(action)){
            Toast.makeText(context, "手机不充电了", Toast.LENGTH_SHORT).show();
        }
    }
}
```

然后需要注册广播, 告诉系统在当哪一个`action`动作发生的时候需要回调我们自定的接收者的`onReceive()`方法. 这里有两种方法, `静态注册`和`动态注册`.

**静态注册**

在清单文件中声明这个广播组件, 并设置`intent-filter`即可

```
<receiver android:name="broadcast.ChargerConnectedReceiver">
  <intent-filter>
      <action android:name="android.intent.action.ACTION_POWER_CONNECTED"/>
      <action android:name="android.intent.action.ACTION_POWER_DISCONNECTED"/>
  </intent-filter>
</receiver>
```

**动态注册**

一般情况动态注册都是在`Activity`中的`onCreate()`和`onResume()`同时出现的. 例如:

```
public class ChargerConnectedActivity extends Activity {

    private ChargerConnectedReceiver chargerConnectedReceiver;

    @Override
    protected void onResume() {
        super.onResume();
        // 生成对于广播的 intent过滤条件
        IntentFilter intentFilter = new IntentFilter();
        intentFilter.addAction(Intent.ACTION_POWER_CONNECTED);
        intentFilter.addAction(Intent.ACTION_POWER_DISCONNECTED);
        chargerConnectedReceiver = new ChargerConnectedReceiver();
        registerReceiver(chargerConnectedReceiver, intentFilter);
    }

    @Override
    protected void onPause() {
        super.onPause();
        unregisterReceiver(chargerConnectedReceiver);
    }
}
```

如果只在应用程序处于运行或活动状态时才关心广播事件时, 可以选择在代码中注册广播. 这样可以让应用程序消耗更少的资源; 如果在清单文件中声明, 则每当有事件发生时, 广播接收器都会启动, 因此会消耗更多资源.

### 本地BroadcastReceiver

如果只是在应用程序进程内发送和接收广播, 那么可以使用`LocalBroadcastManager`而不是更常用的`Context.sendBroadcast()`方法. 这种方法更高效, 因为不需要跨进程管理操作, 也不需要考虑广播通常涉及的安全问题. 标准IPA中没有包含`LocalBroadcastManager`类, 但是可以在支持包(support_V4)中找到. 下面演示如何使用:

```
public static final String LOCAL_BROADCAST_ACTION = "localBroadcast";
private BroadcastReceiver mLocalReceiver;

// 注册本地广播
private void initLocalBroadcast() {
   LocalBroadcastManager instance = LocalBroadcastManager.getInstance(getApplicationContext());
   IntentFilter intentFilter = new IntentFilter(LOCAL_BROADCAST_ACTION);

   mLocalReceiver = new BroadcastReceiver() {
       @Override
       public void onReceive(Context context, Intent intent) {
           Toast.makeText(getApplicationContext(), "本地广播接收到", Toast.LENGTH_SHORT).show();
       }
   };

   instance.registerReceiver(mLocalReceiver, intentFilter);
}

// 发送本地广播
findViewById(R.id.btn_send).setOnClickListener(new View.OnClickListener() {
  @Override
  public void onClick(View v) {
      LocalBroadcastManager instance = LocalBroadcastManager.getInstance(getApplicationContext());
      // 上面定义的通电状态action
      Intent intent = new Intent(LOCAL_BROADCAST_ACTION);
      instance.sendBroadcast(intent);
  }
});
```

在应用程序内部使用本地广播来广播消息和状态也非常方便. 本地广播比标准的全局广播更高效和安全, 因为它不会把数据泄露给其他应用程序. 切记要和正常的接收器一样, 在对应的方法中要移除注册, 否则可能会有内存泄漏.

### 普通广播和粘性广播

广播分为两种类型: **普通广播**和**有序广播**.

- **普通广播**会以异步方式发送给所有的接收者, 并且没有指定的接收顺序. 该方式更加高效, 但是缺少有序广播额一些高级功能, 比如不能发送结果反馈.
- **有序广播**按照特定的顺序分发, 每次只发给一个接收者, 开发者可以在清单文件中设置接收者的`intent-filter`标签的`android:priority`属性来控制广播的接收顺序. 有序广播还有另外一个特性: 通过使用`abortBroadcast()`, `setResultCode()`和`setResultData()`方法, 接收者可以把结果回传给广播, 或者终止广播的分发, 这样`Intent`就不会传递给下一个广播接收者.

有序广播由`Context.sendOrderedBroadcast()`发起, 在接收者的`onReceive()`回调中, 通过`isOrderedBroadcast()`来判断该广播是否是有序广播. 如果是, 可以通过上面`setXxxx()`方法设置要传递下去的数据.

日常开发很少需要在自己的应用程序发送有序广播, 但如果要跟其他应用程序通信(比如插件), 有序广播就有用途. 在Android系统中, 有序广播最常见的场景就是监听传入的短信(隐藏API)的一部分. 后面篇幅会说.

### 粘性广播

`粘性广播(sticky broadcast)` 是一个普通广播的变体, 它和普通广播有细微的区别. 粘性广播在使用`Context.sendStickyBroadcast()`发送`Intent`之后, 该`Intent`还会”继续保留”, 允许之后匹配由该`Intent`新注册的广播接收者, 并发送`Intent`.([查看验证代码](https://github.com/suzeyu1992/AndroidProgrammingPushingTheLimits/commit/ec42e2e44877f5b4b3c5363600646c4e808929e3))

粘性广播的一个例子是`Intent.ACTION_BATTERY_CHANGED`, 它用来指示设备中电池电量的变化. 另一个列子是`Intent.ACTION_DOCK_EVENT`, 用来只是设备是否放在了底座. 更多的粘性广播请参考google文档. 下面的代码展示如何

```
private void myRegisterBattery(){
   // 构建广播接收者要接收的action
   IntentFilter intent = new IntentFilter();
   intent.addAction(Intent.ACTION_BATTERY_CHANGED);
   intent.addAction(Intent.ACTION_BATTERY_OKAY);
   intent.addAction(Intent.ACTION_BATTERY_LOW);
   
   // 创建监听
   BroadcastReceiver broadcastReceiver = new BroadcastReceiver() {
       @Override
       public void onReceive(Context context, Intent intent) {
           if (isInitialStickyBroadcast()) {
               Log.e("sususu", "这是一个粘性广播");
           } else {
               Log.e("sususu", "这是不是粘性广播");
           }
       }
   };
   // 注册接收者
   registerReceiver(broadcastReceiver, intent);
}
​``` 

该方法在广播全系统的状态时特别有用, 如果你需要发送粘性广播, 那么**请添加权限**在清单文件中`<uses-permission android:name="android.permission.BROADCAST_STICKY"/>`权限, 并使用`Context.sendStickyBroadcast()`发送粘性广播. 

> 对于粘性广播一定要慎用, 因为它比普通广播更消耗资源.


### 定向广播

普通广播的另一个变体是`定向广播(directed broadcast)`. 定向广播使用过了`intent-filter`的一个特性, 通过在`Intent`设置`ComponentName`来显示指定接收者. 它把注册接收者的类名和包名结合在了一起. 如下:


​```java
Intent intent = new Intent();
intent.setComponent(new Component(packName, className));
sendBroadcast(intent);
```

这个例子只会指定的class类的广播接收者才可以收到广播, 即便其他接收器也注册了相同的`Intent`操作. **注意:使用定向广播需要同时知道接收者的包名和类名.**. 使用场景很少.

### 启动和禁用广播接收器

如果广播接收者在清单文件中注册的, 还有另外一种减少对系统负载的影响的方法. 通过`PackageManager`, 开发者可以启动和禁用应用程序的组件, 这在用户比如在应用设置更改后使用此方法即可. 代码如下:

```
/**
* 设置组件
* @param setClass 要设置改变的组件
* @param isEnable true为启用, false为禁用
*/
public void setComponentEnable(boolean isEnable, Class setClass){
   PackageManager pm = getPackageManager();
   // 构建要改变组件的Component
   ComponentName componentName = new ComponentName(getApplicationContext(), setClass);
   pm.setComponentEnabledSetting(componentName,
           isEnable ? PackageManager.COMPONENT_ENABLED_STATE_ENABLED : PackageManager.COMPONENT_ENABLED_STATE_DISABLED,
           PackageManager.DONT_KILL_APP);
}
```

`记住, 这里仅适用静态注册就是清单文件上注册, 不支持动态注册的会抛出异常, 并且这里接收的class同样适用于Activity, Service, ContentProvider`. 其本质就是改变在清单文件中组件标签的`<android:enable='true/false'>`.

关于`setComponentEnabledSetting()`方法的最后一个参数`PackageManager.DONT_KILL_APP`的使用. 这回防止平台杀死应用, 如果不设置该值平台默认会杀死应用.

> 可以用在应用程序启动图标的切换, 比如,开发者可以在安装应用程序后只显示设置Activity界面, 在设置完成之后使用该方法把启动图标隐藏

### 系统广播Intent

`Android API`定义了许多不同的系统广播事件. 例如电池电量变化, 是否连接了设备电源. 并且还有一些可能会用到的广播, 但是action并没有在API中公开, 所以后面的篇幅也会对一些隐藏的API进行一定记录.这里说一下常用的系统事件

------

**自动启动应用程序**

关于自启动这个问题在国内比较蛋疼, 由于各种厂商对`ROM`的修改, 各式各样. 目前为我自己的实验是只有`Google Nexus4`可以直接声明`BOOT_complete`重启的广播, `魅族4`,`三星S6`都无法做到重启可以监听到重启广播. 需要对应用通过手机提供的应用管理, 把`自启动开启才可以达到预期的效果`. 这里尝试的做法是`注册监听重启广播的接收`, `添加权限<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>`, `添加<manifest>标签的内属性android:installLocation="internalOnly"`确保应用正确安装到内存储位置.

广播的监听的对应action:`android.intent.action.BOOT_COMPLETED`

还有一个应用程序包替换的时候的广播, 例如升级`android.intent.action.MY_PACKAGE_REPLACED`

------

**用户状态和屏幕状态**

虽然当按下关机键和锁屏键会触发屏幕熄灭, `Activity`会调用对应的焦点失去或者获取的回调. 但是如果服务`Service`需要注意此动作的时候, 我们通过屏幕的状态广播来监听这是很方便的.

相关广播`action`

- `action.intent.action.SCREEN_OFF`
- `action.intent.action.SCREEN_ON`
- `action.intent.action.SCREEN_PRESENT`

开启和关闭设备屏幕时, 系统会分别发送`Intent.ACTION_SCREEN_ON`和`Intent.ACTION_OFF`广播事件. 当用户解锁屏幕时系统会发送`Intent.ACTION_USER_PRESENT`广播事件.

------

**网络和连接变化**

大多数`Android`设备都支持两种类型的网络: **蜂窝网络**和**Wi-Fi网络**. 如果应用程序过度依赖网络操作, 开发者可能要在蜂窝网络中推迟数据的传输, 知道设备连接到`Wi-Fi网络`; 否则, 如果使用`3G`, `LTE`之类的移动网络传输可能会产生相当可观的流量.

连接的相关广播和网络相关的广播分别由不同的API负责. 每当有通用的网络连接变化发生时, 比如从`Wi-Fi`切换到移动数据, 系统就会发送`ConnectivityManager.CONNECTIVITY_ACTION`广播, 接下来可以使用`Context.getService()`方法来检索`ConnectivityManager`服务, 它允许开发者获取当前网络的更多信息.

然而, 要获取当前网络更细粒度的信息, 开发者还需要监听来自`TelephonyManager`和`WifiManager`的广播事件. `TelephonyManager`允许查询移动数据连接的类型, `WiFiManager`允许检索`WiFi`连接状态并访问和`WiFi`相关的不同`ID(SSID是wifi名称和BSSID对应mac地址)`

以下代码会检测设备是否连接到了预先设置的某一个`WiFi`. 使用此方法可以有效地和服务器或者只是支持特定的`Wi-Fi`的媒体进行通信.

```
<!-- 检查wifi广播-->
<receiver
  android:name="broadcast.CheckForHomeWifi">

  <intent-filter>
      <!--监听wifi的连接状态是否连接上一个有效无线路由-->
      <action android:name="android.net.wifi.WIFI_STATE_CHANGED" />
      <!--监听wifi的打开和关闭, 和wifi具体的连接不关心-->
      <action android:name="android.net.wifi.STATE_CHANGE" />
  </intent-filter>

</receiver>
```

如果只需要关心wifi的开启, 和wifi连接到某一个路由, 那么这两个广播监听足够了.

```
@Override
public void onReceive(Context context, Intent intent) {

   // 需要判断的路由名字 对应ssid 
   String name = "\"ziroom502\"";

   // 1.首先判断wifi是否开启, 并连接
   NetworkInfo networkInfo = intent.getParcelableExtra(WifiManager.EXTRA_NETWORK_INFO);
   if (networkInfo != null && networkInfo.getState().equals(NetworkInfo.State.CONNECTED)) {

       // 2.再判断连接的wifi的具体信息
       WifiInfo WifiInfo = intent.getParcelableExtra(WifiManager.EXTRA_WIFI_INFO);
       if (WifiInfo != null && name.equals(WifiInfo.getSSID())) {
           Log.d("sususu", "连接到指定wifi");
       } else {
           Log.d("sususu", "连接到其他wifi");
       }
   }
}
```

这是一个简单的初步判断, 如果需要结合手机连接判断, 结合下面的另一个监听, 整合起来就差不多了.

------

这个监听来自`ConnectivityManager`的变化, 并确定当前连接的是否为移动数据网络. 如果收到移动数据, 接下来在使用`TelephonyManager`检查是否在使用`3G`或者`LTE`网络.

监听广播的清单文件

```
 <!--判断手机连接-->
<receiver
  android:name="broadcast.WhenOn3GorLTE">
  <intent-filter>
      <!--此广播监听, 网络连接的设置包括wifi和数据的打开和关闭-->
      <action android:name="android.net.conn.CONNECTIVITY_CHANGE" />
  </intent-filter>
</receiver>
```

```
public class WhenOn3GorLTE extends BroadcastReceiver {
    private static final String TAG = WhenOn3GorLTE.class.getSimpleName();

    @Override
    public void onReceive(Context context, Intent intent) {
        String action = intent.getAction();
        if (ConnectivityManager.CONNECTIVITY_ACTION.equals(action)){
            boolean noConnectivity = intent.getBooleanExtra(ConnectivityManager.EXTRA_NO_CONNECTIVITY, false);
            
            if (noConnectivity){
                Log.e(TAG, "没有连接" );
            }else{
                int networkType = intent.getIntExtra(ConnectivityManager.EXTRA_NETWORK_TYPE, ConnectivityManager.TYPE_DUMMY);
                
                if (networkType == ConnectivityManager.TYPE_MOBILE){
                    checkfor3GorLte(context);
                }else{
                    Log.i(TAG, "不是移动连接");
                }
            }

        }
    }


    /**
     *  当前如果移动数据开启, 那么显示出移动数据的连接类型
     */
    private void checkfor3GorLte(Context context){

        TelephonyManager telephonyManager = (TelephonyManager) context.getSystemService(Context.TELEPHONY_SERVICE);

        switch (telephonyManager.getNetworkType()){
            case TelephonyManager.NETWORK_TYPE_HSDPA:
                Log.d(TAG, "连接类型: NETWORK_TYPE_HSDPA");
                break;

            case TelephonyManager.NETWORK_TYPE_HSPA:
                Log.d(TAG, "连接类型: NETWORK_TYPE_HSPA");
                break;

            case TelephonyManager.NETWORK_TYPE_HSPAP:
                Log.d(TAG, "连接类型: NETWORK_TYPE_HSPAP");
                break;

            case TelephonyManager.NETWORK_TYPE_HSUPA:
                Log.d(TAG, "连接类型: NETWORK_TYPE_HSUPA");
                break;

            case TelephonyManager.NETWORK_TYPE_LTE:
                Log.d(TAG, "连接类型: NETWORK_TYPE_LTE");
                break;

            default:
                Log.d(TAG, "连接类型: 未知类型, 可能传输速度会慢");
                break;

        }
    }
}
```

**此广播稍微有2秒左右延迟, 当wifi开关,和移动数据开关变化的时候不会立即响应**

一下记录一下测试的结果广播接收情况,以上判断条件是获取的`intent中的ConnectivityManager.EXTRA_NO_CONNECTIVITY`(测试手机三星S6), 可能会因为不同的条件产生不同的结果. 这里只说明代码中的条件说明:

- `当wifi连接开启`: 这时打开或者关闭移动数据, 不会发送广播.
- `当移动数据开启`: 这时操作wifi开关会接收到两个广播.
- - 如果从`wifi关闭`->`wifi开启`那么首先会发送一个`移动数据的状态info广播`,紧接着会发送一个`wifi连接`相关的广播(总共两个有效连接)
- - 如果`wifi开启` -> `wifi关闭`那么首先会收到一个`无连接`的广播, 然后会接收到一个`移动数据`连接类型的广播.
- `如果当都关闭的时候`: 开启任意一个只能接收到一个广播.

这种方式也可以获取`wifi`连接的信息, 但是**延迟性**是这个方式的特点.