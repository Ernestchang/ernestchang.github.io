---
title: 《Android开发艺术探索》Chap1_Activity生命周期和启动模式
date: 
categories: 
- android 
- 《Android开发艺术探索》
tags:  
- android

---

注：此篇笔记只记录重难点，对于基础和详细内容请自行学习《Android开发艺术探索》

---

## 1.1 Activity的生命周期
### 1.1.1 典型情况下的生命周期
- onStart和onResume的区别是onStart可见，还没有出现在前台，无法和用户进行交互。onResume获取到焦点可以和用户交互。

- 当启动新Activity是透明主题时，旧Activity不会走onStop。

- Activity切换时，旧Activity的onPause会先执行，然后才会启动新的Activity。

### 1.1.2 异常情况下的生命周期

- Activity在异常情况下被回收时，onSaveInstanceState方法会被回调，回调时机是在onStop之前，当Activity被重新创建的时候，onRestoreInstanceState方法会被回调，时序在onStart之后；

- 当Activity异常情况下的情况下需要重新创建时，系统会默认我们保存当前Activity的视图结果，并在Activity重启后为我们恢复这些数据，比如文本框用户输入的数据，ListView的滚动位置等，具体可以查看相对应View的源码，查看onSaveInstanceState和onRestoreInstanceState方法。

- 关于保存和恢复View的层次结构，系统的**工作流程**：首先Activity会调用onSaveInstanceState去保存数据，然后Activity会**委托**Window去保存数据，接着Window会委托它上面的顶级容器去保存数据，顶层容器一般是DecorView（ViewGroup)；最后顶层容器再去一一通知它的子元素保存数据，这样整个数据的保存过程就完成了。
- 如果资源内存不足优先级低的Activity会被杀死，**优先级**从高到低：
- 前台Activity-正在和用户交互优先级最高；
- 可见但非前台的Activity-比如Activity中弹了一个对话框，导致Activty可见但位于后台；
- 后台Activity-已经被暂停的Activity，已经执行了onStop，优先级最低。

- 如果我们不想配置发生改变就Activity重新创建，可以使用Activity的**configChanges**属性；常用的有locale、orientation和keyboardHidden这三个选项；指定了configChanges后，Activity发生对应改变后，不会重启Activity，只会调用onConfigurationChanged方法。

## 1.2 Activity的启动模式
### 1.2.1 Activity的LaunchMode
**任务栈**是一个“后进先出”的栈结构，每次finish()处于前台的Activity就会出栈，直到栈为空为止，当栈中无任何Activity的时候，系统就会回收这个任务栈。

- **standard**：标准模式，系统默认模式，每次启动都会创建一个新的实例；在这种模式下，谁启动了这个Activity，这个Activity就在启动它的那个Activity所在的栈中。当我们使用ApplicationContext去启动standard模式的Activity就会报错，因为standard模式的Activity会默认进入启动它的Activity所属的任务栈中，而非Activity类型的Context并没有任务栈。解决的办法是为这个待启动的Activity指定FLAG_ACTIVITY_NEW_TASK标记位，这样启动的时候就会为它创建一个新的任务栈。

- **singleTop**: 栈顶复用模式。这种模式下，如果新的Activity已经位于栈顶，那么此Activity不会创建，同时他的onNewIntent方法会被回调，onCreate、onStart不会被调用。如果新的Activity的实例存在但不是位于栈顶，那么新的Activty依然会重新创建。

- **singleTask**: 栈内复用模式。这是一种单实例模式，这种模式下，Activity在一个栈中存在，那么多次启动该Activity都不会重新创建实例，和sinleTop一样，系统会回调其onNewIntent。如果启动的Activity没有所需要的任务栈，就会先创建任务栈再创建Activity。singleTask默认具有clearTop的效果，具有该模式的Activity会让其之上的Activity全部出栈。

