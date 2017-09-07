title: 《Android编程权威指南》Chap4_ 组件等
date: 

categories: 
- android 
- 《Android编程权威指南》
tags:  
- android
---

> 这本书属于入门, 有很多内容可以当做扩展来了解一些API, 并且有的时候可以适当的利用Google提供好的API来做一些高效开发节约时间成本. 原理东西本书偏少. 可以学习本书中的代码的编写风格, 书中代码都是采用**MVC**模型来编写的. 就写这么多, 下面开始整理一些小知识点.

## Message与Message Handler

**消息Message**

消息是`Message`类的一个实例. 包含好几个实例变量. 其中有三个需要在实现时定义.

- `what` 用户定义的int类型消息代码, 用来描述消息
- `obj` 随消息发送的用户指定对象
- `target` 处理消息的`Handler`

Message的目标是Handler类的一个实例. `Message`在创建时, 会自动与一个`Handler`相关联. Message在准备处理状态下, Handler是负责让消息处理行为发生的对象.

**Handler**

要处理消息以及消息指定的任务, 首先需要一个消息`Handler`实例. `Handler`不仅仅是处理`Message`的目标(target), 也是创建和发出`Message`的接口.

`Looper` 拥有`Message`对象的收件箱, 所以`Message`必须在`Looper`上发布或读取. 基于`Looper`和`Message`的这种关系, 为了与`Looper`协同工作, `Handler`总是引用着它.

- 一个`Handler`仅于一个`Looper`相关联.
- 一个`Message`也仅于一个目标`Handler`相关联
- 多个`Handler`可能都关联一个`Looper`(这也意味着一个`Handler`的`Message`可能与另一个`Handler`的`Message`存在在同一个消息队列中)
- `Looper`拥有整个队列

**Handler的使用**

消息的目标`Handler`通常不需要手动设置. 一个比较理想的方式是调用`Handler.obtainMessage()`方法创建消息并传入其他消息字段, 然后该方法自动完成目标`Handler`的设置.

为避免创建新的`Message`对象, `Handler.obtainMessage()`方法会从公共循环池里获取消息. 因此相比重新创建, 这种复用的选择会更加高效.

一旦取得了`Message`就可以调用`sendToTarget()`方法将其发送给它的`Handler`. 然后`Handler`会将`Message`放置在`Looper`消息队列的尾部.

`Looper`取得到消息队列中的特定消息后, 会将它发送给消息目标去处理. 消息一般是在目标的`Handler.handleMessage()`中进行处理的.

**传递Handler**

`HandlerThread`能在主线程上完成任务的一种方式是, 让主线程将其自身的`Handler`传递给`HandlerThread`.

主线程是一个拥有`Handler`和`Looper`的消息循环. 主线程上创建的`handler`会自动与它的`Looper`相关联. 可以将主线程的`handler`传递给另一个线程. 传递出去的`Handler`与创建线程的`Looper`始终保持着联系. 因此任何已传出的`Handler`负责处理的消息都将在主线程的消息队列中处理.

**关于AsyncTask**

`AsyncTask` 主要应用于那些短暂且较少重复的任务. 并且在`Android 3.2`时其内部进行了代码的改动. 改动之后的效果是`AsyncTask`不再为每一个`AsyncTask`的实例单独创建一个线程. 相反, 它使用了`Executor`在单一的后台线程上运行所有的`AsyncTask`的后台任务. 这意味着每个`AsyncTask`都需要排队逐个运行. (虽然可以通过特定的方法改变其内部的顺序调用, 但是或许应该想想是否有其他的更好的方式来选择)

## 启动模式于新的Intent

启动模式决定了`activity`在收到新的`intent`时的启动方式.

