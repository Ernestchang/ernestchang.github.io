title: 《Android开发艺术探索》Chap13_综合技术
date: 

categories: 
- android 
- 《Android开发艺术探索》
tags:  
- android
---

> crash的异常处理, Android中的方法限制, 反编译等也是需要了解的技术.
> 注：本文主要参考[http://szysky.com](http://szysky.com)


[blog相关代码](https://github.com/suzeyu1992/Notes_AndroidDevSeek)

## 捕捉Crash信息

应用总会不可避免的发生Crash, 有可能是因为底层bug, 或者是适配问题, 代码逻辑问题等等. 当发生Crash的时候, 系统会kill掉正在执行的程序, 现象就是闪退或者提示用户程序已经停止运行, 这对用户来说是很不友好, 也是开发者不愿意看到的. 系统提供了在**程序Crash之前可以保存异常信息的能力, 这样在崩溃前可以保存异常信息到文件中, 传回服务器,这样可以针对各种各样的用户发生的错误对应用进行完善**的方法, 这个类在`Thread`中的`setDefaultUncaughtException()`方法.

```
/**
* Sets the default uncaught exception handler. This handler is invoked in
* case any Thread dies due to an unhandled exception.
*
* @param handler The handler to set or null.
*/
public static void setDefaultUncaughtExceptionHandler(UncaughtExceptionHandler handler) {
   Thread.defaultUncaughtHandler = handler;
}
```

从方法名的意思看出设置默认未捕获异常的处理器. 就是当Crash发生的时候, 系统会回调`UncaughtExceptionHandler#uncaughtException()`方法, 在这个方法中可以获取到异常的信息, 可以选择把信息写到SD卡中, 然后在合适的时机通过网络将Crash信息上传到服务器上. 也可以在这里弹出一个对话框告诉用户, 会比直接闪退好那么一点.

首先要实现出一个`UncaughtExceptionHandler`的子类. 直接看实现:

```
public class MyCrashHandler implements Thread.UncaughtExceptionHandler {

    private static final String TAG = MyCrashHandler.class.getSimpleName();

    /**
     * 定义异常文件要存储的路径
     */
    private static final String PATH = Environment.getExternalStorageDirectory().getPath() + "/szyCrash/log/";

    private static final String FILE_NAME = "crash-";

    private static final String FILE_NAME_SUFFIX = ".trace";

    private Thread.UncaughtExceptionHandler mDefaultUncaughtExceptionHandler;

    private Context mContext;

    /**
     * 单例模式三部曲    懒汉式
     */
    private static MyCrashHandler sInstance = new MyCrashHandler();

    private MyCrashHandler() {
    }

    public static MyCrashHandler getsInstance() {
        return sInstance;
    }


    /**
     * 设置本类为当前进行的默认未捕捉异常处理器
     */
    public void init(Context context) {
        mContext = context.getApplicationContext();
        mDefaultUncaughtExceptionHandler = Thread.getDefaultUncaughtExceptionHandler();
        Thread.setDefaultUncaughtExceptionHandler(this);

    }


    @Override
    public void uncaughtException(Thread thread, Throwable ex) {
        try {
            // 导出信息到sd卡上
            dumpExceptionToSDCard(ex);

            // 对本地的信息进行上传服务器处理
            uploadExceptionToServer();

        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            // 也把异常打印到控制台 防止系统没有默认异常处理器导致的没有崩溃信息
            ex.printStackTrace();

            // 如果系统有默认的异常处理器那么就给系统处理  否则自己关闭掉
            // 如果不交给系统 或者 自己手动杀掉, 那么应用就会进入假死, 点击会出现ANR
            if (mDefaultUncaughtExceptionHandler != null) {
                ex.printStackTrace();
                mDefaultUncaughtExceptionHandler.uncaughtException(thread, ex);
            } else {
                Process.killProcess(Process.myPid());
            }
        }


    }

    /**
     * 上传发生未捕获的错误日志
     */
    private void uploadExceptionToServer() {
        // TODO: 16/8/25   看具体需求, 也可以在每次启动app后检测本地是否有异常日志
    }

    /**
     * 开始异常信息进行本地存储处理
     *
     * @param ex 程序发生的异常对象
     * @throws IOException 写入文件发生异常
     */
    private void dumpExceptionToSDCard(Throwable ex) throws IOException {
        // 首先判断sd卡是否被挂起
        if (!Environment.getExternalStorageState().equals(Environment.MEDIA_MOUNTED)) {
            Log.w(TAG, "sd卡不可用, 跳过 dump Exception");
            return;
        }

        File dir = new File(PATH);
        if (!dir.exists()) {
            dir.mkdirs();
        }

        // 获取日志记录时间
        long currentTimeMillis = System.currentTimeMillis();
        String formatTimeStr = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss").format(new Date(currentTimeMillis));

        // 创建存储异常的文件
        File file = new File(PATH + FILE_NAME + formatTimeStr + FILE_NAME_SUFFIX);


        try {
            // 创建写入流, 开始哗哗写东西
            PrintWriter pw = new PrintWriter(new BufferedWriter(new FileWriter(file)));
            pw.println(formatTimeStr);
            dumpPhoneInfo(pw);
            pw.println();
            pw.println("**********************异常信息**************************");
            ex.printStackTrace(pw);
            pw.close();


        } catch (PackageManager.NameNotFoundException e) {
            Log.e(TAG, "存储未捕获异常失败了.");
        }


    }

    /**
     * 读取当前设备信息  并追加到异常日志中
     */
    private void dumpPhoneInfo(PrintWriter pw) throws PackageManager.NameNotFoundException {
        // 获得应用包管理者 并获取存储当前应用的信息对象
        PackageManager pm = mContext.getPackageManager();
        PackageInfo pi = pm.getPackageInfo(mContext.getPackageName(), PackageManager.GET_ACTIVITIES);

        pw.println("**********************设备信息**************************");
        // 开始写入 应用信息
        pw.println("App Version ");
        pw.print("    VersionName: ");
        pw.println(pi.versionName);
        pw.print("    VersionCode: ");
        pw.println(pi.versionCode);

        // 开始写入 Android版本信息
        pw.println("OS Version ");
        pw.println("    SDK_NAME: " + Build.VERSION.RELEASE);
        pw.println("    SDK_INT: " + Build.VERSION.SDK_INT);


        // 开始写入 手机制造商
        pw.println("Vendor ");
        pw.println("    " + Build.MANUFACTURER);


        // 开始写入 手机型号
        pw.println("Model ");
        pw.println("    " + Build.MODEL);


        // 开始写入 CPU架构
        pw.println("CPU ABI ");
        String[] supportedAbis = SUPPORTED_ABIS;
        for (int i = 0; i < supportedAbis.length; i++) {
            pw.println("    " + supportedAbis[i]);
        }
        pw.println("*************************end***************************");


    }

}
```

核心方法就是复写了这个类的`uncaughtException()`方法. 别忘了在`Application`中进行这个类的`init()`方法调用. 这样才会把自定义的捕捉未捕获的异常这个类注册到系统.

看一下模拟机和真机的两个log日志

![img](http://szysky.com/2016/08/25/%E3%80%8AAndroid-%E5%BC%80%E5%8F%91%E8%89%BA%E6%9C%AF%E6%8E%A2%E7%B4%A2%E3%80%8B-13-%E7%BB%BC%E5%90%88%E6%8A%80%E6%9C%AF/uncaughtException_1.png)
![img](http://szysky.com/2016/08/25/%E3%80%8AAndroid-%E5%BC%80%E5%8F%91%E8%89%BA%E6%9C%AF%E6%8E%A2%E7%B4%A2%E3%80%8B-13-%E7%BB%BC%E5%90%88%E6%8A%80%E6%9C%AF/uncaughtException_2.png)

## multidex解决方法数越界

Android中单个**dex**文件所能包含的最大方法数为65536, 这包含了FrameWork, 依赖的jar包以及应用本身的代码中的所有方法. 会爆出:

`com.android.dex.DexIndexOverflowException: method ID not in[0, 0xffff] :65536`

可能在一些低版本的手机, 即使没有超过方法数的上限却还是出现错误

```
E/dalvikvm: Optimization failed
E/installd: dexopt failed on '/data/dalvik-cache/.....'
```

这个现象, 首先`dexpot`是一个程序, 应用在安装时, 系统会通过`dexopt`来优化dex文件, 在优化过程中dexopt采用一个固定大小的缓冲区来存储应用中所有方法消息, 这个缓冲区就是`linearAlloc`. `LinearAlloc`缓冲区在新版本的Android系统中大小为8MB或者16MB. 在Android 2.2和2.3中却只有5MB. 这是如果方法过多, 即使方法数没有超过`65535`也有可能会因为存储空间失败而无法安装.

**解决方案**

- **插件化**: 是一套重量级的技术方案, 通过将一个`dex`拆分成两个或者多个`dex`,可以在一定程度上解决方法数的越界问题. 但是还有兼容性问题需要考虑, 所以需要权衡是否需要使用这个方案.
- **multidex**: 这是Google在2014年提出的解决方案

= =. 略过

## Android的动态加载技术

动态加载也叫插件化. 当项目越来越大的时候, 可以通过插件化来减轻应用的内存和CPU占用. 还可以实现热插播, 即可以在不发布新版本的情况下更新某些模块.

插件化基本都要面临解决的基本问题:

1. 资源访问
2. Activity声明周期的管理
3. ClassLoader的管理

说明**宿主**和**插件**的概念, 宿主是指普通的apk, 而插件一般指经过处理的dex或者apk. 在主流的插件化框架中多采用经过处理的apk来作为插件, 处理方式往往和编译以及打包环节有关, 另外很多插件化框架都需要用到代理Activity的概念, 插件Activity的启动大多数是借助一个代理Activity来实现.

= =. 略过

## 反编译

= =. 略过