- **singleInstance**: 单实例模式。这是一种加强的singleTask模式，除了具备singleTask的特性之外，具有该模式的Activity只能单独位于一个任务栈中；比如Activity A是singleInstance模式的，当A启动后，系统会为它创建一个新的任务栈，后续的启动均不会创建新的Activity，除非这个任务栈被系统销毁了。

- 参数TaskAffinity用于指定Activity栈，TaskAffinity属性经常和singleTask启动模式或allowTaskReparenting属性配对使用，其他情况没有意义。当应用A启动了应用B的某个Activity后，如果这个Activity的allowTaskReparenting属性为true的话，那么当应用B被启动后，此Activity会直接从应用A的任务栈转移到应用B的任务栈中。

- 两种方法都可以为Activity指定启动模式，但还是有区别。
- Activity的启动模式可以通过AndroidMenifest为其指定启动模式
```
<activity android:name=".MainActivity" android:launchMode="singleTask" />
```
- 还可以通过在Intent中设置标记位来为Activity指定启动模式
```
Intent intent = new Intent(this,MainActivity.class); 
intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
 startActivity(intent);
```
- 第二种方式的**优先级**高于第一种，如果都设置了只有第二种会生效。第一种方式无法直接为Activity设置FLAG_ACTIVITY_CLEAR_TOP标记，而第二种方式无法为Activity指定singleInstance模式。

- 通过adb shell dumpsys activity命令可以详细的了解当前任务栈情况。

### 1.2.2 Activity的Flags
- FLAG_ACTIVITY_NEW_TASK为Activity指定“singleTask”启动模式，其效果和xml中指定该启动模式相同。

- FLAG_ACTIVITY_SINGLE_TOP为Activity指定“singleTop”启动模式，其效果和xml中指定该启动模式相同。

- FLAG_ACTIVITY_CLEAR_TOP具有此标记位的Activity，当他启动时，在同一个任务栈中所有位于它上面的Activity都要出栈。这个模式一般与FLAG_ACTIVITY_NEW_TASK配合使用，singleTask启动模式默认具有此标记为的效果。

- FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS具有这个标记为的Activity不会出现在历史Activity的列表中，当某些情况不希望用户通过历史列表回到我们Activity的时候这边标记比较有用。等同于在XML中指定Activity的属性android:excludeFromRecents="true"。

## 1.3 IntentFilter的匹配规则

启动Activity分为两种，显式调用（明确地指定被启动对象的组件信息，包括包名和类名）和隐式调用（不需要明确指定组件信息，需要Intent能匹配上目标组件的IntentFilter中所设置的过滤信息）。IntentFilter的过滤信息有action、category、data。为了匹配过滤列表，需同时匹配过滤列表中的action、category、data信息，否则匹配失败；一个Activity中可以有多个intent-filter，一个Intent只要能匹配任何一组intent-filter即可成功启动对应的Activity。

- action匹配规则：要求intent中的action存在且必须和过滤规则中的其中一个相同，区分大小写。

- category匹配规则：系统会默认加上一个android.intent.category.DEAFAULT，所以intent中可以不存在category，但如果存在就必须匹配其中一个。同时为了我们的Activity能够支持隐式调用，就必须要在intent-filter中指定“android.intent.category.DEFAULT”这个category。

- data匹配规则：data由两部分组成，mimeType和URI，要求和action相似。如果没有指定URI，URI但默认值为content和file（schema）

- 如果要为Intent指定完整的data，必须调用setDataAndType方法，不能先调用setData再调用setType，因为这两个方法都会清除对方的值。

- 采用PackageManager的resolveActivity方法或Intent的resolveActivity方法，如果找不到匹配的Activity就会返回null，我们通过判断返回值就可以规避抛出android.content.ActivityNotFoundException异常。

- 在intent-filter中声明了<category android:name="android.intent.category.DEFAULT"/>这个category的Activity，才可以接收隐式意图。

- 有一类action和category的共同作用是标明这是一个入口Activity。
```
<action android:name="android.intent.action.MAIN" />
<category android:name="android.intent.category.LAUNCHER" />
```