| 启动模式           | 行为                                       |
| -------------- | ---------------------------------------- |
| standard       | 默认行为, 针对每一个收到的新Intent, 都会启动新的activity    |
| singleTop      | 如果activity实例已经处在回退栈的顶部, 则不重新创建新的activity, 而直接路由新的intent给现有的activity |
| singleTask     | 在自身task中启动activity, 如果task中activity已经存在, 则清除回退站中该activity上的任何activity, 然后路由新的intent给现有的activity |
| singleInstance | 在自身task中启动activity, 该activity是task中唯一的activity, 如果任何其他activity从该task中启动, 他们都将启动到自己的task中. 如果task中的activity已经存在. 那么就路由新的intent给现有的activity |

上面说到路由新的`intent`给现有的`activity`. 这个新的`intent`通过复写`activity#onNewIntent()`可以获得.

## 使用SP实现轻量级数据存储

对于存储可以采用序列化对象并保存至外部存储设备的方式, 实现持久化存储. 然后对于**轻量级数据的持久化**可以使用`shared preferences`会更加简单高效.

`shared preferences`本质上就是文件系统中的文件. 可使用`SharedPreferences`类对其进行读写. `SharedPreferences`实例使用起来更像一个键值对仓库如`Bundle`, 但它可以通过持久化存储保存数据. 键值对中的键为**字符串**. 而值是**原子数据类型**. 其本质上就是一种简单的XML文件. 但`SharedPreferences`类已屏蔽了读写文件的实现细节.

**关于使用:** 可以使用`Context.getSharePreferences(String, int)`方法. 然而在实际开发中, 由于`sharedpreferences`共享整个应用. 并不太关心获取的特定实例是什么. 这种情况下, 可以使用`PreferenceManager.getDefaultSharedPreferences(Context)`, 该方法会返回具有私有权限与默认名称的实例.

而对于添加数据, 通过SP对象的`edit()`方法可以获得一个`Editor`实例, 通过这个实例, 就可将一组数据操作放入一个事务中, 添加之后. 通过`Editor.commit()`最终写入数据.

## 后台服务

**IntentService**

`IntentService`并不是`Android`提供唯一服务, 但却是最常用的. `IntentService`也是一个`Content`. 并能够响应intent. 服务的`intent`又称作**命令**. 每一条命令都要求服务完成某项具体的任务.

`IntentService`逐个执行命令队列的命令. 接收到首个命令时, `IntentService`即完成启动, 并触发一个后台线程, 然后将命令放入队列.

随后`IntentService`继续按顺序执行每一条命令, 并同时为每一条命令在后台线程上调用`onHandlerIntent()`方法. 新进命令总是放置在队列尾部. 最后, 执行完队列中全部命令后, 服务也随即停止并被销毁.

## AlarmManager延迟运行服务

有时为了保证服务在后台的可用, 当没有`activity`在运行时, 需通过某种方式在后台执行一些任务. 比如说, 设置一个5分钟间隔的定时器.

一种实现方式是调用`Handler`的`sendMessageDelayed()`或者`postDelayed()`方法. 但如果用户离开了当前应用. 进程就会停止, `Handler`消息也会随之消亡, 因此该解决方案并不算可靠.

这样可以尝试使用`AlarmManager`. `AlarmManager`是可以发送`Intent`的**系统服务**. 那么如何告诉`AlarmManager`发送`Intent`呢? 使用`PendingIntent`. 利用`PendingIntent`打包`Intent`, 然后将其发送给系统中的其他部件, 如`AlarmManager`. 如下代码:

```
public static void setServiceAlarm(Context context, boolean isOpen){
   Intent intent = new Intent(context, 要打开服务的类名.class);

   PendingIntent pi = PendingIntent.getService(context, 0, intent, 0);

   AlarmManager alarmManager = (AlarmManager) context.getSystemService(Context.ALARM_SERVICE);
   
   if (isOpen){
       // 设置定时器
       alarmManager.setRepeating(AlarmManager.RTC, System.currentTimeMillis(), 1000 * 15 , pi);
   }else{
       alarmManager.cancel(pi);
       pi.cancel();
   }
   
}
```

如上代码中只要替换创建`Intent`指定的服务类名. 那么当调用了`setServiceAlarm`传递`true`那么. 会发现每隔15秒就会启动一次`IntentService`. 即使进程停止, `AlarmManager`依然会不断发送`intent`. 以反复启动服务.

