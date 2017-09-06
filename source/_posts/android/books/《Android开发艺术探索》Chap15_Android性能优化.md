title: 《Android开发艺术探索》Chap15_Android性能优化
date: 

categories: 
- android 
- 《Android开发艺术探索》
tags:  
- android
---

> 通过一些常见的性能优化方法, 这将有助于提高Android程序的性能, 于一些性能能分析等.
> 注：本文主要参考[http://szysky.com](http://szysky.com)


优化主要几个方面:

- 布局优化
- 绘制优化
- 内存泄漏优化
- 相应速度优化
- ListView, Bitmap, 线程优化

## Android性能优化的方法

### 布局优化

> 布局优化的思想就是**尽量较少布局文件的层级,这就可以让Android绘制时的工作量减少**

删除无用的控件和层级, 有选择地使用`ViewGroup`. 例如`RelativeLayout`和`LinearLayout`. 都可以的话那么就采用`LinearLayout`. 因为`RelativeLayout`的功能比较复杂, 它的布局过程需要花费更多的CPU时间. `FrameLayout`和`LinearLayout`都是一种简单高效的`ViewGroup`. **如果需要嵌套才可以实现的布局那么就是用RelativeLayout**.

布局优化的另外一个方法就是采用`<include>`标签, `<merge>`标签和`ViewStub`.

- `<include>`: 主要用于布局的重用
- `<merge>`: 一般和include标签配合使用, 它可以减少布局的层级
- `ViewStub`: 提供了按需加载的功能, 当需要时才会将ViewStub中的布局加载到内存,这可以提高程序的初始化.

**<include>标签**

可以将一个指定的布局文件加载到当前布局文件中:

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal">

    <include
        android:id="@+id/hah"
        android:layout="@layout/layout_titlebar"
        android:layout_height="match_parent"
        android:layout_width="match_parent"
        android:visibility="invisible"/>

</LinearLayout>
```

这个标签里面支持的属性很少, 根据编辑器的提示只有5个, 而且`width和height`如果要出现需要同时出现, 也可以不写, 最重要的就是一个必须指定导入的布局`layout="xxxxxx"`.

**<merge>标签**

这个标签一般和`include`标签一起使用从而减少布局的层级. 有时候会有这样一个场景, 如果`include`导入的布局的根布局是竖直方向的, 而当前布局也是竖直方向的, 那么和显然有一个层级是多余的. 这个时候使用`<merge>`就可以去掉重复布局.

**ViewStub**

`ViewStub`继承了`View`, 这是一个非常轻量级的且宽高都是0, 因此它本身不参与任何的布局和绘制过程. 而`ViewStub`存在的意义在于按需加载所需的布局文件, 在实际开发中, 有很多布局文件在正常情况系不会显示, 如网络异常等. 这个时候就没有必要再整个界面初始化的时候将其加载.

首先布局中添加`<ViewStub>`

```
<ViewStub
   android:id="@+id/stub_import"
   android:inflatedId="@+id/stin_root"
   android:layout="@layout/layout_stubview"
   android:layout_width="match_parent"
   android:layout_height="60dp"/>
```

这里`android:layout`属性还是导入外部布局的意思. `inflatedId`这个添加的id的属性是给导入进来的布局`layout_stubview`的根布局设定了一个id值.

然后在代码中有两种方式让其显示

```
// 方式一 通过设置visibility
((ViewStub)findViewById(R.id.stub_import)).setVisibility(View.VISIBLE);
 // 方式二 通过inflate加载显示
 //View inflate =  ((ViewStub) findViewById(R.id.stub_import)).inflate();

 // 通过inflatedId这个id可以得到加载进来的布局的根布局
 LinearLayout commLv = (LinearLayout) findViewById(R.id.stin_root);
```

### 绘制优化

绘制优化是指`View#onDraw()`方法要避免执行大量的操作.两个方面

1. onDraw中不要创建新的局部对象, 因为`onDraw()`方法可能会被频繁调用, 这样就会在一瞬间产生大量的临时对象, 这不仅占用了过多的内存而且还会导致系统频繁的gc, 降低了程序的执行效率.
2. 不要做耗时任务, 也不能执行成千上万次的循环操作. 即使每次循环都很轻量级, 但是大量的循环仍然十分抢占CPU的时间片, 这会造成View的绘制流程不流畅. 按照官方的规范, View的绘制帧率保证60fps最佳. 也就是16ms的为每一阵帧的绘制时间.

### 内存泄漏优化

**情况1. 静态变量导致的内存泄漏**

```
private static View sView;

@Override
protected void onCreate(Bundle savedInstanceState) {
   super.onCreate(savedInstanceState);
   setContentView(R.layout.activity_main);

   sView = new View(this);
}
```

如非必须传递Activity的引用不要这么做, 如果需要上下文可以传递`getApplicationContext()`返回的上下文

------

**情况2. 单例模式导致的内存泄漏**

常见的就是在使用注册监听的时候, 往往会往一个单例类中传入`this`本类对象,进行注册, 然后却没有解注册的动作. 那么这个Activity被引用的时间也就是和`Application`的生命周期持平.

------

**情况3. 属性动画导致的内存泄漏**

在Android 3.0中加入了属性动画, 属性动画有一类**无限循环**的动画, 如果在Activity中播放此类动画且没有在Activity退出的时候没有停止动画. 尽管无法界面上看到效果, 但是创建这个动画所关联的`View`被动画所持有, 而`View`又持有了`Activity`, 最终Activity无法释放. **解决方案, 就是在onDestroy()中调用动画的cancel()来停止动画.**

### 响应速度和ANR日志分析

响应速度的优化核心就是避免主线程做耗时操作, 响应速度过慢更多体现在`Activity`启动的速度上. 如果主线程内做太多的事情, 会导致Activity启动时出现黑屏现象, 甚至出现ANR.

Android中规定如果`Activity`5秒钟之内无法响应屏幕事件或者键盘输入事件就会出现**ANR**. 而`BroadCastReceiver`如果10秒之内还未执行完操作也会出现**ANR**.

如果进程发生了ANR以后, 系统会在`data/anr`目录下创建一个文件`traces.txt`. 通过分析这个文件就定位出原因.(这个文件很长如果需要分析, 请先删除文件生成一个在分析来进行了解)

通过一个例子来了解如何去分析文件, 首先在`onCreate()`添加如下代码, 让主线程等待一个锁,然后点击返回5秒后会出现ANR, 贴代码

```
@Override
protected void onCreate(Bundle savedInstanceState) {
   super.onCreate(savedInstanceState);
   setContentView(R.layout.activity_main);
   // 以下代码是为了模拟一个ANR的场景来分析日志
   new Thread(new Runnable() {
       @Override
       public void run() {
           testANR();
       }
   }).start();
   SystemClock.sleep(10);
   initView();
}


/**
*  以下两个方法用来模拟出一个稍微不好发现的ANR
*/
private synchronized void testANR(){
   SystemClock.sleep(3000 * 1000);
}

private synchronized void initView(){}
```

这样会出现ANR, 然后导出`/data/anr/straces.txt`文件. 因为内容比较多只贴出关键部分

```
DALVIK THREADS (15):
"main" prio=5 tid=1 Blocked
  | group="main" sCount=1 dsCount=0 obj=0x73db0970 self=0xf4306800
  | sysTid=19949 nice=0 cgrp=apps sched=0/0 handle=0xf778d160
  | state=S schedstat=( 151056979 25055334 199 ) utm=5 stm=9 core=1 HZ=100
  | stack=0xff5b2000-0xff5b4000 stackSize=8MB
  | held mutexes=
  at com.szysky.note.androiddevseek_15.MainActivity.initView(MainActivity.java:0)
  - waiting to lock <0x2fbcb3de> (a com.szysky.note.androiddevseek_15.MainActivity) 
  - held by thread 15
  at com.szysky.note.androiddevseek_15.MainActivity.onCreate(MainActivity.java:42)
```

这段可以看出最后指明了`ANR`发生的位置在`ManiActivity的42行`. 并且通过上面看出`initView`方法正在`等待一个锁<0x2fbcb3de>`锁的类型是一个`MainActivity`对象. 并且这个锁已经被线程id为15(tid=15)的线程持有了. 接下来找一下线程15

```
"Thread-404" prio=5 tid=15 Sleeping
  | group="main" sCount=1 dsCount=0 obj=0x12c00f80 self=0xeb95bc00
  | sysTid=19985 nice=0 cgrp=apps sched=0/0 handle=0xef34be80
  | state=S schedstat=( 391248 0 1 ) utm=0 stm=0 core=2 HZ=100
  | stack=0xe2bfe000-0xe2c00000 stackSize=1036KB
  | held mutexes=
  at java.lang.Thread.sleep!(Native method)
  - sleeping on <0x2e3896a7> (a java.lang.Object)
  at java.lang.Thread.sleep(Thread.java:1031)
  - locked <0x2e3896a7> (a java.lang.Object)
  at java.lang.Thread.sleep(Thread.java:985)
  at android.os.SystemClock.sleep(SystemClock.java:120)
  at com.szysky.note.androiddevseek_15.MainActivity.testANR(MainActivity.java:50)
  - locked <0x2fbcb3de> (a com.szysky.note.androiddevseek_15.MainActivity)
```

tid = 15 就是相关信息如上, 首行已经标出线程的状态为`Sleeping`, 原因在**50行**, 就是`SystemClock.sleep(3000 * 1000);`这句话. 也就是`testANR()`. 而最后一行也表明了持有的`locked<0x2fbcb3de>`就是主线程在等待的那个锁对象.

### ListView和Bitmap优化

- `ListView`: 在前面已经说过了, 主要三个方面: 采用`ViewHolder`避免在getView中执行耗时操作; 其次要根据列表的滑动状态来控制任务的执行频率; 最后可以尝试开启硬件加速是`ListView`滑动更加流畅. `ListView`的优化策略也完全适用于`GridView`
- `Bitmap`: 也已经说过, 主要是通过`BitmapFactory.Options`根据需要对图片进行采样, 采样率的设置通过`inSampleSize`属性.

### 线程优化

主要思想就是采用线程池, 避免程序中存在大量的`Thread`. 线程池可以重用内部的线程, 避免了线程创建和销毁的性能开销. 同时线程池还能有效的控制线程的最大并发数, 避免了大量线程因互相巷战系统资源从而导致阻塞现象的发生.

### 额外的性能优化建议

- 避免创建过多的对象
- 不要过多使用枚举, 枚举占用的内存空间比整形还要大,使用Android官方提供的方法可参考另一篇博客[链接跳转](http://szysky.com/2016/05/20/Android-%E4%B8%AD%E5%A6%82%E4%BD%95%E4%BD%BF%E7%94%A8annotion%E6%9B%BF%E4%BB%A3Enum/)
- 常量请使用 static final 来修饰
- 使用一些Android特有的数据结构, 比如`SparseArray`和`Pair`等
- 适当的使用软引用和弱引用
- 采用内存缓存和磁盘缓存
- 尽量采用静态内部类, 避免潜在的由于内部类而导致的内存泄漏

### 内存泄漏分析工具MAT

MAT全程Eclipse Memory Analyzer, 是一个内存泄漏分析工具. 下载后解压即可. 下载地址`http://www.eclipse.org/mat/downloads.php`. 这里仅简单说一下. 这个我没有手动去实践, 就当个记录, 因为现在`Android Studio`可以直接分析`hprof`文件.

可以手动写一个会造成内存泄漏的代码, 然后打开`DDMS`, 然后选中要分析的进程, 然后单击`Dump HPROF file`这个按钮. 等一小段会生成一个文件. 这个文件不能被`MAT`直接识别. 需要使用`Android SDK`中的工具进行格式转换一下.这个工具在`platform-conv`文件夹下

`hprof-conv 要转换的文件名 输出的文件名` 文件名的签名有包名.

然后打开`MAT`通过菜单打开转换后的这个文件. 这里常用的就有两个

- `Histogram`: 可以直观的看出内存中不同类型的buffer的数量和占用内存大小
- `Dominator Tree`: 把内存中的对象按照从大到小的顺序进行排序, 并且可以分析对象之间的引用关系, 内存泄漏分析就是通过这个完成的.

------

分析内存泄漏的时候需要分析`Dominator Tree`里面的内存信息, 一般会不直接显示出来, 可以按照从大到小的顺序去排查一遍. 如果发生了了泄漏, 那么在泄漏对象处右键单击`Path To GC Roots->exclude wake/soft references`. 可以看到最终是什么对象导致的无法释放. 刚才的操作之所以排除软引用和弱引用是因为,大部分情况下这两种类型都可以被gc回收掉,所以基本也就不会造成内存泄漏.

同样这里也可以使用搜索功能, 假如我们手动模拟了内存泄漏, 泄漏的对象就是`Activity`那么我们`back`退出重进循环几次, 会发现其实很多个`Activit`对象.

更多的东西我也不会,作者也没有说.. 不过这些以后Android Studio都会很有好用对应功能.

## 提高程序的可维护性

这里主要说`Android`的程序设计思想. 主旨是如何提高代码的可维护性和可扩展性, 而程序的可维护性也包含可扩展性. 这里的切入点为: **代码风格**, **代码的层次性和单一职责原则**, **面向扩展编程以及设计模式**

可读性是代码可维护性的前提, 一段只能让机器读懂的代码即使可以跑也属于”坏味道的代码”, 而良好的代码风格在一定程度上可以提高从程序的可读性. 代码的风格有 命名规范, 代码排版, 注释说明.

1. 命名要规范, 正确传达出变量或者方法的定义, 少用缩写除非业界通用的缩写如`String->str`.能让人一眼明白的. 私有成员要以`m`开头. 静态成员要以`s`开头. 常量要全部大写.
2. 代码排版上留出合理的空白来区分不同的代码块, 其中同类变量的声明放在一组, 两类变量之间留出一行作为空白.
3. 仅为非常关键的代码添加注释, 其他地方不写注释, 这就对变量和方法的命名风格提出了很高的要求. 一个合理的命名风格可以让读者阅读源码的时候就如阅读注释一样. 因此根本不需要为代码额外写注释

------

代码的层次是指代码要有分层的概念, 对于一段业务逻辑, 不要试图在一个方法或者一个类中去全部实现, 而是将其分成几个子逻辑, 然后每个逻辑做自己的事情, 这样即显得代码层次分明, 又可以分解任务从而实现简单逻辑的效果.

单一职责是和层次性相关联的. 代码分层以后, 每一层仅仅关注少量的逻辑, 这样就做到了单一职责.

程序的扩展性, 由于很多时候在开发过程中无法保证已经做好的需求不在后面的版本发生更改, 因此在写程序的时候要时刻考虑到扩展的问题, 考虑如果这个逻辑以后发生了改变那么哪些需要修改, 以及怎样在以后修改的时候降低工作量, 而面向扩展编程可以让程序具有很好的扩展性.

适当使用设计模式可以提高代码的可维护性和可扩展性. 但是一定控制设计的度, 千万别过度设计.

------

> 这本书终于抄完了, 看了两遍. 全部理解透彻是没达到. 不过这本书挺不错的. 以后针对一些不是很理解,并且在Android中是一些贯穿始终的东西必须要认真琢磨. good night. 感谢任玉刚.