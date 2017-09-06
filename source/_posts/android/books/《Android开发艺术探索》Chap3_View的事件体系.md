title: 《Android开发艺术探索》Chap3_View的事件体系
date: 

categories: 
- android 
- 《Android开发艺术探索》
tags:  
- android
---

> Anroid中的事件是怎样进行的
>
>  注：本文主要参考[http://szysky.com](http://szysky.com)

[blog相关代码](https://github.com/suzeyu1992/Notes_AndroidDevSeek)

## View的基础知识

### View的位置参数

一个`View`的位置主要由四个顶点构成, 或者可以就是两个点就可以确定. 分别为**左上点**,**右下角**每个点都对应x,y两个属性. 因为默认都是矩形, 所以两个点就可以确定.

一个`View`的大小可以利用四个属性可知. 分别对应`getLeft()`,`getRight()`,`getTop()`,`getBottom`系统提供的函数.

- 一个控件的宽: getRight() - getLeft()
- 一个控件的高: getTop() - getBottom()

**在Android3.0中, View增加了几个属性:x , y, translationX, translationY**

- `x` , `y`: 表示View的左上角坐标点(最终坐标点).
- `translationX`, `translationY`: 表示View的左上点相对于父容器的偏移量(默认是0).

而这些参数的换算关系为:

```
x = left + translationX;
y = top + translationY;
```

### MotionEvent和TouchSlop

**MotionEvent是指手指在接触屏幕之后产生的一系列事件**

最常见事件类型是`ACTION_DOWN`,`ACTION_MOVE`,`ACTION_UP`

一次事件可以有不同的**持续时间**, 和不同的**事件类型**. 例如

- 按下抬起 : DOWN –> UP
- 按下移动抬起 : DOWN -> MOVE -> MOVE -> … ->UP
- ….

而在移动时可以根据`MotionEvent`提供的参数获对应的xy取值.

*`getX/getY`: 返回相对于当前View左上角的x,y坐标.
`getRawX/getRawY`: 返回的是针对整个屏幕的左上角的x,y坐标.

**TouchSlop是系统可以识别的最小滑动距离单位**

只有手指两次滑动大于这个`TouchSlop`,系统才认为是滑动.

`ViewConfiguration.get(getContent).getSealedTouchSlop()`可以获得这个系统值默认**8dp**.

用途: 在自定义的时候, 可以参考系统的默认值, 来作为实际的滑动定义.

### VelocityTracker GestureDetector 和Scroller

**VelocityTracker 速度追踪**

> 用于追踪手指在滑动过程中的速度,包括水平和数值方向的速度

使用方式: 在View的OnTouchEvent方法中:

```
//获得速度追踪对象
VelocityTracker velocity = VelocityTracker.obtain();
velocity.addMovement(event);

//计算速度 并获取计算值
velocity.computeCurrentVelocity(1000); //设定一个时间间隔值
float xVelocity = velocity.getXVelocity();
float yVelocity = velocity.getYVelocity();
```

必须要先计算并设定计算速度的时间单元值,才可以获得速率.

公式: `速度 = (终点位置 - 起点位置) / 时间间隔值`

可以看到, 计算的速度是根据我们自己添加的时间间隔值计算的. 并且速度可以为负值,如果向左滑动.

当不需要的时候, 调用`clear()`重置并回收内存.

```
velocity.clear();
velocity.recycle();
```

**GestureDetector 手势检测**

> 用于辅助检测用户的单击, 滑动, 长按, 双击等行为.

使用如下

创建`GestureDetector`对象并实现`OnGestureDetector`接口.

```
GestureDetector mGestureDetector = new GestureDetector(this);
// 解决长按屏幕后无法拖动的现象
mGestureDetector.setIsLongpressEnabled(false);
```

然后接管目标View的`onTouchEvent()`方法. 在`onTouchEvent()`方法中

```
boolean consume = mGestureDetector.onTouchEvent(event);
return consume;
```

然后根据需求可以选择性的实现`OnGestureListener`和`OnDoubleTapListener`接口

接口的方法说明:

| 方法名                    | 描述                                     | 所属接口                  |
| ---------------------- | -------------------------------------- | --------------------- |
| `onDown`               | 按下                                     | `OnGestureListener`   |
| `onShowPress`          | 按下 但是未松开或者拖动.强调状态                      | `OnGestureListener`   |
| `onSingleTapUp`        | 抬起 表示单击行为, 双击中也会触发                     | `OnGestureListener`   |
| `onScroll`             | 按下并拖动 拖动行为                             | `OnGestureListener`   |
| `onLongPress`          | 长按                                     | `OnGestureListener`   |
| `onFling`              | 按下屏幕病快速滑动后松开                           | `OnGestureListener`   |
| `onDoubleTap`          | 双击,两次连续单击组成, 与onSingleTapConfirmed无法共存 | `OnDoubleTapListener` |
| `onSingleTapConfirmed` | 严格意义上的单击 双击中的单击无法触发                    | `OnDoubleTapListener` |
| `onDoubleTapEvent`     | 表示发生了行为                                | `OnDoubleTapListener` |

实际开发中:根据喜好来使用. 即使不使用`GestureDetector`辅助手势检测类,一样可以实现.

建议: 如果要监听双击这种行为就是用此类.

**Scroller 弹性滑动对象**

> 用于实现View的弹性滑动.

在开发中, 当需要把View从一个点移动到另一个点的时候. 如果使用`scrollTo/scrollBy`进行滑动时, 都是瞬间完成. 没有过度动画, 给用户感觉很生硬. 使用`Scroller` 可以实现有过渡的滑动.Scroller本身无法让View弹性滑动, 需要和View的`computerScroll`进行配合使用.

下面会说到

## View的滑动

实现滑动的方式有三种:

- 通过View本身的`scrollTo/scrollBy`方法实现滑动
- 通过动画给View施加平移效果来实现动画
- 通过改变View的`LayoutParams`使View重新布局实现滑动

### scrollTo/scrollBy

首先要明确一点: **这两个方法只能改变View的内容位置,而不能改变View本身在布局中的位置**

而且方法中都是以像素值来进行移动的.

- scrollTo: 针对当前View的绝对位置进行移动.
- scrollBy: 根据当前View的内容值进行相对位置移动.

看一下`scrollBy`的源码调用

```
public void scrollBy(int x, int y) {
   scrollTo(mScrollX + x, mScrollY + y);
}
```

其实本质上`scrollBy`调用了`scrollTo`方法

而`mScrollX/mScrollY`是什么? 这个就是当前View的内容 与这个View实际布局位置(原始位置)的差值.
而当前View内容这个东西就是让用户看到的效果发生改变. 但是如果这个View可以被点击. 那么能触发点击的位置是View的**实际所在布局位置**. 而不是View的内容显示的位置.

### 使用动画

> 使用动画来对View进行移动,主要就是操作View的**translationX/translationY**属性

可以使用普通动画和属性动画.

普通动画是对View进行影像的移动. 可以通过设置`fillAfter=true`,来让影像在动画结束时候保留最终结果.而不是还原到起始位置.

而属性动画会对真实位置也进行改变.

`ObjectAnimator.ofFloat(tagerView,"translationX",0,100).setDuration(100).start()`

### 改变布局参数

这个比较简单, 获得View的`LayoutParams`参数.进行修改,改好之后再赋值回去.

```
MarginLayoutParams params = (MarginLayoutParams)mTextView.getLayoutParams();
params.width += 100;
params.leftMargin += 100;
mTextView.requestLayout();
//或者mTextview.setLayoutParams(params);
```

**关于这三种方式的简单总结**

- **scrollTo/scrollBy**: 操作简单, 适合对View内容的滑动
- **动画**: 操作简单,主要适用于没有交互的View和实现复杂的动画效果.
- **修改布局参数**: 操作稍微复杂,适用于有交互的View.

## 弹性滑动

**使用Scroller**

一个简单的使用方法如下:

```
Scroller mScroller = new Scroller(mContent);

// 封装一个方法, 接收要移动到的目标点 x和y
private void smoothScrollTo(int destX, int destY){
    int scrollX = getScrollX();
    int deltaX = destX - scrollX;
    // 1000ms内逐渐滑向destX
    mScroller.startScroll(scrollX, 0, deltaX, 0, 1000);
    invalidate();
}

//复写View的computeScroll方法
public void computeScroll(){
    if(mScroller.computeScrollOffset()){
        scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
        postInvalidate()
    }
}
```

源码中`Scroller类中startScroll()`方法,其实没有实际操作什么,只是保存了调用方法时,传递的几个参数. 如: 开始结束点,时间等. *那动画究竟是怎么实现的? 复写的computeScroll()又有什么用?*

流程顺序这样的: 当调用了`startScroll()`系统只是保存了一些信息, 但是下面调用`invalidate()`. 这个方法都知道是会导致View的重绘, 在View的`draw()`方法中又会去调用`computeScroll()`方法,本身`computeScroll()`是一个空实现,但是这里进行了复写. 而这个方法我们复写的时候调用了`scrollTo()`方法! ok这样View就会真正的移动了! 但是还有一点这次滚动只是整个滚动事件的一个小部分,后续的怎么触发的? 就是下面又调用了`postInvalidate()`, 又会重新绘制重新调用computeScroll()这个复写过的空实现方法.

而`Scroller类中的computeScrollOffset()`可以直接返回这个滚动的动作是否全部完成. 源码实现思路就是根据时间的流逝的百分比来计算出当前ScrollX和ScrollY的值.

```
// 核心代码  x就是时间流逝的百分比
mCurrX = mStartX + Math.round(x * mDeltaX);
mCurrY = mStartY + Math.round(x * mDeltaY);
```

小结Scroller的工作原理:

`Scroller`本身不可以实现滑动, 需要和`View的ComputeScroll()`配合使用来完成弹性滑动. 通过不断的在`computeScroll()`调用View的重绘方法. 每次绘制时候的当前时间与开始时间的时间差与设定的**执行动画时间**的百分比,算出每一次需要scroll到的坐标点, 然后通过调用**scrollTo()**来实现每一次的小滚动效果. 通过一连串的滚动达到了平滑的效果. 这就是`Scroller`工作机制. 完全实现了解耦操作. 这个过程没有任何一处对View进行引用,甚至连内部计时器都没有.

**通过动画**

可以直接使用`ObjectAnimator.ofFloat(tagerView,"translationX",0,100).setDuration(100).start()`

也可以利用动画的特性, 实现与Scroller原理近似的方法.

```
final int startX = 100;
final int endX = 200;
ValueAnimator valueAnimator = ValueAnimator.ofInt(0, 1).setDuration(1000);
valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
  @Override
  public void onAnimationUpdate(ValueAnimator animation) {
      int offset = (int) animation.getAnimatedFraction();
      mTextview.scrollTo(startX+offset, 0);
  }
});
```

让系统算出每个时间片我们需要移动的距离, 并回调给我们.让我们自己实现. 如果是一组动画在相同的时间执行的绝对值相同我们就可以在`onAnimationUpdate()`一起进行调用.

**使用延时策略**

> 核心思想就是通过发送一些列延时消息从而达到一种渐进的效果.

可以使用: `Handler`, `View的postDelayed()`方法, 或者线程的`sleep()`

## View事件分发机制

### 点击事件传递规则

> 所谓点击事件的事件分发,就是对MotionEvent事件的分发过程,传递给某一个View.

在事件传递中有三个方法是贯穿始终的

- `dispatchTouchEvent()`:进行事件的分发 如果事件能传递到View那么此方法一定会被调用,返回值受当前View的`onTouchEvent()`和下级View的`dispatchTouchEvent()`的影响. 表示是否消耗当前事件.
- `onInterceptTouchEvent()`: 判断是否拦截事件,如果当前View拦截了某个事件,那么在同一个事件序列中, 此方法不会被再次调用, 返回结果表示是否拦截当前事件.
- `onTouchEvent()`: 在dispatchTouchEvent中被调用. 用来处理点击事件, 返回结果表示是否消耗当前事件,如果不消耗, 则在同一个事件序列中, 当前View无法再次接收到事件.

如果把源码中的各种判断去掉, 只留最核心的代码, 那么就如下:

```
public boolean dispatchTouchEvent(MotionEvent event) {
   boolean consume = false;   //表示这个事件最终的处理结果
   
   if (onInterceptTouchEvent(event)){
        //事件被拦截自己处理
       consume = onTouchEvent(event);
   }else{
        //事件被分发到子view的dispatchTouchEvent()中
       consume = child.dispatchTouchEvent(event);
   }
   
   return consume;
}
```

**OnTouchListener() , onTouchEvent() , OnClickListener() 的优先级**

上面说了普遍情况下的事件分发. 如果这个View同时还添加了`OnTouchListener`和`OnClickListener`. 此时的优先级如下:

> OnTouchListener –> onTouchEvent –> OnClickListener

而`onTouchEvent()`能否被最终调用取决于**设置了OnTouchListener()中的onTouch()的返回值**, 如果`onTouch()`返回的结果是false,那么`onTouchEvent()`会被调用. 如果返回true那么`onTouchEvent()`不会被调用.

而最后被调用的`OnClickListener()`方法是在`onTouchEvent()`被调用的. 所以如果`onTouchEvent()`方法如果执行, 那么对应的添加的`onClickLisener()`才会被调用. 所以如果在`OnTouchListener()中的onTouch()返回true`那么`onTouchEvent()不会被调用,内部调用OnClickListener也就更无法被调用`.

**一个事件的传递过程遵循如下**

> Activity -> Window -> View

如果事件一直不拦截,传递到了最里层的View而最里层的View的`onTouchEvent()`也返回false不消费, 那么事件就会向上级的`onTouchEvent()`传递,如果还返回false就依次传递.

对于事件机制的规则:

1. **事件序列**是指按下到抬起之间发生的一系列事件.
2. 默认一个事件序列只能被一个View拦截并消耗. (例外:采用非常规,在onTouchEvent强行传递给其他View. 不推荐)
3. 如果View决定拦截,那么这个事件序列只能由它自己处理. 并且它的`onInterceptTouchEvent()`不会再被调用
4. 如果View不在`ACTION_DOWN`事件时返回true, 那么同一个事件序列都不会再交给它来处理.并且事件会重新传递到父元素的`onTouchEvent()`再次调用方法.
5. 如果View不消耗除`ACTION_DOWN`以外的事件,那么这个点击事件会消失,而父元素的`onTouchEvent()`不会被调用,并且当前View可以持续收到后续的事件,最终这些消失的事件会传递到activity处理.
6. `ViewGroup`默认不拦截任何事件, 源码中ViewGroup的`onInterceptTouchEvent()`默认返回false
7. `View`没有`onInterceptTouchEvent()`, 因为它没有子View,所以直接调用`onTouchEvent()`
8. `View`的`onTouchEvent`默认都会消耗事件返回true. 除非它不可点击的(需要`clickable`,`longClickable`同时为false). View的`longClickable`默认都为false. 而`clickable`需要区分控件, 如`Button`默认为true, `TextView`默认为false.
9. `View`的`enable`属性不影响`onTouchEvent`的默认返回值, 哪怕一个View是`disable`状态. 只要它的`clickable`或者`longClickable`有一个为true. 那么它的`onTouchEvent()`就返回true.
10. `onClick`会发生的前提是当前View为可点击, 并且他收到了down和up事件.
11. 事件传递的过程是由外向内的. 通过`requestDisallowInterceptTouchEvent()`可以在子元素中干预父元素的事件分发过程,但是`ACTION_DOWN`事件除外.

### 事件分发的源码解析

**1.Activity对点击事件的分发过程**

事件的起始`Activity`的`dispatchTouchEvent()`进行分发,具体的工作交由内部的**Window**来完成. **window**会将事件传递给**decor view**. 而**decor view**一般是当前界面的底层容器(平常setContentView中传递的布局),可通过`Activity.getWindow.getDecorView()`获得.

**window**是怎样将事件传递给`ViewGroup`的? 首先Window类为一个抽象类,而类中的调用的分发方法也为抽象方法. 所以需要找到实现类. Window的唯一实现类`PhoneWindow`. 这个类会在被实例化的时候会被重构.

`PhoneWindow#superDispatchTouchEvent(ev)`方法中将事件传递给了DecorView.

decorView就是挂载我们的layout布局的**顶级View**,继承`FrameLayout`.

`((ViewGroup)getWindow().getDecorView().findViewById(android.R.id.content)).getChildAt(0);`

这个方法可以获取到Activity所设置的View. 所以之间的关系很清楚了. 事件先交给最顶级的DecorView然后交由我们设置的View.

**2.ViewGroup对事件的处理**

在父元素中判断子元素是否能接收点击事件的主要由两个因素衡量: 子元素是否在播放动画和点击事件的坐标是否落在子元素的区域内.`dispatchTransformedTouchEvent()`实际就是调用子View的`dispatchTouchEvent`. 而在事件从孩子到父元素(子View在`onTouchEvent`返回false). 其实也是调用了`dispatchTransformedTouchEvent()`. 区别就在于向内传递参数3是传入的不是空值, 向外传传入的是null.看下面代码:

```
 private boolean dispatchTransformedTouchEvent(MotionEvent event, boolean cancel,
            View child, int desiredPointerIdBits) {
    if (child == null) {
     handled = super.dispatchTouchEvent(event);
    } else {
     handled = child.dispatchTouchEvent(event);
    }
}
```

**3.View对事件的处理**

1. View对事件的判断首先是检测是否有`onTouchListener`如果有那么就调用其中的`onTouch()`方法.
2. 然后执行`onTouchEvent()`这里有个判断条件,如果在`onTouchListener()`返回true. 那么if中的条件判断第一个就不会成立,也就不会再调用`onTouchEvent()`方法.如下:

```
if (li != null && li.mOnTouchListener != null
               && (mViewFlags & ENABLED_MASK) == ENABLED
               && li.mOnTouchListener.onTouch(this, event)) {
           result = true;
}

if (!result && onTouchEvent(event)) {
 result = true;
}
```

1. 然后在`onTouchEvent()`中首先是查看View处理不可用状态. 这里需要注意一下, 虽然View不可用但是如果点击标记或者长按点击标记都是`true`. 那么事件也会被消费.如下

```
if ((viewFlags & ENABLED_MASK) == DISABLED) {
       .....
           // A disabled view that is clickable still consumes the touch
           // events, it just doesn't respond to them.
           return (((viewFlags & CLICKABLE) == CLICKABLE
                   || (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)
                   || (viewFlags & CONTEXT_CLICKABLE) == CONTEXT_CLICKABLE);
       }
```

1. 如果View设置有代理, 那么还会执行`TouchDelegate的onTouchEvent()`
2. 然后对点击状态进行处理.首先如果**点击**和**长按**有一个为true那么就会消费事件. 即`onTouchEvent()`返回true. 然后在`ACTION_UP`中会触发`performClick()`如果设置了`onClickListener()`那么就会在此处进行判断并调用`onClick()`.
3. 上面一直说的**LONG_CLICKABLE**与**CLICKABLE**. 长按标记默认为false. 点击标记和View是否是可点击View有关系. 如button可点击默认就是true. 否者反之. 在使用时可以通过`setClickable`和`setLongClickable`对View的这两个标记进行修改. 这里还要注意还有一种赋值方式. 如果设置了`setOnClickListener()`或者`setOnLongClickListener()`监听的话那么会自动将其对应的属性置为true.

## View的滑动冲突

### 滑动冲突的解决方式

**外部拦截法**

> 是指点击事件都是需要先经过父容器的拦截处理, 如果父容器需要此事件就拦截,不需要就下放. 外部拦截需要重写父容器的onInterceptTouchEvent方法

简述一下: 如果使用这样拦截法. 那么首先

- `ACTION_DOWN`这个事件,父容器必须返回false, 即不拦截`ACTION_DOWN`事件, 因为一旦父容器拦截了这个事件, 那么后续的`ACTION_MOVE`,`ACTION_UP`事件都会交由父容器来处理了. 这个时候这个**事件序列**剩余部分无法传递给子元素了.
- `ACTION_MOVE`这个事件,就可以根据实际的需求来决定是否需要拦截. 如果需要拦截就返回true.否则false.
- `ACTION_UP`这个事件必须返回false, 因为`ACTION_UP`事件本身没有太多意义.

**内部拦截法**

> 是指父容器不拦截任何事件, 所有的事件都需要传递给子元素, 如果子元素需要此事件就直接消费. 否则就交由父容器进行处理, 由于这种方法和Android中的事件分发机制不一致, 需要配合requestDisallowInterceptTouchEvent()方式才能正常工作. 需要重写子元素的dispatchTouchEvent

这种拦截法的使用规则:

子View中的`dispatchTouchEvent()`进行复写.

- `ACTION_DOWN`事件中: 让父容器拒绝拦截所有事件, 调用`parent.requestDisallowInterceptTouchEvent(true)`
- `ACTION_MOVE`事件中: 进行条件的拦截判断, 如果在某一种场景需要拦截,那么就调用方法允许父容器拦截事件.
- `return` 时, 调用`super.dispatchTouchEvent(event)`

父容器的`onInterceptTouchEvent()`进行`ACTION_DOWN`返回false, 其余都是返回true的复写.

说明一点, 为什么父容器不连`Action_down`一并的用true复写. 因为`ACTION_DOWN`这个事件是不受`INTERCEPT_FLAG`这个标记影响的的, 就是不管拦截标记是否是何值, 按下事件必然会执行, 所以如果这里返回true, 那么就代表着, 这个事件序列的后续部分将由父容器进行处理, 而子容器无法收到这个事件.

## 实例演练

[实例演练的代码](https://github.com/suzeyu1992/Notes_AndroidDevSeek)

**实现效果:类似ViewPager中嵌套ListView的效果**

首先需要一个父容器, 这里使用作者提供的一个类`MyHorizontalScrollView`,这个类类似于`ViewPager`是继承`ViewGroup`自定义的, 支持左右滑动. 因为这个类对拦截进行了处理,所以这里把`onInterceptTouchEvent()`实现的方法注释掉让这个自定义ViewGroup走默认的分发模式. 达到一会使用可以出现滑动冲突的场景. (关于这个类的其他代码,会在下一章笔记中说明,这里不需要关心)

然后子容器是一个`TextView`+`ListView`.

**分析存在的问题**

由于`ViewGroup`默认的是不拦截事件的. 在自己的`onTouchEvent`进行了事件的处理来实现左右滑动. 但是这里有有一个前提, 那就是事件一直传递到最里层并且最里层的`View`不会把事件消费掉.这样父容器才会实现预期的效果.

但是这里放了一个`listview`, `TextView`. 运行代码看一下实际情况. 发现如下情况:

![img](http://szysky.com/2016/08/08/Android%E5%BC%80%E5%8F%91%E8%89%BA%E6%9C%AF%E6%8E%A2%E7%B4%A2-%E7%AC%94%E8%AE%B003-View%E7%9A%84%E4%BA%8B%E4%BB%B6%E4%BD%93%E7%B3%BB/outside.png)

### 外部拦截解决

*外部拦截法相关代码在仓库的outside包中*

接下来就解决这个问题. 既然打算外部拦截法,那么首先继承这个`MyHorizontalScrollView`类, 复写其中的`onInterceptTouchEvent()`进行相应的处理.

需要解决的就是在:**滑动过程中水平距离差比垂直距离差大,父容器就要拦截事件**

```
@Override
public boolean onInterceptTouchEvent(MotionEvent ev) {
boolean intercept = false;
int x = (int) ev.getX();
int y = (int) ev.getY();

   switch (ev.getAction()){
       case MotionEvent.ACTION_DOWN:
           if (!mScroller.isFinished()){
               mScroller.abortAnimation();  //优化滑动效果
               intercept = true;
           }
           break;

       case MotionEvent.ACTION_MOVE:
           int deltaX = x - mLastYIntercept;
           int deltaY = y - mLastYIntercept;
           //根据绝对值判断是否需要拦截
           if (Math.abs(deltaX) < Math.abs(deltaY)){
               intercept = false;
           }else{
               intercept = true;
           }
           break;

       case MotionEvent.ACTION_UP:
           intercept = false;
           break;

       default:
           break;
   }
   //赋值给mLast是防止在onTouchEvent第一次move移动时候跳屏
   mLastX = mLastXIntercept = x;
   mLastY = mLastYIntercept = y;
   return intercept;
}
```

### 内部拦截解决

*内部拦截法相关代码在仓库的inside包中*

上面总结过使用内部拦截法. 主要就是允许父容器在`onIntercept()`中的`MOVE和UP事件中一直返回true`表示拦截事件. 而在子容器中进行对父容器的`requestDisallowIntercept`标记的修改. 来控制事件的分发.(`DOWN`事件不能返回true,会导致子View永远收不到事件)

子容器这里`ListView`进行继承并复写`dispatchTouchEvent()`,来控制父容器的`requestDisallowIntercept`标记.

```
@Override
public boolean dispatchTouchEvent(MotionEvent ev) {
    int x = (int) ev.getX();
    int y = (int) ev.getY();
    
    switch (ev.getAction()){
      case MotionEvent.ACTION_DOWN:
          //相当于进行初始化每次发生一个事件序列时, 都对不容器进行重置不允许拦截
          mInSide2HorizontalScrollview.requestDisallowInterceptTouchEvent(true);
          break;
      case MotionEvent.ACTION_MOVE:
          int deltaX = x - mLastX;
          int deltaY = y - mLastY;
    
          if (Math.abs(deltaX) > Math.abs(deltaY)){
              //当水平距离大的时候 允许父容器拦截
              mInSide2HorizontalScrollview.requestDisallowInterceptTouchEvent(false);
          }
    
          break;
    
      default:
          break;
    
    }
    mLastX = x;
    mLastY = y;
    return super.dispatchTouchEvent(ev);
}
```

同时修改一下父容器的`onInterceptTouchEvent()`方法

```
@Override
public boolean onInterceptTouchEvent(MotionEvent ev) {
   int x = (int) ev.getX();
   int y = (int) ev.getY();

   switch (ev.getAction()){
       case MotionEvent.ACTION_DOWN:
           if (!mScroller.isFinished()){
               mScroller.abortAnimation();
               return true;
           }
           mLastX = x;
           mLastY = y;
           return false;

       default:
          return true;
   }

}
```