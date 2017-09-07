title: 《Android编程权威指南》Chap3_ 媒体与Intent
date: 

categories: 
- android 
- 《Android编程权威指南》
tags:  
- android
---

> 这本书属于入门, 有很多内容可以当做扩展来了解一些API, 并且有的时候可以适当的利用Google提供好的API来做一些高效开发节约时间成本. 原理东西本书偏少. 可以学习本书中的代码的编写风格, 书中代码都是采用**MVC**模型来编写的. 就写这么多, 下面开始整理一些小知识点.
> 注：本文主要参考[http://szysky.com](http://szysky.com)

> 关于Camera和SurfaceView的知识点练习都保存在仓库中的**Criminal**项目中.
> [git地址](https://github.com/suzeyu1992/AndroidProgramminGuide)

## MediaPlay播放音频

> `MediaPlayer`是一个支持音频及视频文件播放的Android类. 可播放不同的来源(本地或网络流媒体).多种格式(WAV, MP3, MPEG-4, 3GPP等)的多媒体文件.

直接贴出代码

```
/**
 * Author :  suzeyu
 * Time   :  2016-09-07  上午10:17
 * Blog   :  http://szysky.com
 * GitHub :  https://github.com/suzeyu1992
 *
 * ClassDescription : 主要用来管理MediaPlayer的实例, 和对该实例的播放和停止等功能
 */

public class AudioPlayer {

    private MediaPlayer mPlayer;

    public void stop(){
        if (mPlayer != null){
            // 销毁, 否则MediaPlayer将一直占用着音频解码硬件及其他系统资源
            mPlayer.release();
            mPlayer = null;
        }
    }

    public void play(Context context){
        // 防止过多的创建MediaPlayer实例, 第一步先销毁已经存在的
        stop();

        //  设置要播放的音频文件, 如果音频来自其他渠道如网络或者URI, 则使用其他的create(...)函数
        mPlayer = MediaPlayer.create(context, R.raw.ss_small);
        mPlayer.start();

        // 防止过多的创建MediaPlayer实例, 第二步设置监听, 存活时间为音频的时长
        mPlayer.setOnCompletionListener(new MediaPlayer.OnCompletionListener() {
            @Override
            public void onCompletion(MediaPlayer mp) {
                // 音频播放完成
                stop();
            }
        });
    }
}
```

记得在`res/raw/`放置要播放的文件. 一个简便的播放就是这样.详细的查看官网的对于`MediaPlayer`的介绍

## 播放视频

关于播放视频, `Android`提供了多种实现方式. 其一便是使用上面说到的`MediaPlayer`, 而我们需要做的只是设置在哪里播放即可.

在`Android`系统中, 快速刷新显示的可视图像(如视频)是在`SurfaceView`中显示的. 准确的说, 是在`SurfaceView`内嵌的`Surface`中显示的. 通过获取`SurfaceView的SurfaceHolder`可是实现在`Surface`上显示视频. 简单的说就是通过`MediaPlayer.setDisplay(SurfaceHolder)`方法, 将`MediaPlayer`类于`SurfaceHolder`关联起来即可.

通常来说直接使用`VideoView`实例来播放视频会更简单些, 不同于`SurfaceView于MediaPlayer`之间的交互, `VideoView是与MediaController`交互的, 这样可以方便地提供视频播放界面. 而`VideoView`是不接受*ID资源的*. 而只接受文件路径或者URI对象.

创建一个指向**Android资源**的URI, 可使用如下代码:

`Uri resourceUri = Uri.parse("android.resource://包名/raw/文件名称");`

使用了`android.resource`格式, 用包名作为主机名, 资源文件类型与文件名称组成了一个路径用以创建URI.完成后就可以将其传给`VideoView`使用.

## Camera与SurfaceView

- `Camera:`提供了对设备相机硬件级别的调用. 相机是一种**独占性资源**: 一次只能有一个activity调用(如果你的应用没有释放相机资源,那么系统的相机也就无法调起使用)
- `SurfaceView:`这是一种特殊的视图, 可直接将要显示的内容渲染输出到设备的屏幕上.

首先既然使用了相机那么就需要添加相机的使用权限

```
<!--增加相机使用权限-->
<uses-permission android:name="android.permission.CAMERA"/>
<!--指定应用使用的某项特色功能, 这个属性可以保证那些配备相机功能的设备才能看到发布在GooglePlay上的此应用-->
<uses-feature android:name="android.hardware.camera"/>
```

`uses-feature`对我们貌似没什么用处.

既然是拍照那么很多都是横屏的,所以清单文件中把所在的`<activity>`设置横屏模式

```
<activity android:name=".activity.CrimeCameraActivity"
            android:screenOrientation="landscape"/>
```

------

**开始了解相机使用**

- `public static Camera open(int)`
- `public static Camera release()`

这两个方法是管理`Camera`的方法. 最好用户可以与界面进行交互分别调用. 如`onResume()`和`onPause()`

```
@Override
public void onResume() {
  super.onResume();
  // 0为打开后置摄像头, 如果没有后置摄像头那么就打开前置摄像头
  mCamera = Camera.open(0);
}
    
@Override
public void onPause() {
  super.onPause();
  if(mCamera != null){
      mCamera.release();
      mCamera = null;
  }
}
```

**相机打开了现在轮到Surface,因为Camera的拍照需要它**

`SurfaceView`类实现了`SurfaceHolder`接口. 首先我们先获取`SurfaceHolder`实例.(在`Activity#onCreate`或者`Fragment#onCreateView()`只要能保证布局已经加载就可以)

```
/**
*  对SurfaceView控件进行一些初始化和绑定客户端Camera
*/
private void initSurfaceView(View rootView) {
   mV_sfv_camera = (SurfaceView) rootView.findViewById(R.id.sfv_camera_display);
   SurfaceHolder holder = mV_sfv_camera.getHolder();
   // setType设置只为兼容旧版本3.0之前,可以选择考虑是否需要
   holder.setType(SurfaceHolder.SURFACE_TYPE_PUSH_BUFFERS);

   //  对SurfaceView的声明周期进行关联
   holder.addCallback(new SurfaceHolder.Callback() {

       /** SurfaceView的视图层级结构被放在屏幕上时候被调用, 这里也是SurfaceView与客户端(Camera)进行关联的地方*/
       @Override
       public void surfaceCreated(SurfaceHolder holder) {
           try {
               if (mCamera != null) {
                   mCamera.setPreviewDisplay(holder);
               }
           } catch (IOException e) {
               Log.e(TAG, "@@-> Camera设置关联SurfaceView预览显示失败!" );
           }
       }

       /**  Surface首次显示在屏幕上的时候被动调用的方法, 通过此参数可以知道Surface的像素格式以及他的宽高.
        通过此方法可以通知Surface客户端, 有多大的绘制区域可以使用. */
       @Override
       public void surfaceChanged(SurfaceHolder holder, int format, int width, int height) {
           if (mCamera == null) return;
           // The surface has change size, update the camera preview size
           Camera.Parameters parameters = mCamera.getParameters();
           Camera.Size size = getSupportedSize(parameters.getSupportedPreviewSizes(), width, height);
           //  设置图片尺寸大小
           parameters.setPictureSize(size.width, size.height);

           mCamera.setParameters(parameters);

           try {
               //  开始在Surface上绘制
               mCamera.startPreview();
           }catch (Exception e){
               //  如果开始预览绘制失败那么我们通过这里来释放相机资源
               Log.e(TAG, "@@-> startPreview()启动失败, 准备释放相机资源!" );
               mCamera.release();
               mCamera = null;

           }
       }

       /** SurfaceView从屏幕上移除时, Surface也随之被销毁, 通过客户端停止使用Surface*/
       @Override
       public void surfaceDestroyed(SurfaceHolder holder) {
           if (mCamera != null){
               mCamera.stopPreview();
           }
       }
   });
}

/**
*  找出具有最大数目的像素的尺寸
*/
private Camera.Size getSupportedSize(List<Camera.Size> sizes, int width, int height){
   Camera.Size bestSize = sizes.get(0);
   int largestArea = bestSize.width * bestSize.height;
   for (Camera.Size s : sizes) {
       int area = s.width * s.height;
       if (area > largestArea){
           bestSize = s;
           largestArea = area;
       }
   }

   return bestSize;

}
```

`SurfaceHolder`是我们与`Surface`对象联系的纽带. `Surface`对象代表着原始像素数据的缓冲区.

`Surface`对象也有生命周期: `SurfaceView`出现在屏幕上时, 会创建`Surface`; `SurfaceView`从屏幕上消失的时候,`Surface`随即被销毁. `Surface`不存在的时候, 必须保证没有任何内容要在它上面绘制.

不像其他视图一样, `SurfaceView`及其协同工作对象都不会自我绘制内容. 对于任何想要将内容绘制到`Surface`缓冲区的对象, 我们都称其为`Surface`的客户端. 比如这里的`Camera`

为了对应`Surface`各个生命周期, `SurfaceHolder`提供了另外一个接口`SurfaceHolder.Callback`来关联.

**确定预览界面的大小**

可能看到了`surfaceChanged()`的生命周期回调中设置了`Camera`实例的一个属性大小.

首先要知道**相机的预览大小不能随意设置**, 如果设置了不可接受的值有可能会出现崩溃. 所以我们应该先获取到设备相机所支持的预览尺寸大小. 通过`Camera.Parameters`类中的`getSupportedPreviewSizes()`就可以获取到**相机的支持的预览尺寸列表**.

这个方法返回的是一个`List<Camera.Size>`, 泛型`Camera.Size`每一个实例都封装了一个具体的图片宽高尺寸. 利用`getSupportedSize()`简便的比较方法可以达到选择一个最大的像素尺寸.

**有可能你会需要先检测设备是否有前后相机**

可以获取到`PackageManager`后, 调用`hasSystemFeature(String)`方法并传入表示设备特色功能的常量即可根据布尔值来判断. `FEATURE_CAMERA`常量代表后置相机, `FEATURE_CAMERA_FRONT`常量代表前置相机.

**目前为止相机开启并渲染值Surface应该没问题了,下面就是利用Camera进行照片**

主要逻辑就是从相机的实时预览中捕获一帧图像, 然后将其保存为JPEG格式的文件. 而要进行拍摄需要`Camera#takePicture(Camera.shutterCallback shutter, Camera.PictureCallback raw, Camera.PictureCallback jpeg)`

- `shutterCallback` 此回调里面的方法会在相机捕获图像的时候调用, 但此时图像数据还未处理完成.
- `第一个PictureCallback` 回调方法是在原始图像数据可用时调用, 通常来说, 是在加工处理原始图像数据且没有存储之前.
- `第二个PictureCallback` 回调方法是在JPEG版本的图像可用时候调用.

如果不需要某一个步骤的回调可以直接传递null. 这里我们实现参数一和参数三的回调, 首先定义回调如下

```
/**
*  在进行快门的时候进行进度条View的显示
*/
private Camera.ShutterCallback mShutterCallback = new Camera.ShutterCallback() {
   @Override
   public void onShutter() {
       //  这里我做了照相开始时候显示进度条, 如果为了练习这里可以不需要
       mVFlProgress.setVisibility(View.VISIBLE);
   }
};

/**
*  Camera如果快门之后有数据处理生成成功,  那么会调此回调, 可以进行把数据写入到本地的动作
*/
private Camera.PictureCallback mJpegCallback = new Camera.PictureCallback() {
   @Override
   public void onPictureTaken(byte[] data, Camera camera) {
       // create a filename
       String filename = UUID.randomUUID().toString()+".jpg";
       //  save the jpeg date to disk
       FileOutputStream out = null;
       try {
           // 获得输出流进行数据的写入
           out = mContext.openFileOutput(filename, Context.MODE_PRIVATE);
           out.write(data);
       } catch (IOException e) {
           Log.e(TAG, "onPictureTaken: @@-> 图片写入disk失败, 失败文件地址:"+filename, e );
       }finally {
           // 进行清扫动作, 关流
           if (out != null){
               try {
                   out.close();
               } catch (IOException e) {
                   Log.e(TAG, "onPictureTaken: @@-> 流文件关流失败, 失败文件地址:"+filename, e );
                  
               }
           }
       }

      Log.i(TAG, "onPictureTaken(): 照片 JPEG 保存成功, 即将关闭activity, 保存地址:"+filename);
};
```

回调定义好了之后, 剩下的只需要搞一个按钮, 添加点击事件通过调用`Camera`的实例的`takePicture()`传入自定义的回调方法即可.

**扩展:**

如果需要显示大图可以直接`DialogFragment`.

1. 首先设置`Fragment`的样式为`DialogFragment.STYLE_NO_TITLE`.
2. 因为想要的效果为一个图片查看放大, 那么不需要显示`AlertDialog`视图自带的标题和按钮. 所以可以直接复写`onCreateView()`方法使用一个简单视图, 比覆盖`onCreateDialog()`方法使用`Dialog`更简单.快捷灵活.
3. 通过文件的路径得到`Bitmap`设置到在`onCreateView()`中创建并返回的`ImageView`即可.

## 隐式Intent

在`Android`系统中, 使用**隐式Intent**可以启动其他应用的activity. 在**显示Intent**中, 需要指定要启动的`activity`类. 操作系统会负责启动它. 而**隐式Intent**中, 只要描述清楚要完成的任务, 操作系统会找到合适的应用. 并在其中启动相应的`activity`.

**典型的隐式Intent的组成**

- **要执行的动作(action)**: 通常以`Intent`类中的常量进行表示. 例如要访问查看某个`URL`,可以使用`Intent_ACTION_VIEW`; 要发送邮件,可以使用`Intent.ACTION_SEND`
- **要发送的数据位置以及数据类型**: 数据位置的话可能会是某个网页的`URL`,也可能是指向某个文件的`URI`, 或者指向`ContentProvider`中某条记录的某个内容`URI`; 数据类型这里指的是`MIME`形式的数据类型, 如`text/html`或者`audio/mpeg3`. 如果一个intent包含某类数据的位置, 那么通常可以从中推测出数据的类型.
- **可选类别(category)**: 如果`action`用于描述具体要做什么, 那么类别通常用来描述我们**何时,何地或者说如何使用**某个activity. `android.intent.category.LAUNCHER`类别表明, activity应该显示在顶级应用启动器中. 而`android.intent.category.INFO`类别表明,虽然activity向用户显示了包信息, 但它不应该显示在启动器中.

**而在配置清单文件中的intent过滤器设置时**

- `<action>`: 告诉操作系统, activity能够处理指定的哪个action动作.
- `<category>`: 一般情况下都是要指定`DEFAULT`类别. 每发起**隐式intent**如果没有指定`category那么系统都会默认的添加`DEFALUT`类别.

操作系统进行`隐式Intent`寻找的时候, 是不需要使用数据(extra)来参与匹配规则的.

**利用Intent选择器**

比如打开一个视频文件的时候, 如果手机上有多个选择的话可能会弹出一个全部列表, 然后选择一个进行播放, 而之后会发现再也不会询问用户使用哪个视频播放软件进行播放. 对于这种情况有时候或许不需要, 这个时候就可以使用**选择器**来创建一个activity来展示可打开的软件, 每次都进行选择. 使用很简单.

```
Intent intent = new Intent();
intent.setType("text/plain");
intent.putExtra(Intent.EXTRA_TEXT, getCrimeReport());
intent.putExtra(Intent.EXTRA_SUBJECT, getString(R.string.crime_report_subject));

// 设置一个选择每一次都显示activity的选择器. 
// 这是重点
intent= intent.createChooser(intent, getString(R.string.send_report));

startActivity(intent);
```

通过`createChooser()`方法重新构建一个`intent`即可, 参数二传递的字符串是用来作为弹出的选择界面的标题.

**再说一下获取联系人信息**

如果要获取手机通讯录, 那么要指定`action`,并且要找指定获取位置; action对应的`Intent.ACTION_PICK`. 获取的位置为`ContactsContract.Contacts.CONTENT_URI`. 就是请求Android协助从手机联系人数据库获取某个具体联系人.

打开手机联系人代码

```
// 打开联系人Contract列表
// 这是通讯录应用将其权限临时给了本应用, 首先通讯录应用对联系人的数据库具有全部权限. 所以在通讯录应用
// 返回包含在Intent中的URI的时候, 他会添加一个Intent.FLAG_GRANT_READ_URI_PERMISSION标识,
// 此标志向系统表示, 我们这个应用可以使用联系人数据一次.
Intent intent = new Intent(Intent.ACTION_PICK, ContactsContract.Contacts.CONTENT_URI);

// 检测是否有接收此Intent的应用
List<ResolveInfo> resolveInfos = getActivity().getPackageManager().queryIntentActivities(intent, 0);

if (resolveInfos.size() > 0){
startActivityForResult(intent, REQUEST_CONTACT);
}else{
Toast.makeText(getActivity().getApplicationContext(), "无法打开Intent", Toast.LENGTH_SHORT).show();
}
```

以上代码很容易懂, 并且做了一定的防错处理. 接下来就准备接收返回结果就可以在`onActivityResult()`中

```
if (requestCode == REQUEST_CONTACT){
  //  打开联系人列表关闭后返回的逻辑

  //  从intent取出URI, 该数据URI是一个指向用户所选联系人的定位符.
  Uri contactUri = data.getData();

  //  specify which fields you want you query to return values for
  //  指定在返回数据的时候所对应查找的字段
  String[] queryFileds = {ContactsContract.Contacts.DISPLAY_NAME};

  //  perform query
  Cursor query = getActivity().getContentResolver().query(contactUri, queryFileds, null, null, null);

  if (query.getCount() == 0){
      query.close();
      return;
  }

  // 获取嫌疑人的姓名 添加到陋习记录中的suspect
  query.moveToFirst();
  String suspect = query.getString(0);

  // 拿到了姓名 可以做后续的事情....

}
```

上面获得了一个`Cursor`, 因为已经知道`Cursor`只是包含一条记录, 所以将`Cursor`移动到第一条记录并获取它的字符串形式.就是姓名.

上面代码中有两句话是对可以响应的`activity`做检查. 通过`PackageManager#queryIntentActivitys()`返回集合的size大小来决定是否可以执行后续的操作.

**打电话的隐式Intent**

电话相关的`Intent`有两种

- `Intent.ACTION_DIAL`: 选择联系人(得到号码发送一个`tel:xxx`数据uri)之后会停止到拨号界面等待用户手动呼叫
- `Intent.ACTION_CALL`: 选择联系人(得到号码发送一个`tel:xxx`数据uri)之后会立即拨打出去. 而不会等待用户的手动拨打.

## 深入了解Intent

利用`隐式的Intent`可以创建一个启动器来替换系统默认的启动器应用. 例如这样:

![img](http://szysky.com/2016/09/09/%E3%80%8AAndroid%E7%BC%96%E7%A8%8B%E6%9D%83%E5%A8%81%E6%8C%87%E5%8D%97%E3%80%8B%E9%9A%8F%E8%AE%B0%E4%B8%89-%E5%AA%92%E4%BD%93%E4%B8%8EIntent%E7%AD%89/Launch.png)

展示出来手机上面的所有应用, 选择一个并可以打开, 有点丑, 你也可以设置上图片. 显示实现这个直接贴出代码.

```
//  创建一个隐式Intent, 从PackageManager中获取匹配intent的activity列表
Intent intent = new Intent(Intent.ACTION_MAIN);
intent.addCategory(Intent.CATEGORY_LAUNCHER);

PackageManager pm = getActivity().getPackageManager();
List<ResolveInfo> resolveInfos = pm.queryIntentActivities(intent, 0);

Log.i(TAG, "onCreate: 可以打开的数量:"+resolveInfos.size());

//  进行排序
Collections.sort(resolveInfos, new Comparator<ResolveInfo>() {
  @Override
  public int compare(ResolveInfo o1, ResolveInfo o2) {
      PackageManager pm = getActivity().getPackageManager();
      //  按照名称进行排序
      int result = String.CASE_INSENSITIVE_ORDER.compare(o1.loadLabel(pm).toString(), o2.loadLabel(pm).toString());
      return result;
  }
});

​``` 



首先利用`PackageManager`对`Main/Launcher`进行匹配, 这个没什么说的, 所有的应用都有一个启动入口就是这个. 可以得到所有应用的入口信息集合. 

这里有一点需要了解: **`MAIN/LAUNCHER`intent过滤器不能`startActivity()`这种方式发送的`MAIN/LAUNCHER`相匹配**.  因为对于`类别category为Launcher`的时候, 系统是不希望你通过隐式Intent的方式去打开. 而是要你使用显示intent. 一般情况下隐式Intent打开的时候系统总是会给你添加`category为default`的类别. 所以可以认为`隐式Intent`打开的基本都是过滤器信息中类别包含为`Default`的类别的Intent. 而这一点在系统的入口`activity`过滤其中却无法得到保证. 

定义了`MAIN/LAUNCHER`过滤器的activity是应用的主要入口, 它只关心作为应用主要入口点要处理的工作. 通常不关心自己是否属于默认的主要入口点, 因此,他也就不必包含`CATEGORY_DEFAULT`类别. 

**但好在我们可以通过隐式的`MAIN/LAUNCHER`**查到匹配的`activity`集合信息, 而不需要先打开. 通过集合中的每个activity的`ResolveInfo`实例得到`ActivityInfo`我们也就可以得到**包名, 类名**. 那么就可以通过**显示Intent打开**! 如下代码

通过上面得到的集合, 随便获取一个`resolveInof`实例.

​```java
ResolveInfo resolveInfo = (ResolveInfo) resolveInfos.get(0);;

//  准备获取要打开activity的包名 类名等信息
ActivityInfo activityInfo = resolveInfo.activityInfo;

if (activityInfo == null) return;

// 创建显示Intent来打开Activity
//  先指定一个action
Intent intent = new Intent(Intent.ACTION_MAIN);
intent.setClassName(activityInfo.applicationInfo.packageName, activityInfo.name);
//  添加新任务标识给intent
intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);

startActivity(intent);
```

上面使用的是`setClassName()`方法, 这个方法可以自动创建**组件名**, 也可以自己通过类名和包名创建一个**ComponentName**, 然后使用`setComponent()`创建一个显示的intent.

上面还添加了一个`FLAG`, 主要的区别就是没有这个`Flag`的话, 那么打开的新应用的界面本质上是存在我们的**应用任务栈中**, 如果有那么就会在**属于自己独立的任务中**. 如下图, 在我们自己的应用打开同一个任务. 然后查看任务管理器. (前面为没有添加flag的)

![img](http://szysky.com/2016/09/09/%E3%80%8AAndroid%E7%BC%96%E7%A8%8B%E6%9D%83%E5%A8%81%E6%8C%87%E5%8D%97%E3%80%8B%E9%9A%8F%E8%AE%B0%E4%B8%89-%E5%AA%92%E4%BD%93%E4%B8%8EIntent%E7%AD%89/taskStack.png)

至此, 现在这个应用可以得到手机的全部应用并可以做为展示列表. 但是还没完, 还差一点**将我们这个应用作为设备主屏幕**.

没有人愿意通过一个应用来启动另一个应用, 所以要做的就是替换`Android主界面`配置我们应用的启动`activity`再添加两个类别category,

```
<activity android:name=".NerdLauncherActivity">
  <intent-filter>
      <action android:name="android.intent.action.MAIN" />
      <category android:name="android.intent.category.LAUNCHER" />

      <category android:name="android.intent.category.HOME"/>
      <category android:name="android.intent.category.DEFAULT"/>
  </intent-filter>
</activity>
```

## 应用图标与任务重排

通过上面的获得的`ResolveInfo.loadLable()`方法, 可以获取各个`activity`的名称, 其中还有一个方法`loadIcon()`可以使用该方法为每一个应用加载显示图标.

关于**任务重排**, 需要使用`ActivityManager`系统服务, 该系统服务提供了当前运行`activity`, 任务以及应用的有用信息. 通过`Activity.getSystemService(Activity.ACTIVITY_SERVICE)`来获取`ActivityManager`然后调用`ActivityManager实例的getRunningTasks()`方法, 得到按照时间由近到久的排序的任务列表. 在调用`moveTaskToFront()`方法实现将任意任务切换到前台. 关于任务切换需要一些权限配置, 具体参考android文档.

## 进程和任务

- 进程: 是操作系统创建的供应用对象生存以及应用运行的地方. 包含了应用的全部运行代码和对象.
- 每一个`activity`实例都仅存在一个进程和一个任务中. 这也是进程与任务的唯一类似的地方.
- 任务: 只包含`activity`, 这些`activity`通常来自不同应用.

`activity`赖以生存的任务和进程有可能会有所不同. 比如上面的`深入了解Intentn`中给出的图片参考. 可以发现. 当我没有指定`new_task`的时候, 新的应用打开的activity是和我们的应用在一个任务栈的. 这也就意味当后退的时候虽然看着只是界面的切换,但是实际上发生了`进程间的切换`

## XmlPullParser使用

`XmlPullParser`接口采用拉的方式从`xml`数据流中获取解析事件. Android内部也使用`XmlPullParser`接口来实例化布局文件.

```
// 模拟一个xml字符串
String textXMLStr = "<school>\n" +
     "    <class name=\"1年级\">\n" +
     "        <student name=\"张三\"/>\n" +
     "        <student name=\"李四\"/>\n" +
     "    </class>\n" +
     "\n" +
     "    <class name=\"2年级\">\n" +
     "        优班\n" +
     "        <student name=\"张三\"/>\n" +
     "        <student name=\"李四\"/>\n" +
     "    </class>\n" +
     "\n" +
     "</school>";

// 利用xmlPullParser的工厂类创建出一个解析流对象
XmlPullParser xmlPullParser = XmlPullParserFactory.newInstance().newPullParser();

// 把要解析的xml格式的字符串设置到解析流对象中
xmlPullParser.setInput(new StringReader(textXMLStr));

// 准备开始解析 移动指针到第一个标签
int eventType = xmlPullParser.next();

// 如果标签类型不是结束文档标签标示, 那么就标示还有内容继续循环
while(eventType != XmlPullParser.END_DOCUMENT){
 // 判断class标签
 if (eventType == xmlPullParser.START_TAG && "class".equals(xmlPullParser.getName())){
     String className = xmlPullParser.getAttributeValue(null, "name");
     Log.e("sususu","获得的class标签的名称属性为: "+className );
 }

 // 判断student标签
 if (eventType == xmlPullParser.START_TAG && "student".equals(xmlPullParser.getName())){
     String studentName = xmlPullParser.getAttributeValue(null, "name");
     Log.e("sususu","获得的student标签的属性: "+studentName );
 }
 // 移动指针到下一个标签
 eventType = xmlPullParser.next();
}
```

运行结果:

![img](http://szysky.com/2016/09/09/%E3%80%8AAndroid%E7%BC%96%E7%A8%8B%E6%9D%83%E5%A8%81%E6%8C%87%E5%8D%97%E3%80%8B%E9%9A%8F%E8%AE%B0%E4%B8%89-%E5%AA%92%E4%BD%93%E4%B8%8EIntent%E7%AD%89/xmlPullParser.png)

代码中的注释已经很详细了, 不用再做解释. 有一点需要注意, 如果要获得便签的内容的话,那么别忘了考虑到空白字符或者换行符的也是存在的.