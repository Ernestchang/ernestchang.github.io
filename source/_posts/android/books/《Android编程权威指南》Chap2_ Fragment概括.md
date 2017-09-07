title: 《Android编程权威指南》Chap2_ Fragment概括
date: 

categories: 
- android 
- 《Android编程权威指南》
tags:  
- android
---


> 这本书属于入门, 有很多内容可以当做扩展来了解一些API, 并且有的时候可以适当的利用Google提供好的API来做一些高效开发节约时间成本. 原理东西本书偏少. 可以学习本书中的代码的编写风格, 书中代码都是采用**MVC**模型来编写的. 就写这么多, 下面开始整理一些小知识点.
>
> 关于Fragment的知识点练习都保存在仓库中的**Criminal**项目中. [git地址](https://github.com/suzeyu1992/AndroidProgramminGuide)

## Fragment的生命周期

`fragment`是一种控制器对象, `activity`可委派其完成一些任务. 通常这些任务就是管理用户界面. 受管理的用户界面可以是一整屏或是整屏的一部分.

`fragment`的生命周期共**有11种**: `onAttach()` -> `onCreate()` -> `onCreateView()` -> `onActivityCreated()` -> `onStart()` -> `onResume()` -> `onPause()` -> `onStop()` -> `onDestroyView()` -> `onDestroy()` -> `onDetach()`

**Fragment的生命周期和Activity生命周期的最主要的区别就是, Activity的生命周期是由操作系统调用的, 而Fragment的生命周期是由托管的Activity调用的**

## Fragment的托管的方式

- 添加`<fragment>`标签到`activity`布局中. 并在`name`属性中指定一个fragment.
- 在`activity`代码中添加`fragment`.

方式1. 即**布局fragment**. 使用简单, 但是灵活性不够. 直接在xml中添加标签. 就等同于`fragment`及其视图于`activity`的视图绑定在了一起, 并且在activity的生命周期过程中无法切换`fragment`视图.

方式2. **事务管理动态添加**. 这种方式可以在运行时控制`fragment`. 根据需求将`fragment`在某一个时刻添加到activity中. 也可以移除`fragment`, 用其他的`fragment`来替代. 下面的**FragmentManager将稍微说明具体使用**

## Fragment#onCreate(Bundle)

这个方法看起来和`Activity#onCreate()`相似. 相似归相似说说不同之处.

- `Fragment#onCreate()`是公共方法,`Activity#onCreate()`是受保护的方法. 方法的**权限不同**, 因为`Fragment`可以被任何的`Activity`调用.
- `Activity#onCreate()`这个方法已经准备生成视图, 而`Fragment#onCreate()`却并没有要生成视图的动作. `Fragment`生成视图的方法是另一个生命周期`onCreateView()`方法. 这个方法会将生成的View返回给托管的`activity`.

------

相同之处, 同样像activity那样具有保存及获取状态的`Bundle`, 和`onSaveInstanceState()`

## FragmentManager及其事务

`FragmentManager`类负责管理`fragment`并将它们的视图添加到`activity`的视图层级结构中. 相当于连接这两个的桥梁.

`FragmentManager`具体管理的是:

- fragment队列.
- fragment事务的回退栈.

**FragmentManager的获取**

如果现在要动态添加一个`fragment`那么首先需要在`activity`中获取`fragmentManager`. 通过`FragmentActivity#getSupportFragmentManager()`.这里调用的是支持库的`FragmentActivity`. 目前官方`Android 24`中使用的默认是`AppCompatActivity`这个扩展兼容类. 其实这个类就是`FragmentActivity`的子类. 当然如果你就是使用`Activity`那么就调用`getFragmentManager()`一样可以.

**管理fragment的事务**

`FragmentManager`管理者拿到了, 就可以开启事务做一些我们想做的事情.

```
// 获得Fragment 的管理者对象
FragmentManager fm = getSupportFragmentManager();
// 先查找管理者中是否已经存在, 如果不存在就new一个
Fragment fragment = fm.findFragmentById(R.id.activity_main);

if (fragment == null) {
  // 方法内部实现要进行添加Fragment的创建并返回
  fragment = createFragment();  
  fm.beginTransaction()
          .add(R.id.activity_main, fragment)
          .commit();
}
```

首先利用了`Fragment#findFragmentById()`去**fragment队列**获取一个`fragment`. 如果有那么就不需要添加.如果没有重新进行事务添加. 为什么要这么做? (例如屏幕旋转, 虽然activity会被销毁重建, 但是`FragmentManager`会将`Fragment`队列保存下来. 这样当重启的时候就可以直接获取队列已经创建好的而不需要重复创建)

这段代码片段主要就是提交了一个**fragment事务**. **fragment事务**被用来添加,移除,附加,分离或替换**fragment队列**中的fragment. `add()`方法中首先指定activity的xml布局中的一个控件用来替换成被创建的`fragment视图`.然后参数2传入了要被添加的`fragment`

## ListFragment使用

是`Fragment`的子类, 并且内置了`ListView`提供给开发者快速创建列表的需求. 默认已经生成了一个全屏的`ListView`布局.

1. 继承ListFragment派生出一个子类. (这个时候如果什么数据也不添加运行程序,会发现中间会有一个progressBar在一直旋转)
2. 由于`ListView`已经存在. 所以按照我们平常的开发流程接下来就应该**创建适配器,并添加到ListView**. (这里使用快速的`ArrayAdapter`,也可以继承`BaseAdapter`来实现一些复杂的布局)

```
// 创建系统预定义的实例
ArrayAdapter<CrimeBean> myAdapter = new ArrayAdapter<>(getActivity().getApplicationContext(),
                    android.R.layout.simple_list_item_1,
                    mCrimes);

//  是ListFragment类提供的一个便利方法, 可以管理内置的listView设置adapter
setListAdapter(myAdapter);
```

创建一个`ArrayAdapter`传入三个参数

- 参数一: 上下文. 直接获取托管的`activity`的全局上下文即可
- 参数二: `ListView`中的`Item`条目要显示的布局. 这里使用了系统提供的布局. 这个布局里面就是一个TextView.
- 参数三: 数据集合`ArrayList`. 可以看到上面生成的`ArrayAdapter`的泛型是`CrimeBean`.这个泛型就是根据传入的数据集合中的泛型所指定. 如果**数据集合中添加的一个自定义类的实例,一定要复写类的toString()确定返回一个类的说明或者名称等. 不然如果数据集合填充的引用数据类型那么默认toString()会就会返回对象的地址值, 这对我们毫无用处**(其实内部就是`Adapter#getView()`方法内进行每个子Item的数据填充的时候获取集合中对应的position的数据然后`.toString()`获取到字符串填充到TextView上)

最后通过内部提供**绑定适配器的方法**添加即可`setListAdapter()`

现在可以正常显示数据了.

------

**添加点击事件**

在`ListFragment`中提供了点击item事件的方法. 直接覆盖`onListItemClick(...)`即可.

在`ListFragment`通过`getListAdater().getItem(int)`即可获得adapter中对应数据集合中对象

------

**如果需要自定义一个布局, 可以直接派生出一个ArrayAdapter的子类. 创建一个构造方法,和复写getView()方法即可, 不需要继承BaseAdapter要复写四个方法(这里看自己的实际需求来决定用那种适配器)**

```
/**
*  定制list的列表项, 继承adapter 重写getView方法
*/
private  class MyCrimeAdapter extends ArrayAdapter<CrimeBean>{

   public MyCrimeAdapter(ArrayList<CrimeBean> crimes) {
       //  由于这里不打算使用预定义的布局, 所以参数2传递了资源id为0,
       super(getActivity().getApplicationContext(), 0, crimes);
   }

   @NonNull
   @Override
   public View getView(int position, View convertView, ViewGroup parent) {
       ViewHolder holder ;
      
       //  这里省略布局实现的逻辑, 想实现什么就实现什么布局
       return convertView;
   }

}
```

## 使用fragment argument

每个`Fragment`实例都可附带一个`Bundle`对象. 附加`argument`给`Fragment`调用`Fragment.setArguments(Bundle)`即可. **需要注意,附加的时机必须在fragment创建之后并且添加到activity之前**

如果想要获取在托管的`activity`的`intent`中的值. 有两种方式.

- 当需要的时候就通过`getActivity().getIntent()`
- 利用`argument`在被托管前就给Fragment自己存储一些属性值.

还是第二种好一些. 我们可以在`Fragment`内部暴露一个方法如下:

```
public static CrimeFragment newInstance(UUID crimeId){
   Bundle bundle = new Bundle();
   bundle.putSerializable(EXTRA_CRIME_ID, crimeId);

   CrimeFragment crimeFragment = new CrimeFragment();
   crimeFragment.setArguments(bundle);
   return crimeFragment;
}
```

这样当`activity`需要托管添加的时候只需要调用我们提供的方法就可以. 具体的封装动作对activity来说是透明的. `activity`只需要获取到`intent`的值在调用`fragment#newInstance()`传入即可. 相对于第一种更好的将数据管理在自己的所在区域. 并且也不像第一种那样比较依赖`Activity`.

**获取argument**

`getArguments()`就可获得Bundle.

## fragment获取返回结果

在`Fragment`中同样可以使用`startActivityForResult()`和`onActivityResult()`.

虽然这两个方法和`Activity`很像, 但是如果你想设置`setResult()`那么请调用`getActivity().setResult()`.

## DialogFragment

是`Fragment`的子类, 和`ListFragment`一样内部封装了`AlertDialog`视图, 为按照Android开发原则, 统一利用`FragmentManager`来进行管理.

不同之处: 当设备旋转的时候独立配置的`AlertDialog`会在旋转之后消失, 而配置封装在`fragment`中的`AlertDialog`则不会消失.

这里实现目的是**弹窗显示日期控件**, 本身`AlertDialog`还有一个专门对日期操作的类`DatePickerDialog`.由于兼容性这里没有使用.

------

先总结一下实现对话框显示, 需要完成的任务:

1. 派生出一个`DialogFragment`的子类.
2. 创建`AlertDialog`
3. 通过`FragmentManager`在屏幕上显示对话框.

第一步略过直接创建类继承就可以. 第二步通过复写`onCreateDialog(Bundle)`方法. 只需要创建一个`AlertDialog`返回即可, 系统会帮助你显示.如下:

```
@NonNull
@Override
public Dialog onCreateDialog(Bundle savedInstanceState) {       
   // 直接返回在Activity中创建Dialog的方式生成的对象
   AlertDialog.Builder builder = new AlertDialog.Builder(getActivity())
           .setTitle("标题")
           .setPositiveButton(android.R.string.ok, new DialogInterface.OnClickListener() {
               @Override
               public void onClick(DialogInterface dialog, int which) {
                   sendResult(Activity.RESULT_OK);
               }
           })
           // 设置中间区域要显示的View , 可以实现自定义View传入
           .setView(inflate);

   AlertDialog alertDialog = builder.create();

   return alertDialog;

}
```

- `setTitle()` 设置标题栏的文字.
- `setView()` 实现在标题栏于按钮之间显示传入的View对象.
- `setPositiveButton()` 设置一个对话框的按钮(Android中有3种对话框按钮可使用:**positive按钮**, **negative按钮**,**neutral按钮**. )旧版本里面positive按钮出现在最左端, 新版本系统则出现则最右边.
- `AlertDialog.Builder.create()` 返回已配置完成的`AlertDialog`实例, 完成对话框创建.

第三部就是**显示对话框**了.

和其他的`fragment`一样, `DialogFragment`实例也是由托管activity的`FragmentManager`管理者的. 通过`DialogFragment`的以下两个方法都可以.

- `public void show(FragmentManager manager, String tag)`
- `public void show(FragmentTransaction transaction, String tag)`

`String`类型参数可唯一识别存放在`FragmentManager`队列中的`DialogFragment`. 可以按需选择使用`FragmentManager`还是`FragmentTransaction`. 如果传入`FragmentManager`参数, 则事务可自动创建并且提交. 代码例子如下:

```
FragmentManager fm = getActivity().getSupportFragmentManager();

DatePickerFragment dialog = new DatePickerFragment; //派生出来的DialogFragment的子类
                
dialog.show(fm, DIALOG_DATE);
```

## Fragment之间的数据传递

对于`activity`之间或者基于`fragment`的`activity`之间的数据传递. 已经了解. 那么如果是同一个`activity`托管的不同`fragment`之间的传递应该如何处理,比如上面如何从`DialogFragment`点击按钮把数据发送给另一个`fragment`?

大体流程分为两步:

1. 设置目标`Fragment`
2. 传递数据给目标`Fragment`
3. 在目标`Fragment`中对`onActivityResult()`进行数据接收处理

对应实现:

1. 通过`dialog.setTargetFragment(CrimeFragment.this, REQUEST_DATE);` 设置弹出的`AlertFragment`要返回数据的目标`Fragment`后, 目标**fragment和请求码**有`FragmentManager`负责跟踪记录, 在需要的时候通过调用**设置**目标`fragment`的`fragment`中的`getTargetFragment()`和`getTargetRequestCode()`方法来获取.
2. 设置信息都已经准备完毕, 现在只需要进行数据传递即可. 在要进行传递的起始`Fragment`中调用`getTargetFragment()`获取到目标(要传送的目标)`Fragment`. 用获取到的对象调用`onActivityResult()`即可, 参数分别是,请求码,结果码,和传递数据的`intent`
3. 就很简单了, 和`activity`一样接收数据并处理即可.

## Fragment的保留问题

**场景:**
还是设配配置发生改变的时候,会出现`Activity`的重建. 那么如果Activity里面托管一个`Fragment`. 而这个Fragment的作用是在播放音乐. 并且在`onDestroy()`的时候会释放掉`MediaPlayer`对象, 那么即使使用`onSaveInstanceState()`来保存意外销毁的进度, 但仍然会发生卡顿.

**解决:**
可在`Fragment#onCreate()`方法中设置一个属性`setRetainInstance(true)`

**方法说明 :**
`Fragment`中的`retainInstance`属性默认是为**false**. 这表明不会被保留. 因此设备旋转的时候. `fragment`会随着托管`activity`一起销毁并且重建. 如果设置了为**true**. 那么会进行保留, 已保留的`fragment`不会随着`activity`的异常重建而一起销毁. 而在`activity`重建之后再把自己原封不动的传递给新建后的activity. (和保留的fragment中的成员属性也是可以继续使用的.)

**保留fragment的工作原理:**
**可以总结为可销毁重建fragment的视图,但无需销毁fragment自身**. 当设备发生改变的时候, `FragmentManager`首先会销毁队列中的`fragment`的视图. 需要销毁重建的原因与`activity`重建的理由是完全一样的:**新的配置可能需要新的资源来匹配,去查找是否有更合适的匹配资源可以利用不,则就需要重新创建视图**. 当把视图都销毁掉以后, `FragmentManager`就会检查每个`Fragment的retainInstance`的属性值. 如果为false那么管理者就会销毁这个`fragment`的实例. 然后为使用新的设备配置, 新的`activity`的新的`FragmentManager`会创建一个新的`fragment`以及视图. 但是如果`retainInstance`的属性为`true`那么, `fragment`本身实例不会被销毁. 当新的`activity`创建后, 新的`FragmentManager`会找到被保留的fragment, 并重新创建它的视图.

有一个特殊的状态时发生`Fragment`的`onDestroyView()`声明周期之后的. 如果不被保留那么就会调用`onDestroy()和onDetach()`. 如果被保留那么就只会调用`onDetach()`. 此时此`fragment`没有任何的`activity`在托管它.

**需要注意Fragment处于保留状态的时间非常短暂, 就是fragment脱离了旧的activity到重新附加给立即创建的新的activity之间一段时间**

`fragment`必须同时满足两个条件才会进入保留状态:

- `Fragment`的`setRetainInstance`的属性为true.
- 因设备配置改变, 托管的activity正在销毁中.
