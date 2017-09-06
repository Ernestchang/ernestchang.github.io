title: 《Android开发艺术探索》Chap4_View工作原理
date: 

categories: 
- android 
- 《Android开发艺术探索》
tags:  
- android
---

> 拨开炫酷的外表, 看看衣服里面的View是怎样工作的
> 注：本文主要参考[http://szysky.com](http://szysky.com)

[blog相关代码](https://github.com/suzeyu1992/Notes_AndroidDevSeek)

## ViewRoot和DecorView

这是在View三大流程之前(`measure`, `layout`, `draw`),需要了解的概念.

`ViewRoot`对应于`ViewRootImpl`, 它是连接`WindowManager`和`DecorView`的纽带. View的三大流程都是通过`ViewRoot`来完成的. 当一个`Activity`对象在`ActivityThread`被创建后. 会将`DecorView`添加到`Window`中, 同时会创建`ViewRootImp`对象, 并将`ViewRootImpl`对象和`DecorView`建立关联.

View绘制流程是从ViewRoot的`PerformTraversals()`开始的. 经过三大流程才能将一个View绘制出来.

`PerformTraversals()`会依次调用`performMeasure`, `performLayout`, `performDraw`. 而前两种内部的调用基本一致,都是先调用`measure()/layout()`,然后再调用`onMeasure()/onLayout()`在这个方法中会对所有子元素进行测量和绘制.依次向内部传递. `performDraw()`有点不同是在`draw`调用的`dispatchDraw()`.

- **measure过程**: 决定了View宽高, measure后可以通过`getMeasureWidth和getMeasureHeight`来获取View的宽高. 一般情况下是最终宽高.
- **layout过程**: 决定了View的顶点坐标和实际View的宽高. 完成后通过`getTop, getBottom, getLeft, getRight`获得四个顶点, 通过`getWidth,和getHeight`获得宽高
- **draw过程**: 只有draw()方法完成之后View的内容才会显示出来.

```
setContentView(R.layout.activity_inside_intercept);
((ViewGroup) getWindow().getDecorView().findViewById(android.R.id.content)).getChildAt(0);
```

上面第一行可以说无时无刻不存在. 而下面这行在上一章说过就是获得我们设置的布局.那`DecorView`布局究竟是怎么样的, 下图.

![img](http://szysky.com/2016/08/10/%E3%80%8AAndroid%E5%BC%80%E5%8F%91%E8%89%BA%E6%9C%AF%E6%8E%A2%E7%B4%A2%E3%80%8B04-View%E7%9A%84%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86/decorView.png)

`DecorView`就是一个FrameLayout. 而一般情况下它的布局就如上面图那样(具体和主题有关系). 而我们经常`setContentView(xxx)`. 就是把我们编写的xml的布局添加到了`DecorView`的`android.R.id.content`的控件布局中. 所以也就能说通为什么`getChildAt(0)`会获得我们的的布局.
并且为什么我们用的关联布局的方法是setContent…

## MeasureSpec

> 很大程度上决定一个View的尺寸规格, 之所以不是绝对, 是因为这个过程还受父容器的影响.

### 理解MeasureSpec

`MeasureSpec`本身是一个32位的int值, 但是却表示了两种信息.

- 高2位: 代表了SpecMode, 测量模式
- 低30位: 代表了SpecSize, 在上述测量模式中的大小

```
public static class MeasureSpec {
    private static final int MODE_SHIFT = 30;
    private static final int MODE_MASK  = 0x3 << MODE_SHIFT;
    public static final int UNSPECIFIED = 0 << MODE_SHIFT;
    public static final int EXACTLY     = 1 << MODE_SHIFT;
    public static final int AT_MOST     = 2 << MODE_SHIFT;
    
    public static int makeMeasureSpec(int size, int mode) {
      if (sUseBrokenMakeMeasureSpec) {
          return size + mode;
      } else {
          return (size & ~MODE_MASK) | (mode & MODE_MASK);
      }
    }
    
    public static int makeSafeMeasureSpec(int size, int mode) {
      if (sUseZeroUnspecifiedMeasureSpec && mode == UNSPECIFIED) {
          return 0;
      }
      return makeMeasureSpec(size, mode);
    }
    
    public static int getMode(int measureSpec) {
      return (measureSpec & MODE_MASK);
    }
    
         
    public static int getSize(int measureSpec) {
      return (measureSpec & ~MODE_MASK);
   }
   .....
}
```

是不是挺有意思. 三种类型分别高二位01, 00, 10来代表. 直接利用位运算. 来实现可以让频繁计算的东西使用最接近计算机的运算方式. 不需要额外的转换. 也避免了过多的对象内存分配.

**说一下SpecMode的三种模式**

- **UNSPECIFIED**: 父容器不对View有任何的限制,要多大就给多大, 这种情况一般用于系统内部,表示一中测量状态
- **EXACTLY**: 父容器已经检测出View所需要的精确大小, 这个时候View的最终大小就是**SpecSize**所指定的值. 对应着LayoutParams中的`match_parent`和具体的数值.
- **AT_MOST**: 父容器制定了一个可用的大小及**SpecSize**, View的大小不能超过这个值, 它对应与LayoutParams中的`wrap_content`

### MeasureSpec和LayoutParams关系

通常设置的`LayoutParams`,系统会在父容器的的约束下转换成对应的`MeasureSpec`,然后根据这个`MeasureSpec`来确定View测量后的宽高. 所以View自身的`MeasureSpec`是需要`LayoutParams`和父容器一起组合生成的.

上面讲述的是普通View, 但是顶级View(DecorView)有所不同. DecorView是物理窗口尺寸和自身的`LayoutParams`决定的. 具体在`ViewRootImpl类measureHierarchy()`进行生成的.

MeasureSpec一旦确定, onMeasure中就可以测量View的宽高.

**对于我们日常操作的View**

View的`measure`过程是由ViewGroup传递而来的. 看`ViewGroup#measureChildWithMargins()`方法

```
protected void measureChildWithMargins(View child,
          int parentWidthMeasureSpec, int widthUsed,
          int parentHeightMeasureSpec, int heightUsed) {
      
      final MarginLayoutParams lp = (MarginLayoutParams) child.getLayoutParams();
      
      final int childWidthMeasureSpec = getChildMeasureSpec(parentWidthMeasureSpec,
              mPaddingLeft + mPaddingRight + lp.leftMargin + lp.rightMargin
                      + widthUsed, lp.width);
      final int childHeightMeasureSpec = getChildMeasureSpec(parentHeightMeasureSpec,
              mPaddingTop + mPaddingBottom + lp.topMargin + lp.bottomMargin
                      + heightUsed, lp.height);

      child.measure(childWidthMeasureSpec, childHeightMeasureSpec);
  }
```

上面会对子元素进行`measure`, 而在此之前,会通过`getChildMeasureSpec()`来得到子元素的`MeasureSpec`. 通过调用方法传入的参数看到. 生成View的`MeasureSpec`和父容器的`MeasureSpec`, View自身方向的`padding``margin`, 和自身的LayoutParams这三个因素相关联.

而其中的`getChildmeasureSpec()`方法: 就是根据父容器的`MeasureSpec`同时结合View自身的LayoutParams来确定子元素的`MeasureSpec`.这个方法总结如下:

- **dp/px**: 不管父容器的`MeasureSpec`是什么. View都是`EXACTLY`(精确模式), 而大小遵循自身`LayoutParams`的大小.
- **match_parent**: 如果父容器是`EXACTLY`(精确模式),那么子View也是`EXACTLY`(精确模式)并且大小是父容器的剩余空间. 如果父容器是`AT_MOST`(最大模式),那么子View也是`AT_MOST`(最大模式)并且大小不会超过父容器的剩余空间.
- **wrap_content**: 不管父容器是什么. View都是`AT_MOST`(最大模式), 并且大小不能超过父容器剩余空间.

上述没有说明`UNSPECIFIED`在`match_parent`和`wrap_content`中. 因为这个模式主要用于系统多次Measure的情形,一般来说不需要关注.

## View的工作流程

> 主要指measure, layout, draw三大流程. 即测量,布局,绘制.

### measure过程

这里面存在两种场景:

- View: 通过了`measure`方法就完成了测量过程
- ViewGroup: 除了测量自己,还会遍历去调用所有子元素的`measure`方法. 各个子元素在递归去执行这个流程

**View的measure过程**

View的**measure**过程由其`measure()`方法来完成, `measure()`方法是一个final类型, 而在内部调用了`onMeasure()`这个可不是final, 所以也可以自定义的时候复写. 看一下内部.

```
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
   setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
           getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}
```

`setMeasureDimension()`会设置View宽高的测量值.

这里需要看一下`getDefaultSize()`这个方法.

```
public static int getDefaultSize(int size, int measureSpec) {
   int result = size;
   int specMode = MeasureSpec.getMode(measureSpec);
   int specSize = MeasureSpec.getSize(measureSpec);

   switch (specMode) {
   case MeasureSpec.UNSPECIFIED:
       result = size;
       break;
   case MeasureSpec.AT_MOST:
   case MeasureSpec.EXACTLY:
       result = specSize;
       break;
   }
   return result;
}
```

看到如果这个view是`EXACTLY`(精准模式), 那么返回的大小就是SpecSize. `UNSPECIFIED`一般用于系统测量先不说. 而`AT_MOST`(最大模式)的时候. 虽然是不同模式但是默认情况下和精确模式是一样的结果.

`getSuggestedMinimumWidth()`和`getSuggestedMinimumHeight()`. 看一下实现.

```
protected int getSuggestedMinimumWidth() {
    return (mBackground == null) ? mMinWidth : max(mMinWidth, mBackground.getMinimumWidth());
}
protected int getSuggestedMinimumHeight() {
    return (mBackground == null) ? mMinHeight : max(mMinHeight, mBackground.getMinimumHeight());

}
```

首先会看是否设置了背景.

- **无背景:** 那么宽度为`mMinWidth`,这个值对应布局中的`android:minWidth`属性,默认为0.
- **有背景:** 那么取`mMinWidth`和`mBackground.getMinimumHeight()`最大值.

而`getMinimumHeight()`根据看一下:

```
public int getMinimumHeight() {
   final int intrinsicHeight = getIntrinsicHeight();
   return intrinsicHeight > 0 ? intrinsicHeight : 0;
}
```

原来`getMinimumHeight()`返回的就是`Drawable`的原始高度. 如果没有就返回0. 关于**原始高度**举个例子`ShapeDrawable`无原始宽高, `BitmapDrawble`有原始宽高就是图片的尺寸.

**整理getDefaultSize()**: 直接继承View的自定义控件需要重写`onMeasure()`方法并设置`wrap_content`时的自身大小,否则在布局中使用`wrap_content`虽然View自身的`MeasureSpec`的低30位保存了父容器计算自身的剩余大小. 但是在**自定义的时候如果不进行处理wrap_content,那么就会调用默认setMeasureDimension()方法. 而默认中方法的实参传递的是getDefaultSize()这个方法中对AT_MOST这种模式没有处理. 直接沿用和精确模式的大小(相当于设置了wrap_content却得到了match_parent的显示结果)**

可以针对这个问题, 做出对应的编码进行解决:

```
@Override
   protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
       super.onMeasure(widthMeasureSpec, heightMeasureSpec);

       int widthSpaceSize = MeasureSpec.getSize(widthMeasureSpec);
       int widthSpaceMode = MeasureSpec.getMode(widthMeasureSpec);
       int heightSpaceSize = MeasureSpec.getSize(heightMeasureSpec);
       int heightSpaceMode = MeasureSpec.getMode(heightMeasureSpec);
       
       //设置两个默认值宽高
       int defaultHeight = 100;
       int defaultWidth = 100;
       
       
       // 针对AT_MOST模式进行特殊处理
       if (widthSpaceMode == MeasureSpec.AT_MOST 
               && heightSpaceMode == MeasureSpec.AT_MOST){
           
           setMeasuredDimension(defaultWidth, defaultHeight);
           
       }else if (widthSpaceMode == MeasureSpec.AT_MOST){
           
           setMeasuredDimension(defaultWidth, heightSpaceSize);
           
       }else if (heightSpaceMode == MeasureSpec.AT_MOST){
           
           setMeasuredDimension(widthMeasureSpec, defaultHeight);
           
       }
   }
```

**ViewGroup的Measure**

> 对于`ViewGroup`不光会测量自己,还会遍历调用所有的子元素的`measure()`. 和`View`不同的是`ViewGroup`是一个抽象类,它没有重写`onMeasure`,但提供了`measureChildren()`的方法.

这个`measureChildren()`方法内部比较简单就是遍历自己的孩子然后调用->`measureChild()`

这个`measureChild()`这个方法前面贴过源码. 就是取出子元素的`LayoutParams`,并调用->`getChildMeasureSpec()`. 通过传入**子元素的LayoutParams里面的宽高属性, 子元素的padding和margin, 父元素当前(当前ViewGroup)的MeasureSpec属性**来计算出子元素的`MeasureSpec`最后调用->`child.measure()`传入之前计算的测量规格.

**ViewGroup为什么没有定义测量的具体过程?** 因为具体的测量过程需要交给子类去实现的. 比如`LinearLayout`,`RelativeLayout`.

看一下`LinearLayout`的`onMeasure()`是如何定义的.

```
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
   if (mOrientation == VERTICAL) {
       measureVertical(widthMeasureSpec, heightMeasureSpec);
   } else {
       measureHorizontal(widthMeasureSpec, heightMeasureSpec);
   }
}
```

根据设置的排列方式这里分之了两种测量方法. 稍微看一下大概轮廓,选择`measureVertical()`不贴源码了这个方法300行呢!

首先这个方法会遍历每个子元素并执行->`measureChildBeforeLayout()`方法.这个方法内部会调用子元素的`measure()`, 这样子元素会依次测量. 并且会通过`mTotalLenght`这个变量来存储LinearLayout在竖直方向上的初步高度, 每测量一个就会增加. 当子元素测量完之后,LinearLayout会测量自己的大小.

------

在对自己进行测量的时候. 如果布局中的高度采用的是`match_parent`或者`具体数值`, 那么它的测量过程和View一样,即高度为`specSize`. 如果布局中采用`wrap_content`那么高度就是所有的子元素总和但是不能超过父元素剩余空间, 还有竖直方向LinearLayout的`padding`. 具体可参考`resolveSizeAndState()`的实现.

到这里基本上`measure`测量过程已经做了比较详细的分析. 这个过程也是三大过程中最复杂的一个. 在`measure`完成之后就可以通过`getMeasuredWidth/Height`方法获取View的测量宽高. **但是请注意**:某些极端情况下,measure可能执行多次. 所以尽量在`onLayout()`方法中去获得最终宽高.

### 正确获取宽高方法

**首先明确一点:View的measure和Activity的生命周期方法不是同步执行.所以无法保证在某个生命周期(onCreate,onStart)获取到正确的测量宽高**

- onWindowFocusChanged()
- view.post(runnable)
- ViewTreeObserve
- view.measure()

1. `onWindowFocusChanged()`:View已经初始化完毕,宽高已经准备好. 这里需要注意只要Activity的焦点发生变化此方法就会被调用.所以如果你的界面会频繁的进行`onPause`和`onResume`.并且里面有很多关联依赖的方法. 那就请注意这不是一个好办法.
2. 通过`post`可以将一个runnable投递到消息队列的尾部,然后等待`Looper`调用此runnable的时候.View已经初始化完毕.
3. 使用`ViewTreeObserver`. 当View的可见性发生了改变的时候.`onGlobalLayout()`将发生回调.注意伴随着View树的状态改变等,这个回调方法可能会被调用多次. 使用代码如下

```
ViewTreeObserver viewTreeObserver = tv_main.getViewTreeObserver();
       viewTreeObserver.addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
           @Override
           public void onGlobalLayout() {
               tv_main.getViewTreeObserver().removeOnGlobalLayoutListener(this);
               tv_main.getMeasuredHeight();
               tv_main.getMeasuredWidth();
           }
       });
```

1. view.measure(widthMeasureSpec, heightMeasureSpec)

也可以手动进行测量,但是需要分情况处理.

**match_parent**

当View是此属性的时候无法使用`measure()`,首先使用这种方法需要的参数,是通过父容器和子元素组合来生成的子元素的`MeasureSpec`属性. 所以在外部我们不知道父元素的参数值得时候只能处理**不需要父元素数据就可以生成子元素的MeasureSpec的模式**

所以很清楚, 这个`match_patch`这个模式,在给其子元素构造`MeasureSpec`的时候需要得值`parentSize`,所以得到的也是无效.

**具体数值px/dx**

假设这里是100px, 首先构成宽高对应的`MeasureSpec`属性

```
int widthSpec = View.MeasureSpec.makeMeasureSpec(100, View.MeasureSpec.EXACTLY);
int heightSpec = View.MeasureSpec.makeMeasureSpec(100, View.MeasureSpec.EXACTLY);
       tv_main.measure(widthSpec, heightSpec);
```

**wrap_content**

```
int widthSpec = View.MeasureSpec.makeMeasureSpec(((1 << 30)-1), View.MeasureSpec.AT_MOST);
int heightSpec = View.MeasureSpec.makeMeasureSpec(((1 << 30)-1), View.MeasureSpec.AT_MOST);
    tv_main.measure(widthSpec, heightSpec);
```

通过(1<<30)-1 可以构成一个`MeasureSpec`低30位的最大值. 用理论上View能支持的最大值去构造

**关于网上一些在make的使用传入UNSPECIFIED,属于违背了内部实现的规范.不用最好**

**关于网上另一种measure()直接传入LayoutParams.WRAP_CONTENT. 其实也只有当子元素为wrap_content和子元素为match_parent并且父元素是wrap_conetnt时会碰巧有效.**

### layout过程

在`ViewGroup`中会先通过`layout()`方法确定本身的位置. 然后调用`onLayout()`方法遍历所有的子元素,并调用子元素的`layout()`方法确定子元素的位置…依次循环.

提出`View`的`layout`方法, 这里抽取部分代码

```
public void layout(int l, int t, int r, int b) {
       int oldL = mLeft;
       int oldT = mTop;
       int oldB = mBottom;
       int oldR = mRight;

       boolean changed = isLayoutModeOptical(mParent) ?
               setOpticalFrame(l, t, r, b) : setFrame(l, t, r, b);

       if (changed || (mPrivateFlags & PFLAG_LAYOUT_REQUIRED) == PFLAG_LAYOUT_REQUIRED) 
       { onLayout(changed, l, t, r, b);}

   }
```

这样来看,大致流程通过`setFrame()`方法来设定View的四个顶点的位置, 即**mLeft,mTop,mBottom,mRight**,这四个顶点一旦确定.当前View的位置也就确定. 然后会调用`onLayout()`方法. 这个方法是确定子元素的View位置.

这里的和`onMeasure()`类似, `onLayout()`具体实现和具体的布局有关, 所以View和ViewGroup均没有真正实现`onLayout()`方法.

看一下`LinearLayout`的`onLayout()`源码

```
@Override
protected void onLayout(boolean changed, int l, int t, int r, int b) {
   if (mOrientation == VERTICAL) {
       layoutVertical(l, t, r, b);
   } else {
       layoutHorizontal(l, t, r, b);
   }
}
```

和`onMeasure()`一样分支,接下来跟进`layoutVertical()`贴出主要代码

```
void layoutVertical(int left, int top, int right, int bottom) {
           //省略一部分...
       for (int i = 0; i < count; i++) {
           final View child = getVirtualChildAt(i);
           if (child == null) {
               childTop += measureNullChild(i);
           } else if (child.getVisibility() != GONE) {
               final int childWidth = child.getMeasuredWidth();
               final int childHeight = child.getMeasuredHeight();
               
               final LinearLayout.LayoutParams lp =
                       (LinearLayout.LayoutParams) child.getLayoutParams();
               
               int gravity = lp.gravity;
               if (gravity < 0) {
                   gravity = minorGravity;
               }
                //省略一部分...

               if (hasDividerBeforeChildAt(i)) {
                   childTop += mDividerHeight;
               }

               childTop += lp.topMargin;
               setChildFrame(child, childLeft, childTop + getLocationOffset(child),
                       childWidth, childHeight);
               childTop += childHeight + lp.bottomMargin + getNextLocationOffset(child);

               i += getChildrenSkipCount(child, i);
           }
       }
   }
```

上面代码大体逻辑: 首先遍历所有孩子并调用`setChildFrame()`来为子元素指定对应的位置. 其中`childTop`会逐渐增大, 这就意味着后面的子元素会被放置在靠下的位置. 而`setChildFrame()`内部仅有一行代码, 就是调用子元素的`layout()`并传入它自身应该存放的位置.

```
private void setChildFrame(View child, int left, int top, int width, int height) {        
     child.layout(left, top, left + width, top + height);
 }
```

而在`setChildFrame()`中传入的宽高就是子元素的测量宽高.

而在子元素的`layout()`中通过`setFrame()`来设置元素的四个顶点.

**getWidth()layout中的宽 和getMeasureWidth()中的宽永远一样么?**

在一般情况下,测量measure和layout时候的值是完全一样的. 因为`layout()`中接受的参数就是通过测量的结果获取到的. 并且内部直接通过`setFrame()`赋值到自己的四个成员变量上. 但是如果对`layout()`进行了复写.如下

```
 @Override
protected void layout(int l, int t, int r, int b) {
   super.layout( l,  t+200,  r,  b+200);
}
```

如果进行了这样的复写, 那么**最终宽高**永远会与**测量的**出来的值相差200.

### draw过程

这个过程只是将View绘制到屏幕上面.

1. 绘制背景`background.draw(canvas)`
2. 绘制自己`onDraw()`
3. 绘制children`dispatchDraw()`
4. 绘制装饰`onDrawScrollBars()`

View绘制过程传递是通过`dispatchDraw()`实现的. 传递了自己的画布. 这个方法会遍历子元素并且调用元素的`draw()`

View一个特有的方法`setWillNotDraw()`, 这个方法是设置了`true`那么系统会进行相应的优化. 在View中默认是关闭的. 而ViewGroup默认是开启的. 如果我们继承了自定义ViewGroup如果还需要绘制自己的内容那么需要显示的关闭此标记.

## 自定义View

### 自定义View的分类

[相关代码](https://github.com/suzeyu1992/Notes_AndroidDevSeek)

**1.继承View重写onDraw方法**

这种方法主要用于实现一些不规则的效果, 不方便组合布局实现,或者又有动态显示的一些图形. 需要自己绘制那么就重写`onDraw()`方法. **这种方法需要自己支持wrap_content和padding**

**2.继承ViewGroup派生特殊Layout**

这种方式用于实现自定义布局, 这种布局的实现稍微复杂,需要合适的处理ViewGroup的测量,布局这两个过程,并同时处理子元素的测量和布局过程.

**3.继承特定的View(TextView)**

比较常见, 一般用于扩展已有的View的功能. 这种不需要自己处理wrap_content和padding

**4.继承特定ViewGroup(LinearLayout)**

当某种效果看起来像几种View的组合在一起的时候,可以采用这种方式. 这种方式不需要自己处理ViewGroup的测量和布局. 其实这种方式和2没什么区别, 主要是2更接近于底层的View实现.

### 自定义View的须知

1. 让View支持`wrap_content`
2. 最好让你的View支持`padding` -> 如果直接继承View,在`draw()`中不处理padding,那么属性是无法起作用的. 还有继承ViewGroup的控件需要在`onMeasure`和`onLayout`中考虑`padding`和子元素的`margin`会造成的影响.
3. 尽量不要在View中使用`Handler` -> 内部已经提供了post系列方法. 除非很明确要是用Handler发送消息.
4. View中如果有线程或者动画,需要及时的停止.-> 当包含此View的Activity退出或者此View被remove的时候,View的`onDetachedFromWindow()`会被调用,可以适当处理防止内存泄漏.
5. View带有的滑动嵌套时,需要处理好滑动冲突.

### 自定义View实例

**1.自定义View派生类**

首先写一个类继承`View`, 并在`ondraw()`画一个圆. 并设置`margin`属性. 效果没有问题,因为`margin`属性是由父容器控制的.

![img](http://szysky.com/2016/08/10/%E3%80%8AAndroid%E5%BC%80%E5%8F%91%E8%89%BA%E6%9C%AF%E6%8E%A2%E7%B4%A2%E3%80%8B04-View%E7%9A%84%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86/cusview01.png)

**问题1:**这里把`android:layout_width="wrap_content"`设置根据内容. 发现无效.

**不管是设置match_parent也好,wrap_content也好父容器都会给分配自己剩余空间的大小给子容器作为specSize的空间大小**这时需要手动处理. 因为不处理那就是相当于和`match_parent`填充父容器的效果一样.

所以增添`对onMeasure()`方法中的`AT_MOST`模式的制定默认大小, 然后在运行, ok,如下

![img](http://szysky.com/2016/08/10/%E3%80%8AAndroid%E5%BC%80%E5%8F%91%E8%89%BA%E6%9C%AF%E6%8E%A2%E7%B4%A2%E3%80%8B04-View%E7%9A%84%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86/cusview02.png)

**问题2:**这时加上属性`padding=20dp`发现无效. 无变化. 之前说过`margin`是交给父容器分配的.`padding`确实要自己要分配处理的. 这时需要在`onDraw()`来处理. 处理后如下

![img](http://szysky.com/2016/08/10/%E3%80%8AAndroid%E5%BC%80%E5%8F%91%E8%89%BA%E6%9C%AF%E6%8E%A2%E7%B4%A2%E3%80%8B04-View%E7%9A%84%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86/cusview03.png)

**问题3:**有时候我们需要提供自定义属性. 例如`android:id=`这种. 接下来添加自定义属性.

1. 在`values`目录下创建自定义属性的xml. 名字随便当最好`attrs.xml`或者`attrs_xxx_xxx.xml`.

```
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="CircleView">
        <attr name="circle_color" format="color"/>
    </declare-styleable>
</resources>
```

上面相当于,定义了一个`CircleView`的属性集合. 在这个集合里面会有自定义属性. 这里的`format`格式可以是指定尺寸的`dimension`, 资源id引用的`reference`, 基本类型`string, integer ,boolean`等.

1. 声明好了属性在我们自定义View中就可以引用处理了. 如构造方法中.

```
public CircleView(Context context, AttributeSet attrs) {
       super(context, attrs, defStyleAttr);
       //获得一个自定义的对应属性值集合
       TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.CircleView);
       //取出属性集合中的某个属性值
       mColor = typedArray.getColor(R.styleable.CircleView_circle_color, Color.GREEN);
       //释放资源
       typedArray.recycle();
       init();

   }
```

1. 在布局中使用即可.

先声明schemas. `xmlns:app="http://schemas.android.com/apk/res-auto"` ,使用`app`来替代之前的类似`android`前缀的引导.

继承View的派生类就到此为止了.

------

**继承ViewGroup就不说了, 可以看第三章中的仓库代码的那个自定义View大致实现ViewPager的滑动**