## PendingIntent

`PendingIntent`是一种**token**对象. 调用`PendingIntentn.getService()`方法获取`PendingIntent`时, 我们告诉操作系统: 要使用`startService()`方法发送这个`intent`. 随后调用`PendingIntent`对象的`send()`方法时, 操作系统会按照我们的要求发送原来封装的`Intent`.

`PendingIntent`真正精妙之处在于, 将`PendingIntent`交给其他应用使用时, 它是代表当前应用发送`token`对象的. 另外`PendingIntent`本身存在于操作系统而不是`token`里, 因此实际上是我们在控制着它. 如果不顾及别人的感受, 也可以在交给别人一个`PendingIntent`对象后, 立即撤销它, 让send()方法什么也不做.

如果使用同一个`intent`请求`PendingIntent`两次, 得到的`PendingIntent`仍会是同一个, 我们可借此测试某个`PendingIntent`是否已存在, 或撤销已发出的`PendingIntent`

**使用PendingIntent管理定时器**

一个`PendingIntent`只能登记一个定时器, 可以通过`AlarmManager.cancel(PendingIntent)`方法来撤销`PendingIntent`的定时器, 然后撤销`PendingIntent`.

还有一种方式可以判断定时器激活与否, 既然撤销定时器也随即撤销了`PendingIntent`, 可通过检查`PendingIntent`是否存在, 来确认定时器是否激活. 具体实现, 传入`PendingIntent.FLAG_NO_CREATE`标记给`PendingIntent.getService()`方法即可, 该标记表示如果`PendingIntent`不存在则返回null.

## 通知消息 Notification

如果服务需要与用户进行信息沟通, **通知消息(notification)**是一个不错的选择, 通知消息是指显示在通知抽屉上的消息条目, 用户可向下滑动屏幕读取.

发送通知消息, 首先需要创建一个`Notification`对象. `notification`需要使用构造对象完成创建.

一个比较完成的`Notification`应该具备如下:

- 首次显示通知消息时, 在状态栏上显示的ticker text.
- ticker text消失后, 在状态栏上显示的图标
- 代表通知信息自身, 在通知抽屉中显示的视图
- 用户点击抽屉中的通知信息, 触发`PendingIntent`

完成`Notification`对象的创建后, 可调用`Notification`系统服务的`notify(int,Notification)`方法发送它.

## 服务的一些说明

**non-sticky服务**

`IntentService`是一种`non-sticky`服务. `non-sticky`服务在完成所有已有任务时停止. 为获得`non-sticky`服务, 应返回`START_NOT_STICKY`或者`START_REDELIVER_INTENT`.

通过调用`stopSelf()`或`stopSelf(int)`方法, 我们告诉Android任务已经完成. `stopSelf()`是一个无条件方法. 不管`onStartCommand()`方法调用多少次, 该方法总是会成功停止服务.

`IntentService`使用的是`stopSelf(int)`方法. 该方法需要来自`onStartCommand()`方法的启动ID. 只要在接收到最新的启动ID后, 该方法才会停止服务.

返回的`START_NOT_STICKY`和`START_REDELIVER_INTENT`的具体不同是, 如果系统需要在服务完成任务之前关闭它, 则服务的具体表现会有所不同. `START_NOT_STICKY`型服务会被关闭. 而`START_REDELIVER_INTENT`型服务, 则会在可用资源不再吃紧时, 尝试再次启动服务. 所以根据操作与应用的重要程度, 进行选择. `start_not_sticky`是`IntentService`的默认行为. 通过`setIntentRedelivery(true)`来切换模式.

**sticky服务**

`sticky`服务会持续运行, 直到外部组件调用`Content.stopService(Intent)`方法让其停止为止. 为启动`sticky`服务, 应返回`START_STICKY`.

`sticky`服务启动后会持续运行, 直到某个组件调用`Content.stopService(Intent)`方法为止. 如因某种原因需终止服务, 可传入一个null intent给`onStartCommand()`方法实现服务的重启.

