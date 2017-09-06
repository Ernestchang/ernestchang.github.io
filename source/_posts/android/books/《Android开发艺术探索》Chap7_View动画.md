title: 《Android开发艺术探索》Chap7_View动画
date: 

categories: 
- android 
- 《Android开发艺术探索》
tags:  
- android
---

> 了解动画并去自定义动画, 在一些场景有哪些注意事项
> 注：本文主要参考[http://szysky.com](http://szysky.com)


[blog相关代码](https://github.com/suzeyu1992/Notes_AndroidDevSeek)

## View动画

View动画作用的对象是View, 它支持四种动画效果**平移, 缩放, 旋转, 透明**. 除了这四种典型的变化效果. **帧动画**也属于View动画.

### View动画的种类

View动画的四种变换效果对应着`Animation`的四个子类:`TranslateAnimation`, `ScaleAnimation`, `RotateAnimation`和`AlphaAnimation`.

对于View动画建议采用XML来定义动画

| 名称    | 标签            | 子类                 | 效果         |
| ----- | ------------- | ------------------ | ---------- |
| 平移动画  | `<translate>` | TranslateAnimation | 移动View     |
| 缩放动画  | `<scale>`     | ScaleAnimation     | 放大或者缩小View |
| 旋转动画  | `<rotate>`    | RotateAnimation    | 旋转View     |
| 透明度动画 | `<alpha>`     | AlphaAnimation     | 改变View的透明度 |

创建的动画的xml文件. 是放在**res/anim**这个文件夹下的. View动画描述文件的固有语法如下

```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    android:shareInterpolator="true"
    android:fillAfter="true">
    
    <alpha 
        android:fromAlpha="float"
        android:toAlpha="float"/>
    
    <scale
        android:fromXScale="float"
        android:toXScale="float"
        android:fromYScale="float"
        android:toYScale="float"
        android:pivotX="float"
        android:pivotY="float"/>
    
    <translate
        android:fromXDelta="float"
        android:toXDelta="float"
        android:fromYDelta="float"
        android:toYDelta="float"/>
    
    <rotate
        android:fromDegrees="float"
        android:toDegrees="float"
        android:pivotY="float"
        android:pivotX="float"/>

</set>
```

关于动画我们可以只设置一种也可以设置多种的组合.

**set标签对应着AnimationSet类, 标签中的属性的意义:**

- **shareInterpolator** 表示集合中的动画是否和集合共享一个插值器. 如果集合不指定插值器, 那么子动画就需要单独制定所需的插值器或者使用默认值
- **fillAfter** 是否保留动画结束之后的状态

**translate标签表示平移动画, 对应着TranslateAnimation类**

属性值的意义就是**from**开头的为开始起点, **to**开头的结束点

**scale标签表示缩放动画, 对应着ScaleAnimation类**

属性值的意思`from`开头的表示开始时原图缩放的百分比. 用浮点数表示1表示100%(无变化),0.5表示50%(原来的一般), 2表示200%(原来的两倍). `to`开头的表示结束时的百分比. `pivot`表示缩放的轴点.

**rotate标签表示旋转动画, 对应着RotateAnimation类**

`fromDegrees`旋转的开始角度, `toDegrees`旋转的结束角度. `pivot`旋转的轴点

**alpha标签表示透明度动画, 对应AlphaAnimation类**

`fromAlpha`表示透明度的起始值, `toAlpha`表示透明度的结束值.

上面这些标签还有一些通用的属性值. 例如`duration`执行时间.

xml如果声明了之后那么我们就该在代码中应用了. 如下:

```
View btn_main = findViewById(R.id.parent);
Animation animation = AnimationUtils.loadAnimation(this, R.anim.temp);
btn_main.startAnimation(animation);
```

同样也可以不需要xml直接在代码中生成动画对象.

```
AlphaAnimation alphaAnimation = new AlphaAnimation(1, 0);
alphaAnimation.setDuration(1000);
btn_main.startAnimation(alphaAnimation);
```

在开始动画之前可以给动画添加一个监听`setAnimationListener()`这样在动画开始结束和每一次循环下一次的时候都可以在回调方法中监听到.

### 自定义View动画

如果需要自定义View动画, 首先应该继承`Animation`这个抽象类来派生出一种新动画. 然后重写`initialize()`和`applyTransformation()`方法. 在`initialize`中做一些初始化动作, 在`applyTransformation()`中进行相应矩阵变换, 很多时候需要采用`Camera`来简化矩阵变换的过程. 而View动画变化主要就是矩阵的变换过程.

这里举一个Android中ApiDemo的一个自定义View动画. 大概效果就是这样可以参照官网的api也可在包中的`MyRotateAnimation`

![img](http://szysky.com/2016/08/13/%E3%80%8AAndroid-%E5%BC%80%E5%8F%91%E8%89%BA%E6%9C%AF%E6%8E%A2%E7%B4%A2%E3%80%8B-07-Andriod%E5%8A%A8%E7%94%BB%E6%B7%B1%E5%85%A5%E5%88%86%E6%9E%90/reote3d.gif)

### 帧动画

帧动画是顺序播放一组预先定义好的图片, 类似于电影. 系统提供了`AnimationDrawable`来使用帧动画.

同样在xml中声明, 在`res/drawable/`包下创建文件, 并替换每个`drawable`图片即可

```
<?xml version="1.0" encoding="utf-8"?>
<animation-list
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:oneshot="false">

    <item android:drawable="@drawable/xx1" android:duration="500"/>
    <item android:drawable="@drawable/xx2" android:duration="500"/>
    <item android:drawable="@drawable/xx3" android:duration="500"/>
    <item android:drawable="@drawable/xx4" android:duration="500"/>

</animation-list>
```

将上述的Drawable作为View的背景并通过Drawable来播放动画.

```
AnimationDrawable background = (AnimationDrawable) iv_main.getBackground();background.start();
```

## View动画的特殊使用场景

前面介绍的View动画都是作用在某一个`View`对象上的. 还可以针对`ViewGroup`控制其子元素. 或者针对`Activity`切换的动画.

### LayoutAnimation

`LayoutAnimation`作用于`ViewGroup`上的. 为`ViewGroup`指定一个动画, 这样当它的子元素出场时都会具有这种动画效果. 常用的使用场景是在ListView和GridView. 使用很简单步骤如下.

1. 在`res/anim/`文件夹下创建xml文件.

```
<?xml version="1.0" encoding="utf-8"?>
<layoutAnimation xmlns:android="http://schemas.android.com/apk/res/android"
    android:delay="0.5"
    android:animationOrder="random"
    android:animation="@anim/layout">
    
</layoutAnimation>
```

**delay**: 子元素开始动画的延迟时间, 传入值是浮点值. 1为100%. 例如如果是0.5 入场动画周期为300ms(下面关联动画的duration时间), 那么每个子元素都需要延迟150ms才能播放入场动画. 而且这个时间会根据item的递增而增加. 比方说第一个为延迟150ms, 第二个就是300ms依次类推.

**animationOrder**: 子元素动画的顺序, 有三种选择`normal`,`reverse`,`random`. `reverse`表示排在后面的元素先执行入场动画. `random`随机子元素执行动画.

**animation**: 为子元素指定具体的入场动画. 里面放的就是针对View的animation动画的xml

`layoutAnimation`声明完成之后, 在要作用的ViewGroup标签中增加`android:layoutAnimation:"@anim/xxx"`进行关联即可. 同样也可以通过代码创建`LayoutAnimation`类来实现.

```
//获得子元素需要执行的View动画
Animation animation = AnimationUtils.loadAnimation(this, R.anim.layout);
   
//创建一个LayoutAnimation动画对象
LayoutAnimationController controller = new LayoutAnimationController(animation);
controller.setDelay(0.5f);
controller.setOrder(LayoutAnimationController.ORDER_RANDOM);
   
//对ViewGrop进行绑定
listView.setLayoutAnimation(controller);
```

### Activity的切换效果

Activity默认是有一种切换效果的. 如果需要自定义切换效果, 主要用到`overridePendingTransition()`这个方法, 这个方法必须在**startActivity()或者finish()之后调用才会生效**

需要的形参有两个, 第一个是被打开时候所需的动画资源id, 第二个是被暂停时,所需的动画资源id.

**= =. 这里有问题, 试了半天不好使, 就没写具体步骤….**

## 属性动画

属性动画是API新加入的特性, 和View动画不同, 它对作用对象进行了扩展, 属性动画可以对任何对象做动画. 属性动画不再像View动画那样只能支持四种简单的交换 . 属性动画中有`valueAnimator`. `ObjectAnimator`, `AnimatorSet`等概念

### 使用属性动画

属性动画可以对任何对象的属性进行动画而不仅仅是View, 动画默认时间间隔为300ms, 默认帧率10ms/帧. 可以达到的效果为: 在一段时间间隔内完成对象从一个属性值到另一个属性值的改变. 属性动画是从**API11**增加的.

如: 改变一个对象的背景色属性, 典型的改变View的背景色, 下面的动画可以让背景颜色的渐变, 动画会无限循环而且会有反转效果.

```
ObjectAnimator colorAnim = ObjectAnimator.ofInt(activity_main, "backgroundColor", 0xffffa000, 0xffffa0ff);
        colorAnim.setDuration(5000);
        colorAnim.setEvaluator(new ArgbEvaluator());
        colorAnim.setRepeatCount(ValueAnimator.INFINITE);
        colorAnim.setRepeatMode(ValueAnimator.REVERSE);
        colorAnim.start();
```

效果这样:

![img](http://szysky.com/2016/08/13/%E3%80%8AAndroid-%E5%BC%80%E5%8F%91%E8%89%BA%E6%9C%AF%E6%8E%A2%E7%B4%A2%E3%80%8B-07-Andriod%E5%8A%A8%E7%94%BB%E6%B7%B1%E5%85%A5%E5%88%86%E6%9E%90/bg1.gif)

**动画集合,5秒内对View旋转平移缩放透明**

```
AnimatorSet animatorSet = new AnimatorSet();
        animatorSet.playTogether(
                ObjectAnimator.ofFloat(iv_main, "rotationX", 0,360),
                ObjectAnimator.ofFloat(iv_main, "rotationY", 0,360),
                ObjectAnimator.ofFloat(iv_main, "rotation", 0,360),
                ObjectAnimator.ofFloat(iv_main, "translationX", 0,200),
                ObjectAnimator.ofFloat(iv_main, "translationY", 0,200),
                ObjectAnimator.ofFloat(iv_main, "scaleX", 1,1.5f),
                ObjectAnimator.ofFloat(iv_main, "scaleY", 1,1.5f),
                ObjectAnimator.ofFloat(iv_main, "alpha", 1, 0.25f, 1)
        );

        animatorSet.setDuration(5*1000).start();
```

也可以使用xml的形式形式来声明

### 理解插值器和估值器

`TimeInterpolator`时间插值器, 作用是根据时间流逝的百分比来计算当前属性值改变的百分比. 系统预置的有

- `LinearInterpolator`(线性插值器:匀速动画)
- `AccelerateDecelerateInterpolator`(加速减速插值器:动画两头慢中间快)
- `DecelerateInterpolator`(减速插值器:动画越来越慢)

`TypeEvaluator` 类型估值算法, 也叫估值器. 作用是根据当前属性改变的百分比来计算改变后的属性值. 系统预置的估值器有

- `IntEvaluator` 整形估值器
- `FloatEvaluator` 浮点型估值器
- `ArgbEvaluator` Color属性估值器

属性动画中的插值器和估值器都很重要, 他们是实现非匀速动画的重要手段

属性动画要求对象的该属性有`set``get`方法. 插值器和估值器算法除了系统提供的外. 也可以自定义. 实现方式也很简单, 因为插值器和估值算法都是一个接口, 且内部都只有一个方法, 我们只要派生一个类实现接口接可以. 具体就是: 自定义插值器需要实现`Interpolator`或者`TimeInterpolator`. 自定义估值算法需要实现`TypeEvaluator`

### 属性动画的监听器

属性动画提供了监听器用于监听动画的播放过程 主要有两个接口`AnimatorUpdateListener`和`AnimatorListener`接口.

- `AnimatorListener` 通过接口的定义可以看出, 监听了动画的开始,结束,取消,以及重复播放. 系统为了方便开发提供了`AnimatorListenerAdapter`类. 他是`AnimatorListener`的适配器. 这样就不需要非得实现四个抽象方法而是按照我们的需要选择复写.
- `AnimatorUpdateListener` 比较特殊, 他会监听整个动画过程, 动画是由许多帧组成的. 每播放一帧`onAnimationUpdate`就会被调用一次

### 对任意属性做动画

**问题: 如果需要把一个button控件的宽增加200px. 应该怎么做?**

**View动画**只是支持四种基本的属性操作, 而**Scale**只是缩放. 并且还会对内容进行拉伸并且伴随着y轴的增加. 所以属性动画在这里就可以派上用场. 但是如果直接对`width`属性进行修改那么不会有效果. 分析一下:

**属性动画的原理:** 属性动画要求动画作用的对象提供该属性的**get和set方法**, 属性动画根据外界传递的该属性值的初始值和最终值, 以动画的效果多次调用`set`每次set的值也是不同. 最终达到终点值.

所以要让动画生效应该满足两个条件:

1. 必须提供`setXXX()`方法, 如果动画没有传递初始值还要提供`getXXX()`方法. 这样系统在需要初始属性的时候在取值时不会因为没有`getXXX()`而发生Crash.
2. set修改的值必须能改通过某种形式反映出来, 比如会带来UI的改变. (如果不满足这条,动画无效果但不会Crash)

那`Button`本身具备`setWidth()`为什么会无效果. 这是因为虽然`Button`提供了方法, 但是这个`setWidth()`方法并不是改变视图大小的, 他是`TextView`新添加的方法, View却没有这样的方法. 而`setWidth()`方法的内部,作用不是设置View的大小, 而是设置`TextView`的最大宽度和最小宽度, 这个和`TextView`的宽是两个东西. 这样说控件的宽度对应`xml中的layout_width`, 而`setWidth()对应的就是xml中的width属性`. 所以综合上述原因, 满足条件一而不满足条件二.

官网文档中给出了三种解决方案:

1. 给你的对象加上get和set方法, 如果你有权限的话.
2. 用一个类来包装原始对象, 间接为其提供get和set方法.
3. 采用valueAnimator, 监听动画过程,自己实现属性的改变.


1. 虽然简单但是没有权限去SDK内部实现去
2. 可以创建一个内部包装类创建**set(),get()方法**对View的`LayoutParams.width`进行修改.
3. 采用`ValueAnimator`, 监听动画过程, 自己实现属性改变. `ValueAnimator`本身不作用于任何对象. 但是他可以对一个值做动画. 通过对每个值的分配并会回调函数返回此值, 可以手动进行实现.

### 属性动画的工作原理

前面说过, 说属性画要求作用的对象提供该属性方法set方法, 属性动画根据传递的该属性的初始值和最终值, 以动画的效果多次去调用set方法. 每次set方法时候传递的值都是不一样的. 也就是随着时间的推移所传递的值会越来越接近终点值.

源码分析: 针对`ObjectAnimator的start()`为入口

```
 @Override
public void start() {
   // See if any of the current active/pending animators need to be canceled
   AnimationHandler handler = sAnimationHandler.get();
   if (handler != null) {
       int numAnims = handler.mAnimations.size();
       for (int i = numAnims - 1; i >= 0; i--) {
           if (handler.mAnimations.get(i) instanceof ObjectAnimator) {
               ObjectAnimator anim = (ObjectAnimator) handler.mAnimations.get(i);
               if (anim.mAutoCancel && hasSameTargetAndProperties(anim)) {
                   anim.cancel();
               }
           }
       }
       numAnims = handler.mPendingAnimations.size();
       for (int i = numAnims - 1; i >= 0; i--) {
           if (handler.mPendingAnimations.get(i) instanceof ObjectAnimator) {
               ObjectAnimator anim = (ObjectAnimator) handler.mPendingAnimations.get(i);
               if (anim.mAutoCancel && hasSameTargetAndProperties(anim)) {
                   anim.cancel();
               }
           }
       }
       numAnims = handler.mDelayedAnims.size();
       for (int i = numAnims - 1; i >= 0; i--) {
           if (handler.mDelayedAnims.get(i) instanceof ObjectAnimator) {
               ObjectAnimator anim = (ObjectAnimator) handler.mDelayedAnims.get(i);
               if (anim.mAutoCancel && hasSameTargetAndProperties(anim)) {
                   anim.cancel();
               }
           }
       }
   }
   if (DBG) {
       Log.d(LOG_TAG, "Anim target, duration: " + getTarget() + ", " + getDuration());
       for (int i = 0; i < mValues.length; ++i) {
           PropertyValuesHolder pvh = mValues[i];
           Log.d(LOG_TAG, "   Values[" + i + "]: " +
               pvh.getPropertyName() + ", " + pvh.mKeyframes.getValue(0) + ", " +
               pvh.mKeyframes.getValue(1));
       }
   }
   super.start();
}
```

这段代码主要就是取消和当前动画相同的动画. 最开始判断了当前动画,等待动画,延迟动画是否有一致的. 如果有那么就给取消. 最后调用了父类方法. 因为`ObjectAnimator`继承了`ValueAnimator`,所以继续看一下父类的`start()`

```
private void start(boolean playBackwards) {
   if (Looper.myLooper() == null) {
       throw new AndroidRuntimeException("Animators may only be run on Looper threads");
   }
   mReversing = playBackwards;
   mPlayingBackwards = playBackwards;
   
   int prevPlayingState = mPlayingState;
   mPlayingState = STOPPED;
   mStarted = true;
   mStartedDelay = false;
   mPaused = false;
   updateScaledDuration(); // in case the scale factor has changed since creation time
   AnimationHandler animationHandler = getOrCreateAnimationHandler();
   animationHandler.mPendingAnimations.add(this);
   if (mStartDelay == 0) {
       // This sets the initial value of the animation, prior to actually starting it running
       if (prevPlayingState != SEEKED) {
           setCurrentPlayTime(0);
       }
       mPlayingState = STOPPED;
       mRunning = true;
       notifyStartListeners();
   }
   animationHandler.start();
}
```

属性动画需要运行在有`Looper`的线程中, 最终会调用`AnimationHandler.start()`方法.`AnimationHandler`并不是Handler, 他是一个`Runnable`. 后面会调到JNI层, 然后JNI层还会调回, 然后`run`方法会被调用, 这个Runable涉及和底层的交互. 略过. 看重点.

**ValueAnimator的doAnimationFrame()方法, 内部最后调用了animationFrame()方法,而animationFrame()内部调用了animateValue()方法**

```
void animateValue(float fraction) {
   fraction = mInterpolator.getInterpolation(fraction);
   mCurrentFraction = fraction;
   int numValues = mValues.length;
   for (int i = 0; i < numValues; ++i) {
       mValues[i].calculateValue(fraction);
   }
   if (mUpdateListeners != null) {
       int numListeners = mUpdateListeners.size();
       for (int i = 0; i < numListeners; ++i) {
           mUpdateListeners.get(i).onAnimationUpdate(this);
       }
   }
}
```

看到了`calculateValue()`方法, 这个就是计算每帧动画所对应的属性的值, 然后看一下set,get方法. 比如之前说的如果没有初始值, 则调用get方法等.. 查看`PropertyValuesHolder类的setupValue()`

```
private void setupValue(Object target, Keyframe kf) {
   if (mProperty != null) {
       Object value = convertBack(mProperty.get(target));
       kf.setValue(value);
   }
   
  if (mGetter == null) {
      Class targetClass = target.getClass();
      setupGetter(targetClass);
      if (mGetter == null) {
          // Already logged the error - just return to avoid NPE
          return;
      }
  }
  Object value = convertBack(mGetter.invoke(target));
  kf.setValue(value);
```

当动画的下一帧到来的时, `setAnimatedValue()`方法会将新的属性值给对象, 调用其set()方法.同样set也是反射调用

```
void setAnimatedValue(Object target) {
   if (mProperty != null) {
       mProperty.set(target, getAnimatedValue());
   }
   if (mSetter != null) {
      mTmpValueArray[0] = getAnimatedValue();
      mSetter.invoke(target, mTmpValueArray);
   }
}
```

## 使用动画的注意事项

1. OOM问题: 在帧动画时候容易发生
2. 内存泄漏: 如果有无限循环的属性动画, 在界面退出的时候一定要停止动画 ,否则activity会无法释放. 而**View动画**并不存在此问题.
3. 兼容性问题: 主要是3.0以下系统
4. View动画问题: 因为是对原始View做的影像效果. 并未真正改变View. 所以在动画完成之后.无法GONE掉. 这个时候调用`view.clearAnimation()`清除View效果即可
5. 不要使用px
6. 动画交互. 系统3.0之前无论是**属性动画**还是**View动画**新的位置都无法触发单击事件.需要注意
7. 硬件加速的使用