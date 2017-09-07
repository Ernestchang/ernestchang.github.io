title: 《Android编程权威指南》Chap1_ 零散整理
date: 

categories: 
- android 
- 《Android编程权威指南》
tags:  
- android
---

> 这本书属于入门, 有很多内容可以当做扩展来了解一些API, 并且有的时候可以适当的利用Google提供好的API来做一些高效开发节约时间成本. 原理东西本书偏少. 不过可以学习本书中的代码的编写风格, 书中代码都是采用**MVC**模型来编写的. 就写这么多, 下面开始整理一些小知识点.
> 注：本文主要参考[http://szysky.com](http://szysky.com)

## Activity的生命周期

有7种分别为: 除去`onRestart()`比较特殊一点的生命周期. 可以把`activity`认为分别在**3种状态**切换, 分别是`运行, 暂停, 停止`.

## 配置文件改变导致Activity重建,如屏幕旋转

**设备配置(device configuration)**是用来描述设备当前状态的一系列特征. 这些特征包括**屏幕的方向, 手机语言, 屏幕的密度, 屏幕的尺寸, 键盘类型, 底座模式等**

在开发中有时候需要为了匹配不同的设备配置, 我们会提供多套副本文件. 比如说:

- 图片放在不同的`dpi`密度的文件夹下;
- 适应手机的横竖屏创建不同的文件夹并编写不同展示风格的xml布局放入.
- 创建不同语言的`strings`文件. 来实现语言国际化
- …..

这里列举一下创建**水平模式下加载另一个布局文件**, 在`res/`资源文件夹下创建一个`layout-land`目录, 创建一个与`layout`文件夹中布局文件`名称相同`的xml. 然后编写想在横屏时候需要显示的布局.

其实当手机处于**横屏的时候**需要加载资源文件. 这个时候系统首先查找开发者是否创建对应横屏的文件夹就是`layout-land`,如果有. 那么继续查找资源ID所对应的文件是否存在. 如果存在那么就加载这个文件. 如果没有就从`layout`文件夹中加载默认的文件(layout文件夹优先级基本上是最低的)

## 异常重建时Activity的数据保存

由于本地手机的配置文件的改变, 如屏幕旋转造成了`Activity`的重建. 这个时候就会导致已经实例对象或者属性赋值全部丢失.

利用`onSaveInstanceState(Bundle)`一般会在`onPause()`之后被调用. 把要存储的数据添加到`onSaveInstanceState()`的参数类型为`Bundle`的参数中. 然后当销毁后的重建时, 会在`onCreate()`方法中会传回一个`Bundle`类型的对象. 可以取出. 如果不是异常销毁重建那么默认情况下`onCreate()`回调方法带回的`Bundle`类型的参数是为`null`的.

## SDK版本小解和兼容处理

- **SDK最低版本:** 当应用安装的时候, 手机会检测如果本身系统等级如果低于应用指定的最低版本那么应用无法安装.
- **SDK目标版本** 应用是设计给哪个API级别去运行的. 大多数情况下, 目标版本即最新发步的Android版本.
- **SDK编译版本** `Compile`. 前两个是通知手机设备的. 而这个属性是告知`Android Studio`编译器使用哪一个SDK版本进行代码编译.

------

有时候我们可能会使用高版本的API. 但是有可能手机的版本会低于这个API的出现版本. 这个时候就需要在代码中进行逻辑判断分支.

- `Build.VERSION.SDK_INT:`代表了当前运行的Android手机的版本号.

通过判断当前手机的版本号, 这样就可以针对不同的场景进行分支处理.

## XML布局属性

**样式,主题, 主题属性**

**样式(style)**是xml资源文件, 含有用来描述组件行为和外观的属性定义的集合. 例如在`res/valus/styles.xml`中声明如下样式.

```
<style name="BigTextStyle" >
   <item name="android:textSize">20sp</item>
   <item name="android:layout_margin">5dp</item>
</style>
```

在以后的所有大的文字控件`textView`,`button`.都可以直接引入一个`style`属性. 当重复的属性越多, 使用的控件也越多, 这种好处是不言而喻的.

------

**主题(theme)**

Android自带了一些供应使用的预定义平台主题. 例如:`android:theme="@style/Theme.AppCompat.Light"`.

引用主题属性:

```
<TextView
  android:layout_width="match_parent"
  android:layout_height="wrap_content"
  style="?android:listSeparatorTextViewStyle"/>
```

例如这样后, 你会发现`textView`会带有**margin**,**字体颜色**,**加粗**等一些属性. 这就在一些场合很方便的设置了一些属性. 并且也能达到我们看着不那么丑的需求.

------

**dp, sp**

- `dp (density-independent):`一般用于设置控件宽高和`margin`,`padding`. dp值代表的实际长度与手机的**屏幕分辨率**和**屏幕尺寸**没有直接关系,但是有间接关系. 是这样. 根据**屏幕的分辨率**和**屏幕尺寸**会得到一个**屏幕的像素密度**. 根据这个屏幕像素密度来决定1dp在当前设备代表着几个像素点.
- `sp (scale-independent pixel):`. 主要用于字体. 和dp相似会根据手机的密度来动态的选择对应的大小. 但是有一点不同. 这种像素会受**手机字体的偏好设置而改变**

------

**布局参数**

对于资源文件xml中标签的属性`android:layout_width`,`android:layout_margin`,`android:text`等等, 为什么有的属性前面有**layout**开头, 而有的没有呢.

其实名称**以layout_**来头的属性是作用于组件的**父组件**. 我们将这些属性统称为**布局参数**. 它们会告知父布局如何在内部安排自己的子元素.

而**不以layout_开头**的属性是作用于组件的. 组件生成时, 会调用某个方法按照属性即属性值进行自我配置.

------

**外边距于内边距**

根据上面的结论, 应该就可以看出`layout_margin`,`padding`这两个属性. 一个是作用父布局一个是作用在自己身上的. 所以在自定义**ViewGroup**的时候不仅要获取子孩子的宽高还要获取`margin`这样才能正确的布局. 而在自定义`View`的时候我们往往要兼顾到padding这个属性. 不然设置`padding`的属性就会无效果.

## ViewPager简单介绍

`ViewPager`在某种程度上有点类似于`AdapterView(ListView的超类)`. `AdapterView`需借助于Adapter才能提供视图. 所以同样的`ViewPager`也需要`PagerAdapter`的支持.

`ViewPager`可能开发中用的很多. 在使用`ViewPager`作为一个`fragment`容器的时候. `FragmentManager`要求其容器视图必须**具备id属性**. 所以如果是代码中动态的创建ViewPager.那么别忘了设置id属性, 如下:

```
protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        //  动态创建ViewPager并设置到setContent
        mViewPager = new ViewPager(getApplicationContext());
        // 如果需要对Fragment进行托管, 那么要求其容器视图必须具备资源ID
        mViewPager.setId(R.id.vp_main);
        setContentView(mViewPager);

}

// 在res/values/ids.xml 没有可以新建一个,为了创建独立资源id
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <item name="vp_main" type="id"/>
</resources>
```

这样就可以实现动态添加`ViewPager`并作为`Fragment`的容器了.

------

由于直接通过`PagerAdapter`派生出子类有很多场景步骤过于复杂. 我们可以根据具体作用场景来选择其子类来作为ViewPager的适配器.比如说`FragmentStatePagerAdapter`

实现很简单如下: 复写两个方法就可以

```
// 在fragment中的onCreate()中直接设置就可以
// 利用ViewPager的子类 FragmentStatesPagerAdapter可以简单的处理并是实现
// 当使用fragment 还有一个选择FragmentPagerAdapter 两者的区别是在卸载不再需要的fragment的时候有所不同
// FragmentPagerAdapter创建的fragment只是有销毁视图的动作, 没有将其从FragmentManager里面删除
FragmentManager fm = getSupportFragmentManager();
mViewPager.setAdapter(new FragmentStatePagerAdapter(fm) {
            @Override
            public Fragment getItem(int position) {
                //...相当于每个pager要展示的fragment.

                return crimeFragment;
            }

            @Override
            public int getCount() {
                // 返回Pager页的大小
                return mCrimes.size();
            }
        });
```

可以看到当创建了`FragmentStatePagerAdapter(..)`传入了一个`FragmentManager`给其构造方法. 因为实际上`FragmentStatePagerAdapter`是我们的代理, 负责管理与`ViewPager`的对话并协同工作.代理需首先将`getItem()`方法返回的`Fragment`添加到`Activity`. 然后才能使用fragment完成自己的工作. 这也是需要传递`fm`的原因.

**如果需要监听ViewPager的滑动事件**

给ViewPager设置`setOnPageChangeListener()`注册一个回调监听, 里面有三个状态回调:

1. `onPageScrollStateChange`: 告知当前页面所处的行为是什么.
2. `onPageScrolled`: 告知当前显示的页面是哪个, 并且现在已经滑动导致的偏移距离是多少.
3. `onPageSelected`: 告知页面滑向了哪里.可以得知是在数据集合中的第几个Pager被显示出来了

**关于FragmentStatePagerAdapter与FragmentPagerAdapter**

主要区别就是: **二者在卸载不再需要的fragment时,所采用的处理方式有所不同**

- `FragmentStatePagerAdapter`: 会销毁不需要的`fragment`. 事务提交后, 可将`fragment`从`activity#FragmentManager`中彻底移除. 而类名中的`state`表明: 在销毁`Fragment`时, 他会将其`onSaveInstanceState()`方法中的`bundle`信息保存下来. 当用户切换回来的时候, 保存实例状态可用于恢复生成新的`fragment`
- `FragmentPagerAdapter`: 对于不再需要的`fragment`, 其对应的`FragmentManager`则选择调用事务的`detach(Fragment)`方法. 而非`remove(Fragment)`方法. 也就是说,`FragmentPagerAdapter`只是销毁了fragment的视图, 但仍将`fragment`实例保留在`FragmentManager`中

而如何取决需要按场景来使用: `FragmentStatePagerAdapter`更节省内存. 如果需要展示的数据很多,如果使用`FragmentPageAdapter`那么会保存大量的信息, 并且占用很多空间就不是很合适.

所以如果用户界面只需要**少量固定**的fragment. 那么使用`FragmentPagerAdapter`比较合适. 否则就反之使用`FragmentStatePagerAdapter`.

## 资源文件的配置与匹配

先看一下以下项目结构目录截图:

![img](http://szysky.com/2016/09/07/%E3%80%8AAndroid%E7%BC%96%E7%A8%8B%E6%9D%83%E5%A8%81%E6%8C%87%E5%8D%97%E3%80%8B%E9%9A%8F%E8%AE%B0%E4%B8%80/local_1.png)

有三个`value`文件夹, 第一个为我们的默认加载的`value`, 第二个为使用了**横屏修饰符修饰声明**, 第三个使用了**指定语言修饰符声明**. 里面都有一个`strings`文件夹看一下内容. 这里只拿`app_name`属性来说明测试, 三个文件都有此属性.

![img](http://szysky.com/2016/09/07/%E3%80%8AAndroid%E7%BC%96%E7%A8%8B%E6%9D%83%E5%A8%81%E6%8C%87%E5%8D%97%E3%80%8B%E9%9A%8F%E8%AE%B0%E4%B8%80/local_2.png)

具有不同的修饰符的**同种功能的文件夹是具有不同优先级的**. 还是以上面的图片为例子. 如果说在首页`Activity`引用了`app_name`这个字符串. 那么根据不同的场景会去加载不同的资源文件.

- **如果本地语言指定了中文:** 那么就会加载`values-zh`文件中的资源(此文件夹的配置修饰符优先级在上面的三种场景最高).
- **如果本地语言非中文, 并且是横屏状态下:** 那么会加载`values-land`文件中的资源
- **如果即没有指定为中文,也不是横屏状态下, 那么就加载默认的资源文件** `values`文件夹

可以看出来app加载资源文件的时候是按照一定优先级去加载的, 首先会先排除掉不兼容当前设备配置的资源目录. 如果有多个文件匹配那么高优先级的文件夹首先会检查当前手机环境对应的配置是否匹配(比如是否横屏,是否设置某种本地语言). 匹配成功之后就会加载对应的文件夹下的内容. 而低优先级的对应功能的文件夹不会再去加载. 但是如果发现匹配不成功或者成功之后没有找到对应的属性那么还会继续往低优先级的查找. 直到找到为止.

所以可以看出一个属性值,最少要保证在资源文件中存在一份. 否则会出现加载错误, 并且需要处理好场景之间无值可选的问题.

下面列出大部分具有配置修饰符的设备特征. 他们的优先级从高到低排列的

| 优先级(数值越低越高) | 特征说明             |
| ----------- | ---------------- |
| 1           | 移动国家码, 通常附有移动网络码 |
| 2           | 语言代码, 通常附有地区代码   |
| 3           | 布局方向             |
| 4           | 最小宽度             |
| 5           | 可用宽度             |
| 6           | 可用高度             |
| 7           | 屏幕尺寸             |
| 8           | 屏幕纵横比            |
| 9           | 屏幕方位             |
| 10          | UI模式             |
| 11          | 夜间模式             |
| 12          | 屏幕显示密度           |
| 13          | 触摸屏类型            |
| 14          | 键盘可用性            |
| 15          | 首选输入法            |
| 16          | 导航键可用性           |
| 17          | 非文本导航方法          |
| 18          | API级别            |

资源文件夹也可以使用**多重配置修饰符**, 例如`values-zh-land`, 各修饰符必须按照优先级高低排序.否则无效. 例如`values-land-zh`就是无效的目录. 这里就不细说. 可在官网**guide/topics/resources/providing-resources**查看.

------

**资源命名**

资源的文件只能有小写字母组成并且不能包含空格.

无论是在XML中还是在代码中引入资源, 引用都不包括文件的扩展名. 所以**在同级目录下不能以文件的扩展名作为依据来区分命名相同的文件. 也不允许同级下名称相同扩展名不同的文件存在,在编译时候会报错**

**布局文件命名**

最好按文件名排序的声明文件名. 通常以其定义的视图类型名为前缀. 如`activity_`,`dialog_`,`list_item_`.这样查找分类就容易多了.

**资源目录结构**

所有的资源都应该保存在`res/`目录的子目录下, 常见的子目录有`drawable/`,`mipmap/`,`layout/`,`values/`,`raw/`, `menu/`等.

`Android`会无视`res/`目录下的其他子目录.

## 操作栏

也可以认为是title bar. 说法很多. 如下

![img](http://szysky.com/2016/09/07/%E3%80%8AAndroid%E7%BC%96%E7%A8%8B%E6%9D%83%E5%A8%81%E6%8C%87%E5%8D%97%E3%80%8B%E9%9A%8F%E8%AE%B0%E4%B8%80/titlebar.png)

可显示在操作栏上的菜单被称作**选项菜单**, 选项菜单类似于`popupWind`.可以作为用户的扩展操作.

**1.首先在XML文件中定义选项菜单**

`res/menu/`目录下创建一个xml. 根标签为`<menu>`如下

![img](http://szysky.com/2016/09/07/%E3%80%8AAndroid%E7%BC%96%E7%A8%8B%E6%9D%83%E5%A8%81%E6%8C%87%E5%8D%97%E3%80%8B%E9%9A%8F%E8%AE%B0%E4%B8%80/menu_1.png)

或者你可以这样

![img](http://szysky.com/2016/09/07/%E3%80%8AAndroid%E7%BC%96%E7%A8%8B%E6%9D%83%E5%A8%81%E6%8C%87%E5%8D%97%E3%80%8B%E9%9A%8F%E8%AE%B0%E4%B8%80/menu_2.png)

- `showAsAction`属性用于指定菜单选项是显示在操作栏上, 还是隐藏到**溢出菜单**中. 比如上图中的`app:showAsAction="ifRoom|withText"`是一个组合属性. 代表着只要操作栏可用的控件足够,菜单项图标和文字描述都会显示在操作栏中. 这个属性值不设定的情况下默认是**溢出菜单**模式显示的. 如果空间仅够显示图片那么文字不会显示, 如果剩余空间即不够显示文字也不够显示图标, 那么菜单项会被转移隐藏到菜单中.
- 上面这个属性值还有两个可设置项`always`和`never`. 不推荐使用`always`, 应尽量使用更方便的`ifRoom`属性值, 让操作系统系统决定如何显示菜单项. 对于那些很少用到的菜单项, 使用`never`是个不错的选择.
- `icon:` 属性使用了系统的图标资源. 这个资源只存在于设备上.

**2. 菜单文件创建完之后,就该代码设置了**

首先`Activity`提供了管理选项菜单的回调函数. 在需要选项菜单的时候, `Android`会调用其`onCreateOptionsMenu()`方法.

如果你使用的是`Fragment`那么也没关系因为`Fragment`也有同名的函数为你使用. 接下来以在`Fragment`为例说明.

复写`onCreateOptionsMenu()`函数. 通过`inflate`进行填充布局到`menu`对象中. 如下

```
@Override
public void onCreateContextMenu(ContextMenu menu, View v, ContextMenu.ContextMenuInfo menuInfo) {
   // 为操作栏创建浮动的上下文菜单
   mActivity.getMenuInflater().inflate(R.menu.crime_list_item_context, menu);
}
```

然后在`Fragment#onCreate()`方法中添加一段代码 `setHasOptionsMenu(true);`

**说明:** `Fragment#onCreateOptionsMenu()`方法是由`FragmentManager`负责调用的. 因此当`activity`接收到来自操作系统的`onCreateOptionsMenu()`方法回调请求时, 我们需要明确的告诉`FragmentManager`需接收选项菜单的方法回调.就是`setHasOptionsMenu()`.

现在我们的选项菜单就已经可以显示出来.

**3. 响应菜单选项选择的事件**

既然添加了扩展功能的选项, 那么必定需要实现某些点击事件. 通过复写`onOptionsItemSelected()`来进行处理. 有于我们可能在`menu.xml`中声明了多个item. 为了区分点击的是哪一个. 可以通过id来识别不同的点击触发. 如下

```
@Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()){
            case R.id.menu_item_new_crime:
                // 做一些想做的事情
                return true;

            case R.id.menu_item_show_subtitle:
                //  设置操作栏的子标题
                ActionBar supportActionBar = ((AppCompatActivity) getActivity()).getSupportActionBar();

                if (supportActionBar.getSubtitle() == null){
                    supportActionBar.setSubtitle(sub_description);
                    item.setTitle(R.string.hide_subtitle);
                }else{
                    supportActionBar.setSubtitle(null);
                    item.setTitle(R.string.show_subtitle);
                }

                return true;

            default:
                return super.onOptionsItemSelected(item);
        }
    }
```

你可能看到上面有一行注释写的设置**操作栏的子标题**这个的效果如下图.

![img](http://szysky.com/2016/09/07/%E3%80%8AAndroid%E7%BC%96%E7%A8%8B%E6%9D%83%E5%A8%81%E6%8C%87%E5%8D%97%E3%80%8B%E9%9A%8F%E8%AE%B0%E4%B8%80/subtitle.png)

**4. 层级式导航,向左后退箭头**

虽然大多数都是自定义的`titlebar`, 但是对于系统的默认后退也应该了解.

通过`Activity#getSupportActionBar().setDisplayHomeAsUpEnable()`. **(通过测试发现当在二级界面的时候默认都会出现向左的后退箭头但是默认的是没有处理的, 需要在onOptionsItemSelected()做对应的处理,我这里测试的环境为sdk为24,使用的AppCompatActivity派生的子类)**

所以如果是`fragment`开发为基础, 那么别忘了`setHasOptionsMenu(true)`, 允许`FragmentManager`对自己的`fragment`队列进行菜单选项的事件分发回调. 如下:

```
@Override
public boolean onOptionsItemSelected(MenuItem item) {
   switch (item.getItemId()){
       // android.R.id.home 就是默认的向左箭头组件的id
       case android.R.id.home:
           getActivity().finish();
           return true;

       default:
           return super.onOptionsItemSelected(item);
   }
}
```

**还有一个问题!**

Android编程, 有时候你以为已经丝入扣, 天衣无缝, 可以高枕无忧的时候上线, 总是会被人提醒到好像还有一个问题!

上面的二级标题是否还记得. 有这么一个场景如果二级标题显示了, 但是发生了设备旋转! ok 那么重建后你会发现二级标题不存在了. 这时就可以使用`setRetaionInstance(true)`来保存成员属性. 并利用一个布尔值来控制在创建菜单选项的时候二级标题是否需要显示.

## Android的文件系统与Java I/O

在操作系统级别, `Android`运行在`Linux`内核之中, 因此,它的文件系统类似于其他一些Linux或Unix系统. 目录名称已正斜杠`/`分隔. 文件名可由各类字符组成, 且区分大小写. 受益于`Linux`的安全模式, 应用都是以特定的用户ID运行的.

`Android`利用应用的java包名, 来决定应用在文件系统中的存放位置. 用于发布与安装, 应用本身被打包成APK文件格式. 被放置在`/data/app`目录. 当然这是需要进行`root`权限才可以的.

**访问文件和目录**

应用访问文件和目录最便捷的方式是使用`Content`类提供的方法. `Content`类是所有关键应用组件的超类.

下面是Content提供的基本文件或目录的便捷方法

| 方法                                       | 使用目的                                     |
| ---------------------------------------- | ---------------------------------------- |
| File getFilesDir()                       | 获取`data/data/<packname>/files`目录         |
| FileInputStream openFileInput(String name) | 打开`data/data/<packname>/files/`目录下的name文件名的文件获得输入流 |
| FileOutputStream openFileOutput(String name, int mode) | 同上, 这个是获得输出流, 不同之处如果没有这个文件那就会自动创建        |
| File getDir(String name, int mode)       | 获取`data/data/<packname>/name名称的文件夹`目录(没有就会先创建) |
| String[] fileList()                      | 获取`data/data/<packname>/files`目录下的所有文件名称列表. |
| File getCacheDir()                       | 获取`data/data/<packname>/cache`目录         |

## 系统的多选模式

如下效果:

![img](http://szysky.com/2016/09/07/%E3%80%8AAndroid%E7%BC%96%E7%A8%8B%E6%9D%83%E5%A8%81%E6%8C%87%E5%8D%97%E3%80%8B%E9%9A%8F%E8%AE%B0%E4%B8%80/multidex.gif)

开始实现步骤:

**1. 首先定义多选菜单的资源文件**

例如上面的垃圾桶图标需要手动去设置, 在`res/menu/`目录下创建一个xml

```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/menu_item_delete_crime"
        android:icon="@android:drawable/ic_menu_delete"
        android:title="@string/delete_crime"/>
</menu>
```

同样使用系统的资源图片, 就是右上角的垃圾桶.

**2. 实现Content菜单(进入编辑模式)**

**这里面有两种,在旧版本11之前上编辑模式是单项操作模式, 后续版本是多选模式, 下面简单说一下旧版本的模式然后在回到上面图中的多选模式, 可以直接跳过直接看多选模式, 毕竟11版本已经很旧了**

于之前的`操作栏里面的步骤大致相同`. 这里复写的方法是有两个(fragment类中)

- `onCreateContextMenu(...)`: 实例化一个Content多选菜单
- `onContextItemSelected(...)`: 响应菜单选择按钮

这里还是以在`fragment`中的代码举例.

```
@Override
public void onCreateContextMenu(ContextMenu menu, View v, ContextMenu.ContextMenuInfo menuInfo) {
   // 为操作栏创建浮动的上下文菜单
   mActivity.getMenuInflater().inflate(R.menu.crime_list_item_context, menu);
}
    
   // 浮动上下文的点击监听回调
@Override
public boolean onContextItemSelected(MenuItem item) {
   switch (item.getItemId()){
       //  显示删除的浮动上下文
       case R.id.menu_item_delete_crime:
           // 获得点击的具体listview的item位置, 并得到对象
           AdapterView.AdapterContextMenuInfo menuInfo = (AdapterView.AdapterContextMenuInfo) item.getMenuInfo();
           // 获得点击的item信息得到posion
           int position = menuInfo.position;
           CrimeBean clickItemBean = ((MyCrimeAdapter) getListAdapter()).getItem(position);

           //
           mCrimes.remove(clickItemBean);
           ((MyCrimeAdapter) getListAdapter()).notifyDataSetChanged();
           return true;
       default:
           return super.onContextItemSelected(item);

   }
}
```

最后一步为`ContextMenu`登记一个`ListView`如下

```
// 在低版本使用浮动上下文菜单
registerForContextMenu(lv_main);
```

------

**回到多选模式, 还是多选好**

接着第二大步开始. 如果要想实现多选模式的`ContextMenu`. 那么需要设置列表视图的选择模式为`ListView.CHOICE_MODE_MULTIPLE_MODAL`, 然后并添加个监听响应即可,其实很简单.

```
lv_main.setChoiceMode(ListView.CHOICE_MODE_MULTIPLE_MODAL);
//  设置列表视图操作模式回调
lv_main.setMultiChoiceModeListener(new AbsListView.MultiChoiceModeListener() {
    // 视图中的选项在选中或者撤销的时候会触发
    @Override
    public void onItemCheckedStateChanged(ActionMode mode, int position, long id, boolean checked) {
    }
    
    // 在ActionMode对象创建后调用, 也是实例化上下文菜单资源, 并显示在上下文操作栏上的任务完成的地方        
    @Override
    public boolean onCreateActionMode(ActionMode mode, Menu menu) {
    MenuInflater menuInflater = mode.getMenuInflater();
    menuInflater.inflate(R.menu.crime_list_item_context, menu);
    return true;
    }
    
    // 在onCreateActionMode()回调之后调用, 以及当前上下文操作栏需要刷新显示新数据时调用
    @Override
    public boolean onPrepareActionMode(ActionMode mode, Menu menu) {
    return false;
    }

    // 在用户选中某个菜单项操作时调用(这里就是垃圾桶图标按钮). 是相应上下文菜单项操作的地方
    @Override
    public boolean onActionItemClicked(ActionMode mode, MenuItem item) {
    switch (item.getItemId()){
        case R.id.menu_item_delete_crime:
            for (int i = myCrimeAdapter.getCount()-1; i >= 0; i--) {
                if (getListView().isItemChecked(i)){
                    mCrimes.remove(myCrimeAdapter.getItem(i));
                }
            }
            mode.finish();
            myCrimeAdapter.notifyDataSetChanged();
            return true;
    }
    return false;
    }

    // 在用户退出上下文操作模式或所选菜单项操作已被响应, 从而导致ActionMode对象将要销毁时调用.
    // 默认的实现会导致已选视图被反选, 也可以完成上下文操作模式下, 响应菜单项操作而引起的响应fragment刷新
    @Override
    public void onDestroyActionMode(ActionMode mode) {
    }
});
```

**3. 最后一步**

这个最后一步是针对**多选模式下的**, 上面两步已经在逻辑功能上完成了之前的需求, 但是有一点, 当进入多选的时候点击或不点击发现没有区别, 我们无法得知哪一个**item**是否已经被点击激活状态, 哪一个没有没点击激活. **所有我们需要给Item添加一个背景, 设置一下item的激活或者非激活的背景色**, 代码如下:

```
// 在drawable/文件夹下创建一个文件
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <!--当引用控件处于激活状态时, 则使用指定的资源文件, 反之不采取任何操作-->
    <item android:state_activated="true"
        android:drawable="@android:color/darker_gray"/>
</selector>
```

把这个`StateListDrawable`状态`drawable`设置到`ListView的item背景属性上`.

ok 关于多选的`ContentMenu`操作就实现完成.

**扩展一下:**

这里实现的方法可以完全作用在`ListView`和`GridView`中(GridView和ListView都是AdapterView的子类). 但是如果不是这两个类型的控件应该如何实现这种多选方式.

1. 首先给一个控件实现**长按监听**
2. 在长按回调中调用`Activity.startActivityMode(..)`并创建一个`ActionMode.Callback`的接口作为参数传入. 这个接口里面的回调很熟悉就是上面`ListView`设置了`setMultiChoiceModeListener`里面创建的匿名类里面对应的回调.

## 设备屏幕尺寸的确定

你可能会发现你的项目目录中存在如下这个文件, 但具体什么意思却不是很清楚

![img](http://szysky.com/2016/09/07/%E3%80%8AAndroid%E7%BC%96%E7%A8%8B%E6%9D%83%E5%A8%81%E6%8C%87%E5%8D%97%E3%80%8B%E9%9A%8F%E8%AE%B0%E4%B8%80/screen_1.png)

这是在`Android 3.2之后引入的新修饰符`, 在3.2之前是使用`small, normal, large, xlarge`来讲设备分为不同的屏幕大小类别.

现在新引入的屏幕尺寸修饰符如下表:

| 修饰符格式   | 描述                                  |
| ------- | ----------------------------------- |
| wXXXdp  | 有效宽度: 宽度大于或等于 XXX dp                |
| hXXXdp  | 有效高度: 高度大于或等于 XXX dp                |
| swXXXdp | 最小宽度: 宽度或者高度(两者中最小的那个) 大于或等于 xxx dp |

如果要指定某一个布局仅适用于屏幕宽度至少**300dp**的设备, 这种情况下, 可以使用宽度修饰符, 并将布局文件放入`res/layout-w300dp`目录下, (w代表宽度width). 同样道理如果是高度那么使用`hxxxdp`

有时候设备方向变换, 也会导致设备的宽高发生变化. 为了确定某个具体的屏幕尺寸可是使用`sw`, 如果不管是横屏`1280*720`还是竖屏的`720*1280`, 其对应的sw的值都是800.

## 以命令行的方式运行activity

要用`adb`命令启动首先需要满足`activity`组件可以被外部启动. 需要在清单文件中配置要被启动的`activity`的属性如下

```
<activity android:name=".activity.CrimeCameraActivity"
            android:exported="true"/>
```

设置`exported`属性. 这个属性默认是**false**. 表示只能从自己的应用中启动. **如果将intent过滤器添加到activity的声明中, 那么该activity的exported将被自动设置为true**

然后使用`adb`命令, 如果你没有设置全局环境那么就到`Android SDK`安装目录`platform-tools`子目录下执行adb命令.

`adb shell am start -n 应用包名/.activity类名`

/ 后面接的和清单文件中`activity`的name属性对应的是一样的. 如果你如上代码在多层包中. 那么实际上是这样的. 我的包名`com.szysky.note.criminal`

演示代码:

`adb shell am start -n com.szysky.note.criminal/.activity.CrimeCameraActivity`

`am(Activity Manager)`是一个在设备上运行的命令行程序. 它支持启动和停止Android组件(component)并从命令行发送intent. 可以运行adb shell am指令. 查看am工具能够完成的所有任务.