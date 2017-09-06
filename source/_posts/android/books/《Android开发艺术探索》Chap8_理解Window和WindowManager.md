title: 《Android开发艺术探索》Chap8_理解Window和WindowManager
date: 

categories: 
- android 
- 《Android开发艺术探索》
tags:  
- android
---

> 所有的视图都是Windown呈现的, 那它都干了什么?
> 注：本文主要参考[http://szysky.com](http://szysky.com)

[blog相关代码保存在第7章项目window包中](https://github.com/suzeyu1992/Notes_AndroidDevSeek)

Window表示一个窗口的概念, 如有需要在桌面上显示一个类似悬浮窗的东西, 那么这种效果就需要`Window`来实现. `Window`是一个抽象类, 具体实现是`PhoneWindow`. 如果想要创建一个`Window`只需要通过`WindowManager`即可完成. `WindowManager`是外界访问`Window`的入口, Window具体实现位于`WindowManagerService`中, **WM**和**WMS**的交互是一个IPC过程. Android中所有的视图都是通过`Window`来呈现的, 不管是Activity, Dialog, Toast他们的视图实际上都是附加在Window上的.

## Window和WindowManager

先演示使用`WindowManger`添加一个Window.

```
public void addWindow(){
        Button button = new Button(getApplicationContext());
        button.setText("动态添加");
        WindowManager.LayoutParams layoutParams = new WindowManager.LayoutParams(WindowManager.LayoutParams.WRAP_CONTENT, WindowManager.LayoutParams.WRAP_CONTENT, 0, 0, PixelFormat.TRANSPARENT);

        layoutParams.flags = WindowManager.LayoutParams.FLAG_NOT_TOUCH_MODAL
                | WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE
                | WindowManager.LayoutParams.FLAG_SHOW_WHEN_LOCKED;

        layoutParams.gravity = Gravity.LEFT ;

        layoutParams.width = 400;
        layoutParams.height = 300;

        getWindowManager().addView(button, layoutParams);

    }
```

**Flag参数表示Window的属性**,这些属性可以控制Window的显示特性. 下面是常用的属性:

- `FLAG_NOT_FOCUSABLE`: 表示Window不需要获取焦点, 而不需要接收各种输入事件, 此标记会同时启动`FLAG_NOT_TOUCH_MODAL`, 最终事件会直接传递给下层的具有焦点的Window.
- `FLAG_NOT_TOUCH_MODAL`: 这种模式下, 系统会将当前Window区域以外的点击事件传递给底层的Window, 当前Window区域以内的单击事件则自己处理. 这个标记很重要, 一般来说都需要开启此标记, 否则其他Window将无法接收到单击事件.
- `FLAG_SHOW_WHEN_LOCKED`: 开启此模式可以让Window显示在锁屏的界面上.

**Type参数表示Window的类型**

Window共有三种类型, 分别是**应用Window**, **子Window**, **系统Window**. 应用类Window对应着一个Activity. 子Window不能单独存在, 他需要附属在特定的父Window中,比如常见的`Dialog`就是一个子Window. 系统Window是需要声明权限才能创建的Window, 比如`Toast`和系统状态栏都是系统的Window.

Window是分层的, 每个Window都有对应的`z-ordered`, 层级大的会覆盖在层级小的Window的上面, 这和HTML中的`z-index`的概念一样. **应用Window**的层级范围是1~99, **子Window**的层级范围是1000~1999, **系统Window**的层级范围是2000~2999.

如果想要在最顶层显示, 可以选择使用`TYPE_SYSTEM_OVERLAY`, `TYPE_SYSTEM_ERROR`. 如果采用了`TYPE_SYSTEM_ERROR`同时要声明权限`android.permission.SYSTEM_ALERT_WINDOW`. 如果不声明那么在创建的时候就会报错.

**WindowManager常用的功能**

在`ViewManager`接口中定义了三个方法. 就是我们常用的方法**添加View**,**删除View**,**修改View**. `WM`继承了这个接口.

## Window的内部机制

`Window`是一个抽象的概念, 每一`Window`都对应着一个View和一个ViewRootImpl, `Window`和View通过`ViewRootImpl`来建立联系, 因此Window并不是实际存在的, 他是以View的形式存在. 通过`WindowManager`的定义和提供的三个接口方法看出都是针对View的. 说明View才是`Windwo`存在的实体. 而在实际的使用中无法直接访问`Window`, 对`Window`的访问都是必须通过WM.

### Window的添加过程

`Window`的添加过程需要通过`WindowManager`的`addView()`来实现, 而`WindowManager`是一个接口, 它的真正实现是`WindowManagerImpl`类, 在WindowManagerImpl中Window的三大操作如下.

```
@Override
public void addView(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
   applyDefaultToken(params);
   mGlobal.addView(view, params, mDisplay, mParentWindow);
}

@Override
public void updateViewLayout(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
   applyDefaultToken(params);
   mGlobal.updateViewLayout(view, params);
}

 
@Override
public void removeView(View view) {
   mGlobal.removeView(view, false);
}
```

`WindowManagerImpl`并没有直接实现Window的三大操作, 而是全部交给了`WindowManagerGlobal`来处理. `WindowManagerGlobal`以工厂的形式向外提供自己的实例. 而`WindowManagerImpl`这种工作模式就典型的桥接模式, 将所有的操作全部委托给`WindowManagerGlobal`来实现.

**WindowManagerGlobal的addView()主要分为**

1. 检查所有参数是否合法, 如果是子Window那么还需要调整一些布局参数.
2. 创建`ViewRootImpl`并将View添加到列表中.
3. 通过`ViewRootImpl`来更新界面并完成Window的添加过程. 这个过程是通过`ViewRootImpl#setView()`来完成的. View的绘制过程是由`ViewRootImpl`来完成的, 在内部会调用`requestLayout()`来完成异步刷新请求. 而`scheduleTraversals()`实际上是View绘制的入口. 接着会通过`WindowSession`完成Window的添加过程(Window的添加过程是一次IPC调用). 最终会通过`WindowManagerService`来实现Window的添加.

`WindowManagerService`内部会为每一个应用保留一个单独的Session.

### Window的删除过程

Window 的删除过程和添加过程一样, 都是先通过`WindowManagerImpl`后, 在进一步通过`WindowManagerGlobal的removeView()`来实现的.

```
public void removeView(View view, boolean immediate) {
   synchronized (mLock) {
       int index = findViewLocked(view, true);
       View curView = mRoots.get(index).getView();
       removeViewLocked(index, immediate);
       if (curView == view) {
           return;
       }
       throw new IllegalStateException("Calling with view " + view
               + " but the ViewAncestor is attached to " + curView);
   }
}
```

方法内首先通过`findViewLocked`来查找待删除的View的索引, 这个过程就是建立数组遍历, 然后调用`removeViewLocked`来做进一步的删除.

```
private void removeViewLocked(int index, boolean immediate) {
   ViewRootImpl root = mRoots.get(index);
   View view = root.getView();

   if (view != null) {
       InputMethodManager imm = InputMethodManager.getInstance();
       if (imm != null) {
           imm.windowDismissed(mViews.get(index).getWindowToken());
       }
   }
   boolean deferred = root.die(immediate);
   if (view != null) {
       view.assignParent(null);
       if (deferred) {
           mDyingViews.add(view);
       }
   }
}
```

这里通过`ViewRootImpl`的`die()`完成来完成删除操作. `die()`方法只是发送了请求删除的消息后就立刻返回了, 这个时候View并没有完成删除操作, 所以最后会将其添加到`mDyingViews`中, `mDyingViews`表示待删除的View的列表.

```
boolean die(boolean immediate) {
   // Make sure we do execute immediately if we are in the middle of a traversal or the damage
   // done by dispatchDetachedFromWindow will cause havoc on return.
   if (immediate && !mIsInTraversal) {
       doDie();
       return false;
   }

   if (!mIsDrawing) {
       destroyHardwareRenderer();
   } else {
       Log.e(TAG, "Attempting to destroy the window while drawing!\n" +
               "  window=" + this + ", title=" + mWindowAttributes.getTitle());
   }
   mHandler.sendEmptyMessage(MSG_DIE);
   return true;
}
```

die方法中只是做了简单的判断, 如果是异步删除那么就发送一个`MSG_DIE`的消息, `ViewRootImpl`中的`Handler`会处理此消息并调用`doDie()`; 如果是同步删除, 那么就不发送消息直接调用`doDie()`方法.

在`doDie()`方法中会调用`dispatchDetachedFromWindow()`方法, 真正删除View的逻辑在这个方法内部实现. 其中主要做了四件事:

1. 垃圾回收的相关工作, 比如清除数据和消息,移除回调.
2. 通过Session的remove方法删除Window: `mWindowSession.remove(mWindow)`, 这同样是一个IPC过程, 最终会调用`WMS`的removeWindow()方法.
3. 调用View的`dispatchDetachedFromWindow()`方法, 内部会调用View的`onDetachedFromWindow()`以及`onDetachedFromWindowInternal()`. 而对于`onDetachedFromWindow()`就是在View从Window中移除时, 这个方法就会被调用, 可以在这个方法内部做一些资源回收的工作. 比如停止动画,停止线程
4. 调用`WindowManagerGlobal#doRemoveView`方法刷新数据, 包括`mRoots`, `mParams`, `mDyingViews`, 需要将当前Window所关联的这三类对象从列表中删除.

### Window的更新过程

`WindowManagerGlobal#updateViewLayout()`方法做的比较简单, 它需要更新View的`LayoutParams`并替换掉老的`LayoutParams`, 接着在更新`ViewRootImpl`中的LayoutParams. 这一步主要是通过`setLayoutParams()`方法实现.

在`ViewRootImpl`中会通过`scheduleTraversals()`来对View重新布局, 包括测量,布局,重绘. 除了View本身的重绘以外, `ViewRootImpl`还会通过`WindowSession`来更新Window的视图, 这个过程最后由`WMS`的`relayoutWindow()`实现同样是一个IPC过程.

## Window的创建过程

View是Android中视图的呈现方式, 但是View不能单独存在, 它必须依附在Window这个抽象的概念上面, 因此有视图的地方就有`Window`.

### Activity的Window创建过程

`Activity`的大体启动流程: 最终会由`ActivityThread`中的`PerformLaunchActivity()`来完成整个启动过程, 这个方法内部会通过类加载器创建`Activity`的实例对象, 并调用其`attach()`方法为其关联运行过程中所依赖的一系列上下文环境变量. 代码如下:

```
private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
  Activity activity = null;
   
  //获得类加载器
  java.lang.ClassLoader cl = r.packageInfo.getClassLoader();
  activity = mInstrumentation.newActivity(
          cl, component.getClassName(), r.intent);
  StrictMode.incrementExpectedActivityCount(activity.getClass());
  r.intent.setExtrasClassLoader(cl);
  r.intent.prepareToEnterProcess();
  if (r.state != null) {
      r.state.setClassLoader(cl);
  }

 Context appContext = createBaseContextForActivity(r, activity);
 CharSequence title = r.activityInfo.loadLabel(appContext.getPackageManager());
 Configuration config = new Configuration(mCompatConfiguration);
     
 activity.attach(appContext, this, getInstrumentation(), r.token,
         r.ident, app, r.intent, r.activityInfo, title, r.parent,
         r.embeddedID, r.lastNonConfigurationInstances, config,
         r.referrer, r.voiceInteractor);

        return activity;
}
```

在`attach()`方法里, 系统会创建`Activity`所属的Window对象并为其设置回调接口, Window对象的创建是通过`PolicyManager#makeNewWindow()`方法实现. 由于`Activity`实现了`Window`的CallBack接口, 因此当`Window`接收到外界的状态改变的时候就会回调Activity方法. 比如说我们熟悉的`onAttachedToWindow()`, `onDetachedFromWindow()`, `dispatchTouchEvent()`等等.代码如下

```
mWindow = new PhoneWindow(this);
       mWindow.setCallback(this);
       mWindow.setOnWindowDismissedCallback(this);
       mWindow.getLayoutInflater().setPrivateFactory(this);
       if (info.softInputMode != WindowManager.LayoutParams.SOFT_INPUT_STATE_UNSPECIFIED) {
           mWindow.setSoftInputMode(info.softInputMode);
       }
       if (info.uiOptions != 0) {
           mWindow.setUiOptions(info.uiOptions);
       }
       mUiThread = Thread.currentThread();
```

那么Activity视图是怎么附属在Window上的呢? 查看经常使用的`setContentView()`方法干了什么

```
public void setContentView(@LayoutRes int layoutResID) {
   getWindow().setContentView(layoutResID);
   initWindowDecorActionBar();
}
```

`Activity`将具体实现交给了Window处理, 而`Window`的具体实现就是`PhoneWindow`, 所以只需要看`PhoneWindow`的相关逻辑分为以下几步.

1. 如果没有DecorView, 那么就创建它. 由`installDecor()-->generateDecor()`触发
2. 将View添加到`DecorView`的`mContentParent`中
3. 回调Activity的`onContentChanged()`通知activity视图已经发生改变

这个时候`DecorView`已经被创建并初始化完毕, Activity的布局文件也已经添加成功到`DecorView的mContentParent`中. 但是这个时候`DecorView`还没有被`WindowManager`正式添加到Window中. 虽然早在`Activity的attach`方法中window就已经被创建了, 但是这个时候由于`DecorView`并没有被`WindowManager`识别, 所以这个时候的Window无法提供具体功能, 因为他还无法接收外界的输入信息.

在`ActivityThread#handleResumeActivity()`方法中, 首先会调用`Activity#onResume()`, 接着会调用`Activity#makeVisible()`, 正是在makeVisible方法中, `DecorView`真正的完成了添加和显示这两个过程. 如下:

```
void makeVisible() {
   if (!mWindowAdded) {
       ViewManager wm = getWindowManager();
       wm.addView(mDecor, getWindow().getAttributes());
       mWindowAdded = true;
   }
   mDecor.setVisibility(View.VISIBLE);
}
```

### Dialog的Window创建过程

`Dialog`的`Window`的创建过程和Activity类似, 有如下几步

**1. 创建Window**

Dialog的创建后的实际就是`PhoneWindow`, 这个过程和Activity的Window创建过程一致.

**2. 初始化DecorView并将Dialog的视图添加到DecorView中**

这个过程也类似, 都是通过`Window`去添加指定的布局文件.

**3. 将DecorView添加到Window中并显示**

在`Dialog`的show方法中, 会通过`WindowManager`将`DecorView`添加Window中.

```
mWindowManager.addView(mDecor, l);
mShowing = true;   
sendShowMessage();
```

普通的`Dialog`有一个特殊之处, 那就是必须采用`Activity`的Content, 如果采用`Application`的Content, 那么就会报错. 报的错**是没有应用token所导致的, 而应用token一般只有Activity才拥有**.

还有一种方法. `系统Window比较特殊, 他可以不需要token`, 因此只需要指定对话框的Window为系统类型就可以正常弹出对话框.

```
//JAVA 给Dialog的Window改变为系统级的Window
dialog.getWindow().setType(WindowManager.LayoutParams.TYPE_SYSTEM_ERROR);


//XML 声明权限
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW"/>
```

### Toast的Window创建过程

`Toast`和`Dialog`不同, 它的工作过程就稍显复杂. 首先`Toast`也是基于Window来实现的. 但是由于`Toast`具有定时取消的功能, 所以系统采用了`Handler`. 在`Toast`的内部有两类IPC过程, 第一类是Toast访问`NotificationManagerService()`后面简称`NMS`. 第二类是`NotificationManagerService`回调Toast里的`TN`接口.

`Toast`属于系统Window, 它内部的视图有两种方式指定, 一种是系统默认的样式, 另一种是通过`setView`方法来指定一个自定义View. 不管如何, 他们都对应`Toast`的一个View类型的内部成员`mNextView`. Toast内部提供了`cancel`和`show`两个方法. 分别用于显示和隐藏`Toast`. 他们内部是一个IPC过程.

显示和隐藏`Toast`都是需要通过`NMS`来实现的. 由于`NMS`运行在系统的进程中, 所以只能通过远程调用的方式来显示和隐藏`Toast`. 而`TN`这个类是一个`Binder`类. 在`Toast`和`NMS`进行IPC的过程中, 当NMS处理Toast的显示或隐藏请求时会跨进程回调`TN`的方法. 这个时候由于`TN`运行在Binder线程池中, 所以需要通过`Handler`将其切换到当前主线程. 所以由其可知, `Toast`无法在没有`Looper`的线程中弹出, 因为Handler需要使用`Looper`才能完成切换线程的功能.

```
 if (!isSystemToast) {
   int count = 0;
   final int N = mToastQueue.size();
   for (int i=0; i<N; i++) {
        final ToastRecord r = mToastQueue.get(i);
        if (r.pkg.equals(pkg)) {
            count++;
            // MAX_PACKAGE_NOTIFICATIONS == 50
            if (count >= MAX_PACKAGE_NOTIFICATIONS) { 
                Slog.e(TAG, "Package has already posted " + count
                       + " toasts. Not showing more. Package=" + pkg);
                return;
            }
        }
   }
}
```

对于非系统应用来说, 最多能同时存在对`Toast`封装的`ToastRecord`上限为50个. 这样做是为了防止**DOS(Denial of Service)**. 如果不这样, 当通过大量循环去连续的弹出Toast, 这将会导致其他应用没有机会弹出Toast, 那么对于其他应用的Toast请求, 系统的行为就是拒绝服务, 这就是拒绝服务攻击的含义.

在`ToastRecord`被添加到`mToastQueue()`中后, `NMS`就会通过`showNextToastLocked()`方法来显示当前的Toast.

Toast的显示是由`ToastRecord`的`callback`来完成的. 这个`callback`实际上就是`Toast`中的`TN`对象的远程Binder. 通过callback来访问TN中的方法是需要跨进程的. 最终被调用的`TN`中的方法会运行在发起`Toast`请求的应用的Binder线程池.

```
private void scheduleTimeoutLocked(ToastRecord r){
   mHandler.removeCallbacksAndMessages(r);
   Message m = Message.obtain(mHandler, MESSAGE_TIMEOUT, r);
   long delay = r.duration == Toast.LENGTH_LONG ? LONG_DELAY : SHORT_DELAY;
   mHandler.sendMessageDelayed(m, delay);
}
```

如上代码就是在`Toast`显示以后, `NMS`通过这个方法来发送一个延时消息, 具体取决Toast的时长. `LONG_DELAY`, `SHORT_DELAY`分别对应着3.5秒和2秒. 当延时时间达到的时候. `NMS`会通过`cancelToastLocked()`方法来隐藏`Toast`并将其从`mToastQueue`中移除, 这个时候如果`mToastQueue`中还有其余`Toast`那么`NMS`就继续显示其他.

**Toast的隐藏也会通过ToastRecord的callback完成的**.同样是一次IPC过程. 方式和Toast显示类似.

```
void cancelToastLocked(int index) {
ToastRecord record = mToastQueue.get(index);
try {
  record.callback.hide();
} catch (RemoteException e) {
  Slog.w(TAG, "Object died trying to hide notification " + record.callback
          + " in package " + record.pkg);
  // don't worry about this, we're about to remove it from
  // the list anyway
}
mToastQueue.remove(index);
keepProcessAliveLocked(record.pid);
if (mToastQueue.size() > 0) {
  // Show the next one. If the callback fails, this will remove
  // it from the list, so don't assume that the list hasn't changed
  // after this point.
  showNextToastLocked();
}
}
```

以上基本说明`Toast`的显示和影响过程实际上是通过`Toast`中的`TN`这个类来实现的. 他有两个方法`show()`, `hide()`. 分别对应着`Toast`的显示和隐藏. 由于这两个方法是被`NMS`以跨进程的方式调用的, 因此他们运行在`Binder线程池中`. 为了将执行环境切换到Toast请求所在线程中, 在他们内部使用了`handler`,如下

```
@Override
public void show() {
  if (localLOGV) Log.v(TAG, "SHOW: " + this);
  mHandler.post(mShow);
}

/**
* schedule handleHide into the right thread
*/
@Override
public void hide() {
  if (localLOGV) Log.v(TAG, "HIDE: " + this);
  mHandler.post(mHide);
}
```

上面代码中, `mShow`, `mHide`是两个Runnable, 他们内部分别调用了`handleShow`和`handleHide`方法. 所以这两个方法才是真正完成隐藏和显示`Toast`的地方.

`TN`的**handleShow**中会将`Toast`的视图添加到`Window`中.

`TN`的**handleHide**中会将`Toast`的视图从`Window`中移除.

具体实现代码如下:

```
//handleShow()
 mWM = (WindowManager)context.getSystemService(Context.WINDOW_SERVICE);
 mWM.addView(mView, mParams);
 
//handleHide()
if (mView != null) {
  if (mView.getParent() != null) {
     if (localLOGV) Log.v(TAG, "REMOVE! " + mView + " in " + this);
     mWM.removeView(mView);
  }
  mView = null;
}
```

关于`Toast`流程已经完事. 除了说到的`Activity`, `Dialog`, `Toast`. 还有`PopupWindow`
菜单栏, 状态栏都是通过`Window`来实现的.