`sticky`服务适用于长时间运行的服务, 如音乐播放器这种启动后一直保持运行状态, 直到用户主动停止服务.

## broadcast Intent

Android设备中, 各种事件一直频繁的发生着. WIFI型号时有时无, 各种软件包获得安装, 电话的呼入呼出, 短信接收等.

许多的系统组件需要知道某些事件的发生. 为满足这样的需求, android提供了`broadcast intent`组件.

------

**设备重启而重启的定时器**

设备重启后, 那些持续运行的应用通常也需要重启. 通过监听具有`BOOT_COMPLETED`操作的broadcast intent, 可得知设备是否已完成启动.

如果要实现这个功能, 首先定义一个广播, 继承`BroadcastReceiver`. 并在清单文件中声明`<receiver>`并添加intent过滤器设置一个`action`. 因为要监听系统开机所以需要添加一个使用权限`<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED">`

于`activity`和服务不同, 在配置文件中声明的`broadcast receiver`几乎总是需要声明`intent-filter`. `broadcast intent`就是为发送信息给多个监听者而生. 但显示的intent只有一个`receiver`. 因此显示的`broadcast intent`很少见.

------

**如何使用receiver**

`broadcast receiver`的存在很短暂, 因此它的作用也就受到限制, 例如无法使用任何异步API或登记任何的监听器, 因为`onReceiver()`回调一旦运行完, `receiver`也就不存在了. 而因为`onReceiver()`同样运行在主线程, 因此不可以执行耗时操作.

但是存在既有道理, 对于轻型任务代码的运行而言, `receiver`非常有用. 系统重启后, 定时运行的定时器也需要重置. 显示, 使用`broadcast receiver`处理这种小型任务再合适不过了(这里只是列举一个例子).

------

**使用私有权限**

使用动态广播接收存在一个问题, 即系统中的任何应用均可监听并触发我们的`receiver`. 对于这种情况应该避免. 有两种方案可以选择. 如果`receiver`声明在`manifest`配置文件里, 且仅限应用内部使用, 则可在`receiver`标签上添加`android:exported="false"`属性. 这样系统中的其他应用就再也无法接触到该`receiver`. 另外也可以创建自己的使用权限. 通过在清单文件中添加一个`<permission>`标签来完成. 例如

```
<permission android:name="com.suzeyu.android.TEST_PERMISSION"
    android:protectionLevel="signature">
```

这段代码使用了`protection level`签名, 定义了自己的定制权限. 权限本身只是一行字符串. 即使是自定义的权限, 也必须要在使用前获取它, 这是规则.

如果给定义的广播设置了权限, 那么在发送广播时(sendBroadcast), 和注册广播时. 和应用清单添加使用权限的声明. 3处需要注意.

------

**深入了解protection level**

自定义权限必须指定`android:protectionLevel`属性值, Android根据`protectionLevel`属性值确定自定义权限的使用方式. 权限的所选值有四种. 对于仅限应用内部使用的权限, 通常会选择`signature`安全级别.

关于四种可选值的特征如下:

| 可选值               | 用法描述                                     |
| ----------------- | ---------------------------------------- |
| normal            | 用于阻止应用执行危险操作, 如访问个人隐私数据,联网传送数据等. 应用安装前, 用户可看到相应的安全级别, 但无需他们主动授权. `android.permission.RECEIVE_BOOT_COMPLETED`使用该安全级别. 同样, 手机振动也使用该安全级别. 虽然这些安全级别没有危险, 但最好让用户知晓可能带来的影响 |
| dangerous         | normal安全级别控制以外的任何危险操作, 如访问个人隐私数据, 通过网络接口收发数据, 使用可监视用户的硬件功能等. 总之, 包括一切可能为用户带来麻烦的行为. 网络使用权限, 相机使用权限以及联系人信息使用权限都属于危险操作. 需要`dangerous`权限级别时, Android会明确要求用户授权 |
| signature         | 如果应用签署了于声明应用一致的权限证书, 则该权限由系统授予. 否则, 系统则作相应的拒绝. 权限授予时, 系统不会通知用户. 它通常适应于应用内部. 只要拥有证书, 则只有签署了同样证书的应用才能拥有该权限, 因此可自由控制权限的使用. 这里我们使用它阻止其他应用监听到应用发出的`broadcast`. 不过如有需要, 可定制开发能够监听他们的专有应用 |
| signatureOrSystem | 类似于`signature`授权界别. 但该授权级别针对Android系统镜像中的所有包权限. 该授权级别用于系统镜像内应用间的通信, 因此用户通常无需关心 |

