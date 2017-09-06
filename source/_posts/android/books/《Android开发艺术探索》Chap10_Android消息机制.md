title: 《Android开发艺术探索》Chap10_Android消息机制
date: 

categories: 
- android 
- 《Android开发艺术探索》
tags:  
- android
---

> Handler究竟是如何运行的.
> 注：本文主要参考[http://szysky.com](http://szysky.com)

[blog相关代码](https://github.com/suzeyu1992/Notes_AndroidDevSeek)

Android的消息机制主要指`Handler`的运行机制, `Handler`的运行需要底层的`MessageQueue`和`Lopper`的支撑. **MessageQueue消息机制**, 内部存储了一组消息, 以队列的形式对外提供插入和删除工作. 虽然叫消息队列,但是其内部存储结构并不是真正的队列,而是采用单链表的数据结构来存储消息列表. **Looper消息循环**, 因为`MessageQueue`本质只是一个消息的存储单元, 它不能去处理消息, 而`Looper`就是为实现处理而产生的. `Looper`会以无限循环的形式去查找是否有新消息, 如果有的话就处理消息, 否在就一直等待.

`Looper`中有一个特殊的概念`ThreadLocal`, `ThreadLocal`并不是一个线程, 它可以在每个线程中进行数据的存储. 我们使用的Handler创建的时候都会采用当前线程的`Looper`来构造消息循环系统, 而这个**当前线程**就是通过`ThreadLocal`来进行操作的.

有一点需要记住: 一个新的线程默认是没有`Looper`的, 如果要使用`Handler`就必为线程创建`Looper`, 而我们使用主线程的时候之所以不需要创建`Looper`是因为. `UI线程也就是ActivityThread`在被创建的时候就会初始化`Looper`, 所以我们在主线程也就可以直接使用Handler的原因.

## Android 的消息机制描述

在日常开发中如果不小心在子线程更新了UI那么就会抛出异常, 这一步骤是在`ViewRootImpl#checkThread()`方法完成的.

```
void checkThread() {
   if (mThread != Thread.currentThread()) {
       throw new CalledFromWrongThreadException(
               "Only the original thread that created a view hierarchy can touch its views.");
   }
}
```

那Handler可以认为是为了在子线程处理完操作可以切回到主线程进行UI操作的用途. 那为什么不能再子线程进行UI处理呢?

这是因为Android的**UI线程控件**不是线程安全, 如果在多线程中并发的访问可能会导致UI控件处于不可预期的状态, 虽然可以用加锁的形式让上述问题消失, 但是有两个弊端:

1. 加锁机制会让UI访问的逻辑变得复杂
2. 锁机制会降低UI的访问的效率, 因为锁机制会阻塞某些线程的执行.

因为这样, 最简单且高效的方法就是采用单线程模型来处理UI操作.

当`Handler`, `Looper`, `MessageQueue`都准备好之后. 就可以通过`Handler#post()`方法将一个`Runnable`投递到`Handler内部的Looper`中去处理, 也可以通过`send()`方法来发送一个消息, 这个消息同样会在`Looper`中处理. 而post()内部也是通过send()来发送的.

## Android消息机制分析

### ThreadLocal的工作原理

`ThreadLocal`是一个线程内部的数据存储类, 通过他可以在指定的线程中存储数据. 数据存储以后, 只能在指定线程中可以获取到存储的数据. 而其他线程无法获取.

而Android中的`Looper` ,`ActivityThread`, `AMS`都用到了**ThreadLocal**. 可以这样来说: 当某些数据是以线程为作用域并且不同线程具有不同的数据副本的时候, 可以采用`ThreadLocal`.

可以自己用三个线程分别对一个`ThreadLocal`对象进行操作, 虽然操作的是一个对象, 但是每个线程在获取值的时候却不相同. 这是因为: 不同的线程访问同一个`ThreadLocal#get()`方法的时候, `ThreadLocal`内部会从各自的线程中取出一个数组, 然后再从数组中根据当前`ThreadLocal`的索引去查找出对应的value值. 可以看出不同的线程中的数组是不相同的. 所以也就是为什么使用`ThreadLocal`可以在不同的线程中维护一套数据的副本并且彼此互不影响.

**还是用源码来梳理流程**

`ThreadLocal`是一个泛型类, 它的定义为`public class ThreadLocal<T>`, 看一下set()或者get()就明白了.

```
public void set(T value) {
   Thread currentThread = Thread.currentThread();
   Values values = values(currentThread);
   if (values == null) {
       values = initializeValues(currentThread);
   }
   values.put(this, value);
}
```

首先会通过`value()`方法来获取当前线程中的`ThreadLocal`数据. 就是在`Thread`类的内部有一个成员专门用于存储线程的`ThreadLocal`的数据`ThreadLocal.Values localValues`. 如果value的值为null那么就会进行初始化, 初始化结束之后再将`ThreadLocal`的值进行存储. 具体看一下是如何在`localValues`中进行存储的.

在`localValues`中有一个数组`private Object[] table`, `ThreadLocal`的值就存在在这个table数组中. 看一下`put()`方法.

```
void put(ThreadLocal<?> key, Object value) {
  cleanUp();

  // Keep track of first tombstone. That's where we want to go back
  // and add an entry if necessary.
  int firstTombstone = -1;

  for (int index = key.hash & mask;; index = next(index)) {
      Object k = table[index];

      if (k == key.reference) {
          // Replace existing entry.
          table[index + 1] = value;
          return;
      }

      if (k == null) {
          if (firstTombstone == -1) {
              // Fill in null slot.
              table[index] = key.reference;
              table[index + 1] = value;
              size++;
              return;
          }

          // Go back and replace first tombstone.
          table[firstTombstone] = key.reference;
          table[firstTombstone + 1] = value;
          tombstones--;
          size++;
          return;
      }

      // Remember first tombstone.
      if (firstTombstone == -1 && k == TOMBSTONE) {
          firstTombstone = index;
      }
  }
}
```

这里只说看得出的规则, 不需要具体看实现的算法. 规则就是`ThreadLocal`的值在`table`数组中的存储位置总是为`ThreadLocal`的**reference**字段所标示的对象的下一个位置. 例如如果`ThreadLocal的reference`对象在table中的索引为`index`, 那么`ThreadLocal`的值在table数组中的索引就是`index+1`.

接着看`get()`方法

```
public T get() {
       // Optimized for the fast path.
       Thread currentThread = Thread.currentThread();
       Values values = values(currentThread);
       if (values != null) {
           Object[] table = values.table;
           int index = hash & values.mask;
           if (this.reference == table[index]) {
               return (T) table[index + 1];
           }
       } else {
           values = initializeValues(currentThread);
       }

       return (T) values.getAfterMiss(this);
   }
```

而`ThreadLocal#get()`方法, 同样是取出当前线程的`Values`对象, 同样的通过reference的角标+1来获取. 如果取出的`Values`对象为null那么就返回初始值, 初始值由`ThreadLocal#initialValue()`方法来描述, 默认情况下为null(在Android 6.0的源码已经不直接返回null, 而是创建一个Values来返回). 也可以重写这个方法.

从上面这两个方法也可以看出, 他们所操作的对象都是当前线程的`localValues`对象的`table`数组. 因为不同的线程中访问同一个`ThreadLocal#set() get()`他们对`ThreadLocal`所做的读写操作仅限于各自线程的对应数据.

### 消息队列的工作原理

消息队列在android中指的是`MessageQueue`, `MessageQueue`主要包含两个操作: 插入和读取. 读取操作本身会伴随着删除的操作, 插入和读取对应的方法分别为`enqueueMessage()`,`next()`. 前者就是往消息队列中插入一条消息, 而后者就是取出一条消息并将其从消息队列中消除. 上面说过虽然`MessageQueue`称为消息队列, 但是内部实现使用的是单链表的数据结构来维护的消息列表. 单链表在插入和删除上比较有优势.

```
boolean enqueueMessage(Message msg, long when) {
// 省略...
synchronized (this) {
  if (mQuitting) {
      IllegalStateException e = new IllegalStateException(
              msg.target + " sending message to a Handler on a dead thread");
      Log.w(TAG, e.getMessage(), e);
      msg.recycle();
      return false;
  }

  msg.markInUse();
  msg.when = when;
  Message p = mMessages;
  boolean needWake;
  if (p == null || when == 0 || when < p.when) {
      // New head, wake up the event queue if blocked.
      msg.next = p;
      mMessages = msg;
      needWake = mBlocked;
  } else {
      needWake = mBlocked && p.target == null && msg.isAsynchronous();
      Message prev;
      for (;;) {
          prev = p;
          p = p.next;
          if (p == null || when < p.when) {
              break;
          }
          if (needWake && p.isAsynchronous()) {
              needWake = false;
          }
      }
      msg.next = p; // invariant: p == prev.next
      prev.next = msg;
  }
  if (needWake) {
      nativeWake(mPtr);
  }
}
return true;
}
```

可以看到`enqueueMessage`的实现主要操作就是单链表的插入操作. 继续看一下`next()`方法.

```
Message next() {
   final long ptr = mPtr;
   int pendingIdleHandlerCount = -1; // -1 only during first iteration
   int nextPollTimeoutMillis = 0;
   for (;;) {
       if (nextPollTimeoutMillis != 0) {
           Binder.flushPendingCommands();
       }
       nativePollOnce(ptr, nextPollTimeoutMillis);
       synchronized (this) {
           // Try to retrieve the next message.  Return if found.
           final long now = SystemClock.uptimeMillis();
           Message prevMsg = null;
           Message msg = mMessages;
           if (msg != null && msg.target == null) {
             do {
                   prevMsg = msg;
                   msg = msg.next;
               } while (msg != null && !msg.isAsynchronous());
           }
           if (msg != null) {
               if (now < msg.when) {
                   // Next message is not ready.  Set a timeout to wake up when it is ready.
                   nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
               } else {
                   // Got a message.
                   mBlocked = false;
                   if (prevMsg != null) {
                       prevMsg.next = msg.next;
                   } else {
                       mMessages = msg.next;
                   }
                   msg.next = null;
                   if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                   msg.markInUse();
                   return msg;
               }
           } else {
               // No more messages.
               nextPollTimeoutMillis = -1;
           }

           // Process the quit message now that all pending messages have been handled.
           if (mQuitting) {
               dispose();
               return null;
           }
           if (pendingIdleHandlerCount < 0
                   && (mMessages == null || now < mMessages.when)) {
               pendingIdleHandlerCount = mIdleHandlers.size();
           }
           if (pendingIdleHandlerCount <= 0) {
               // No idle handlers to run.  Loop and wait some more.
               mBlocked = true;
               continue;
           }

           if (mPendingIdleHandlers == null) {
               mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
           }
           mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
       }

       // Run the idle handlers.
       // We only ever reach this code block during the first iteration.
       for (int i = 0; i < pendingIdleHandlerCount; i++) {
           final IdleHandler idler = mPendingIdleHandlers[i];
           mPendingIdleHandlers[i] = null; // release the reference to the handler

           boolean keep = false;
           try {
               keep = idler.queueIdle();
           } catch (Throwable t) {
               Log.wtf(TAG, "IdleHandler threw exception", t);
           }

           if (!keep) {
               synchronized (this) {
                   mIdleHandlers.remove(idler);
               }
           }
       }

       // Reset the idle handler count to 0 so we do not run them again.
       pendingIdleHandlerCount = 0;

       // While calling an idle handler, a new message could have been delivered
       // so go back and look again for a pending message without waiting.
       nextPollTimeoutMillis = 0;
   }
}
```

`next()`是一个无限循环的方法, 如果消息队列中没有消息, 那么next方法会一直阻塞在这里. 当有新消息到来时, next方法会返回这条消息并将其从单链表中移除.

### Looper的工作原理

`Looper`在Android的消息机制中扮演者消息循环的角色, 具体来说就是他会不停地从`MessageQueue`中查看是否有新消息. 如果有新消息就会处理. 否则就一直阻塞在那里. 先从构造方法开始, 在构造方法中他会创建一个`MessageQueue`即消息队列, 然后将当前线程的对象保存起来.

```
private Looper(boolean quitAllowed) {
   mQueue = new MessageQueue(quitAllowed);
   mThread = Thread.currentThread();
}
```

构造函数是**私有权限**, 而内部使用的地方就是`Looper#prepare()`方法.

```
private static void prepare(boolean quitAllowed) {
   if (sThreadLocal.get() != null) {
       throw new RuntimeException("Only one Looper may be created per thread");
   }
   sThreadLocal.set(new Looper(quitAllowed));
}
```

这也就是为什么在使用`Handler`之前要有`Looper`的节奏, 而当调用了`Looper.prepare()`. 就不会出现异常的原因.

`Looper`除了`prepare()`方法外, 还提供了`prepareMainLooper()`方法, 这个方法主要是给主线程也就是`ActivityThread`创建Looper使用的. 本质也是通过`prepare()`来实现的. 由于主线程的Looper比较特殊, 所以`Looper`提供了一个`getMainLooper()`方法, 通过它可以在任何地方获取到主线程的Looper.

**Looper的退出:**

- `quit()`: 这个方法会直接退出`Looper`
- `quitSafely()`: 设定一个退出标记, 然后把消息队列中的已有消息处理完毕后才安全的退出.

如果`Looper`退出, 通过`Handler`发送的消息会失败, 这个时候`Handler`发送的消息会失败, 而`Handler#send()`方法这个时候回返回**false**. 在子线程中, 如果手动为其创建了`Looper`, 那么在所有的事情完成以后应该调用`quit()`方法来终止消息循环. 否则这个线程会一直处于等待的状态, 而如果退出了`Looper`以后, 这个线程就会立刻终止.

**Looper最重要的一个方法loop()方法, 只有调用了loop后, 消息循环系统才会真正的起作用,如下**

```
public static void loop() {
   final Looper me = myLooper();
   if (me == null) {
       throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
   }
   final MessageQueue queue = me.mQueue;

   // Make sure the identity of this thread is that of the local process,
   // and keep track of what that identity token actually is.
   Binder.clearCallingIdentity();
   final long ident = Binder.clearCallingIdentity();

   for (;;) {
       Message msg = queue.next(); // might block
       if (msg == null) {
           // No message indicates that the message queue is quitting.
           return;
       }

       // This must be in a local variable, in case a UI event sets the logger
       Printer logging = me.mLogging;
       if (logging != null) {
           logging.println(">>>>> Dispatching to " + msg.target + " " +
                   msg.callback + ": " + msg.what);
       }

       msg.target.dispatchMessage(msg);

       if (logging != null) {
           logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
       }

       // Make sure that during the course of dispatching the
       // identity of the thread wasn't corrupted.
       final long newIdent = Binder.clearCallingIdentity();
       if (ident != newIdent) {
           Log.wtf(TAG, "Thread identity changed from 0x"
                   + Long.toHexString(ident) + " to 0x"
                   + Long.toHexString(newIdent) + " while dispatching to "
                   + msg.target.getClass().getName() + " "
                   + msg.callback + " what=" + msg.what);
       }

       msg.recycleUnchecked();
   }
}
```

首先这个`loop()`方法是一个死循环, 唯一跳出循环的方式就是`MessageQueue#next()`方法返回**null**. 当`Looper#quit()`被调用时, `Looper`就会调用`MessageQueue#quit()或者quitSafely()`方法来通知消息队列退出, 当消息队列被标识为退出状态时, 它的`next()`方法就会返回null. 也就是说`Looper`必须退出, 否则`loop`方法就会无限循环下去. `loop()`会调用`MessageQueue#next()`方法来获取新消息. 而next是一个阻塞操作, 当没有消息时, next方法就会一直阻塞在那里. 这也导致`loop()`会一直阻塞在那里. 如果`MessageQueue#next()`返回了新消息, `Looper`就会处理这条消息: `msg.target.dispatchMessage(msg)`, 这里的msg.target是发送这条消息的`Handler`对象, 这样`Handler`发送的消息最终又交给它的`dispatcherMessage()`来处理. 但是这里不同的是, `Handler#dispatcherMessage()`方法是在创建Handler时所使用的`Looper`中执行的. 这样就成功的将代码逻辑切换到指定的线程中去执行了.

### Handler的工作原理

`Handler`主要包含消息的发送和接收过程. 消息的发送可以通过post的一系列方法以及send的一系列方法来实现. post的一系列方法最终就是还是通过send方法来实现的.

```
public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
   MessageQueue queue = mQueue;
   if (queue == null) {
       RuntimeException e = new RuntimeException(
               this + " sendMessageAtTime() called with no mQueue");
       Log.w("Looper", e.getMessage(), e);
       return false;
   }
   return enqueueMessage(queue, msg, uptimeMillis);
}
```

上面这段代码是`Handler#send()`系列的最终调用. 可以看出, `Handler`发送消息的过程仅仅是向消息队列中插入了一条消息, `MessageQueue#next()`方法就是返回这条消息给`Looper`, Looper收到消息后就开始处理. 最终消息有`Looper`交由`Handler`处理, 即`Handler#dispatchMessage()`方法会被调用, 这个时候`Handler`就会进入了处理消息的阶段.

```
public void dispatchMessage(Message msg) {
   if (msg.callback != null) {
       handleCallback(msg);
   } else {
       if (mCallback != null) {
           if (mCallback.handleMessage(msg)) {
               return;
           }
       }
       handleMessage(msg);
   }
}
```

整理一下: 先检查`msg.callback`属性是否为null, 不为null就通过`handleCallback()`来处理消息. `msg.callback`是一个Runnable接口, 实际上就是`post()`中传递的Runnable参数.

`handleCallback()`实现如下:

```
private static void handleCallback(Message message) {
   message.callback.run();
}
```

其次检查`mCallback`是否为null, 不为null就调用`mCallback.handleMessage(msg)`方法来处理消息. `Callback`是一个接口, 定义如下:

```
public interface Callback {
   public boolean handleMessage(Message msg);
}
```

通过`Callback`可以采用如下方式来创建Handler对象: `Handler handler = new Handler(callback)`. 通过源码注释了解: 这个接口可以用来创建一个Handler的实例但并不需要派生`Handler`的子类并重写其handleMessage方法来处理具体的消息, 而`CallBack`给我们提供了另外一种方式使用Handler. 当我们不想派生子类时, 就可以通过`Callback`来实现.

最后, 调用`Handler#handleMessage()`方法来处理消息.

`Handler`还有一个特殊的构造方法, 那就是通过一个特定的`Looper`来构造Handler,

最常用的就是直接new 出一个`Handler`, 这个构造方法会调用下面的构造函数. 很明显这就是为什么当前线程没有`Looper`的话, 就会抛出`Can't create handler inside thread that has not called Looper.prepare()`这个异常.

```
public Handler(Callback callback, boolean async) {
   //....

   mLooper = Looper.myLooper();
   if (mLooper == null) {
       throw new RuntimeException(
           "Can't create handler inside thread that has not called Looper.prepare()");
   }
   mQueue = mLooper.mQueue;
   mCallback = callback;
   mAsynchronous = async;
}
```

## 主线程的消息循环

Android的主线程就是`ActivityThread`, 主程序的入口方法为`main()`, 在`main()`中系统通过`Looper.prepareMainLooper()`来创建主线程的Looper以及`MessageQueue`, 并通过`Looper.loop()`来开启主线程的消息循环.

当主线程的消息循环开始以后, `ActivityThread`还需要一个`Handler`来和消息队列进行交互, 这个`Handler`就是`ActivityThread.H`, 它的内部定义了一组消息类型, 主要包含了四大组件的启动和停止等过程.

```
private class H extends Handler {
   public static final int LAUNCH_ACTIVITY         = 100;
   public static final int PAUSE_ACTIVITY          = 101;
   public static final int PAUSE_ACTIVITY_FINISHING= 102;
   public static final int STOP_ACTIVITY_SHOW      = 103;
   public static final int STOP_ACTIVITY_HIDE      = 104;
   public static final int SHOW_WINDOW             = 105;
   public static final int HIDE_WINDOW             = 106;
   public static final int RESUME_ACTIVITY         = 107;
   public static final int SEND_RESULT             = 108;
   public static final int DESTROY_ACTIVITY        = 109;
   public static final int BIND_APPLICATION        = 110;
   public static final int EXIT_APPLICATION        = 111;
   public static final int NEW_INTENT              = 112;
   public static final int RECEIVER                = 113;
   public static final int CREATE_SERVICE          = 114;
   public static final int SERVICE_ARGS            = 115;
   public static final int STOP_SERVICE            = 116;

   public static final int CONFIGURATION_CHANGED   = 118;
   public static final int CLEAN_UP_CONTEXT        = 119;
   public static final int GC_WHEN_IDLE            = 120;
   public static final int BIND_SERVICE            = 121;
   public static final int UNBIND_SERVICE          = 122;
}
```

`ActivityThread`通过`ApplicationThread`和`AMS`进行进程间通信, `AMS`以进程间通信的方式完成`ActivityThread`的请求后回调`ApplicationThread`中的`Binder()`方法, 然后`Application`会向`H`发送消息, `H`收到消息后会将`ApplicationThread`中的逻辑切换到`ActivityThread`中去执行, 即切换到主线程去执行, 这个过程就是主线程的消息循环模型.