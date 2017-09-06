title: 《Android开发艺术探索》Chap9_四大组件的工作过程
date: 

categories: 
- android 
- 《Android开发艺术探索》
tags:  
- android
---

> 还原四大组件的本质
> 注：本文主要参考[http://szysky.com](http://szysky.com)

[blog相关代码](https://github.com/suzeyu1992/Notes_AndroidDevSeek)

## 四大组件的运行状态

四大组件中除了`BroadcastReceiver`以外, 其余的三种组件都必须在**AndroidManifest**中注册, 对于`BroadcastReceiver`来说, 既可以在清单文件中注册, 也可以通过代码来注册. 在调用形式上除了`ContentProvider`不需要借助`Intent`. 其余的三大组件都需要Intent

**Activity**是一种展示型组件, 用于向用户直接地展示一个界面, 并且可以接收用户的输入信息从而进行交互. `Activity`是最重要的一种组件, 对于用户来说它就是应用的全部, 因为其他三大组件对用户来说是无法感知的. `Activity`的启动由`Intent`触发, 其中`Intent`可以分为**显式和隐式**. **显式Intent**可以明确的指向一个`Activity`, 而**隐式Intent**则指向一个或者多个Activity组件. 或者是没有`Activity`组件可以处理这个隐式的Intent. 一个`Activity`具有特定的启动模式. 也可以通过`finish`来停止. 总结来说, `Activity`组件的主要作用是展示一个界面并和用户交互, 它扮演的是一种前台界面的角色.

------

**Service**是一种计算型组件, 用于后台执行一系列计算任务. 运行在后台,用户是无法感知的. `Service`和`Activity`的不同: `Activity`组件只有一种运行模式,即`Activity`处于启动状态, 但是`Service`组件有两种状态: **启动状态**和**绑定状态**.

- 当`Service`处于启动状态时, 这个时候Service内部可以做一些后台计算. 尽管`Service`组件是用于执行后台计算的, 但是它本身是运行在主线程的. 因此单独的耗时操作仍然需要单独的线程去执行.

- 当`Service`处于绑定状态时, 内部同样可以进行后台计算, 但是处于这种状态时, 外界可以很方便的和`Service`组件进行通信.

  `Service`可以停止, 需要灵活采用`stopService`和`unBindService`

------

**BroadcastReceiver**是一种消息型组件, 用于在不同的组件乃至不同的应用之间传递消息. 同样无法被用户感知, 因为是运行在系统内部, 广播的注册方式有两种:**静态注册**和**动态注册**

- **静态注册**: 在清单文件中进行注册广播, 这种广播在应用安装时会被系统解析, 此种形式的广播不需要应用启动就可以接收到相应的广播.

- **动态注册**: 需要通过`Context.registerReceiver()`来实现, 并在不需要的时候通过`Context.unRegisterReceiver()`来解除广播. 此种形态的广播要应用启动才能注册和接收广播. 在实际开发中通过`Context`的一系列的`send`方法来发送广播, 被发送的广播会被系统发送给感兴趣的广播接收者, 发送和接收的过程的匹配是通过广播接收者的`<intent-filter>`来描述的.

  可以实现低耦合的观察者模式, 观察者和被观察者之间可以没有任何耦合. 但广播不适合来做耗时操作.

------

**ContentProvider**是一种数据共享组件, 用于向其他组件乃至其他应用共享数据. 无法被用户感知. 对于内容提供者来说, 它只需要实现增删改查四种基本操作, 在它内部维持着一份数据集合, 这个数据集合既可以通过数据库来实现, 也可以采用其他任何类型来实现, 例如list或者map. `ContentProvider`对数据集合的具体实现并没有任何要求.

要注意处理好内部的`insert`, `delete`, `update`, `query`方法的线程同步, 因为这几个方法是在`Binder`线程池被调用.

## Activity的工作过程

虽然要打开一个`Activity`很简单, 但是不应该只是局限于表面. 了解其内部走向构成.所以一切从`startActivity(intent)`这个方法开始.

`startActivity()`有好几种重载方式但是最终都是调用`startActivityForResult()`方法.

**注意这里分析的是5.0版本的源码, 和6.0源码实现稍微不同**

首先看`startActivityForResult()`

```
public void startActivityForResult(Intent intent, int requestCode, @Nullable Bundle options) {
   if (mParent == null) {
       Instrumentation.ActivityResult ar =
           mInstrumentation.execStartActivity(
               this, mMainThread.getApplicationThread(), mToken, this,
               intent, requestCode, options);
       if (ar != null) {
           mMainThread.sendActivityResult(
               mToken, mEmbeddedID, requestCode, ar.getResultCode(),
               ar.getResultData());
       }
       if (requestCode >= 0) {
           mStartedActivity = true;
       }

       final View decor = mWindow != null ? mWindow.peekDecorView() : null;
       if (decor != null) {
           decor.cancelPendingInputEvents();
       }
       // TODO Consider clearing/flushing other event sources and events for child windows.
   } else {
       if (options != null) {
           mParent.startActivityFromChild(this, intent, requestCode, options);
       } else {
           // Note we want to go through this method for compatibility with
           // existing applications that may have overridden it.
           mParent.startActivityFromChild(this, intent, requestCode);
       }
   }
   if (options != null && !isTopOfTask()) {
       mActivityTransitionState.startExitOutTransition(this, options);
   }
}
```

关注`mParent==null`的分支. `mParent`代表的是`ViewGroup`, `ActivityGroup`最开始被用来在一个界面中嵌入多个子`Activity`, 在**API13**已经被废弃. 系统推荐使用`Fragment`代替`ActivityGroup`. 注意`mMainThread.getApplicationThread()`这个参数, 它的参数类型是`ApplicationThread`, `ApplicationThread`是`ActivityThread`的一个内部类.

**看一下Instrumentation#execStartActivity()这个方法**

```
public ActivityResult execStartActivity(
       Context who, IBinder contextThread, IBinder token, Activity target,
       Intent intent, int requestCode, Bundle options) {
       
   IApplicationThread whoThread = (IApplicationThread) contextThread;
   if (mActivityMonitors != null) {
       synchronized (mSync) {
           final int N = mActivityMonitors.size();
           for (int i=0; i<N; i++) {
               final ActivityMonitor am = mActivityMonitors.get(i);
               if (am.match(who, null, intent)) {
                   am.mHits++;
                   if (am.isBlocking()) {
                       return requestCode >= 0 ? am.getResult() : null;
                   }
                   break;
               }
           }
       }
   }
   try {
       intent.migrateExtraStreamToClipData();
       intent.prepareToLeaveProcess();
       // 启动Activity的真正实现
       int result = ActivityManagerNative.getDefault()
           .startActivity(whoThread, who.getBasePackageName(), intent,
                   intent.resolveTypeIfNeeded(who.getContentResolver()),
                   token, target != null ? target.mEmbeddedID : null,
                   requestCode, 0, null, options);
       checkStartActivityResult(result, intent);
   } catch (RemoteException e) {
   }
   return null;
}
```

代码中真正启动`Activity`的真正实现是由`ActivityManagerNative.getDefault().startActivity()`方法完成的. 后面对`ActivityManagerService`简称`AMS`. `AMS`继承自`ActivityManagerNative()`, 而`ActivityManagerNative()`继承自`Binder`并实现了`IActivityManager`这个`Binder`接口, 因此`AMS`也是一个`Binder`, 它是`IActivityManager`的具体实现.

由于`ActivityManagerNative.getDefault()`本质是一个`IActivityManager`类型的`Binder`对象, 因此具体实现是`AMS`. 在`ActivityManagerNative`中, `AMS`这个Binder对象采用单例模式对外提供, `Singleton`是一个单例封装类. 第一次调用它的`get()`方法时会通过create方法来初始化`AMS`这个`Binder`对象, 在后续调用中会返回这个对象. 具体实现如下代码.

```
static public IActivityManager getDefault() {
   return gDefault.get();
}

private static final Singleton<IActivityManager> gDefault = new Singleton<IActivityManager>() {
   protected IActivityManager create() {
       IBinder b = ServiceManager.getService("activity");
       if (false) {
           Log.v("ActivityManager", "default service binder = " + b);
       }
       IActivityManager am = asInterface(b);
       if (false) {
           Log.v("ActivityManager", "default service = " + am);
       }
       return am;
   }
};
```

由上可以看到关于Activity的启动是由`ActivityManagerNative.getDefault()`来启动的, 而`ActivityManagerNative.getDefault()`实际上是`AMS`, 所以`Activity`的启动过程又被转移到了`AMS`中, 接下来查看`AMS`中的`startActivity()`方法.

在分析`AMS#startActivity()`之前, 是否在开始时候碰到过`Activity`没有在清单文件中声明然后崩溃的现象? 这个步骤是在`Instrumentation#execStartActivity()`刚才分析`ActivityManagerNative.getDefault().startActivity()`的下一步. 有一个`checkStartActivityResult()`,看名字应该是检查的类, 看一下实现.

```
public static void checkStartActivityResult(int res, Object intent) {
   if (res >= ActivityManager.START_SUCCESS) {
       return;
   }
   
   switch (res) {
       case ActivityManager.START_INTENT_NOT_RESOLVED:
       case ActivityManager.START_CLASS_NOT_FOUND:
           if (intent instanceof Intent && ((Intent)intent).getComponent() != null)
               throw new ActivityNotFoundException(
                       "Unable to find explicit activity class "
                       + ((Intent)intent).getComponent().toShortString()
                       + "; have you declared this activity in your AndroidManifest.xml?");
           throw new ActivityNotFoundException(
                   "No Activity found to handle " + intent);
       case ActivityManager.START_PERMISSION_DENIED:
           throw new SecurityException("Not allowed to start activity "
                   + intent);
       case ActivityManager.START_FORWARD_AND_REQUEST_CONFLICT:
           throw new AndroidRuntimeException(
                   "FORWARD_RESULT_FLAG used while also requesting a result");
       case ActivityManager.START_NOT_ACTIVITY:
           throw new IllegalArgumentException(
                   "PendingIntent is not an activity");
       case ActivityManager.START_NOT_VOICE_COMPATIBLE:
           throw new SecurityException(
                   "Starting under voice control not allowed for: " + intent);
       default:
           throw new AndroidRuntimeException("Unknown error code "
                   + res + " when starting " + intent);
   }
}
```

看出这个方法是一言不合就抛异常. 看**ActivityManager.START_CLASS_NOT_FOUND**这个判断分支抛出的异常是不是很眼熟? 对就是没有在清单文件中注册就在这里抛出. 所以这个方法就是检查启动Activity的结果.

**回到AMS的startActivity()**

```
@Override
public final int startActivity(IApplicationThread caller, String callingPackage,
  Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
  int startFlags, ProfilerInfo profilerInfo, Bundle options) {
  // 直接调用下面方法
return startActivityAsUser(caller, callingPackage, intent, resolvedType, resultTo,
  resultWho, requestCode, startFlags, profilerInfo, options,
  UserHandle.getCallingUserId());
}

@Override
public final int startActivityAsUser(IApplicationThread caller, String callingPackage,
       Intent intent, String resolvedType, IBinder resultTo, String resultWho, int requestCode,
       int startFlags, ProfilerInfo profilerInfo, Bundle options, int userId) {
   enforceNotIsolatedCaller("startActivity");
   userId = handleIncomingUser(Binder.getCallingPid(), Binder.getCallingUid(), userId,
           false, ALLOW_FULL_ONLY, "startActivity", null);
           
   // TODO: Switch to user app stacks here.
   return mStackSupervisor.startActivityMayWait(caller, -1, callingPackage, intent,
           resolvedType, null, null, resultTo, resultWho, requestCode, startFlags,
           profilerInfo, null, null, options, userId, null, null);
}
```

`Activity`启动过程经过两次转移, 最后又转移到了`mStackSupervisor.startActivityMayWait()`这个方法, 所属类为`ActivityStackSupervisor`. 在`startActivityMayWait()`内部又调用了`startActivityLocked()`这里会返回结果码就是之前`checkStartActivityResult()`用到的. 继续跟进方法最后会调用`startActivityUncheckedLocked()`, 然后又调用了`ActivityStack#resumeTopActivityLocked()`. 这个时候启动过程已经从`ActivityStackSupervisor`转移到了`ActivityStack`类中.

**具体实现**

```
final boolean resumeTopActivityLocked(ActivityRecord prev, Bundle options) {
   if (mStackSupervisor.inResumeTopActivity) {
       // Don't even start recursing.
       return false;
   }

   boolean result = false;
   try {
       // Protect against recursion.
       mStackSupervisor.inResumeTopActivity = true;
       if (mService.mLockScreenShown == ActivityManagerService.LOCK_SCREEN_LEAVING) {
           mService.mLockScreenShown = ActivityManagerService.LOCK_SCREEN_HIDDEN;
           mService.updateSleepIfNeededLocked();
       }
       result = resumeTopActivityInnerLocked(prev, options);
   } finally {
       mStackSupervisor.inResumeTopActivity = false;
   }
   return result;
}
```

从上可以看到result是根据调用的`resumeTopActivityInnerLocked()`返回, 而`resumeTopActivityInnerLocked()`又调用了`ActivityStackSupervisor#startSpecificActivityLocked()`方法. 看一下这个方法:

```
void startSpecificActivityLocked(ActivityRecord r,
       boolean andResume, boolean checkConfig) {
   // Is this activity's application already running?
   ProcessRecord app = mService.getProcessRecordLocked(r.processName,
           r.info.applicationInfo.uid, true);

   r.task.stack.setLaunchTime(r);

   if (app != null && app.thread != null) {
       try {
           if ((r.info.flags&ActivityInfo.FLAG_MULTIPROCESS) == 0
                   || !"android".equals(r.info.packageName)) {
               app.addPackage(r.info.packageName, r.info.applicationInfo.versionCode,
                       mService.mProcessStats);
           }
           // 继续进入!!!
           realStartActivityLocked(r, app, andResume, checkConfig);
           return;
       } catch (RemoteException e) {
           Slog.w(TAG, "Exception when starting activity "
                   + r.intent.getComponent().flattenToShortString(), e);
       }

       // If a dead object exception was thrown -- fall through to
       // restart the application.
   }

   mService.startProcessLocked(r.processName, r.info.applicationInfo, true, 0,
           "activity", r.intent.getComponent(), false, false, true);
}
```

这个方法又进入了`realStartActivityLocked()`. 在这个方法中有如下一段代码:

```
app.thread.scheduleLaunchActivity(new Intent(r.intent), r.appToken,
                   System.identityHashCode(r), r.info, new Configuration(mService.mConfiguration),
                   r.compat, r.launchedFromPackage, r.task.voiceInteractor, app.repProcState,
                   r.icicle, r.persistentState, results, newIntents, !andResume,
                   mService.isNextTransitionForward(), profilerInfo);
```

先整理一下刚才的调用过程, 我已经照着书上跟的不知所以然了…. = =

![img](http://szysky.com/2016/08/16/%E3%80%8AAndroid-%E5%BC%80%E5%8F%91%E8%89%BA%E6%9C%AF%E6%8E%A2%E7%B4%A2%E3%80%8B-09-%E5%9B%9B%E5%A4%A7%E7%BB%84%E4%BB%B6%E7%9A%84%E5%B7%A5%E4%BD%9C%E8%BF%87%E7%A8%8B/activity_01.png)

继续看刚才`app.thread.scheduleLaunchActivity()`这个方法. 其中`app.thread`类型为`IApplicationThread`, 这个玩意继承了`IInterface`接口, 所以他也是一个`Binder`类型的接口接口. 从`IApplicationThread`声明的接口方法可以看出, 其内部包含了大量启动和停止`Activity`的接口, 此外还包含了`Service`的启动与停止. 可以猜想, `IApplicationThread`这个`Binder`接口的是实现者完成了大量和`Activity`以及`Service`启动和停止的相关功能.

**那IApplicationThread的实现者是哪个类? 在ActivityThread中的内部类有一个ApplicationThread,看看定义**

```
private class ApplicationThread extends ApplicationThreadNative

public abstract class ApplicationThreadNative extends Binder
        implements IApplicationThread
```

可以看到`ApplicationThread`的继承关系, 而查看`ApplicationThreadNative`的作用其实和系统AIDL文件生成的类是一样的.

在`ApplicationThreadNative`的内部, 还有一个`ApplicationThreadProxy`类, 眼熟吧在第二章讲解的时候**aidl生成的java文件中**也有一个内部的代理类. 所以`ApplicationThreadNative`就是`IApplicationThread`的实现者, 由于`ApplicationThreadNative`被系统定义为抽象类, 所以`ApplicationThread`就成了`IApplicationThread`的实现者

------

绕了一大圈, **Activity**启动过程最终回到了`ApplicationThread`中, `ApplicationThread`通过`scheduleLaunchActivity()`来启动Activity.

```
public final void scheduleLaunchActivity(Intent intent, IBinder token, int ident,
                ActivityInfo info, Configuration curConfig, CompatibilityInfo compatInfo,
                String referrer, IVoiceInteractor voiceInteractor, int procState, Bundle state,
                PersistableBundle persistentState, List<ResultInfo> pendingResults,
                List<ReferrerIntent> pendingNewIntents, boolean notResumed, boolean isForward,
                ProfilerInfo profilerInfo) {

            updateProcessState(procState, false);

            ActivityClientRecord r = new ActivityClientRecord();
            r.token = token;
            r.ident = ident;
            r.intent = intent;
            r.referrer = referrer;
            r.voiceInteractor = voiceInteractor;
            r.activityInfo = info;
            r.compatInfo = compatInfo;
            r.state = state;
            r.persistentState = persistentState;
            r.pendingResults = pendingResults;
            r.pendingIntents = pendingNewIntents;
            r.startsNotResumed = notResumed;
            r.isForward = isForward;
            r.profilerInfo = profilerInfo;
            updatePendingConfiguration(curConfig);
            
            sendMessage(H.LAUNCH_ACTIVITY, r);
}
```

这段代码做的就是封装一个`Activity的记录信息`交给名字叫**H**的一个Handler对象去处理, 继续跟进看看里面的实现

```
private void sendMessage(int what, Object obj, int arg1, int arg2, boolean async) {
   if (DEBUG_MESSAGES) Slog.v(
       TAG, "SCHEDULE " + what + " " + mH.codeToString(what)
       + ": " + arg1 + " / " + obj);
   Message msg = Message.obtain();
   msg.what = what;
   msg.obj = obj;
   msg.arg1 = arg1;
   msg.arg2 = arg2;
   if (async) {
       msg.setAsynchronous(true);
   }
   mH.sendMessage(msg);
}
```

额, 没啥好瞅的就是封装一个`message`发送了一个消息. 好吧 那看看`Handler`接收的时候是如何处理的吧. 这个类里面对`Handler`进行了包装, 包装的类就是一个`H`的内部类.

```
private class H extends Handler {
   public static final int LAUNCH_ACTIVITY         = 100;
   public static final int PAUSE_ACTIVITY          = 101;
   public static final int PAUSE_ACTIVITY_FINISHING= 102;
   public static final int STOP_ACTIVITY_SHOW      = 103;
   public static final int STOP_ACTIVITY_HIDE      = 104;
 //这里省略一堆常量声明
 //在省略一堆switch分支, 直接看具体的消息处理
   
   public void handleMessage(Message msg) {
       if (DEBUG_MESSAGES) Slog.v(TAG, ">>> handling: " + codeToString(msg.what));
       switch (msg.what) {
           case LAUNCH_ACTIVITY: {
               Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityStart");
               final ActivityClientRecord r = (ActivityClientRecord) msg.obj;

               r.packageInfo = getPackageInfoNoCheck(
                       r.activityInfo.applicationInfo, r.compatInfo);
               handleLaunchActivity(r, null);
               Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
           } break;
           case PAUSE_ACTIVITY:
               Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityPause");
               handlePauseActivity((IBinder)msg.obj, false, (msg.arg1&1) != 0, msg.arg2,
                       (msg.arg1&2) != 0);
               maybeSnapshot();
               Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
               break;
           case PAUSE_ACTIVITY_FINISHING:
               Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "activityPause");
               handlePauseActivity((IBinder)msg.obj, true, (msg.arg1&1) != 0, msg.arg2,
                       (msg.arg1&1) != 0);
               Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
               break;
            //这里又省去了一大坨的case分支
        }
       if (DEBUG_MESSAGES) Slog.v(TAG, "<<< done: " + codeToString(msg.what));
   }
```

我们目前只关心`LAUNCH_ACTIVITY`这个标记, 这个内部我们可以看到通过`handleLaunchActivity()`方法来实现. 这个方法内部通过**PerformLaunchActivity()**方法最终完成了`Activity`对象的创建和启动过程. 并且`ActivityThread`通过`handleResumeActivity()`方法来调用被启动的`onResume()`这一生命周期方法.

**那么PerformLaunchActivity()究竟做了那几件事情完成了Activity的形成呢.**

------

**1. 从ActivityClientRecord中获取待启动的Activity信息**

```
if (r.packageInfo == null) {
  r.packageInfo = getPackageInfo(aInfo.applicationInfo, r.compatInfo,
          Context.CONTEXT_INCLUDE_CODE);
}

ComponentName component = r.intent.getComponent();
if (component == null) {
  component = r.intent.resolveActivity(
      mInitialApplication.getPackageManager());
  r.intent.setComponent(component);
}

if (r.activityInfo.targetActivity != null) {
  component = new ComponentName(r.activityInfo.packageName,
          r.activityInfo.targetActivity);
}
```

**2.通过Instrumentation的newActivity方法使用类加载器创建Activity对象**

```
java.lang.ClassLoader cl = r.packageInfo.getClassLoader();
activity = mInstrumentation.newActivity(
        cl, component.getClassName(), r.intent);
StrictMode.incrementExpectedActivityCount(activity.getClass());
r.intent.setExtrasClassLoader(cl);
r.intent.prepareToEnterProcess();
if (r.state != null) {
    r.state.setClassLoader(cl);
}
```

**3.通过LoadedApk的makeApplication()方法来尝试创建Application对象**

```
public Application makeApplication(boolean forceDefaultAppClass,
        Instrumentation instrumentation) {
    if (mApplication != null) {
        return mApplication;
    }

    Application app = null;

    String appClass = mApplicationInfo.className;
    if (forceDefaultAppClass || (appClass == null)) {
        appClass = "android.app.Application";
    }

    try {
        java.lang.ClassLoader cl = getClassLoader();
        if (!mPackageName.equals("android")) {
            initializeJavaContextClassLoader();
        }
        ContextImpl appContext = ContextImpl.createAppContext(mActivityThread, this);
        app = mActivityThread.mInstrumentation.newApplication(
                cl, appClass, appContext);
        appContext.setOuterContext(app);
    } catch (Exception e) {
        if (!mActivityThread.mInstrumentation.onException(app, e)) {
            throw new RuntimeException(
                "Unable to instantiate application " + appClass
                + ": " + e.toString(), e);
        }
    }
    mActivityThread.mAllApplications.add(app);
    mApplication = app;

    if (instrumentation != null) {
        try {
            instrumentation.callApplicationOnCreate(app);
        } catch (Exception e) {
            if (!instrumentation.onException(app, e)) {
                throw new RuntimeException(
                    "Unable to create application " + app.getClass().getName()
                    + ": " + e.toString(), e);
            }
        }
    }

    // Rewrite the R 'constants' for all library apks.
    SparseArray<String> packageIdentifiers = getAssets(mActivityThread)
            .getAssignedPackageIdentifiers();
    final int N = packageIdentifiers.size();
    for (int i = 0; i < N; i++) {
        final int id = packageIdentifiers.keyAt(i);
        if (id == 0x01 || id == 0x7f) {
            continue;
        }

        rewriteRValues(getClassLoader(), packageIdentifiers.valueAt(i), id);
    }

    return app;
}
```

从这个方法中可以看到, 如果`Application`这个对象已经被创建过, 那么就不会再重复创建了, 这也就意味着一个应用只有一个`Application`对象的原因. `Application`对象的创建也是通过`Instrumentation`类来完成, 这个过程和Activity对象的创建是一致的, 都是通过类加载器来实现. `Application`创建完毕后, 系统会通过`Instrumentation`的`callApplicationOnCreate()`方法来调用`Application#onCreate()`方法.

**4. 创建ContextImpl对象并通过Activity的attach方法来完成重要数据的初始化**

```
Context appContext = createBaseContextForActivity(r, activity);
               CharSequence title = r.activityInfo.loadLabel(appContext.getPackageManager());
               Configuration config = new Configuration(mCompatConfiguration);
               if (DEBUG_CONFIGURATION) Slog.v(TAG, "Launching activity "
                       + r.activityInfo.name + " with config " + config);
               activity.attach(appContext, this, getInstrumentation(), r.token,
                       r.ident, app, r.intent, r.activityInfo, title, r.parent,
                       r.embeddedID, r.lastNonConfigurationInstances, config,
                       r.referrer, r.voiceInteractor);
```

`ContextImpl`是一个很重要的数据结构, 它是Context的具体实现, `Context`中的大部分逻辑都是由`ContentImpl`来完成的. ContextImpl是通过Activity的`attach()`方法来和Activity建立关联的,除此之外, 在`attach()`中Activity还会完成Window的创建并建立自己和Window的关联, 这样当Window接收到外部输入事件收就可以将事件传递给Activity.

**5. 调用Activity的onCreate()方法**

`mInstrumentation.callActivityOnCreate(activity, r.state);`, 由于Activity的`onCreate()`已经被调用, 这也意味着Activity已经完成整个启动过程.

## Service的工作流程

### Service的启动过程

同样从`ContextImpl#startService()`这个方法作为入口.

这个方法会调用`startServiceCommon()`并返回. 这个方法内部通过`ActivityManagerNative.getDefault()`获得一个`AMS`并调用`startService()`开启一个服务. 在这里通过`AMS`来启动一个服务的行为是属于远程调用的过程.

看一下`startService()`

```
public ComponentName startService(IApplicationThread caller, Intent service,
        String resolvedType, int userId) {
    enforceNotIsolatedCaller("startService");
    // Refuse possible leaked file descriptors
    if (service != null && service.hasFileDescriptors() == true) {
        throw new IllegalArgumentException("File descriptors passed in Intent");
    }

    if (DEBUG_SERVICE)
        Slog.v(TAG, "startService: " + service + " type=" + resolvedType);
    synchronized(this) {
        final int callingPid = Binder.getCallingPid();
        final int callingUid = Binder.getCallingUid();
        final long origId = Binder.clearCallingIdentity();
        ComponentName res = mServices.startServiceLocked(caller, service,
                resolvedType, callingPid, callingUid, userId);
        Binder.restoreCallingIdentity(origId);
        return res;
    }
}
```

这段主要就是`AMS`通过`mServices`这个对象来完成`Service`后续的启动过程. 这里`mService`的对象类型是`ActivityServices`(这是一个辅助AMS进行Service管理的类, 包括Service的启动,绑定和停止等).

这里调用了`mServices.startServiceLocked()`然后这个方法最后又调用了`startServiceInnerLocked()`, 实现如下.

```
ComponentName startServiceInnerLocked(ServiceMap smap, Intent service,
           ServiceRecord r, boolean callerFg, boolean addToStarting) {
       ProcessStats.ServiceState stracker = r.getTracker();
       if (stracker != null) {
           stracker.setStarted(true, mAm.mProcessStats.getMemFactorLocked(), r.lastActivity);
       }
       r.callStart = false;
       synchronized (r.stats.getBatteryStats()) {
           r.stats.startRunningLocked();
       }
       String error = bringUpServiceLocked(r, service.getFlags(), callerFg, false);
       if (error != null) {
           return new ComponentName("!!", error);
       }

       if (r.startRequested && addToStarting) {
           boolean first = smap.mStartingBackground.size() == 0;
           smap.mStartingBackground.add(r);
           r.startingBgTimeout = SystemClock.uptimeMillis() + BG_START_TIMEOUT;
           if (DEBUG_DELAYED_SERVICE) {
               RuntimeException here = new RuntimeException("here");
               here.fillInStackTrace();
               Slog.v(TAG, "Starting background (first=" + first + "): " + r, here);
           } else if (DEBUG_DELAYED_STARTS) {
               Slog.v(TAG, "Starting background (first=" + first + "): " + r);
           }
           if (first) {
               smap.rescheduleDelayedStarts();
           }
       } else if (callerFg) {
           smap.ensureNotStartingBackground(r);
       }

       return r.name;
   }
```

`ServiceRecord`描述的是一个Service记录, `ServiceRecord`一直贯穿着整个Service的启动过程. `startServiceInnerLocked()`方法并没有完成具体的启动工作, 而是把后续的工作交给了`bringUpServiceLocked()`,在`bringUpServiceLocked()`又调用了`realStartServiceLocked()`方法. 这个方法算是真正的启动一个`Service`.

**realStartServiceLocked()**首先通过`app.thread.scheduleCreateService()`方法来创建`Service`对象并调用其`onCreate()`, 接着再通过`sendServiceArgsLoceked()`方法来调用Service的其他方法, 比如`onStartCommond`这两个过程均是进程间通信. `app.thread`是一个`Binder`对象. 具体实现看`ApplicationThread`即可. 在`Activity`启动流程的时候已经解释过了.

所以只查看`Application`对`Service`的启动过程的处理即可. 这对应着它的`scheduleCreateService()`.

```
public final void scheduleCreateService(IBinder token,
        ServiceInfo info, CompatibilityInfo compatInfo, int processState) {
    updateProcessState(processState, false);
    CreateServiceData s = new CreateServiceData();
    s.token = token;
    s.info = info;
    s.compatInfo = compatInfo;

    sendMessage(H.CREATE_SERVICE, s);
}
```

这个过程和`Activity`类似, 都是通过`H`来完成. 最终处理消息接收结果会调用`handleCreateService`.

```
private void handleCreateService(CreateServiceData data) {
  
    unscheduleGcIdler();

    LoadedApk packageInfo = getPackageInfoNoCheck(
            data.info.applicationInfo, data.compatInfo);
    Service service = null;
    try {
        java.lang.ClassLoader cl = packageInfo.getClassLoader();
        service = (Service) cl.loadClass(data.info.name).newInstance();
    } catch (Exception e) {
        if (!mInstrumentation.onException(service, e)) {
            throw new RuntimeException(
                "Unable to instantiate service " + data.info.name
                + ": " + e.toString(), e);
        }
    }

    try {
        if (localLOGV) Slog.v(TAG, "Creating service " + data.info.name);

        ContextImpl context = ContextImpl.createAppContext(this, packageInfo);
        context.setOuterContext(service);

        Application app = packageInfo.makeApplication(false, mInstrumentation);
        //----------------
        service.attach(context, this, data.info.name, data.token, app,
                ActivityManagerNative.getDefault());
        service.onCreate();
        //---------------
        mServices.put(data.token, service);
        try {
            ActivityManagerNative.getDefault().serviceDoneExecuting(
                    data.token, SERVICE_DONE_EXECUTING_ANON, 0, 0);
        } catch (RemoteException e) {
            // nothing to do.
        }
    } catch (Exception e) {
        if (!mInstrumentation.onException(service, e)) {
            throw new RuntimeException(
                "Unable to create service " + data.info.name
                + ": " + e.toString(), e);
        }
    }
}
```

**handleCreateService做了如下事情**

1. 通过类加载器创建`Service`的实例
2. 创建`Application`对象并调用其onCreate(), 当然`Application`创建过程只会有一次.
3. 接着创建`ContextImpl`对象并通过`Service`的attach方法建立二者之间的关系, 这个过程和`Activity`实际上是类似的. 毕竟Service和Activity都是一个Context.
4. 调用`onCreate()`并将Service对象存储到`ActivityThread`中的一个列表. 就是`final ArrayMap<IBinder, Service> mServices = new ArrayMap<IBinder, Service>();`

`onCreate()`方法被执行了也就意味着`Service`已经启动了. 除此之外, `ActivityThread`中还会通过`handleServiceArgs()`方法调用Service的`onStartCommand()`方法.

### Service的绑定过程

和启动过程一样, 绑定过程同样是从`ContextImpl`开始的. 先查看`bindServiceCommon()`.

```
private boolean bindServiceCommon(Intent service, ServiceConnection conn, int flags,
        UserHandle user) {
    IServiceConnection sd;
    if (conn == null) {
        throw new IllegalArgumentException("connection is null");
    }
    if (mPackageInfo != null) {
        sd = mPackageInfo.getServiceDispatcher(conn, getOuterContext(),
                mMainThread.getHandler(), flags);
    } else {
        throw new RuntimeException("Not supported in system context");
    }
    validateServiceIntent(service);
    try {
        IBinder token = getActivityToken();
        if (token == null && (flags&BIND_AUTO_CREATE) == 0 && mPackageInfo != null
                && mPackageInfo.getApplicationInfo().targetSdkVersion
                < android.os.Build.VERSION_CODES.ICE_CREAM_SANDWICH) {
            flags |= BIND_WAIVE_PRIORITY;
        }
        service.prepareToLeaveProcess();
        int res = ActivityManagerNative.getDefault().bindService(
            mMainThread.getApplicationThread(), getActivityToken(),
            service, service.resolveTypeIfNeeded(getContentResolver()),
            sd, flags, user.getIdentifier());
        if (res < 0) {
            throw new SecurityException(
                    "Not allowed to bind to service " + service);
        }
        return res != 0;
    } catch (RemoteException e) {
        return false;
    }
}
```

这段代码首先将客户端的`ServiceConnection`对象转化成为`ServiceDispatcher.InnerConnection`对象. 不能直接使用`ServiceConnection`对象必须借助于Binder才能让远程服务回调自己的方法. 而`ServiceDispatcher`的内部类`InnerConnection`刚好充当了Binder这个角色.

`ServiceDispatcher`的作用就是连接`ServiceConnection`和`InnerConnection`的作用. 这个过程由`LoadedApk`的`getServiceDispatcher()`方法完成. 实现如下:

```
public final IServiceConnection getServiceDispatcher(ServiceConnection c,
       Context context, Handler handler, int flags) {
   synchronized (mServices) {
       LoadedApk.ServiceDispatcher sd = null;
       ArrayMap<ServiceConnection, LoadedApk.ServiceDispatcher> map = mServices.get(context);
       if (map != null) {
           sd = map.get(c);
       }
       if (sd == null) {
           sd = new ServiceDispatcher(c, context, handler, flags);
           if (map == null) {
               map = new ArrayMap<ServiceConnection, LoadedApk.ServiceDispatcher>();
               mServices.put(context, map);
           }
           map.put(c, sd);
       } else {
           sd.validate(context, handler);
       }
       return sd.getIServiceConnection();
   }
}
```

`mService`是一个`ArrayMap`, 它存储了一个应用当前活动的`ServiceConnection`和`ServiceDispatcher`的映射关系.

系统首先会查找是否存在相同的`ServiceConnection`, 如果不存在就会重新创建一个`ServiceDispatch`对象并将其存储在`mService`中, 其中的key是`ServiceConnection`,value是`ServiceDispatcher`, 在`ServiceDispatcher`的内部又保存了`ServiceConnection`和`InnerConnection`对象. 当`Service`和客户端建立连接后, 系统会通过InnerConnection来调用`ServiceConnection`中的`onServiceConnected()`方法. 这个过程可能是跨进程的. 当`ServiceDispatcher`创建好了以后, `getServiceDispatcher`会返回其保存的`InnerConnection`对象.

接着`bindServiceCommon`方法会通过AMS完成Service的具体绑定, 这对应着`AMS#bindService()`方法.

```
public int bindService(IApplicationThread caller, IBinder token,
        Intent service, String resolvedType,
        IServiceConnection connection, int flags, int userId) {
    enforceNotIsolatedCaller("bindService");

    // Refuse possible leaked file descriptors
    if (service != null && service.hasFileDescriptors() == true) {
        throw new IllegalArgumentException("File descriptors passed in Intent");
    }

    synchronized(this) {
        return mServices.bindServiceLocked(caller, token, service, resolvedType,
                connection, flags, userId);
    }
}
```

然后`AMS`会调用`ActivityService#bindServiceLocked()`方法. 然后调用`bringUpServiceLocked()`, 继续调用. 发现调到了`realStartServiceLocked`. 这里面的逻辑和**启动过程**类似. 最终都是通过`ApplicationThread`来完成`Service`实例的创建并执行其onCreate()方法. 这里不再重复说明.

**与Service启动稍微不同的是**:

- 绑定过程: 会调用到`app.thread(ActivityThread)`的`scheduleBindService()`方法. 而这个过程的实现是在`ActiveService#requestServiceBindingLocked()`方法.

```
private final boolean requestServiceBindingLocked(ServiceRecord r,
        IntentBindRecord i, boolean execInFg, boolean rebind) {
    if (r.app == null || r.app.thread == null) {
        // If service is not currently running, can't yet bind.
        return false;
    }
    if ((!i.requested || rebind) && i.apps.size() > 0) {
        try {
            bumpServiceExecutingLocked(r, execInFg, "bind");
            r.app.forceProcessStateUpTo(ActivityManager.PROCESS_STATE_SERVICE);
            r.app.thread.scheduleBindService(r, i.intent.getIntent(), rebind,
                    r.app.repProcState);
            if (!rebind) {
                i.requested = true;
            }
            i.hasBound = true;
            i.doRebind = false;
        } catch (RemoteException e) {
            if (DEBUG_SERVICE) Slog.v(TAG, "Crashed while binding " + r);
            return false;
        }
    }
    return true;
}
```

`app.thread`实际上就是`ApplicationThread`. `ApplicationThread`的一系列以`schedule`开头的方法, 其内部都是通过Handler H来中转的.

**H的内部** 接收到`BIND_SERVICE`消息需要处理时, 会交给`ActivityThread#handleBindService()`. 在`handlerBindService`中, 首先根据`Service`的token取出`Service`对象. 然后调用`Service#onBind()`方法, Service的onBinder方法返回一个Binder对象给客户端使用. 原则上来说, `Service#onBind()`方法被调用后, `Service`就处于绑定状态, 但是`onBind`方法是`Service`的方法, 这个时候客户端并不知道已经成功连接`Service`, 所以还必须调用客户端的`ServiceConnection`中的`onServiceConnected()`, 这个过程是由`AMS#publishService()`来完成.

`handleBindService()`实现如下:

```
private void handleBindService(BindServiceData data) {
    Service s = mServices.get(data.token);
    if (DEBUG_SERVICE)
        Slog.v(TAG, "handleBindService s=" + s + " rebind=" + data.rebind);
    if (s != null) {
        try {
            data.intent.setExtrasClassLoader(s.getClassLoader());
            data.intent.prepareToEnterProcess();
            try {
                if (!data.rebind) {
                    IBinder binder = s.onBind(data.intent);
                    // 看我看我
                    ActivityManagerNative.getDefault().publishService(
                            data.token, data.intent, binder);
                } else {
                    s.onRebind(data.intent);
                    ActivityManagerNative.getDefault().serviceDoneExecuting(
                            data.token, SERVICE_DONE_EXECUTING_ANON, 0, 0);
                }
                ensureJitEnabled();
            } catch (RemoteException ex) {
            }
        } catch (Exception e) {
            if (!mInstrumentation.onException(s, e)) {
                throw new RuntimeException(
                        "Unable to bind to service " + s
                        + " with " + data.intent + ": " + e.toString(), e);
            }
        }
    }
}
```

还记得多次绑定会有什么效果么? `Service#onBind()`方法只会执行一次, 除非Service被终止了. 当Service的`onBind()`执行之后, 系统还需要告知客户端已经成功连接Service了. 这些过程是在`AMS#publishService()`实现.

```
public void publishService(IBinder token, Intent intent, IBinder service) {
    // Refuse possible leaked file descriptors
    if (intent != null && intent.hasFileDescriptors() == true) {
        throw new IllegalArgumentException("File descriptors passed in Intent");
    }

    synchronized(this) {
        if (!(token instanceof ServiceRecord)) {
            throw new IllegalArgumentException("Invalid service token");
        }
        mServices.publishServiceLocked((ServiceRecord)token, intent, service);
    }
}
```

可以看到这里将具体的工作交给了`mServices.publishServiceLocked()`它是一个`ActiveService`类型. 其核心代码就是:

```
c.conn.connected(r.name, service);
//c的类型是ConnectionRecord
//c.conn的类型是ServiceDispatcher.InnerConnection
//service参数就是Service的onBind返回的Binder对象
```

而`InnerConnection#connected()`方法内又调用`ServiceDispatcher的connected()`内部就是创建一个`RunConnection()`发送到`mActivityThread`的消息中.

对于`Service`的绑定过程来说, `ServiceDispatcher`的`mActivityThread`是一个Handler, 就是`ActivityThread#H`, 从前面的`ServiceDispatcher`的创建过程来说, `mActivityThread`不会为null, 所以`RunConnection`就可以经由`H`的post方法从而运行在主线程. 因此客户端的`ServiceConnection`中的方法是在主线程被回调的.

**RunConnection**是一个`Runnable`接口, `run()`方法也是简单调用`ServiceDispatcher#doConnected方法`, 由于`ServiceDispatcher`内部保存了客户端的`ServiceConnection`对象, 因此他可以很方便调用`ServiceConnection`对象的`onServiceConnected()`

------

解绑和停止过程, 基本类似… 恩 书上这么说的.

## BroadcastReceiver的工作流程

简单回顾一下广播的使用方法, 首先定义广播接收者, 只需要继承`BroadcastReceiver`并重写`onReceive()`方法即可. 定义好了广播接收者, 还需要注册广播接收者, 分为两种静态注册或者动态注册. 注册完成之后就可以发送广播了.

### 广播的注册过程

广播的注册有两种**静态注册**, **动态注册**. 其中**静态注册**的广播在应用安装时由系统自动完成注册, 具体来说是有**PMS**(PackageManagerService)来完成整个注册过程的. 除了广播外, 其他三大组件也都是在应用安装时由`PMS`解析并注册的.

**动态注册**的过程是从`ContextWrapper#registerReceiver()`开始的. 和`Activity`或者`Service`一样. `ContextWrapper`并没有做实际的工作, 而是将注册的过程直接交给了`ContextImpl`来完成.

`ContextImpl#registerReceiver()`方法调用了本类的`registerReceiverInternal()`方法.

```
private Intent registerReceiverInternal(BroadcastReceiver receiver, int userId,
        IntentFilter filter, String broadcastPermission,
        Handler scheduler, Context context) {
    IIntentReceiver rd = null;
    if (receiver != null) {
        if (mPackageInfo != null && context != null) {
            if (scheduler == null) {
                scheduler = mMainThread.getHandler();
            }
            rd = mPackageInfo.getReceiverDispatcher(
                receiver, context, scheduler,
                mMainThread.getInstrumentation(), true);
        } else {
            if (scheduler == null) {
                scheduler = mMainThread.getHandler();
            }
            rd = new LoadedApk.ReceiverDispatcher(
                    receiver, context, scheduler, null, true).getIIntentReceiver();
        }
    }
    try {
        return ActivityManagerNative.getDefault().registerReceiver(
                mMainThread.getApplicationThread(), mBasePackageName,
                rd, filter, broadcastPermission, userId);
    } catch (RemoteException e) {
        return null;
    }
}
```

上述代码中, 系统首先从`mPackageInfo`获取到`IIntentReceiver`对象, 然后再采用跨进程的方式向AMS发送广播注册的请求. 之所以采用`IIntentReceiver`而不是直接采用`BroadcastReceiver`, 这是因为上述注册过程中是一个进程间通信的过程. 而`BroadcastReceiver`作为Android中的一个组件是不能直接跨进程传递的. 所有需要通过`IIntentReceiver`来中转一下.

`IIntentReceiver`作为一个`Binder`接口, 它的具体实现是`LoadedApk.ReceiverDispatcher.InnerReceiver`, `ReceiverDispatcher`的内部同时保存了`BroadcastReceiver`和`InnerReceiver`, 这样当接收到广播的时候, `ReceiverDispatcher`可以很方便的调用`BroadcastReceiver#onReceive()`方法. 这里和Service很像有同样的类, 并且内部类中同样也是一个Binder接口.

看一下`LoadedApk.ReceiverDispatcher#getIIntentReceiver()`的实现, 很显然`getReceiverDispatcher()`重新创建了一个`ReceiverDispatcher`对象并将其保存的`InnerReceiver`对象作为返回值返回, 其中`InnerReceiver`对象和`BroadcastReceiver`都是在`ReceiverDispatcher`的构造方法中被保存起来的.

```
public IIntentReceiver getReceiverDispatcher(BroadcastReceiver r,
        Context context, Handler handler,
        Instrumentation instrumentation, boolean registered) {
    synchronized (mReceivers) {
        LoadedApk.ReceiverDispatcher rd = null;
        ArrayMap<BroadcastReceiver, LoadedApk.ReceiverDispatcher> map = null;
        if (registered) {
            map = mReceivers.get(context);
            if (map != null) {
                rd = map.get(r);
            }
        }
        if (rd == null) {
            rd = new ReceiverDispatcher(r, context, handler,
                    instrumentation, registered);
            if (registered) {
                if (map == null) {
                    map = new ArrayMap<BroadcastReceiver, LoadedApk.ReceiverDispatcher>();
                    mReceivers.put(context, map);
                }
                map.put(r, rd);
            }
        } else {
            rd.validate(context, handler);
        }
        rd.mForgotten = false;
        return rd.getIIntentReceiver();
    }
}
```

由于注册广播真正实现过程是在`AMS`中, 因此跟进AMS中, 首先看`registerReceiver()`方法, 这里只关心里面的核心部分. 这段代码最终会把远程的**InnerReceiver**对象以及**IntentFilter**对象存储起来, 这样整个广播的注册就完成了.

```
public Intent registerReceiver(IApplicationThread caller, String callerPackage,
       IIntentReceiver receiver, IntentFilter filter, String permission, int userId) {
    //...
    mRegisteredReceivers.put(receiver.asBinder(), rl);
    BroadcastFilter bf = new BroadcastFilter(filter, rl, callerPackage,permission, callingUid, userId);
    rl.add(bf);
   
    mReceiverResolver.addFilter(bf);  
}
```

### 广播的发送和接收过程

当通过`send()`发送广播时, AMS会查找出匹配的广播接收者并将广播发送给他们处理. 广播的发送种类有: **普通广播**, **有序广播**, **粘性广播**. 这里分析普通广播.

广播的发送和接收, 本质就是一个过程的两个阶段. 广播的发送仍然开始于`ContextImpl#sendBroadcase()`方法, 之所以不是`Context`, 那是因为`Context#sendBroad()`是一个抽象方法. 和广播的注册过程一样, `ContextWrapper#sendBroadcast()`仍然什么都不做, 只是把事情交给了`ContextImpl`去处理, `ContextImpl#sendBroadcast()`源码如下

```
@Override
public void sendBroadcast(Intent intent) {
   warnIfCallingFromSystemProcess();
   String resolvedType = intent.resolveTypeIfNeeded(getContentResolver());
  
   intent.prepareToLeaveProcess();
   ActivityManagerNative.getDefault().broadcastIntent(
          mMainThread.getApplicationThread(), intent, resolvedType, null,
          Activity.RESULT_OK, null, null, null, AppOpsManager.OP_NONE, null, false, false,
          getUserId());
}
```

看到`ContextImpl`里面也几乎什么都没有做, 内部直接向`AMS`发起了一个异步请求用于发送广播. 接下来看`AMS#broadcastIntent()`方法.

```
public final int broadcastIntent(IApplicationThread caller,
       Intent intent, String resolvedType, IIntentReceiver resultTo,
       int resultCode, String resultData, Bundle resultExtras,
       String[] requiredPermissions, int appOp, Bundle options,
       boolean serialized, boolean sticky, int userId) {
   enforceNotIsolatedCaller("broadcastIntent");
   synchronized(this) {
       intent = verifyBroadcastLocked(intent);

       final ProcessRecord callerApp = getRecordForAppLocked(caller);
       final int callingPid = Binder.getCallingPid();
       final int callingUid = Binder.getCallingUid();
       final long origId = Binder.clearCallingIdentity();
       int res = broadcastIntentLocked(callerApp,
               callerApp != null ? callerApp.info.packageName : null,
               intent, resolvedType, resultTo, resultCode, resultData, resultExtras,
               requiredPermissions, appOp, null, serialized, sticky,
               callingPid, callingUid, userId);
       Binder.restoreCallingIdentity(origId);
       return res;
   }
}
```

看到这里, 又继续调用`broadcastIntentLocked()`方法, 这个方法有点长. 在代码开始处

```
// By default broadcasts do not go to stopped apps.
intent.addFlags(Intent.FLAG_EXCLUDE_STOPPED_PACKAGES);
```

这个表示默认情况下广播不会发送给已经停止的应用, android5.0中. 而android 3.1开始就增添了两个标记为. 分别是`FLAG_INCLUDE_STOPPED_PACKAGES`, `FLAG_EXCLUDE_STOPPED_PACKAGES`. 用来控制广播是否要对处于停止的应用起作用.

- `FLAG_INCLUDE_STOPPED_PACKAGES`: 包含停止应用, 广播会发送给已停止的应用.
- `FLAG_EXCLUDE_STOPPED_PACKAGES`: 不包含已停止应用, 广播不会发送给已停止的应用

在**android 3.1**开始, 系统就为所有广播默认添加了`FLAG_EXCLUDE_STOPPED_PACKAGES`标识, **为了防止广播无意间或者不必要的时候调起已经停止运行的应用**. 当这两个标记共存的时候以`FLAG_INCLUDE_STOPPED_PACKAGES`(非默认项为主).

------

应用处于停止分为两种

1. 应用安装后未运行
2. 被手动或者其他应用强停

开机广播同样受到了这个标志位的影响. 从Android 3.1开始处于停止状态的应用同样无法接受到开机广播, 而在`android 3.1`之前处于停止的状态也是可以接收到的开机广播的.

在`broadcastIntentLocked()`内部, 会根据intent-filter查找出匹配的广播接收者并经过一系列的条件过滤. 最终会将满足条件的广播接收者添加到`BroadcastQueue`中, 接着`BroadcastQueue`就会将广播发送给相应广播接收者.

```
if ((receivers != null && receivers.size() > 0)
      || resultTo != null) {
  BroadcastQueue queue = broadcastQueueForIntent(intent);
  BroadcastRecord r = new BroadcastRecord(queue, intent, callerApp,
          callerPackage, callingPid, callingUid, resolvedType,
          requiredPermissions, appOp, brOptions, receivers, resultTo, resultCode,
          resultData, resultExtras, ordered, sticky, false, userId);

  if (DEBUG_BROADCAST) Slog.v(TAG_BROADCAST, "Enqueueing ordered broadcast " + r
          + ": prev had " + queue.mOrderedBroadcasts.size());
  if (DEBUG_BROADCAST) Slog.i(TAG_BROADCAST,
          "Enqueueing broadcast " + r.intent.getAction());

  boolean replaced = replacePending && queue.replaceOrderedBroadcastLocked(r);
  if (!replaced) {
      queue.enqueueOrderedBroadcastLocked(r);
      queue.scheduleBroadcastsLocked();
  }
}
```

跟进`BroadcastQueue#scheduleBroadcastsLocked()`

```
public void scheduleBroadcastsLocked() {
   if (DEBUG_BROADCAST) Slog.v(TAG_BROADCAST, "Schedule broadcasts ["
           + mQueueName + "]: current="
           + mBroadcastsScheduled);

   if (mBroadcastsScheduled) {
       return;
   }
   mHandler.sendMessage(mHandler.obtainMessage(BROADCAST_INTENT_MSG, this));
   mBroadcastsScheduled = true;
}
```

方法内并没有立即发送广播, 而是发送了一个`BROADCAST_INTENT_MSG`类型的消息, `BroadcastQueue`收到消息后会调用`processNextBroadcast()`方法. 这个方法对普通广播的处理如下:

```
// First, deliver any non-serialized broadcasts right away.
  while (mParallelBroadcasts.size() > 0) {
      r = mParallelBroadcasts.remove(0);
      r.dispatchTime = SystemClock.uptimeMillis();
      r.dispatchClockTime = System.currentTimeMillis();
      final int N = r.receivers.size();
      if (DEBUG_BROADCAST_LIGHT) Slog.v(TAG_BROADCAST, "Processing parallel broadcast ["
              + mQueueName + "] " + r);
      for (int i=0; i<N; i++) {
          Object target = r.receivers.get(i);
          if (DEBUG_BROADCAST)  Slog.v(TAG_BROADCAST,
                  "Delivering non-ordered on [" + mQueueName + "] to registered "
                  + target + ": " + r);
          deliverToRegisteredReceiverLocked(r, (BroadcastFilter)target, false);
      }
      addBroadcastToHistoryLocked(r);
      if (DEBUG_BROADCAST_LIGHT) Slog.v(TAG_BROADCAST, "Done with parallel broadcast ["
              + mQueueName + "] " + r);
  }
```

无序广播存储在`mParallelBroadcasts`中, 系统会遍历这个集合并将其中的广播发送给他们所有的接收者, 具体的发送过程是通过`deliverToRegisteredReceiverLocked()`方法实现. `deliverToRegisteredReceiverLocked()`负责将一个广播发送给一个特定的接收者, 它的内部调用了`performReceiverLocked`方法来完成具体发送过程.

`performReceiverLocked()`方法实现如下, 由于接收广播会调起应用程序, 因为`app.thread`不为null, 根据前面的总结`app.thread`仍然指`ApplicationThread`.

```
private static void performReceiveLocked(ProcessRecord app, IIntentReceiver receiver,
            Intent intent, int resultCode, String data, Bundle extras,
            boolean ordered, boolean sticky, int sendingUser) throws RemoteException {
        // Send the intent to the receiver asynchronously using one-way binder calls.
        if (app != null) {
            if (app.thread != null) {
                // If we have an app thread, do the call through that so it is
                // correctly ordered with other one-way calls.
                app.thread.scheduleRegisteredReceiver(receiver, intent, resultCode,
                        data, extras, ordered, sticky, sendingUser, app.repProcState);
            } else {
                // Application has died. Receiver doesn't exist.
                throw new RemoteException("app.thread must not be null");
            }
        } else {
            receiver.performReceive(intent, resultCode, data, extras, ordered,
                    sticky, sendingUser);
        }
    }
```

而调用的`ApplicationThread#scheduleRegisteredReceiver()`实现比较简单, 它通过`InnerReceiver`来实现广播的接收

```
public void scheduleRegisteredReceiver(IIntentReceiver receiver, Intent intent,
        int resultCode, String dataStr, Bundle extras, boolean ordered,
        boolean sticky, int sendingUser, int processState) throws RemoteException {
    updateProcessState(processState, false);
    receiver.performReceive(intent, resultCode, dataStr, extras, ordered,
            sticky, sendingUser);
}
```

上面的`receiver.performReceive()`中的`receiver`对应着`IIntentReceiver`类型的接口. 而具体的实现就是`ReceiverDispatcher$InnerReceiver`. 这两个嵌套的内部类是所属在`LoadedApk`中的. 好了看一下`performReceiver()`

```
public void performReceive(Intent intent, int resultCode, String data,
                    Bundle extras, boolean ordered, boolean sticky, int sendingUser) {
                LoadedApk.ReceiverDispatcher rd = mDispatcher.get();
                if (ActivityThread.DEBUG_BROADCAST) {
                    int seq = intent.getIntExtra("seq", -1);
                    Slog.i(ActivityThread.TAG, "Receiving broadcast " + intent.getAction() + " seq=" + seq
                            + " to " + (rd != null ? rd.mReceiver : null));
                }
                if (rd != null) {
                    rd.performReceive(intent, resultCode, data, extras,
                            ordered, sticky, sendingUser);
                } else {
                    // The activity manager dispatched a broadcast to a registered
                    // receiver in this process, but before it could be delivered the
                    // receiver was unregistered.  Acknowledge the broadcast on its
                    // behalf so that the system's broadcast sequence can continue.
                    if (ActivityThread.DEBUG_BROADCAST) Slog.i(ActivityThread.TAG,
                            "Finishing broadcast to unregistered receiver");
                    IActivityManager mgr = ActivityManagerNative.getDefault();
                    try {
                        if (extras != null) {
                            extras.setAllowFds(false);
                        }
                        mgr.finishReceiver(this, resultCode, data, extras, false, intent.getFlags());
                    } catch (RemoteException e) {
                        Slog.w(ActivityThread.TAG, "Couldn't finish broadcast to unregistered receiver");
                    }
                }
}
```

上面又调用了`LoadedApk$ReceiverDispatcher#performReceive()`的方法.

```
public void performReceive(Intent intent, int resultCode, String data,
                Bundle extras, boolean ordered, boolean sticky, int sendingUser) {
            if (ActivityThread.DEBUG_BROADCAST) {
                int seq = intent.getIntExtra("seq", -1);
                Slog.i(ActivityThread.TAG, "Enqueueing broadcast " + intent.getAction() + " seq=" + seq
                        + " to " + mReceiver);
            }
            Args args = new Args(intent, resultCode, data, extras, ordered,
                    sticky, sendingUser);
            if (!mActivityThread.post(args)) {
                if (mRegistered && ordered) {
                    IActivityManager mgr = ActivityManagerNative.getDefault();
                    if (ActivityThread.DEBUG_BROADCAST) Slog.i(ActivityThread.TAG,
                            "Finishing sync broadcast to " + mReceiver);
                    args.sendFinished(mgr);
                }
            }
        }
```

在`performReceiver()`这个方法中, 会创建一个`Args`对象并通过`mActivityThread`的**post**方法执行`args`中的逻辑. 而这些类的本质关系就是:

- Args: 实现类Runnable
- mActivityThread: 是一个Handler, 就是`ActivityThread`中的mH. mH就是`ActivityThread$H`. 这个内部类H以前说过.

而实现Runnable接口的`Args`中有如下代码:

```
ClassLoader cl =  mReceiver.getClass().getClassLoader();
intent.setExtrasClassLoader(cl);
setExtrasClassLoader(cl);
receiver.setPendingResult(this);
receiver.onReceive(mContext, intent);
```

这个时候`BroadcastReceiver#onReceive()`方法被执行了, 也就是说应用已经接收到了广播, 同时`onReceive()`方法是在广播接收者的主线程中被调用的.

## ContentProvider的工作机制

`ContentProvider`是一种内容共享型组件, 它通过`Binder`向其他组件乃至其他应用提供数据. 当`ContentProvider`所在的进程启动时, `ContentProvider`会同时启动并发布到AMS中. 要注意:**这个时候ContentProvider的onCreate()方法是先于Application的onCreate()执行的**这一点在四大组件是少有的现象.

当一个应用启动的时候, 入口的方法为`ActivityThread#main()`方法, main方法为一个静态方法, 在main方法中会创建`ActivityThread`的实例, 并创建主线程的消息队列, 然后在`ActivityThread#attach()`方法中会远程调用`AMS#attachApplication()`并将`ApplicationThread`对象提供给AMS.

`ApplicationThread`是一个`Binder`对象, 它的Binder接口是`IApplicationThread`, 主要用于`ActivityThread`和`AMS`之间的通信, 这一点在前面多次提到. 在`AMS`的`attachApplication()`中, 会调用`ApplicationThread#bindApplication()`. 这个过程同样是跨进程的. `bindApplication()`中会经过 `ActivityThread`中的mh(Handler) 切换到`ActivityThread`中去执行, 具体的方式是`handleBindApplication()`.

在`handleBindApplication()`方法中, `ActivityThread`会创建`Application`对象并加载`ContentProvider`. 需要注意的是, `ActivityThread`会先加载`ContentProvider`, 然后在调用`Application#onCreate()`方法

------

以上就是`ContentProvider`的启动过程, `ContentProvider`启动后, 外界就可以通过它所提供的增删改查这四个接口来操作`ContentProvider`中的数据源, 这四个方法都是通过Binder来调用的, 外界无法直接访问`ContentProvider`, 它只能通过`AMS`根据`URI`来获取到对应的`ContentProvider`的`Binder`接口`IContentProvider`, 然后再通过`IContentProvider`来访问`ContentProvider`中的数据源.

**ContentProvider是否属于单实例?**

具体`ContentProvider`是否是单实例取决于`android:multiprocess`属性来决定的, 当其值为**false**的时候, 就是单实例也是默认值. 如果为**true**那就为多实例. 这个时候在每一个调用者的进程中都会存在一个`ContentProvider`对象.

------

**通过单实例的ContentProvider来分析一下启动过程**

首先访问`ContentProvider`需要通过`ContentResolver`, `ContentResolver`是一个抽象类, 通过`Content#getContentResolver()`方法获取的实际上是`ApplicationContentResolver`对象, 而这个类继承了`ContentProvider`并实现了其抽象方法. 当`ContentProvider`所在的进程未启动时, 第一次访问它的时候就会触发`ContentProvider`的创建, 当然这也伴随着`ContentProvider`所在的进程的启动. 通过四个对数据的操作方法中的任何一个, 都可以触发`ContentProvider`的启动过程.

**四种操作过程差不多, 那么这里以query方法为例**

首先会获取`IContentProvider`对象, 不管是通过`acquireUnstableProvider()`方法还是直接通过`acquireProvider()`方法, 他们的本质都是一样的, 最终都是通过`acquireProvider`方法来获取ContentProvider.

`ApplicationContentResolver#acquireProvider()`方法并没有处理任何逻辑, 它直接调用了`ActivityThread#acquireProvider()`, 这个方法如下:

```
public final IContentProvider acquireProvider(
            Context c, String auth, int userId, boolean stable) {
        final IContentProvider provider = acquireExistingProvider(c, auth, userId, stable);
        if (provider != null) {
            return provider;
        }

        IActivityManager.ContentProviderHolder holder = null;
        try {
            holder = ActivityManagerNative.getDefault().getContentProvider(
                    getApplicationThread(), auth, userId, stable);
        } catch (RemoteException ex) {
        }
        if (holder == null) {
            Slog.e(TAG, "Failed to find provider info for " + auth);
            return null;
        }

        // Install provider will increment the reference count for us, and break
        // any ties in the race.
        holder = installProvider(c, holder, holder.info,
                true /*noisy*/, holder.noReleaseNeeded, stable);
        return holder.provider;
}
```

这段代码主要从`ActivityThread`中查找是否已经存在了`ContentProvider`了, 如果存在那么就直接返回. `ActivityThread`中通过`mProviderMap`来存储已经启动的`ContentProvider`对象, 这个集合的存储类型`ArrayMap<ProviderKey, ProviderClientRecord> mProviderMap`. 如果目前`ContentProvider`没有启动, 那么就发送一个进程间请求给`AMS`让其启动项目目标`ContentProvider`, 最后再通过`installProvider()`方法来修改引用计数.

那么AMS是如何启动`ContentProvider`的呢?

关于`ContentProvider`被启动的时候会伴随着进程的启动, 在`AMS`中, 首先会启动`ContentProvider`所在的进程, 然后再启动`ContentProvider`. 启动进程是由`AMS#startProcessLocked()`方法来完成, 其内部主要是通过`Process#start()`方法来完成一个新进程的启动, 新进程启动后其入口方法为`ActivityThread#main()`方法. 如下:

```
    public static void main(String[] args) {
        Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ActivityThreadMain");
        SamplingProfilerIntegration.start();

        // CloseGuard defaults to true and can be quite spammy.  We
        // disable it here, but selectively enable it later (via
        // StrictMode) on debug builds, but using DropBox, not logs.
        CloseGuard.setEnabled(false);

        Environment.initForCurrentUser();

        // Set the reporter for event logging in libcore
        EventLogger.setReporter(new EventLoggingReporter());

        AndroidKeyStoreProvider.install();

        // Make sure TrustedCertificateStore looks in the right place for CA certificates
        final File configDir = Environment.getUserConfigDirectory(UserHandle.myUserId());
        TrustedCertificateStore.setDefaultUserDirectory(configDir);

        Process.setArgV0("<pre-initialized>");

        Looper.prepareMainLooper();

        ActivityThread thread = new ActivityThread();
        thread.attach(false);

        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }

        if (false) {
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }

        // End of event ActivityThreadMain.
        Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
        Looper.loop();

        throw new RuntimeException("Main thread loop unexpectedly exited");
}
```

`ActivityThread#main()`是一个静态方法, 在它的内部首先会创建`ActivityThread`实例并调用`attach()`方法来进行一系列初始化, 接着就开始进行消息循环. `ActivityThread#attach()`方法会将`Application`对象通过`AMS#attachApplication`方法跨进程传递给AMS, 最终AMS会完成`ContentProvider`的创建过程.

`ActivityThread#attach()`部分代码:

```
try {
      mgr.attachApplication(mAppThread);
  } catch (RemoteException ex) {
      // Ignore
  }
```

`AMS#attachApplication()`方法调用了`attachApplication()`, 然后又调用了`ApplicationThread#bindApplication()`, 这个过程也属于进程通信.

而上面的`bindApplication()`方法会发送一个`BIND_APPLICATION`类型的消息给`mH`, 这是一个Handler, 它收到消息后会调用`ActivityThread#handleBindApplication()`方法. `bindApplication()`发送源码如下:

```
AppBindData data = new AppBindData();
data.processName = processName;
data.appInfo = appInfo;
data.providers = providers;
data.instrumentationName = instrumentationName;
data.instrumentationArgs = instrumentationArgs;
data.instrumentationWatcher = instrumentationWatcher;
data.instrumentationUiAutomationConnection = instrumentationUiConnection;
data.debugMode = debugMode;
data.enableOpenGlTrace = enableOpenGlTrace;
data.restrictedBackupMode = isRestrictedBackupMode;
data.persistent = persistent;
data.config = config;
data.compatInfo = compatInfo;
data.initProfilerInfo = profilerInfo;
sendMessage(H.BIND_APPLICATION, data);
```

`ActivityThread#handlerBindApplication()`则完成了Application的创建以及`ContentProvider` 可以分为如下四个步骤:

1. 创建`ContentProvider`和`Instrumentation`
2. 创建`Application`对象
3. 启动当前进程的`ContentProvider`并调用onCreate()方法. 主要内部实现是`installContentProvider()`完成了`ContentProvider`的启动工作, 首先会遍历当前进程的`ProviderInfo`的列表并一一调用`installProvider()`方法来启动他们, 接着将已经启动的`ContentProvider`发布到AMS中, AMS会把他们存储在`ProviderMap`中, 这样一来外部调用者就可以直接从AMS中获取到`ContentProvider`. **installProvider()**内部通过类加载器创建的`ContentProvider`实例并在方法中调用了`attachInfo()`, 在这内部调用了`ContentProvider#onCreate()`
4. 调用`Application#onCreate()`

经过了上述的四个步骤, `ContentProvider`已经启动成功, 并且其所在的进程的`Application`也已经成功, 这意味着`ContentProvider`所在的进程已经完成了整个的启动过程, 然后其他应用就可以通过`AMS`来访问这个`ContentProvider`了.

当拿到了`ContentProvider`以后, 就可以通过它所提供的接口方法来访问它. 这里要注意: **这里的ContentProvider并不是原始的ContentProvider**. 而是`ContentProvider`的Binder类型对象`IContentProvider`, 而`IContentProvider`的具体实现是`ContentProviderNative`和`ContentProvider.Transport`. 后者继承了前者.

如果还用query方法来解释流程: 那么最开始其他应用通过`AMS`获取到`ContentProvider`的`Binder`对象就是`IContentProvider`. 而`IContentProvider`的实际实现者是`ContentProvider.Transport`. 因此实际上外部应用调用的时候本质上**会以进程间通信的方式调用ContentProvider.Transport的query()方法**