------

**使用ordered broadcast接收结果**

常规的`Broadcast`可同时被其他应用所接收. 而如果希望他们按照某种顺序依次运行, 或知道他们什么时候全部结束运行是无法做到的. 所以为了解决这种情况, 可使用**有序broadcast intent**实现双向通信. 有序广播允许多个广播接收者依序处理`broadcast intent`. 另外 通过传入一个名为`result receiver`的特别`broadcast receiver`, 有序广播还可以实现让**广播的发送者**接收到**广播接收者**发送的返回结果.

通过`sendOrderedBroadcast(...)`发送有序广播. 和常规的广播相比有序广播的参数还多出五个. 一次为: 一个result receiver; 一个支持result receiver运行的Handler; 结果代码初始值; 结果数据; 有序broadcast的结果附加内容

`result receiver`比较特殊, 只有在所有有序广播接收者结束运行后, 它才可以运行.

有序发送顺序是通过`receiver`的优先级作为依据.

关于优先级数值越高优先级越高. 如果要设置最后接收的接收者可以设定其优先级为**-999**(-1000及以下的值为系统保留值)

------

**receiver长时运行任务**

如果不想受限于主线程的时间限制, 并希望`broadcast intent`可触发一个长时运行任务,有两种方式可以选择:

- 将任务交给服务去处理, 然后再通过`broadcast receiver`启动服务. 服务可以运行很久, 直到完成需要处理的任务. 同时服务可将请求放在队列中, 然后依次进行处理, 或按其自认为合适的方式管理全部任务请求.
- 使用`BroadcastReceiver.goAsync()`方法. 该方法返回一个`BroadcastReceiver.PendingResult`对象, 随后, 可使用该对象提供结果. 因此, 可将`PendingResult`交给`AsyncTask`去执行长时运行任务, 然后再调用`PendingResult`的方法相应`broadcast`

`goAsync()`方法两个弊端: 不支持旧设备; 其次它不够灵活: 仍需快速响应broadcast.

## WebView

首先在xml布局中添加`WebView`控件, 没什么好说的. 接下来. 开始代码设置. 应该先完成三件事情.

- 指定`WebView`要打开的URL
- 启动`JavaScript`, `JavaScript`默认是禁用的. 最好启动以免不必要的错误.
- 覆盖`WebViewClient`类的`shouldOverrideUrlLoading(WebView, String)`方法. 并返回false.

例如:

```
mWebView.getSettings().setJavaScriptEnabled(true);
   
mWebView.setWebViewClient(new WebViewClient(){
  @Override
  public boolean shouldOverrideUrlLoading(WebView view, String url) {
      return false;
  }
});
   
mWebView.loadUrl(要加载的网址);
```

加载Url必须等到`WebView`配置完成之后. `WebSetting`是修改`WebView`配置的三种途径之一. 他还有一些其他可设置属性, 如用户代理字符串和显示文字大小.

配置`WebViewClient`. `WebViewClient`是一个事件接口, 通过提供自己实现的`WebViewClient`, 可响应各种渲染事件. 例如可检测渲染器何时开始从特定URL加载图片, 或决定是否需要向服务器重新提交POST请求.
`WebViewClient`有多个方法可供覆盖, 其中大多数用不到. 然而必须覆盖其`shouldOverrideUrlLoading()`的默认方法. 当有新的URL加载到`WebView`时(如点击了某个链接),如果返回true那么有系统浏览器处理. 如果为false那么就是`webview`去加载.

