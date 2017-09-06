title: 《Android开发艺术探索》Chap5_RemoteView的应用
date: 

categories: 
- android 
- 《Android开发艺术探索》
tags:  
- android
---

> 桌面小部件和使用RemoteViews跨进程更新界面
> 注：本文主要参考[http://szysky.com](http://szysky.com)

[blog相关代码](https://github.com/suzeyu1992/Notes_AndroidDevSeek)

## RemoteView的应用

**简介:**在开发中, 通知栏都知道是通过`NotificationManager`的`notify`方法实现. 桌面小部件则是通过`AppWidgetProvider`实现. 后者本质上是一个广播.更新他们无法像以前那样.这是因为不是一个进程,小部件是`SystemServer`进程. 为了跨进程更新界面,RemoteViews提供了一系列的set方法…

### RemoteViews通知栏的应用

先使用系统默认的样式.

–! 先不记录`notification`了. 发现书上的方法在编译环境23版本以上无效. 23以下是没有问题的.
`notification.setLatestEventInfo()`此方法已经被删除了.

贴出自定义布局通知栏代码利用remoteViews

```
/**
 * 打开自定义布局的通知栏
 */
private void displayRemoteViews() {
    Notification notification = new Notification();
    notification.icon = R.mipmap.ic_launcher;
    notification.tickerText = "我是小部件";
    notification.when = System.currentTimeMillis();
    notification.flags = Notification.FLAG_AUTO_CANCEL;
    Intent intent = new Intent(getApplicationContext(), MainActivity.class);
    PendingIntent pedingIntent = PendingIntent.getActivity(getApplicationContext(), 0, intent, PendingIntent.FLAG_UPDATE_CURRENT);

    RemoteViews remoteViews = new RemoteViews(getPackageName(), R.layout.layout_notification);
    remoteViews.setTextViewText(R.id.tv_msg, "我是文字信息");
    remoteViews.setImageViewResource(R.id.iv_icon, R.mipmap.favicon);

    notification.contentView = remoteViews;
    notification.contentIntent = pedingIntent;

    PendingIntent openActivity2PendingIntent = PendingIntent.getActivity(this,
            0, new Intent(this, OpenActivity.class), PendingIntent.FLAG_UPDATE_CURRENT);
    remoteViews.setOnClickPendingIntent(R.id.tv_open, openActivity2PendingIntent);


    NotificationManager manager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
    manager.notify(2, notification);
}
```

传入了一个自定义布局里面有一个`imageView`两个`textView`. 如下图:

![img](http://szysky.com/2016/08/11/%E3%80%8AAndroid%E5%BC%80%E5%8F%91%E8%89%BA%E6%9C%AF%E6%8E%A2%E7%B4%A2%E3%80%8B-05-%E7%90%86%E8%A7%A3RemoteViews/notify01.png)

### RemoteView在桌面小部件上的应用

`AppWidgetProvider`是系统提供的用于实现桌面小部件的类, 继承`BroadcaseReceiver`.可以当成广播理解.

**桌面小部件的开发步骤**

**1.定义小部件界面**

在创建一个布局xml当做这个小部件要展示的样子

**2.定义小部件配置信息**

在`res/xml`文件夹下新建一个xxx_info.xml的文件

```
<?xml version="1.0" encoding="utf-8"?>
<appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
    android:initialLayout="@layout/layout_widget"
    android:minHeight="100dp"
    android:minWidth="100dp"
    android:updatePeriodMillis="60000"
    >
</appwidget-provider>
```

- `initiaLayout`: 小工具所要使用的初始化布局
- `minHeight``minWidth`: 指定小工具的尺寸
- `updatePeriodMillis`: 自动刷新的时间, 单位毫秒

**3.定义小部件的实现类**

```
/**
 * Created by suzeyu on 16/8/11.
 * 定义小部件的实现类
 */
public class MyAppWidgetProvider extends AppWidgetProvider {
    public static final String TAG = MyAppWidgetProvider.class.getName();
    public static final String CLICK_ACTION = "com.szysky.note.androiddevseek_05.action.CLICK";

    @Override
    public void onReceive(final Context context, Intent intent) {
        super.onReceive(context, intent);
        Log.i(TAG, "onReceive: 接收到广播-->"+intent.getAction());

        //是触发的自己点击时发送的action那么就让小部件旋转
        if (intent.getAction().equals(CLICK_ACTION)){
            Toast.makeText(context, "准备旋转", Toast.LENGTH_SHORT).show();

            AsyncTask.execute(new Runnable() {
                @Override
                public void run() {
                    Bitmap srcBmp = BitmapFactory.decodeResource(context.getResources(), R.mipmap.favicon);
                    AppWidgetManager widgetManager = AppWidgetManager.getInstance(context);

                    for (int i = 0; i < 37; i++) {
                        float degree = (i * 10) % 360;
                        RemoteViews remoteViews = new RemoteViews(context.getPackageName(), R.layout.layout_widget);
                        remoteViews.setImageViewBitmap(R.id.iv_main, rotateBmp(context, srcBmp, degree));

                        if (i==36){
                            Intent intentClick = new Intent();
                            intentClick.setAction(CLICK_ACTION);
                            PendingIntent peddingIntent = PendingIntent.getBroadcast(context, 0, intentClick, 0);
                            remoteViews.setOnClickPendingIntent(R.id.iv_main, peddingIntent);
                        }


                        widgetManager.updateAppWidget(new ComponentName(context, MyAppWidgetProvider.class), remoteViews);
                        SystemClock.sleep(50);
                    }
                }
            });


        }
    }



    /**
     * 旋转一个bitmap
     */
    private Bitmap rotateBmp(Context context, Bitmap srcBmp, float degree) {
        Matrix matrix = new Matrix();
        matrix.reset();
        matrix.setRotate(degree);
        return Bitmap.createBitmap(srcBmp, 0, 0, srcBmp.getWidth(), srcBmp.getHeight(), matrix, true);
    }

    /**
     * 当自定义的小桌面被添加 这个方法只有在本实例中只有被添加的时候才调用
     */
    @Override
    public void onUpdate(Context context, AppWidgetManager appWidgetManager, int[] appWidgetIds) {
        super.onUpdate(context, appWidgetManager, appWidgetIds);

        final int counter = appWidgetIds.length;
        Log.i(TAG, "小桌面更新了   counter="+counter);

        for (int i = 0; i < counter; i++) {
            int appWidgetID = appWidgetIds[i];
            onWidgetUpdate(context, appWidgetManager, appWidgetID);
        }
    }

    /**
     * 桌面小部件更新  这个方法只有在本实例中只有被添加的时候才调用
     */
    private void onWidgetUpdate(Context context, AppWidgetManager appWidgetManager, int appWidgetID) {
        Log.i(TAG, "onWidgetUpdate: id=="+appWidgetID);

        RemoteViews remoteViews = new RemoteViews(context.getPackageName(), R.layout.layout_widget);
        remoteViews.setImageViewBitmap(R.id.iv_main, BitmapFactory.decodeResource(context.getResources(), R.mipmap.favicon));


        Intent intentClick = new Intent();
        intentClick.setAction(CLICK_ACTION);
        PendingIntent pendingIntent = PendingIntent.getBroadcast(context, 0, intentClick, 0);
        remoteViews.setOnClickPendingIntent(R.id.iv_main, pendingIntent);
        appWidgetManager.updateAppWidget(appWidgetID, remoteViews);
    }
}
```

直接说用途把, 小部件被添加到桌面的时候, 会先走`onUpdate()`回调,这个时候执行方法通过`RemoteViews()`构建一个布局,并更新桌面上新的布局和设置了点击事件,然后走`onReceive()` .

如果当我们点击的小部件的时候, 会触发广播中的`onReceive()`然后进行图片的旋转.这就是上述代码的大体流程.

**4.最后要在清单文件中声明小部件**

```
<receiver android:name=".MyAppWidgetProvider">
            <meta-data android:name="android.appwidget.provider"
                        android:resource="@xml/widget_provider_info"/>

            <intent-filter>
                <action android:name="android.appwidget.action.APPWIDGET_UPDATE"/>
                <action android:name="com.szysky.note.androiddevseek_05.action.CLICK"/>
            </intent-filter>
            
</receiver>
```

第一个action则是作为小部件的表示而必须存在的. 第二个action就是要识别设定的单击行为.

**然后就可以在主屏幕上长按添加小部件查看效果了**

[嫌麻烦就直接扣代码,这是链接](https://github.com/suzeyu1992/Notes_AndroidDevSeek)

`AppWidgetProvider`这个类还有其他生命周期的回调, 其实就是当广播到来之后, `AppWidgetProvider`会自动根据广播的Action通过`onReceive()`来自动进行分发广播.

- `onEnable()`: 当该窗口小部件第一次添加到桌面时调用该方法, 可添加多次但只在第一次调用.
- `onUpdate()`: 小部件被添加时或者每次小部件更新时都会调用一次该方法, 小部件的更新时机由`updatePeriodMillis`来指定, 每个周期小部件都会自动更新一次.
- `onDeleted()`: 每删除一次小部件就会调用一次.
- `onReceive()`: 这是广播的内置方法, 用于分发具体的事件给其他方法.

------

> 下面就是分析了

### PendingIntent概述

PendingIntent和Intent的区别:

- `PendingIntent`: 等待意图, 有一个Intent将在某个待定的时刻发生.
- `Intent`: 是立刻发生.

**使用场景** 最典型的就是给`RemoteViews`添加单击事件, 因为`RemoteViews`运行在远程进程中, 因此`RemoteViews`不同于普通的View, 所以无法直接向View那样通过`setOnClickListener()`方法那样设置单击事件. 要想给`RemoteViews`设置单击事件, 就必须使用`PendingIntent`, PendingIntent通过`send()`和`cancel()`来发送和取消特定的待定Intent.

`PendingIntent`支持三种待定意图: 启动Activity, 启动Service, 和发送广播

对应着`PendingIntent`三个静态方法

`getActivity()`,`getService()`, `getBroadCast()`. 当这三种方法返回的`PendingIntent`待定意图发生时候, 对应的效果就是我们日常开启这三大组件的情形.

上述三个方法都需要四个参数. 需要说一下第二个参数`requestCode`和第四个参数`flags`. 其中`requestCode`表示PendingIntent发送方的请求码, 多数情况下设为0即可, 另外`requestCode`会影响到`flags`的效果.

flags: 常用的类型有: `FLAG_ONE_SHOT`, `FLAG_UPDATE_CURRENT`, `FLAG_NO_CREATE`, `FLAG_CANCEL_CURRENT`. 在此之前首先要明确一个概念, `PendingIntent`的匹配规则, 在什么情况下两个`PendingIntent`是相同的.

**PendingIntent匹配规则**: 如果两个`PendingIntent`的内部Intent相同并且requestCode也相同那么这两个`PendingIntent`就是想同的. `requestCode`是int值不需要解释.

而`Intent`匹配规则是: 如果两个Intent的`ComponentName`和`intent-filter`都相同, 那么这两个Intent就是相同的. `Extras`是不参与Intent的匹配规则.

- **FLAG_ONE_SHOT**: 当前描述的`PendingIntent`只能被使用一次, 然后它就会被自动cancle, 如果后续还有相同的`PendingIntent`, 那么它们的send方法就会调用失败. 对于通知栏消息来说, 如果采用此标记, 那么同类的通知只能使用一次, 后续的通知单击后将无法打开.
- **FLAG_NO_CREATE**: 当前描述的`PendingIntent`不会主动创建, 如果当前`PendingIntent`之前不存在, 那么getActivity, getService, getBroadcast方法会直接返回null, 即获取`PendingIntent`失败. 这个标记很少见, 它无法单独使用,因此日常中没有太多意义.
- **FLAG_CANCEL_CURRENT**: 当前描述的`PendingIntent`如果已经存在, 那么他们都会被cancel, 然后系统会创建一个新的`PendingIntent`. 对于通知栏消息来说, 那些被cancel的消息单击后将无法打开.
- **FLAG_UPDATE_CURRENT**: 当前描述的`PendingIntent`如果已经存在, 那么他们都会自动被更新, 即它们的Intent中的Extra会被换成新的.

规则说了接下来结合实际使用说明:

如果`manager.notify(1, notification)`,如果参数1的id是**常量**,那么多次调用`notify()`只能弹出一个通知, 后续的通知会把前面的通知全部替代, 如果每次id都是不一样的, 那么多次调用`notify()`就会弹出多个通知.

所以如果`notify()`是常量, 那么不管`PendingIntent`是否匹配, 后面的通知都会直接替换前面的通知.

如果`notify()`每次不同, 那么当`pendingIntent`不匹配时(这里指的匹配就是上面介绍的Intent和requestCode是否同时相同), 不管采用何种标记, 这些通知之间都不会互相干扰. **但是如果PendingIntent匹配时就要用到去按照之前说的标记区别来划分**

- **FLAG_ONE_SHOT**–> 那么后续通知中的`PendingIntent`会和第一条通知保持一致, 包括`Extras`, 单击任何一条通知后, 剩下的通知均无法再打开, 当所有的通知都被清除后, 会再次重复这个过程.
- **FLAG_CANCEL_CURRENT**–> 那么只有最新的通知可以打开, 之前弹出的所有通知均无法打开
- **FLAG_UPDATE_CURRENT**–> 那么之前弹出的通知中的`PendingIntent`会被更新, 最终他们和最新的一条通知保持完全的一致, 包括其中的`Extras`,并且这些通知都是可以打开的.

## RemoteViews的内部机制

`RemoteViews`的作用是在其他进程中显示并更新View界面.

最常用的构造函数就是`public RemoteViews(String packageName, int layoutId)`, 注意`RemoteViews`目前并不能支持所有的View类型, 目前支持如下(不包括其子类):

**Layout**

`FrameLayout`, `LinearLayout`, `RelativeLayout`, `GridLayout`

**View**

`TextView`, `ImageView`, `ImageButton`, `Button`, `AnalogClock`, `Chronometer`, `ProgressBar`, `ViewFlipper`, `ListView`, `GridView`, `StackView`, `AdapterViewFlipper`, `ViewStub`

`RemoteViews`没有提供`findviewById()`方法, 只有一系列的`set()`方法.

| 方法名                         | 作用                               |
| --------------------------- | -------------------------------- |
| `setTextViewText()`         | 设置TextView的文本                    |
| `setTextViewSize()`         | 设置TextView的字体大小                  |
| `setTextColor()`            | 设置TextView的字体颜色                  |
| `setImageViewResource()`    | 设置imageView的图片资源                 |
| `setImageViewBitmap()`      | 设置imageView的图片                   |
| `setInt()`                  | 反射调用View对象的参数类型为int的方法           |
| `setLong()`                 | 反射调用View对象的参数类型为long的方法          |
| `setBoolean()`              | 反射调用View对象的参数类型为boolean的方法       |
| `setOnClickPendingIntent()` | 为View添加单击事件, 事件类型只能PendingIntent |

**RemoteViews的工作流程**

通知栏和桌面小部件分别由`NotificationManager`和`AppWidgetManager`管理, 而这两个管理者都是通过`Binder`分别和`SystemServer`进程中的`NotificationManagerService`以及`AppWidgetService`进行通信. 由此可见,通知栏和桌面小部件中的布局文件实际上是在`NotificationManagerService`以及`AppWidgetService`中被加载的, 而他们运行在系统的`SystemServer`中, 这就和我们的进程构成了进程间通信.

最开始`RemoteViews`会通过`Binder`传递到`SystemServer`进程, `RemoteViews`实现了`Parcelable`接口. 系统根据`RemoteViews`中的包名等信息去得到该应用的资源, 然后通过`LayoutInflate`去加载`RemoteViews`中的布局文件. 在`SystemServer`进程中加载后的布局文件是一个普通的View, 只不过相对于我们的进程他是一个`RemoteViews`而已. 接着系统会对View执行一系列界面更新任务, 这些任务就是之前的设置的`set()`. set方法对View所做的更新不是立即执行, 在`RemoteViews`内部会记录所有的更新操作, 具体的执行时机要等到`RemoteViews`被加载以后才能执行, 这样`RemoteViews`就可以在`SystemServer`进程中显示, 这就是我们看到的通知栏或者桌面小部件. 当需要更新`RemoteViews`时, 我们需要调用`set`方法并通过`NotificationManager`和`AppWidgetManager`来提交更新任务, 具体的更新操作也是在`SystemServer`进程中完成的.

**为什么不支持所有的View和其操作?** 因为代价太大, View的方法太多, 另外就是大量的IPC操作会影响效率. 为了解决这个问题, 系统并没有通过Binder直接支持View的跨进程访问, 而是提供了一个`Action`的概念, `Action`代表一个View操作, `Action`同样实现了`Parcelable`接口. 系统首先将View操作封装到`Action`对象并将这些对象跨进程传输到远程进程, 接着在远程进程中执行`Action`对象中的具体操作. 在我们的应用中每调用一次`set()`, `RemoteViews`中就会添加一个对应的`Action`对象, 当我们通过`NotificationManager`和`AppWidgetManager`来提交我们的更新时, 这些Action对象就会传输到远程进程并在远程进程中一次执行. 如图

![img](http://szysky.com/2016/08/11/%E3%80%8AAndroid%E5%BC%80%E5%8F%91%E8%89%BA%E6%9C%AF%E6%8E%A2%E7%B4%A2%E3%80%8B-05-%E7%90%86%E8%A7%A3RemoteViews/remoteview.png)

远程进程通过`RemoteViews`的apply方法来进行View的更新操作, `RemoteViews`的apply方法内部则会去遍历所有的`Action`对象并调用他们的`apply`方法, 具体的View更新操作是由Action对象的apply方法来完成的. 上述做法的好处是显而易见的, 首先不需要定义大量的`Binder`接口, 其次通过远程进程中批量执行`RemoteViews`的修改操作从而避免了大量的IPC操作, 这就提高了程序的性能.

接下来从源码角度分析.

首先最长用到的`setTextViewText()`,源码如下

```
public void setTextViewText(int viewId, CharSequence text) {
   setCharSequence(viewId, "setText", text);
}
```

接收的参数比较简单,继续跟进`setCharSequence()`方法.

```
public void setCharSequence(int viewId, String methodName, CharSequence value) {
   addAction(new ReflectionAction(viewId, methodName, ReflectionAction.CHAR_SEQUENCE, value));
}
```

从这里实现看到, 内部并没有对View进程直接的操作, 而是添加一个`ReflectionAction()`一个看名字类似反射类型的对象. 接下看`addAction()`

```
private void addAction(Action a) {
    //省略部分代码...
    if (mActions == null) {
        mActions = new ArrayList<Action>();
    }
    mActions.add(a);

    // update the memory usage stats
    a.updateMemoryUsageEstimate(mMemoryUsageCounter);
}
```

这里看到, 在`RemoteViews`内部有一个`mActions`成员, 它是一个ArrayList, 外界每调用一次`set()`, `RemoteViews`就会为其创建一个`Action`对象并加入到这个集合中, 这里仅仅将`Action`对象保存了起来, 并未对View进行实际的操作, 这一点在上面的理论分析中已经提到过.

接下来再看`ReflectionAction`的实现之前, 先看一下`RemoteViews`的`apply()`方法以及Action类的实现.

```
public View apply(Context context, ViewGroup parent, OnClickHandler handler) {
        RemoteViews rvToApply = getRemoteViewsToApply(context);

        View result;
        
        final Context contextForResources = getContextForResources(context);
        Context inflationContext = new ContextWrapper(context) {
            @Override
            public Resources getResources() {
                return contextForResources.getResources();
            }
            @Override
            public Resources.Theme getTheme() {
                return contextForResources.getTheme();
            }
            @Override
            public String getPackageName() {
                return contextForResources.getPackageName();
            }
        };

        LayoutInflater inflater = (LayoutInflater)
                context.getSystemService(Context.LAYOUT_INFLATER_SERVICE);

        // Clone inflater so we load resources from correct context and
        // we don't add a filter to the static version returned by getSystemService.
        inflater = inflater.cloneInContext(inflationContext);
        inflater.setFilter(this);
        result = inflater.inflate(rvToApply.getLayoutId(), parent, false);

        rvToApply.performApply(result, parent, handler);

        return result;
    }
```

这段代码首先通过`LayoutInflate`去加载`RemoteViews`中的布局文件, `RemoteViews`中的布局文件可以通过`getLayoutId()`这个方法获得, 加载完布局文件后会通过`performApply()`去执行一些更新操作,如下:

```
private void performApply(View v, ViewGroup parent, OnClickHandler handler) {
       if (mActions != null) {
           handler = handler == null ? DEFAULT_ON_CLICK_HANDLER : handler;
           final int count = mActions.size();
           for (int i = 0; i < count; i++) {
               Action a = mActions.get(i);
               a.apply(v, parent, handler);
           }
       }
   }
```

这个实现就是遍历`mActions`并执行每个`Action`对象的`apply()`方法, 这里猜想Action对象的apply方法就是真正操作View的地方.

`RemoteViews`在通知栏和桌面小部件中的工作过程和上面描述的过程是一致的. 当调用了`RemoteViews`的set方法时, 并不会立刻更新他们的界面, 而必须要通过`NotificationManager`的`notify`方法以及`AppWidgetManager`的`updateAppWidget`才能更新他们的界面. 实际上在`AppWidgetManager`的`updateAppWidget`内部实现中, 他们就是通过`RemoteViews`的`apply`以及`reapply`方法来加载或者更新布局的. `apply`和`reApply`的区别在于:前者会加载布局并更新界面, 而后者只会更新界面. 通知栏和桌面小部件在初始化界面的时候回调用`apply()`方法, 而在后续的更新界面时则会调用`reapply()`方法.

了解了`apply()`以及`reapply()`的作用后, 接着看`Action`的子类具体实现, 先看`ReflectionAction`的具体实现.

```
private final class ReflectionAction extends Action {
   //省略部分代码    ...
  

   String methodName;
   int type;
   Object value;

   ReflectionAction(int viewId, String methodName, int type, Object value) {
       this.viewId = viewId;
       this.methodName = methodName;
       this.type = type;
       this.value = value;
   }
   @Override
   public void apply(View root, ViewGroup rootParent, OnClickHandler handler) {
       final View view = root.findViewById(viewId);
       if (view == null) return;

       Class<?> param = getParameterType();
       if (param == null) {
           throw new ActionException("bad type: " + this.type);
       }

       try {
           getMethod(view, this.methodName, param).invoke(view, wrapArg(this.value));
       } catch (ActionException e) {
           throw e;
       } catch (Exception ex) {
           throw new ActionException(ex);
       }
   }
    // ...
}
```

`ReflectionAction`表示的是一个反射动作, 通过它对View的操作会以反射的方式来调用, 其中`getMethod`就是根据方法名来得到反射所需要的`Method对象`. 除了`ReflectionAction`, 还有其他的`Action`. 例如: `TextViewSizeAction`, `ViewPaddingAction`, `SetOnClickPendingIntent`等. 看一下`TextViewSizeAction`

```
private class TextViewSizeAction extends Action {
   public TextViewSizeAction(int viewId, int units, float size) {
       this.viewId = viewId;
       this.units = units;
       this.size = size;
   }
   
   @Override
   public void apply(View root, ViewGroup rootParent, OnClickHandler handler) {
       final TextView target = (TextView) root.findViewById(viewId);
       if (target == null) return;
       target.setTextSize(units, size);
   }

   public String getActionName() {
       return "TextViewSizeAction";
   }

   int units;
   float size;

   public final static int TAG = 13;
}
```

这个类没有使用反射, 因为setTextSize的方法有两个参数,因此无法复用`ReflectionAction`, 因为这个反射调用只能有一个参数.

关于单击事件, `RemoteViews`只支持发起`PendingIntent`,不支持`onClickListener()`这种模式.

`setOnClickPendingIntent`,`setPendingIntentTemplate`,`setOnClickFillIntent`这三个的区别.

`setOnClickPendingIntent`: 只支持普通View设置点击事件, 不能给集合(`ListView`,`StackView`)中的View设置点击事件,如item. 因为开销比较大, 系统禁止了这种方式. 如果要给集合中的item添加点击事件,则必须使用后两种组合使用才可以.

## RemoteViews的意义

可以模拟一个通知栏效果并实现跨进程的UI更新.

恩, 详情去看书吧, 不写了在书的*239页*