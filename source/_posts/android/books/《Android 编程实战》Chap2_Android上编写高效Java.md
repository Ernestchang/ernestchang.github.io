title: 《Android 编程实战》Chap2_Android上编写高效Java
date: 

categories: 
- android 
- 《Android 编程实战》
tags:  
- android
---


> 阅读《Android 编程实战》一书的随记笔记
> 注：本文主要参考[http://szysky.com](http://szysky.com)

## Dalvik Java和Java SE

在`Android`设备上运行的`VM`成为`Dalvik`. 适用于CPU和内存受限的移动设备. `Java SE`和`Dalvik Java`存在一些差异, 这些差异主要体现在虚拟机上.

- `Java SE`: 使用了栈机设计
- `Dalvik`: 使用了基于寄存器的机器的设计

`Android SDK`中个有一个`dx`工具, 它会把`Java SE`栈机器的字节码转换成基于寄存器的`Dalvik`机器字节码, 这个步骤平时是由`AS`自动完成的.

虽然基于寄存器的虚拟机最多可以比基于栈的虚拟机快32%, 但这只限于执行解释代码的虚拟机. 在`Android 2.2`之前, `Dalvik`虚拟机都是纯解释性的. 之后才引入了`JIT`编译器.

**JIT编译**

也称`即时编译或者动态编译`. 它在执行前把字节码翻译成本地代码, 这样的两个好处:

- 消除了那些纯解释性虚拟机的开销
- 它能对本机代码执行优化, 这通常是静态编译代码无法做到的.

另一个区别是: `Dalvik`在开机时会启动一个`zygote`的进程, 该进程会创建第一个`Dalvik`实例, 由这个实例创建所有其他的实例. 当应用程序启动的时候, `zygote`进行会收到一个创建新虚拟机实例的请求, 并给该应用程序创建一个新进程.

------

对于两个方向的API也不是一模一样, 比方说`Android`移除了`Java SE`中`Swing/AWT`包. 因为`Android`有自己的UI框架. 还有一些例如`RMI`, `CORBA`,`JMX`等.

## 优化Android上的Java代码

**1.更安全的队列**

例如如果需要实现一个**生产者/消费者**模式, 关于所使用的管理存储的集合可能需要选择队列(`LinkedList`)并且具体的存取操作要套上`synchronize`防止并发问题. 但是如果通过`LinkedBlockingQueue`就可以实现上面的手动编写的步骤. 对于做同样的事情但是更快的实现方法,我们没有例如不去使用

其实在`JDK1.5`之后, 提供了`java.util.concurrent`包, 这个包不仅有相关的类, 此外还包含信号量, 锁, 以及对单个变量进行原子操作的类.

------

**2.更好的锁**

对于Java提供的`synchronize`关键字允许创建线程安全的代码块和方法. `synchronize`关键字虽然便于使用, 但也很容易滥用, 对性能造成损耗. 当需要区分读数据和写数据时, `synchronize`也不是最有效的.

```
public class ReadWriteLockDemo {

    /**
     * 控制读写操作并发的锁
     */
    private final ReentrantReadWriteLock mLock ;
    private String mName;

    public ReadWriteLockDemo(){
        mLock = new ReentrantReadWriteLock();
    }
    
    public void setPersonData(String name){
        // 获取写操作的锁
        ReentrantReadWriteLock.WriteLock writeLock = mLock.writeLock();
        
        try {
            //  获取 写锁使用权, 其余线程需要操作只有在写锁被关闭才可以
            writeLock.lock();
            mName = name;    
        }finally {
            //  释放 写锁使用权 
            writeLock.unlock();
        }
    }
    
    public String getName(){
        ReentrantReadWriteLock.ReadLock readLock = mLock.readLock();
        
        try {
            readLock.lock();
            return mName;
        }finally {
            readLock.unlock();
        }
    }
    // 省略其余实现
}
```

上面代码允许多个并发线程对数据进行只读方法, 并确保同一时间只有一个线程写入数据.

## 管理和分配内存

Java的自动内存管理有效消除了开发过程中的许多问题. 不再需要记住为每个创建的对象进行手动释放对象内存. 当时同时也需要付出一定的性能代价. **自动垃圾收集器会和程序并行运行. 垃圾回收器会一直运行, 并检查是否有可以回收的内存. 这种行为意味着应用程序进程会和垃圾收集器竞争CPU时间,即使垃圾回收器不会运行太长时间,但是也是一种性能损失. 虽然我们不能抛弃这中垃圾收集方式, 但是可以有效的控制, 例如:**

**减少对象分配**

如果可以不要在循环内部创建对象, 在循环的外部创建对象, 并复用此对象. 减少对象的生成造成的空间开辟, 和无用对象的回收.

## Android中的多线程

确保不再`UI线程`做耗时操作, 这基本已经印在`Android`开发的脑袋里了. 那么如何知道一个方法是否在主线程执行? `Android`文档指出, **默认情况下, 应用的所有组件都运行在同一个进程和线程中(主线程)**. 更确切点的说三个组件(`Activity`, `Service`, `Broadcast`, `Application`)的所有回调基本上所有的`onXXXX()`这种命名的方法都是运行在主线程的.

------

**那么什么样的代码在主线程执行才是安全的?**

只有那些必须在主线程执行的方法才能放在主线程中. 其他一切操作都应该放在另一个单独的线程中执行. 实际情况下, 那些不会耗时的操作也可以放在主线程中. 如何能确保在另一个单独的线程中执行文件, 数据库或者网络操作, 通常主线程会是安全的. 当然也要确保同一时间不会运行太多线程, 原因是`CPU`切换线程也会造成性能损失.

### Thread

`Thread`类是`Android`中所有线程的基类, `Java SE`中也包含它. 如果要在线程中执行代码, 即可以创建一个继承, 也可以把实现`Runnable`接口的类对象传给`Thread`的构造函数.

对于这个基础的类, 尽量能不用就不用. 除非配合缓冲池, 不然频繁的创建新的线程开销很大.

### AsyncTask

这是`Android`中比较流行的一种, 因为使用简单,并且允许定义一个运行在单独线程中的任务, 并且提供了不同阶段的回调, 如`doInBackground()`会被回调在单独的线程. 而其余的三个回调都是被回调在主线程. 这都是为了让开发者可以把代码编写的重点放在实现的功能上, 而不是考虑是否应该避免系统级的问题发生.

使用`AsyncTask`创建的实例只能使用一次, 这也就意味着每次执行操作都新建一个实例对象. 虽然是个轻量级的类(实际的线程是由`ExecutorService`管理的), **但是不适合那些频繁的操作**, 因为这会快速聚集需要垃圾回收的对象, 并最终导致应用程序的卡顿.

并且, `AsyncTask`不能对操作设置执行时间, 也无法间隔一段时间去执行操作. 它适合文件下载, 以及不会频繁发生或通过用户交互等类似情况的操作. 但是使用简单嘛.

### Handler

当需要更细粒度地控制一个单独的线程中执行操作的时候, `Handler`类会是一个很有用的工具. 该类允许准确地控制操作的执行时间. 还可以重复多次的使用. 执行操作的线程会一直运行, 直到被显示地终止. `Loop`会处理幕后的事情, 但是作为开发人员却很少和其打交道.

使用`Handler`类最常见的方式是发送`Message`. 当向后台线程传递数据和参数时, 这些消息对象简单,易于创建, 并且可以重用. `Message`对象通常是由它的共有整型成员变量`what`定义的. 还有两个名为`arg1和arg2`的整型成员变量, 他们用于创建低开销的的参数, 以及`obj`成员变量(可以存储任意单个对象的引用). 如果你需要存储更复杂的数据那么通过`setData(Bundle)`方法设置更复杂的数据.

------

**发送消息**

既可以使用`Message#sendToTarget()`也可以使用`Handler#sendXXX()`等方法. 如下

```
// 创建一个带有data参数的Message, 然后立刻把它发送到handler执行
Message.obtain(mHandler, FLAG_SYNC_DATA, data).sendToTarget();

// 立刻给handler发送一个空消息
mHandler.sendEmptyMessage(FLAG_SYNC_DATA);

// 给handler发送一个空消息, 在30秒后执行
mHandler.sendEmptyMessageAtTime(FLAG_SYNC_DATA, 30 * 1000);

// 给handler发送带有arguments和obj参数的消息, 并在10秒后执行
Message msg = mHandler.obtainMessage(FLAG_SYNC_DATA, 参数1, 参数2, data);
mHandler.sendMessageDelayed(msg, 10 * 1000);
```

------

在创建`Handler`的时候构造函数传入`Looper`对象可以为`Handler`分配线程, 所以可以创建一个主线程消息的`Handler`, 避免使用`runOnUiThread()`这种丑陋缩进的代码并且还低效.

只要确保把消息发送给正确的`handler`即可.

### 选择合适的线程

之前说过了三种在`Android`上创建和使用线程的方式, `API`中和线程相关的类还有`ExecutorService`和`Loader`.

- `ExecutorService`适合处理并行运行的多个任务, 这个非常适合编写相应多客户端的服务器应用. `AsyncTask`内部同样使用了`ExecutorService`处理多线程

三种中尽量少的直接使用`Thread`类, 大多数情况下推荐使用`AsyncTask`和`Handler`类, 具体使用哪一个取决于具体的需求. 如果不是很频繁地执行操作, 比如超过每分钟一次, 那么`AsyncTask`是个选择的对象. 如果需要安排操作时间或者需要快速间隔地执行操作, `Handler`是个更好的选择. 从长远的角度来看, `Handler`生成的代码较少, 而`AsyncTask`更容易使用.