**使用WebChromeClient优化WebView的显示**

如果说`WebViewClient`是响应渲染事件的接口, 那么`WebChromeClient`就是一个响应改变浏览器中装饰元素的事件接口, 包括`JavaScript`警告信息, 网页图标, 状态条架子啊,以及当前网页标题的刷新.

进度条和标题栏的更新都有各自的回调方法, 即`onProgressChange()`和`onReceivedTitle()` 方法.

------

**处理WebView的设备旋转问题**

由于`WebView`包含了太多的数据, 以至无法再`onSaveInstanceState()`方法保存所有数据. 对于一些类似的类如`VideoView`. Android文档推荐让`activity`自己处理设备配置的变更. 也就是说, 不销毁重建. 那么就要让`activity`不会因设备配置变更而发生重建动作,而是自己去处理配置更改后的问题. 那么就需要在清单文件中声明属性.

```
android:configChanges="keyboardHidden|orientation|screenSize"
```

这个属性加在`activity`标签内部表明, 如果因键盘开关, 屏幕方向改变, 屏幕大小改变而发生的设配配置更改, 那么`activity`应自己处理配置更改,

## LocationManager

Android系统中的地理位置数据是由`LocationManager`系统服务提供的. 该系统服务向所有需要地理位置数据的应用提供数据更新. 更新数据的传送通常采用两种方式.

- 使用`LocationListener`接口
- 使用`PendingIntent`获取地理位置更新

`LocationListener`接口可能是最直接的一种方式. 通过`onLocationChanged()`方法, 该接口提供的信息有:地理位置数据更新, 状态更新, 以及定位服务提供者启停状态的通知消息. 如只需将地理位置数据发送给应用中的单个组件, 使用`LocationListener`接口会很方便. 通过实现`LocationManager`类的`requestLocationUpdates()`或者`requestSingleUpdate()`方法即可.

使用`PendingIntent`来获取地理位置数据更新, 实际是要求`LocationManager`在将来某个时点帮忙发送某种类型的`Intent`. 这样即使应用组件甚至整个应用进程都销毁了, `LocationManager`仍会一直发送`intent`, 直到要求它停止并按需启动新组件响应他们. 利用这种优势, 即使持续进行设备定位, 也可以避免应用消耗过多资源.

`LocationManager`是通过`Context.getSystemService(Context.LOCATION_SERVICE)`来获得的.
通过调用其`requestLocationUpdates(String, long, float, PendingIntent)`数值参数为: 最小等待时间(ms); 最短移动距离(m);

之后定义接收定位数据的广播接收者, 为了保证无论前台还是后台都可以接收更新数据, 最好使用清单文件进行注册. 在`onReceive()`实现方法中. `LocationManager`打包了附加额外信息的intent. `LocationManager.KEY_LOCATION_CHANGED`键值可指定一个表示最新更新的Location实例. 通过这个实例可以获取到服务提供者名字以及相应的经纬度数据. **别忘了添加使用地理位置的权限**. `<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION">`

也可以通过`LocationManag#getLastKnownLocation()`获取最近地理位置信息. 如下:

```
String provider = LocationManager.GPS_PROVIDER;
Location lastKnown = mLocationManager.getLastKnownLocation(provider);
```

## SQLite本地数据库

Android内置了操作SQLite的Java前端, 该前端的`SQLiteDatabase`类负责提供`Cursor`实例形式的结果集.

Android提供了一个帮助类, `SQLiteOpenHelper`类封装了一些存储应用数据的常用数据库操作, 如创建打开以及更新数据库等.

继承`SQLiteOpenHelper`通常要覆盖两个方法:

- `onCreate()`: 为新建数据库创建表结构
- `onUpgrade()`: 可执行迁移代码, 实现不同版本间的数据库结构升级或转换.

通常还需要实现一个父类的构造函数. 为父类提供必要的初始化参数, 数据库文件名, 可选`CursorFactory`参数的null值, 以及数据库版本号.

