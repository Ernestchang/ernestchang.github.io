title: 《Android开发艺术探索》Chap2_IPC机制
date: 

categories: 
- android 
- 《Android开发艺术探索》
tags:  
- android
---

> 第二章: Android中的IPC机制,深入记录Bundle, 文件共享, AIDL, Messenger, ContentProvider, Socket等进程间的通信.
>
>  注：本文主要参考[http://szysky.com](http://szysky.com)

[blog相关代码](https://github.com/suzeyu1992/Notes_AndroidDevSeek)

## Android IPC简介

IPC是**Inter-Process Communication**缩写,含义为进程间通信. 按照操作系统中的描述,线程是cpu调度的最小单元,而进程一般指一个执行单元. 进程中可以有一个或者多个线程.

不同的操作系统有着不同的IPC机制:

- **Windows**: 通过剪切板, 管道, 信号量来进行进程间通信
- **Linux**: 通过命名管道, 共享内存, 信号量等来进行进行进程间通信
- **android**: 虽然基于Linux内核,但是使用了独有的Binder机制, 也可以Socket进行通信

使用场景: 可能有些模块因为特殊原因需要运行在单独的进程中; 或者为了加大一个应用可使用的内存; 又或者我们需要去另外一个进程去获取数据,必然需要跨进程.

## Android中的多进程的模式

### 开启多进程模式

如果你想在一个应用中使用多个进程,通过清单文件给四大组件添加`android:process`属性,就可以很方便的开启多进程.

还有一种非常规的创建方式,通过JNI在native层去fork一个新的进程.这种只做了解.

![img](http://szysky.com/2016/08/02/Android%E5%BC%80%E5%8F%91%E8%89%BA%E6%9C%AF%E6%8E%A2%E7%B4%A2-%E7%AC%94%E8%AE%B002-IPC%E6%9C%BA%E5%88%B6/sample_1.png)

例如这样,当我们依次打开**MainActivity**, **SecondActivity**, **ThirdActivity**.此时应该打开了三个进程.

我们来检测一下, 你可以直接使用DDMS来查看进程,这里使用命令行来测试

`$ adb shell ps | grep com.szysky` 你可以直接使用`adb shell ps`这会把系统所有进程展示出来, 你可以加上过滤信息`| grep xxx` xxx替换你需要过滤出来信息即可

![img](http://szysky.com/2016/08/02/Android%E5%BC%80%E5%8F%91%E8%89%BA%E6%9C%AF%E6%8E%A2%E7%B4%A2-%E7%AC%94%E8%AE%B002-IPC%E6%9C%BA%E5%88%B6/psnum.png)

你可能已经发现在创建新进程的时候使用两种不同的方式

- 当以`:`开头的进程,属于当前应用的私有进程,其他应用的组件不可以和它跑在同一个进程
- 当不以`:`开头,那么进程属于全局进程,其他应用通过`ShareUID`方法可以和它跑在同一个进程

Android系统会为每一个应用分配唯一的**UID**. 相同**UID**的应用才能共享数据. 但是两个应用通过**ShareUID**跑在同一个进程是有要求的. 除了具有相同的**ShareUID**并且还要**签名相同**才可以. 这时如果不在同一进程他们之间可以**共享data目录**,**组件信息**等. 如果还在同一进程, 那么他们还能共享**内存数据**.

### 进程模式的运行机制

开启多进程简单,但是如果不能处理好其中的特性,那么受伤的总会是你.

先说第一个比较严重的问题. **静态变量不在共享**. 还是直接三个类的例子,如果在**mainActivity**中对静态变量进行修改, 在**SecondActivity**取出这个静态发现是main没修改之前的. 这说明两个进程间即使是静态属性也是无法共享.

其实这是因为这两个类运行在两个进程间,而每个单独的进程又会分配一个独立的虚拟机, 所以每个虚拟机在内存分配上有不同的地址空间.对于不同虚拟机访问同一个对象就会产生多份副本. 副本之间互相独立不干扰彼此.

一般情况下多进程可能面临的问题:

1. 静态成员和单例模式完全失效
2. 线程同步机制完全失效
3. SharedPreferences的可靠性下降
4. Application会多次创建

2中因为不是一块内存,所以不管是锁对象还是锁全局都无法保证线程同步,因为不是同一个对象. 3中因为Sp不支持两个进程同时读写,因为底层是通过读写XML文件实现的,并发可能会触发异常. 4中运行在多个进程中,那么就会创建多个虚拟机,每个虚拟机都有一个对应Application并需要启动加载这个文件.

一个应用的多进程:**它就相当于两个不同的应用采用了ShareUID的模式. 每个进程都会拥有独立的虚拟机, Application以及内存空间**

## IPC基础概念

关于IPC主要包含三方面的内容: **Serializable接口**, **Parcelable接口**, 以及**Binder**

### Serializable接口

Serializable是Java提供的一个序列化接口,这个一个空接口. 如果我们想使用只需要实现`Serializable`接口,并声明一个long类型的常量`serialVersionUID`(不声明也是可以,但是在反序列化会出现错误).

javabean 的实现

```
public class Student implements Serializable{
    public static final long serialVersionUID = 123456789L;
    //.....省略创建属性,打印等操作
}
```

序列化的代码如下图,并附上结果.

![img](http://szysky.com/2016/08/02/Android%E5%BC%80%E5%8F%91%E8%89%BA%E6%9C%AF%E6%8E%A2%E7%B4%A2-%E7%AC%94%E8%AE%B002-IPC%E6%9C%BA%E5%88%B6/serial_1.png)

好了说一下`serialVersionUID`这个属性. 即使我们不声明系统会根据当前类结构(成员变量等)生成一个hash为`serialVersionUID`, 虽然这样也可以但是如果在你把一个对象序列化的到磁盘的一个文件的时候. 对这个对象增加了一个成员变量,那么在反序列的时候就会报错. 因为当你反序列化的时候对象如果没有`serialVersionUID`还会重新计算.这时反序列化的hash和序列化的hash就不一致了.

关于根据当前类结构计算hash值,有两点需要注意:

- 静态成员变量属于类不属于对象,所以不参与序列化的过程
- 其次用`transient`关键字标记的成员变量不参与序列化的过程.

系统默认的序列化过程是可以改变的,通过实现`writeObject`和`readObject`可以重写默认的序列化和反序列化过程. 这里就不详细说明

### Parcelable接口

系统已经为我们提供了很多实现了Parcelable接口的类,他们都可以直接序列化. 例如`intent`, `Bundle`, `Bitmap`, 同时List和Map也可以序列化.前提是他们里面的每个元素都可以序列化.

实现Parcelable接口主要复写四个, 我们可以直接定义好javabean直接让AS帮我们实现.

- **writeToParcel()** 主要完成序列化功能
- **CREATOR** 主要完成反序列化
- **接收参数parcel的构造函数** 用于从序列化后的对象中创建原始对象
- **describeContents()** 几乎所有情况下都返回0,只有当前对象中存在文件描述符时返回1

**关于Parcelable和Serializable的取舍**

- Serializable: 适合序列化到设备或者序列化后通过网络传输.
- Parcelable: 主要用在内存序列化上. 不需要大量的I/O操作,所以在内存中使用高效.

### 了解Binder

- **代码层面**: Binder是Android中的一个类,它实现了IBinder接口
- **IPC角度**: Binder是Android中的一种跨进程通信方式.
- **物理设备角度**: Binder也可以认为是一种虚拟的物理设备,设备驱动是/dev/binder
- **Framework角度**: Binder是ServiceManager连接各种Manager和相应的ManagerService的桥梁.
- **Android应用角度**:Binder是客户端和服务端进行通信的媒介.

日常开发中,Binder主要用在`Service`包括`AIDL`和`Messenger`. 而普通的Service中的Binder不涉及进程间的通信,无法触及Binder的核心. 而Messenger底层其实就是AIDL.所以我们利用AIDL来分析Binder的工作机制

### 创建AIDL实例

新建三个文件 `Book.java` `Book.aidl` 和`IBookManager.aidl`

首先创建一个Book类并实现Parcelable接口,然后在这个类所在的包上右键,如图所示

![img](http://szysky.com/2016/08/02/Android%E5%BC%80%E5%8F%91%E8%89%BA%E6%9C%AF%E6%8E%A2%E7%B4%A2-%E7%AC%94%E8%AE%B002-IPC%E6%9C%BA%E5%88%B6/createAIDL.png)

如果名字不能为Book,可以先随便写一个,创建之后修改. 然后按图修改,

![img](http://szysky.com/2016/08/02/Android%E5%BC%80%E5%8F%91%E8%89%BA%E6%9C%AF%E6%8E%A2%E7%B4%A2-%E7%AC%94%E8%AE%B002-IPC%E6%9C%BA%E5%88%B6/createAIDL_2.png)

文件声明完,我们只需要重新Make一下工程就可以.`Build --> Rebuild Project或者Make Project`

![img](http://szysky.com/2016/08/02/Android%E5%BC%80%E5%8F%91%E8%89%BA%E6%9C%AF%E6%8E%A2%E7%B4%A2-%E7%AC%94%E8%AE%B002-IPC%E6%9C%BA%E5%88%B6/createAIDL_3.png)

Make之后会在app -> build -> generated -> source -> aidl -> debug -> … 出现系统自动生成好的java类. 我们需要对其进行分析

看下图了解一个大体结构

![img](http://szysky.com/2016/08/02/Android%E5%BC%80%E5%8F%91%E8%89%BA%E6%9C%AF%E6%8E%A2%E7%B4%A2-%E7%AC%94%E8%AE%B002-IPC%E6%9C%BA%E5%88%B6/aidl_1.png)

上图圈出了两个部分,部分二应该很清楚就是我们定义在aidl中的两个抽象方法. 而部分一在图上的内部类Stub写了说明. 我们自定义的两个抽象方法,在内部类中用了两个**整形int值**来标识两个抽象方法,用在`transact()`中可以识别客户端请求哪个方法.

这个继承了**IInterface**的接口的核心实现:**就是内部类Stub和Stub的内部代理Proxy**

先看一下内部类的结构图:

![img](http://szysky.com/2016/08/02/Android%E5%BC%80%E5%8F%91%E8%89%BA%E6%9C%AF%E6%8E%A2%E7%B4%A2-%E7%AC%94%E8%AE%B002-IPC%E6%9C%BA%E5%88%B6/aidl_2.png)

说明直接放图,写在代码上,在git上的根目录的aidl的java类说明文件夹也有添加了注释的类.

![img](http://szysky.com/2016/08/02/Android%E5%BC%80%E5%8F%91%E8%89%BA%E6%9C%AF%E6%8E%A2%E7%B4%A2-%E7%AC%94%E8%AE%B002-IPC%E6%9C%BA%E5%88%B6/aidl_3.png)

有两点需要注意:

1. 客户端发起远程请求时,当前线程会被挂起直到服务器进程返回数据,所以注意线程是否在意耗时
2. 由于服务端Binder方法运行在Binder线程池中,所以不管Binder方法是否耗时都应该采用同步方式,因为已经在一个线程中了

我们也可以手动实现Binder类,这里不再细说,在git仓库的项目中有一个`manual`包里面是关于手动实现Binder的代码.

**其实** 不管是手动实现Binder也好,或者AIDL文件实现Binder也好. 其实两者的工作原理都是一样的, AIDL文件的存在意义是系统为我们提供了一种快速实现Binder的工具,仅此而已.

### Binder生命状态的监听

由于Binder是运行在服务端,如果服务端进程异常终止,那么我们到服务端的Binder连接也就断裂(Binder死亡).就会导致调用失败,所里系统提供了**死亡代理的方法** 就是当Binder死亡时,我们就会收到通知,这个时候就可以重新发起连接请求而恢复连接

首先创建监听的DeathRecipient对象

```
IBinder.DeathRecipient mDeat = new IBinder.DeathRecipient() {
            // 当Binder死亡的时候,系统会回调binderDied()方法
            @Override
            public void binderDied() {
                if (mBookManager == null)
                    return ;
                //清除掉已经无用的Binder连接
                mBookManager.asBinder().unlinkToDeath(mDeat,0);
                mBookManager == null;
                //TODO  进行重新绑定远程服务
            }
        };
```

当客户端绑定远程服务成功的时候,给binder设置死亡代理

```
xxxxManager.Stub.asInterface(binder);
binder.linkToDeath(mDeat,0);
```

linkToDeath的第二个参数是个标志位,直接设0即可. 另外也可以通过Binder的方法`isBindAlive`也可以判断Binder是否死亡.

## Android的几种跨进程的方式

### 使用Bundle

由于Bundle实现了**Parcelable**接口,所以在四大组件中的三大组件(`Activity`, `Service`, `Receiver`)都支持在Intent中传递Bundle.

所以如果在一个进程中启动了另一个进程的三大组件,就可以在Bundle中附加我们需要的信息通过Intent发送出去. 当然传递的类型必须是能够被序列化的, 例如基本数据类型,实现了Parcelable和Serializable接口的对象和一些Android支持的特殊对象.

### 使用文件共享

练习代码对应仓库中的`userbundle`包中.

**文件共享适合在对数据同步要求不高的进程之间进行通信,并且要妥善的处理并发读写的问题.**

两个进程通过**读/写**同一个文件来交换数据. 例如进程A把数据写入文件中,而进程B从文件中读取出来数据.

Android是基于Linux系统, 所以对于并发读写文件可以没有限制的执行. 这里不像Windows系统,对于一个文件如果加了排斥锁将会导致其他线程无法对其进行访问.

关于这部分的练习, 在之前练习Binder的时候已经练习过了. *代码在项目中的MainActivity中*

虽然序列化反序列达到的效果是可以恢复对象里面的属性值,但是反序列每回都是一个新的对象.

**SharePreferencess**是Android提供的一个轻量级方案,通过键值对存储数据,底层采用XML文件来进行存储. 存储路径`/data/data/package name/shared_prefs`目录下. 也属于文件的一种,但是由于系统对SP的读写存在一定的缓存策略,内存中会有一份缓存,所以多进程下,系统对它的读写也就变得不可靠.

### 使用Messenger

`Messenger`(信使). 不同的进程中可以传递Message对象, 在Message中放入我们需要传递的数据,就可实现进程间传递. Messenger是一种轻量级的IPC方案,它的底层实现`AIDL`. 看一些构造函数

```
public Messenger(Handler target) {
        mTarget = target.getIMessenger();
    }
    
public Messenger(IBinder target) {
   mTarget = IMessenger.Stub.asInterface(target);
}
```

无论是IMessenger还是Stub.asInterface. 可以明显看出AIDL的痕迹.

因为Messenger对`AIDL`进行了封装,使得在使用时更加简单,并且它的处理方式是一次处理一个请求,因此服务器端不用考虑线程同步因为服务端不存在并发执行的情形.

**具体实现Messenger**

**1.服务端**

创建一个`Service`作为服务端来处理客户端的请求, 同时创建一个`Handle`并通过它来创建一个`Messenger`对象,然后在Service的`onBind()`方法中返回这个`Messenger`对象底层的`Binder`.
最后这个组件在清单文件中声明加上`android:process="com.szysky.test"`属性.已达到模拟多进程的场景

```
public class MessengerService extends Service {


    /**
     * 编写一个类继承Handler,并对客户端发来的消息进行处理操作进行添加
     */
    private static  class MessengerHandler extends Handler{
        private static final String TAG = "MessengerHandler";

        @Override
        public void handleMessage(Message msg) {
            switch (msg.what){
                //客户端发来的信息标识
                case MessengerActivity.FROM_CLIENT:
                    Log.d(TAG, "handleMessage: receive msg form clinet-->" +msg.getData().getString("msg"));

                    //对客户端进行reply回答
                    // 1. 通过接收到的到客户端的Message对象获取到Messenger信使
                    Messenger client = msg.replyTo;
                    // 2. 创建一个信息Message对象,并把一些数据加入到这个对象中
                    Message replyMessage = Message.obtain(null, MessengerActivity.FROM_SERVICE);
                    Bundle bundle = new Bundle();
                    bundle.putString("reply", "我是服务端发送的消息,我已经接收到你的消息了,你应该在你的客户端可以看到");
                    replyMessage.setData(bundle);
                    // 3. 通过信使Messenger发送封装好的Message信息
                    try {
                        client.send(replyMessage);
                    } catch (RemoteException e) {
                        e.printStackTrace();
                    }

                    break;
                default:
                    super.handleMessage(msg);
            }
        }
    }

    /**
     * 创建一个Messenger信使
     */
    private final Messenger mMessenger = new Messenger(new MessengerHandler());

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return mMessenger.getBinder();
    }
}
```

别忘了清单文件

```
<service android:name=".message.MessengerService"
            android:process="com.szysky.test"/>
```

**2.客户端**

直接用一个`activity`作为客户端, 首先绑定之前创建的服务端的`Service`, 绑定成功时通过`ServiceConnection`对象接收到服务端返回的`IBinder`,用`IBinder`对象创建一个`Messenger`. 通过这个`Messenger`就可以往服务端发送消息. 如果我们需要服务端也能够回应客户端. 那么就要在客户端同之前服务端一样通过`Handle`创建一个`Messenger`对象, 并把这个`Messenger`在通过连接成功返回的IBinder创建的Message对象通过`replyTo`参数传递给服务器. 这个服务器就可以通过`replyTo`参数来回应客户端.

```
/**
* 声明一个本进程的信使 用来监听并处理服务端传入的消息
*/
private Messenger mGetReplyMessenger =  new Messenger(new Handler(){
   @Override
   public void handleMessage(Message msg) {
       switch (msg.what){
           case FROM_SERVICE:
               Log.d(TAG, "handleMessage: 这里是客户端:::"+msg.getData().getString("reply"));
               break;
           default:
               super.handleMessage(msg);
       }
   }
});
/**
* 创建一个服务监听连接对象 并在成功的时候给服务器发送一条消息
*/
private ServiceConnection mConnection = new ServiceConnection() {
   //绑定成功回调
   @Override
   public void onServiceConnected(ComponentName name, IBinder service) {
       // 利用服务端返回的binder对象创建Messenger并使用此对象想服务端发送消息
       Messenger mService = new Messenger(service);
       Message obtain = Message.obtain(null, FROM_CLIENT);
       Bundle bundle = new Bundle();
       bundle.putString("msg", "你好啊,  我是从客户端来");
       obtain.setData(bundle);
       // 需要把接收服务端回复的Messenger通过Message的replyTo传递给服务端
       obtain.replyTo = mGetReplyMessenger;
       try {
           mService.send(obtain);
       } catch (RemoteException e) {
           e.printStackTrace();
       }
   }
   @Override
   public void onServiceDisconnected(ComponentName name) {

   }
};


//进行远端服务的连接
 Intent intent = new Intent(MessengerActivity.this, MessengerService.class);
 bindService(intent, mConnection, Context.BIND_AUTO_CREATE);
```

[**本例测试代码在仓库对应项目的message包中**](https://github.com/suzeyu1992/Notes_AndroidDevSeek)

在使用Messenger进行数据传递必须将数据放入到Message中. 而Messenger和Message都实现了序列化接口. 所以可以在进程间通信.

`Message`的能使用的载体只有**what**, **arg1**, **arg2**, **Bundle** 以及**replyTo**. 这里有一个载体需要注意**object**,它在同一个进程很实用,但是在版本2.2之前是不支持跨进程的,虽然进行了改进之后,但是也只是支持系统提供实现的某些对象才可以. 所以使用的时候需要注意.

顺便绘制了一个Messenger通信的流程图, 可以对代码的调用顺序理解的更清楚.

![img](http://szysky.com/2016/08/02/Android%E5%BC%80%E5%8F%91%E8%89%BA%E6%9C%AF%E6%8E%A2%E7%B4%A2-%E7%AC%94%E8%AE%B002-IPC%E6%9C%BA%E5%88%B6/messenger.png)

### 使用AIDL

虽然Messenger使用方便, 但是要清楚它是以串行的方式处理客户端发来的消息,如果有大量并发的请求. 或者需求是跨进程调用服务端的方法时. 就无法使用Messenger. 这个时候就该**AIDL**

对于使用AIDL的流程简单梳理一遍

**服务端**

服务端创建一个`Service`用来监听客户端的连接请求, 然后创建一个AIDL文件,将暴露给客户端的接口在这个AIDL文件中声明,最后在Service中实现这个AIDL接口并在`onBind()`返回即可.

**客户端**

绑定服务端的Service,绑定成功后,将服务端返回来的Binder对象转成AIDL接口所属的类型,接着就可以直接调用AIDL中的方法了.

[**AIDL相关代码在仓库项目的java路径下的aidl包中,包中可能是包含实现订阅的代码,可以通过git回退到’使用AIDL实现简单的进程通信’版本查看基础实现**](https://github.com/suzeyu1992/Notes_AndroidDevSeek)

详细代码在上面仓库中都有,这里不全部贴出来,记录一下需要注意的地方.

**AIDL中所支持的类型**

- 基本数据类型
- String 和 CharSequence
- List: 只支持ArrayList, 里面每个元素都必须能被AIDL支持
- Map: 只支持HashMap, 里面的每个元素都必须被AIDL支持
- Parcelable: 所有实现了Parcelable接口的对象
- AIDL: 所有的AIDL接口本身也可以在AIDL文件中使用

**这里请注意,上面支持类型中Parcelable和AIDL比较特殊,自定义的Parcelable对象和AIDL对象必须要显示的import引入, 这是AIDL的规范需要遵循, 如下Book类**

```
import com.szysky.note.androiddevseek_02.aidl.Book;//必须Book的全限定名
interface IBookManager {
   List<Book> getBookList();
   void addBook(in Book book);  //这里标明输入型
}
```

**如果用到了自定义对象实现了Parcelable那么就需要创建一个同名的aidl文件**

```
package com.szysky.note.androiddevseek_02.aidl;

parcelable Book;
```

AIDL中除了基本数据类型外,其他类型的`参数`必须标上方向**out** , **in**, **inout**.分别表示输入,输出,输入输出型. 按需而定可以节省不必要的操作在底层实现的开销. 最后一点AIDL接口中只支持方法,不支持声明静态常量.

在服务端用了`CopyOnWriteArrayList`数组来保存所有书籍. 这个集合的特性是**支持并发读写**. 在说Binder的时候提到过, AIDL方法是在服务端Binder线程池中执行的, 所以当多个客户端同时连接,会存在多线程并发的问题. 所以使用`CopyOnWriteArrayList`集合可以进行自动的线程同步.与之相似的还有`ConcurrentHashMap`这个在**LRU机制中使用到过**

这里有知识点. 之前说过AIDL中能过使用的只有ArrayList. 而`CopyOnWriteArrayList`也并不是ArrayList的子类. 其实AIDL所支持的是抽象的List, 而List只是一个接口, 虽然服务端返回的是`CopyOnWriteArrayList`,但是在Binder中会按照List的规范去访问数据并最终形成一个新的ArrayList传递给客户端.

看下面的log图在客户端接收返回的`CopyOnWriteArrayList`实际上是ArrayList类型

![img](http://szysky.com/2016/08/02/Android%E5%BC%80%E5%8F%91%E8%89%BA%E6%9C%AF%E6%8E%A2%E7%B4%A2-%E7%AC%94%E8%AE%B002-IPC%E6%9C%BA%E5%88%B6/aidl_4.png)

git仓库的代码的`aidl包中` 最后保存的是实现了客户端和服务端的观察者模式(可以通过git版本切换之前代码), 通过客户端注册监听接口,
在服务端每当有新书来的时候,通知已经注册了的客户端.

**需要注意的几点**

**线程问题**

当有新书的时候,服务端回调的是客户端实现的接口里面的方法. 这个方法实际是在客户端的线程池中执行的. 所以要处理处理UI的问题, 解决方案可以创建一个Handler,将其切换到客户端的主线程中

```
private INewBookArrivedListener mNewBookListener = new INewBookArrivedListener.Stub() {
        @Override
        public void onNewBookArrived(Book book) throws RemoteException {
            // 如果有新书 那么此方法会被回调,  并且由于调用处服务端的Binder线程池, 所以给主线程的Handler发送消息,以切换线程
            mhandler.obtainMessage(NEW_BOOK_ARRIVED, book).sendToTarget();
        }
    };
```

**对象不一致,导致接触绑定失败**

服务端不能再用**CopyOnWriteArrayList**来记录绑定过的客户端. 因为这里一定要清楚**对象是不能跨进程的**当我们客户端注册监听传入一个监听对象到服务端, 在解绑的时候再次传入一个进行判断与注册时相同的对象时删除达到解除绑定效果时是无效的. 因为服务端在注册和解绑的时候是两个反序列化的对象完全不一致.

`RemoteCallbackList`是系统专门提供的用于删除跨进程listener的接口. 接收的是一个泛型,支持管理任意的AIDL接口,从声明就可以看出, 因为AIDL接口都继承`IInterface`

内部实现是一个Map结构 key是`IBinder`类型, value是`Callback`类型.

```
ArrayMap<IBinder, Callback> mCallbacks= new ArrayMap<IBinder, Callback>();

IBinder key = listener.asBinder();
Callback value = new Callback(listener, cookie);    //这里的Callback封装了真正的监听对象
```

不管是注册还是解注册,多进程到服务端都会生成不同的对象. 但是这些不同的对象有一个共同点, 底层的Binder对象是同一个, 利用这个特性可解决上面的问题.

`RemoteCallbackList` 当客户端进程终止后, 它能够自动移出客户端所注册的listener. 并且内部实现了线程同步的功能, 所以在注册和解注册的时候不需要做额外的线程工作.

在使用的使用,虽然名字有List但是他并不是一个List我们要遍历的通知监听者的时候,要使用**bigenBroadcast**和**finishBroadcase**成对出现.

```
//遍历集合  去调用客户端方法
int N = mListeners.beginBroadcast();

for (int i = 0; i<N; i++){
  INewBookArrivedListener listener = mListeners.getBroadcastItem(i);
  if (listener != null){
      listener.onNewBookArrived(newBook);
  }
}

mListeners.finishBroadcast();
```

当客户端调用远程服务的方法,被调用的方法运行在服务端的Binder线程池中,同时**客户端会被挂起**, 所以你如果在主线程(客户端的`onServiceConnected`和`onServiceDisconnected`就是UI线程)调用服务端的耗时方法, 你多点几次就很容易出现ANR. 比方说在服务端的`getBookList()`睡上十秒,可以复现ANR.

监听死亡状态: 在整理Binder的时候有说了一种**DeathRecipient**的方式,下两种都可以

- `onServiceDisconnected()` UI线程被回调
- `binderDied()` 在客户端的Binder线程池中被回调

还记得在绑定的时候`bindService(intent,mConnection, Context.BIND_AUTO_CREATE);`其中参数3如果设置这个模式, 当服务或线程死亡,还会重新启动的.

**权限验证**

1. 在服务端的**onBinder()**回调中判断权限.
2. 在服务端实现的AIDL接口中的**onTransact()**进行包名判断或者权限

**第一种:**

先清单文件中注册一个自定义的权限

```
<permission
        android:name="com.szysky.permission.ACCESS_BOOK_SERVICE"
        android:protectionLevel="normal"/>
```

在清单文件中添加这个权限的使用资格

```
<uses-permission android:name="com.szysky.permission.ACCESS_BOOK_SERVICE"/>
```

然后在**onBinder()**进行判断,如果没有那么就返回null, 这样客户端是无法绑定服务的

```
public IBinder onBind(Intent intent) {
    //做一下权限的验证  在清单文件中声明了一个,  并添加了使用权限
    int check = checkCallingOrSelfPermission("com.szysky.permission.ACCESS_BOOK_SERVICE");
    if (check == PackageManager.PERMISSION_DENIED){
      return  null;
    }
    return mBinder;
}
```

**第二种**

可以判断客户端的包名是否满足我们的需求,这里用**com.szysky**开头为例. 如果不符合方法返回false.那么调用服务的方法也会失效

```
@Override
public boolean onTransact(int code, Parcel data, Parcel reply, int flags) throws RemoteException {
  String packageName = null;
  String[] packagesForUid = getPackageManager().getPackagesForUid(getCallingUid());

//下面是获得客户端的包名
  if (packagesForUid != null && packagesForUid.length >0){
      packageName = packagesForUid[0];
  }
  Log.e(TAG, "onTransact: -----------------------------" + packageName);
  if (!packageName.startsWith("com.szysky")){
      return false;
  }

  return super.onTransact(code, data, reply, flags);
}
```

### 使用ContentProvider

[**相关代码在仓库项目的java路径下的provider包中**](https://github.com/suzeyu1992/Notes_AndroidDevSeek)

`ContentProvider`是Android提供专门用于不用应用进行数据共享的方式. 它的底层同样也是Binder. 因为系统封装, 所以它的使用比起AIDL要简单很多.

要实现一个内容提供者, 只需要写一个类继承**ContentProvider**,并复写六个抽象方法. 其中有四个是CURD操作方法. 一个**onCreate()**用来做初始化. 一个**getType()**用来返回一个Uri请求所对应的MIME类型,比如图片还是视频等. 如果我们不关心那么可是直接返回`NULL`或者`*/*`.

这六个方法根据Binder工作原理,都是运行在ContentProvider的进程中. 除了**onCreate()**是被系统回调运行在主线程, 其余的都在Binder的线程池中.

主要存储方式是**表格**的形式, 也可以支持**文件**格式,例如图片视频, 可以返回这类文件的句柄给外界来访问ContentProvider中的文件信息.

**清单文件的声明**

```
<provider
           android:authorities="com.szysky.note.androiddevseek_02.provider"
           android:name=".provider.BookProvider"
           android:permission="com.szysky.PROVIDER"/>
```

- `authorities`: 后面的值是指定这个ContentProvider的唯一标识.
- `permission`: 添加一个权限认证, 对于访问者必须添加了这个使用权限的声明.

查询的时候通过`Uri`对`authorities`声明值得解析就可以找到对应的ContentProvider

```
Uri uri = Uri.parse("content://com.szysky.note.androiddevseek_02.provider");
getContentResolver().query(uri, null, null, null, null);
```

为了后续操作, 这里利用**SQLiteOpenHelper**来管理数据库,并创建两个表**user**和**book**,代码在仓库有,这里不写实现过程.

由于有两个表支持被访问, 所以应该为每一个不同的表设定单独的Uri和Uri_Code 并将其关联. 这样外界访问的时候可以根据Uri得到Uri_Code. 也就在ContentProvider知道要处理的具体事件.

在新建的**ContentProvider**类中进行关联, 如下

```
private static final String AUTHORITY = "com.szysky.note.androiddevseek_02.provider";

    /**
     * 指定两个操作的Uri
     */
    private static final Uri BOOK_CONTENT_URI = Uri.parse("content://" +AUTHORITY + "/book");
    private static final Uri USER_CONTENT_URI = Uri.parse("content://" +AUTHORITY + "/user");

    /**
     * 创建Uri对应的Uri_Code
     */
    private static final int BOOK_URI_CODE = 1;
    private static final int USER_URI_CODE = 2;

    /**
     * 创建一个管理Uri和Uri_Code的对象
     */
    private static final UriMatcher sUriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
    
    static {
        //进行关联
        sUriMatcher.addURI(AUTHORITY, "book",BOOK_URI_CODE);
        sUriMatcher.addURI(AUTHORITY, "user",USER_URI_CODE);
    }
```

针对**query**方法进行演示,其他三个类似,代码有全部实现的例子, 在**自定义Provider**文件中.

```
/**
* 通过自动以的Uri来判断对应的数据库表名
*/
private String getTableName(Uri uri){
   String tableName = null;
   switch (sUriMatcher.match(uri)){
       case BOOK_URI_CODE:
           tableName = DbHelper.BOOK_TABLE_NAME;
           break;

       case USER_URI_CODE:
           tableName = DbHelper.USER_TABLE_NAME;
           break;
       default:break;
   }
   return tableName;
}

/**
 *   在query中, 获取到Uri传入要查询的具体表名, 使用SQLiteOpenHelper来进行query的查询,并把结果返回
 */
public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder) {
        //获取表名
        String tableName = getTableName(uri);
        if (tableName == null)
            throw new IllegalArgumentException("不被支持的Uri参数-->"+uri );
        return mDb.query(tableName, projection, selection, selectionArgs, null, null, sortOrder,null);

    }
```

如果需要监听**Provider**内容的变化, 那么可以在**Provider中**`update`, `delete`, `insert`. 中操作完数据库之后. 使用`getContentResolver().notifyChange(uri,null);`来通知外界当前ContentProvider中数据已经发生改变. 而外部要想观察其变化. 使用`ContentResolver`的`rigisterContentObserver`方法来注册观察者.

**线程安全问题**

如果只有一个**SQLiteDataBase**对象被使用, 那么增删改查不会出现线程安全问题, 因为其内部对数据库的操作是有同步处理. 但是如果多个**SQLiteDataBase**对象来操作数据库就无法保证其线程安全. 这个时候就要注意了.

### 使用Socket

Socket也称为套接字. 是网络通信中的概念, 它分为流式套接字和用户数据包套接字两种. 分别对应于网络的传输控制层中TCP和UDP协议.

使用Socket通信, 需要在清单文件添加权限的申请

```
<!--Socket通信额外需要的线程-->
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
<uses-permission android:name="android.permission.INTERNET"/>
```

**服务端** 需要启动服务, 并且在线程中建立`TCP`服务, 然后监听一个**端口**. 当客户端建立连接的时候就会生成一个`Socket`流. 可以保持后续的持续通信. 每一个客户端都对应一个Socket. 如果客户端断开连接. 服务端需要做好相应的Socket流关闭并结束通话线程. 可以通过在客户端断开的时候服务端的接收字节流会是**null**来判断连接是否还存活.

首先需要开启一个新线程, `Runnable`接口这样实现, 以下是伪代码, 没有捕捉异常

```
// 监听 3333 端口
 ServerSocket serverSocket = new ServerSocket(3333);
// 判断服务是否断开 没有断开就继续监听端口         
  while (!mIsServiceDestoryed) {
        //这个是阻塞方法, 当有新的客户端连接,才会返回Socket值
        final Socket accept = serverSocket.accept();
        // 有了新的客户端 那就需要创建一个新的线程去维护
        new Thread() {
            public void run() {
                //  这里做对一个Socket的具体操作
                responseClient(accept);
            }}.start();
  }
```

```
private void responseClient(Socket client) throws IOException {
        //接收客户端消息
        BufferedReader in = new BufferedReader(new InputStreamReader(client.getInputStream()));
        
        //发送到客户端 , 设置true参数就不需要手动的刷新输出流
        PrintWriter out = new PrintWriter(new BufferedWriter(new OutputStreamWriter(client.getOutputStream())),true);

        out.println("欢迎来到直播间");

        //判断服务标志是否销毁, 没有销毁那么就一直监听此链接的Socket流
        while (!mIsServiceDestoryed) {
            String str = in.readLine(); //这是一个阻塞方法
            
            //判断如果取出来的是null,那么就说明连接已经断开
            if (str == null)
                break;

            // 对客户端进行回复
            out.println("我是回复消息");
        }
        //准备关闭一系列的流
        ......
    }
```

最后就是要在`onDestroy()`中把循环中判断服务存活的标识置为`false`, 让开启的线程都能自动走完关闭.

**客户端**

在`onCreate()`先startService开启TCP的服务, 然后开启一个线程准备连接**Socket**. 可以加上失败重连的的机制. 只要获取到了**Socket**, 就和之前服务端一样获取输入输出流进行对应的操作.

贴出客户端的核心代码, Runnable接口实现的 ,同样是伪代码

```
Socket socket = null;
// 试图连接服务器, 如果失败休眠一秒重试
while(socket == null){
  try {
      // 如果可以连接 3333 端口成功那么socket就不为null, 此循环也就结束
      socket = new Socket("localhost", 3333);
      mClientSocket = socket;
      // 获得输出流, 并设置true,自动刷新输出流里面的内容
      mPrintWrite = new PrintWriter(new BufferedWriter(new OutputStreamWriter(mClientSocket.getOutputStream())),true );
  } catch (IOException e) {
      SystemClock.sleep(1000);
      e.printStackTrace();
  }
}


//准备接收服务器的消息.
 BufferedReader in = new BufferedReader(new InputStreamReader(mClientSocket.getInputStream()));
  //获得了socket流的读入段  只要activity不关闭一直循环读
  while(!SocketActivity.this.isFinishing()){
      // readLine()同样也是阻塞方法
      String strLine = in.readLine();
      if (strLine != null){
          //TODO 获取到了服务端发来的数据, 做一些事情
          //....
      }
  }

//走到这里 说明界面已经不存在, 进行扫尾动作
in.close();
mPrintWrite.close();
socket.close();
```

demo图例**Socket相关代码存在仓库的socket包中**

![img](http://szysky.com/2016/08/02/Android%E5%BC%80%E5%8F%91%E8%89%BA%E6%9C%AF%E6%8E%A2%E7%B4%A2-%E7%AC%94%E8%AE%B002-IPC%E6%9C%BA%E5%88%B6/socket.png)

## 各种IPC的差异以及选择

| 名称                | 优点                           | 缺点                                       | 使用场景                               |
| ----------------- | ---------------------------- | ---------------------------------------- | ---------------------------------- |
| `Bundle`          | 简单易用                         | 只能传输Bundle支持的数据类型                        | 四大组件间的进程间通信                        |
| `文件共享`            | 简单易用                         | 不适合高并发场景,并且无法做到进程间的即时通                   | 无法并发访问情形, 交换简单的数据实时性不高的场景          |
| `AIDL`            | 功能强大                         | 使用稍复杂,需要处理好线程同步                          | 一对多通信且有RPC需求                       |
| `ContentProvider` | 在数据源访问方面功能强大,支持一对多并发数据共享     | 可以理解为受约束的AIDL,主要提供数据源的CRUD操作             | 一对多的进程间的数据共享                       |
| `Messenger`       | 功能一般, 支持一对多串行通信,支持实时通信       | 不能很好处理高并发,不支持RPC,数据通过Message进行传输, 因此只能传输Bundle支持的数据类型 | 低并发的一对多即时通信,无RPC需求,或者无需要返回结果的RPC需求 |
| `Socket`          | 功能强大,可以通过网络传输字节流,支持一对多并发实时通信 | 实现细节稍微有点繁琐,不支持直接的RPC                     | 网络数据交换                             |