`SQLiteOpenHelper`类有两个访问`SQLiteDatabase`实例的方法:

- `getReadableDatabase()` 需要只读时使用
- `getWritableDatabase()` 需要可写数据库时使用

这两种方法基本没有太大区别, 但在某种特定情况下, 如磁盘空间满了, 则可能无法获取可写数据库, 而只能获取到可读数据库.

查询`SQLiteDatabase`可返回描述结果的`Cursor`实例. `Cursor`将结果集看做是一系列的数据行和数据列, 但仅支持String以及原始数据类型的值 .(可使用CursorWrapper封装当前Cursor对象.下面说明)

------

**使用CursorAdapter**

`CursorAdapter`类的构造方法需要一个Context, 一个Cursor, 一个整型flag. 为了提倡使用`Loader`, 大多数flag已经被废弃或存在一些问题. 因此可直接传入0.

需要实现两个方法:

- `newView(...)`: 会返回一个代表cursor中当前数据行的View. 在这里需要创建一个View并返回供item视图显示做准备
- `bindView(...)`: 进行绑定数据

**关于查询数据操作**

```
//  通过where子句. 限制查询只能返回一条记录, 然后封装到RunCursor中并返回
Cursor wrapped = getReadableDatabase().query(TABLE_RUN,
      null,   // All columns
      COLUMN_RUN_ID + " = ?",           // look for a run ID
      new String[]{String.valueOf(id)}, // with the value
      null,   // group by
      null,   // having
      null,   // order by
      "1"     //  limit 1 row
);
```

就给一个简单的例子. 了解一下查询方法的参数.

## Loader加载异步数据

------

**Loader于LoaderManager**

`Loader`设计用于从数据源加载某类数据(如对象). 数据源可以是磁盘, 数据库, ContentProvider, 网络或者另一个进程. `loader`可在不阻塞主线程的情况下获取并发送结果数据给接收者.

`loader`有三种内置类型: `Loader`, `AsyncTaskLoader`, `CursorLoader`.

- 作为基类`Loader`本身没有多大用处. 它定义了供`LoaderManager`与其他`loader`通讯时使用的API.
- `AsyncTaskLoader`是一个抽象`Loader`. 它使用`AsyncTask`将数据加载任务转移到其他线程中处理. 几乎所有创建的有用的`loader`类都是`AsyncTaskLoader`的子类
- `CursorLoader`. 通过继承`AsyncTaskLoader`类, 它借助`ContentResolver`从`ContentProvider`加载`Cursor`

`LoaderManager`管理着于loader间的所有通讯, 并负责启动, 停止和管理与组件关联的loader的生命周期方法. 在`activity`和`fragment`可通过`getLoaderManager()`方法返回一个实例并进行交互

要初始化`Loader`可使用`initLoader(int, Bundler, LoaderCallbacks<D>)`方法.

参数说明如下:

- 整数类型的loader标识符
- Bundler参数(值可为空)
- 一个接口回调分别对应着创建, 完成, 重启的回调

如果要强制重启现有的`loader`, 可使用`restartLoader()`方法. 在明确知道或怀疑数据比较陈旧时, 通常使用该方法重新加载最新数据.

**为什么使用loader而不直接使用AsyncTask**

有一个比较好的理由: 因设备旋转等原因发生配置更改时, LoaderManager可保证组件的loader及其数据不会丢失.

如果使用`AsyncTask`加载数据, 配置发生改变时, 就必须亲自管理其生命周期并保存它所取得的数据. 虽然可以通过`setRetainInstance(true)`解决了这些麻烦的问题, 但某些场景下, 还是要亲自编写代码处理才可以.

`loader`的设计目的就是要解决部分这样烦恼的问题. 配置发生改变后. 我们初始化一个已经完成数据加载的`loader`, 他会立即发送取得的数据, 而不是尝试再次获取数据. 而且无论`fragment`是否已保留, 他都是如此工作. 这样一来就无需考虑`fragment`带来的生命周期问题.

实现步骤 = =不想写了…. 就这样吧.