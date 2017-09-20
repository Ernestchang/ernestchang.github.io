title: Glide解析5_图片变换
date: 

categories: 
- android
- github

tags: 
- github
- glide

---

> 本文出自[郭霖博客](http://blog.csdn.net/guolin_blog/article/details/71524668)

大家好，又到了学习Glide的时间了。前段时间由于项目开发紧张，再加上后来又生病了，所以停更了一个月，不过现在终于又可以恢复正常更新了。今天是这个系列的第五篇文章，在前面四篇文章的当中，我们已经学习了Glide的基本用法、Glide的工作原理和执行流程、Glide的缓存机制、以及Glide的回调机制等内容。如果你能将前面的四篇文章都掌握好了，那么恭喜你，现在你已经是一名Glide好手了。

如果你还没有阅读过前面四篇文章的话，那么可以点击后面的链接，依次向前阅读 [Android图片加载框架最全解析（四），玩转Glide的回调与监听](http://blog.csdn.net/guolin_blog/article/details/70215985)。

不过Glide的这个框架的功能实在是太强大了，它所能做的事情远远不止于目前我们所学的这些。因此，今天我们就再来学习一个新的功能模块，并且是一个非常重要的模块——Glide的图片变化功能。

# 一个问题

在正式开始学习Glide的图片变化功能之前，我们先来看一个问题，这个问题可能有不少人都在使用Glide的时候都遇到过，正好在本篇内容的主题之下我们顺带着将这个问题给解决了。

首先我们尝试使用Glide来加载一张图片，图片URL地址是：

```
https://www.baidu.com/img/bd_logo1.png1
```

这是百度首页logo的一张图片，图片尺寸是540*258像素。

接下来我们编写一个非常简单的布局文件，如下所示：

```
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Load Image"
        android:onClick="loadImage"
        />

    <ImageView
        android:id="@+id/image_view"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        />
</LinearLayout>
```

布局文件中只有一个按钮和一个用于显示图片的ImageView。注意，ImageView的宽和高这里设置的都是wrap_content。

然后编写如下的代码来加载图片：

```
public class MainActivity extends AppCompatActivity {

    ImageView imageView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        imageView = (ImageView) findViewById(R.id.image_view);
    }

    public void loadImage(View view) {
        String url = "https://www.baidu.com/img/bd_logo1.png";
        Glide.with(this)
             .load(url)
             .into(imageView);
    }
}
```

这些简单的代码对于现在的你而言应该都是小儿科了，相信我也不用再做什么解释。现在运行一下程序并点击加载图片按钮，效果如下图所示。

![img](http://img.blog.csdn.net/20170809205829589?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

图片是正常加载出来了，不过大家有没有发现一个问题。百度这张logo图片的尺寸只有540*258像素，但是我的手机的分辨率却是1080*1920像素，而我们将ImageView的宽高设置的都是wrap_content，那么图片的宽度应该只有手机屏幕宽度的一半而已，但是这里却充满了全屏，这是为什么呢？

如果你之前也被这个问题困扰过，那么恭喜，本篇文章正是你所需要的。之所以会出现这个现象，就是因为Glide的图片变换功能所导致的。那么接下来我们会先分析如何解决这个问题，然后再深入学习Glide图片变化的更多功能。

稍微对Android有点了解的人应该都知道ImageView有scaleType这个属性，但是可能大多数人却不知道，如果在没有指定scaleType属性的情况下，ImageView默认的scaleType是什么？

这个问题如果直接问我，我也答不上来。不过动手才是检验真理的唯一标准，想知道答案，自己动手试一下就知道了。

```
public class MainActivity extends AppCompatActivity {

    private static final String TAG = "MainActivity";

    ImageView imageView;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        imageView = (ImageView) findViewById(R.id.image_view);
        Log.d(TAG, "imageView scaleType is " + imageView.getScaleType());
    }

    ...
}
```

可以看到，我们在onCreate()方法中打印了ImageView默认的scaleType，然后重新运行一下程序，结果如下图所示：

![img](http://img.blog.csdn.net/20170809211645334)

由此我们可以得知，在没有明确指定的情况下，ImageView默认的scaleType是FIT_CENTER。

有了这个前提条件，我们就可以继续去分析Glide的源码了。当然，本文中的源码还是建在第二篇源码分析的基础之上，还没有看过这篇文章的朋友，建议先去阅读 [Android图片加载框架最全解析（二），从源码的角度理解Glide的执行流程](http://blog.csdn.net/guolin_blog/article/details/53939176) 。

回顾一下第二篇文章中我们分析过的into()方法，它是在GenericRequestBuilder类当中的，代码如下所示：

```
public Target<TranscodeType> into(ImageView view) {
    Util.assertMainThread();
    if (view == null) {
        throw new IllegalArgumentException("You must pass in a non null View");
    }
    if (!isTransformationSet && view.getScaleType() != null) {
        switch (view.getScaleType()) {
            case CENTER_CROP:
                applyCenterCrop();
                break;
            case FIT_CENTER:
            case FIT_START:
            case FIT_END:
                applyFitCenter();
                break;
            //$CASES-OMITTED$
            default:
                // Do nothing.
        }
    }
    return into(glide.buildImageViewTarget(view, transcodeClass));
}
```

还记得我们当初分析这段代码的时候，直接跳过前面的所有代码，直奔最后一行。因为那个时候我们的主要任务是分析Glide的主线执行流程，而不去仔细阅读它的细节，但是现在我们是时候应该阅读一下细节了。

可以看到，这里在第7行会进行一个switch判断，如果ImageView的scaleType是CENTER_CROP，则会去调用applyCenterCrop()方法，如果scaleType是FIT_CENTER、FIT_START或FIT_END，则会去调用applyFitCenter()方法。这里的applyCenterCrop()和applyFitCenter()方法其实就是向Glide的加载流程中添加了一个图片变换操作，具体的源码我们就不跟进去看了。

那么现在我们就基本清楚了，由于ImageView默认的scaleType是FIT_CENTER，因此会自动添加一个FitCenter的图片变换，而在这个图片变换过程中做了某些操作，导致图片充满了全屏。

那么我们该如何解决这个问题呢？最直白的一种办法就是看着源码来改。当ImageView的scaleType是CENTER_CROP、FIT_CENTER、FIT_START或FIT_END时不是会自动添加一个图片变换操作吗？那我们把scaleType改成其他值不就可以了。ImageView的scaleType可选值还有CENTER、CENTER_INSIDE、FIT_XY等。这当然是一种解决方案，不过只能说是一种比较笨的解决方案，因为我们为了解决这个问题而去改动了ImageView原有的scaleType，那如果你真的需要ImageView的scaleType为CENTER_CROP或FIT_CENTER时可能就傻眼了。

上面只是我们通过分析源码得到的一种解决方案，并不推荐大家使用。实际上，Glide给我们提供了专门的API来添加和取消图片变换，想要解决这个问题只需要使用如下代码即可：

```
Glide.with(this)
     .load(url)
     .dontTransform()
     .into(imageView);
```

可以看到，这里调用了一个dontTransform()方法，表示让Glide在加载图片的过程中不进行图片变换，这样刚才调用的applyCenterCrop()、applyFitCenter()就统统无效了。

现在我们重新运行一下代码，效果如下图所示：

![img](http://img.blog.csdn.net/20170810213840362?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这样图片就只会占据半个屏幕的宽度了，说明我们的代码奏效了。

但是使用dontTransform()方法存在着一个问题，就是调用这个方法之后，所有的图片变换操作就全部失效了，那如果我有一些图片变换操作是必须要执行的该怎么办呢？不用担心，总归是有办法的，这种情况下我们只需要借助override()方法强制将图片尺寸指定成原始大小就可以了，代码如下所示：

```
Glide.with(this)
     .load(url)
     .override(Target.SIZE_ORIGINAL, Target.SIZE_ORIGINAL)
     .into(imageView);
```

通过override()方法将图片的宽和高都指定成Target.SIZE_ORIGINAL，问题同样被解决了。程序的最终运行结果和上图是完全一样的，我就不再重新截图了。

由此我们可以看出，之所以会出现这个问题，和Glide的图片变换功能是撇不开关系的。那么也是通过这个问题，我们对Glide的图片变换有了一个最基本的认识。接下来，就让我们正式开始进入本篇文章的正题吧。

# 图片变换的基本用法

顾名思义，图片变换的意思就是说，Glide从加载了原始图片到最终展示给用户之前，又进行了一些变换处理，从而能够实现一些更加丰富的图片效果，如图片圆角化、圆形化、模糊化等等。

添加图片变换的用法非常简单，我们只需要调用transform()方法，并将想要执行的图片变换操作作为参数传入transform()方法即可，如下所示：

```
Glide.with(this)
     .load(url)
     .transform(...)
     .into(imageView);
```

至于具体要进行什么样的图片变换操作，这个通常都是需要我们自己来写的。不过Glide已经内置了两种图片变换操作，我们可以直接拿来使用，一个是CenterCrop，一个是FitCenter。

但这两种内置的图片变换操作其实都不需要使用transform()方法，Glide为了方便我们使用直接提供了现成的API：

```
Glide.with(this)
     .load(url)
     .centerCrop()
     .into(imageView);

Glide.with(this)
     .load(url)
     .fitCenter()
     .into(imageView);
```

当然，centerCrop()和fitCenter()方法其实也只是对transform()方法进行了一层封装而已，它们背后的源码仍然还是借助transform()方法来实现的，如下所示：

```
public class DrawableRequestBuilder<ModelType>
        extends GenericRequestBuilder<ModelType, ImageVideoWrapper, GifBitmapWrapper, GlideDrawable>
        implements BitmapOptions, DrawableOptions {
    ...

    /**
     * Transform {@link GlideDrawable}s using {@link com.bumptech.glide.load.resource.bitmap.CenterCrop}.
     *
     * @see #fitCenter()
     * @see #transform(BitmapTransformation...)
     * @see #bitmapTransform(Transformation[])
     * @see #transform(Transformation[])
     *
     * @return This request builder.
     */
    @SuppressWarnings("unchecked")
    public DrawableRequestBuilder<ModelType> centerCrop() {
        return transform(glide.getDrawableCenterCrop());
    }

    /**
     * Transform {@link GlideDrawable}s using {@link com.bumptech.glide.load.resource.bitmap.FitCenter}.
     *
     * @see #centerCrop()
     * @see #transform(BitmapTransformation...)
     * @see #bitmapTransform(Transformation[])
     * @see #transform(Transformation[])
     *
     * @return This request builder.
     */
    @SuppressWarnings("unchecked")
    public DrawableRequestBuilder<ModelType> fitCenter() {
        return transform(glide.getDrawableFitCenter());
    }

    ...
}
```

那么这两种内置的图片变换操作到底能实现什么样的效果呢？FitCenter的效果其实刚才我们已经见识过了，就是会将图片按照原始的长宽比充满全屏。那么CenterCrop又是什么样的效果呢？我们来动手试一下就知道了。

为了让效果更加明显，这里我就不使用百度首页的Logo图了，而是换成必应首页的一张美图。在不应用任何图片变换的情况下，使用Glide加载必应这张图片效果如下所示。

![img](http://img.blog.csdn.net/20170816204655787?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

现在我们添加一个CenterCrop的图片变换操作，代码如下：

```
String url = "http://cn.bing.com/az/hprichbg/rb/AvalancheCreek_ROW11173354624_1920x1080.jpg";
Glide.with(this)
     .load(url)
     .centerCrop()
     .into(imageView);
```

重新运行一下程序并点击加载图片按钮，效果如下图所示。

![img](http://img.blog.csdn.net/20170816205304104?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

可以看到，现在展示的图片是对原图的中心区域进行裁剪后得到的图片。

另外，centerCrop()方法还可以配合override()方法来实现更加丰富的效果，比如指定图片裁剪的比例：

```
String url = "http://cn.bing.com/az/hprichbg/rb/AvalancheCreek_ROW11173354624_1920x1080.jpg";
Glide.with(this)
     .load(url)
     .override(500, 500)
     .centerCrop()
     .into(imageView);
```

可以看到，这里我们将图片的尺寸设定为500*500像素，那么裁剪的比例也就变成1：1了，现在重新运行一下程序，效果如下图所示。

![img](http://img.blog.csdn.net/20170816211113917?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这样我们就把Glide内置的图片变换接口的用法都掌握了。不过不得不说，Glide内置的图片变换接口功能十分单一且有限，完全没有办法满足我们平时的开发需求。因此，掌握自定义图片变换功能就显得尤为重要了。

不过，在正式开始学习自定义图片变换功能之前，我们先来探究一下CenterCrop这种图片变换的源码，理解了它的源码我们再来进行自定义图片变换就能更加得心应手了。

# 源码分析

那么就话不多说，我们直接打开CenterCrop类来看一下它的源码吧，如下所示：

```
public class CenterCrop extends BitmapTransformation {

    public CenterCrop(Context context) {
        super(context);
    }

    public CenterCrop(BitmapPool bitmapPool) {
        super(bitmapPool);
    }

    // Bitmap doesn't implement equals, so == and .equals are equivalent here.
    @SuppressWarnings("PMD.CompareObjectsWithEquals")
    @Override
    protected Bitmap transform(BitmapPool pool, Bitmap toTransform, int outWidth, int outHeight) {
        final Bitmap toReuse = pool.get(outWidth, outHeight, toTransform.getConfig() != null
                ? toTransform.getConfig() : Bitmap.Config.ARGB_8888);
        Bitmap transformed = TransformationUtils.centerCrop(toReuse, toTransform, outWidth, outHeight);
        if (toReuse != null && toReuse != transformed && !pool.put(toReuse)) {
            toReuse.recycle();
        }
        return transformed;
    }

    @Override
    public String getId() {
        return "CenterCrop.com.bumptech.glide.load.resource.bitmap";
    }
}
```

这段代码并不长，但是我还是要划下重点，这样大家看起来的时候会更加轻松。

首先，CenterCrop是继承自BitmapTransformation的，这个是重中之重，因为整个图片变换功能都是建立在这个继承结构基础上的。

接下来CenterCrop中最重要的就是transform()方法，其他的方法我们可以暂时忽略。transform()方法中有四个参数，每一个都很重要，我们来一一解读下。第一个参数pool，这个是Glide中的一个Bitmap缓存池，用于对Bitmap对象进行重用，否则每次图片变换都重新创建Bitmap对象将会非常消耗内存。第二个参数toTransform，这个是原始图片的Bitmap对象，我们就是要对它来进行图片变换。第三和第四个参数比较简单，分别代表图片变换后的宽度和高度，其实也就是override()方法中传入的宽和高的值了。

下面我们来看一下transform()方法的细节，首先第一行就从Bitmap缓存池中尝试获取一个可重用的Bitmap对象，然后把这个对象连同toTransform、outWidth、outHeight参数一起传入到了TransformationUtils.centerCrop()方法当中。那么我们就跟进去来看一下这个方法的源码，如下所示：

```
public final class TransformationUtils {
    ...

    public static Bitmap centerCrop(Bitmap recycled, Bitmap toCrop, int width, int height) {
        if (toCrop == null) {
            return null;
        } else if (toCrop.getWidth() == width && toCrop.getHeight() == height) {
            return toCrop;
        }
        // From ImageView/Bitmap.createScaledBitmap.
        final float scale;
        float dx = 0, dy = 0;
        Matrix m = new Matrix();
        if (toCrop.getWidth() * height > width * toCrop.getHeight()) {
            scale = (float) height / (float) toCrop.getHeight();
            dx = (width - toCrop.getWidth() * scale) * 0.5f;
        } else {
            scale = (float) width / (float) toCrop.getWidth();
            dy = (height - toCrop.getHeight() * scale) * 0.5f;
        }
        m.setScale(scale, scale);
        m.postTranslate((int) (dx + 0.5f), (int) (dy + 0.5f));

        final Bitmap result;
        if (recycled != null) {
            result = recycled;
        } else {
            result = Bitmap.createBitmap(width, height, getSafeConfig(toCrop));
        }

        // We don't add or remove alpha, so keep the alpha setting of the Bitmap we were given.
        TransformationUtils.setAlpha(toCrop, result);

        Canvas canvas = new Canvas(result);
        Paint paint = new Paint(PAINT_FLAGS);
        canvas.drawBitmap(toCrop, m, paint);
        return result;
    }

    ...
}
```

这段代码就是整个图片变换功能的核心代码了。可以看到，第5-9行主要是先做了一些校验，如果原图为空，或者原图的尺寸和目标裁剪尺寸相同，那么就放弃裁剪。接下来第11-22行是通过数学计算来算出画布的缩放的比例以及偏移值。第24-29行是判断缓存池中取出的Bitmap对象是否为空，如果不为空就可以直接使用，如果为空则要创建一个新的Bitmap对象。第32行是将原图Bitmap对象的alpha值复制到裁剪Bitmap对象上面。最后第34-37行是裁剪Bitmap对象进行绘制，并将最终的结果进行返回。全部的逻辑就是这样，总体来说还是比较简单的，可能也就是数学计算那边需要稍微动下脑筋。

那么现在得到了裁剪后的Bitmap对象，我们再回到CenterCrop当中，你会看到，在最终返回这个Bitmap对象之前，还会尝试将复用的Bitmap对象重新放回到缓存池当中，以便下次继续使用。

好的，这样我们就将CenterCrop图片变换的工作原理完整地分析了一遍，FitCenter的源码也是基本类似的，这里就不再重复分析了。了解了这些内容之后，接下来我们就可以开始学习自定义图片变换功能了。

# 自定义图片变换

Glide给我们定制好了一个图片变换的框架，大致的流程是我们可以获取到原始的图片，然后对图片进行变换，再将变换完成后的图片返回给Glide，最终由Glide将图片显示出来。理论上，在对图片进行变换这个步骤中我们可以进行任何的操作，你想对图片怎么样都可以。包括圆角化、圆形化、黑白化、模糊化等等，甚至你将原图片完全替换成另外一张图都是可以的。

但是这里显然我不可能向大家演示所有图片变换的可能，图片变换的可能性也是无限的。因此这里我们就选择一种常用的图片变换效果来进行自定义吧——对图片进行圆形化变换。

图片圆形化的功能现在在手机应用中非常常见，比如手机QQ就会将用户的头像进行圆形化变换，从而使得界面变得更加好看。

自定义图片变换功能的实现逻辑比较固定，我们刚才看过CenterCrop的源码之后，相信你已经基本了解整个自定义的过程了。其实就是自定义一个类让它继承自BitmapTransformation ，然后重写transform()方法，并在这里去实现具体的图片变换逻辑就可以了。一个空的图片变换实现大概如下所示：

```
public class CircleCrop extends BitmapTransformation {

    public CircleCrop(Context context) {
        super(context);
    }

    public CircleCrop(BitmapPool bitmapPool) {
        super(bitmapPool);
    }

    @Override
    public String getId() {
        return "com.example.glidetest.CircleCrop";
    }

    @Override
    protected Bitmap transform(BitmapPool pool, Bitmap toTransform, int outWidth, int outHeight) {
        return null;
    }
}
```

这里有一点需要注意，就是getId()方法中要求返回一个唯一的字符串来作为id，以和其他的图片变换做区分。通常情况下，我们直接返回当前类的完整类名就可以了。

另外，这里我们选择继承BitmapTransformation还有一个限制，就是只能对静态图进行图片变换。当然，这已经足够覆盖日常95%以上的开发需求了。如果你有特殊的需求要对GIF图进行图片变换，那就得去自己实现Transformation接口才可以了。不过这个就非常复杂了，不在我们今天的讨论范围。

好了，那么我们继续实现对图片进行圆形化变换的功能，接下来只需要在transform()方法中去做具体的逻辑实现就可以了，代码如下所示：

```
public class CircleCrop extends BitmapTransformation {

    public CircleCrop(Context context) {
        super(context);
    }

    public CircleCrop(BitmapPool bitmapPool) {
        super(bitmapPool);
    }

    @Override
    public String getId() {
        return "com.example.glidetest.CircleCrop";
    }

    @Override
    protected Bitmap transform(BitmapPool pool, Bitmap toTransform, int outWidth, int outHeight) {
        int diameter = Math.min(toTransform.getWidth(), toTransform.getHeight());

        final Bitmap toReuse = pool.get(outWidth, outHeight, Bitmap.Config.ARGB_8888);
        final Bitmap result;
        if (toReuse != null) {
            result = toReuse;
        } else {
            result = Bitmap.createBitmap(diameter, diameter, Bitmap.Config.ARGB_8888);
        }

        int dx = (toTransform.getWidth() - diameter) / 2;
        int dy = (toTransform.getHeight() - diameter) / 2;
        Canvas canvas = new Canvas(result);
        Paint paint = new Paint();
        BitmapShader shader = new BitmapShader(toTransform, BitmapShader.TileMode.CLAMP, 
                                            BitmapShader.TileMode.CLAMP);
        if (dx != 0 || dy != 0) {
            Matrix matrix = new Matrix();
            matrix.setTranslate(-dx, -dy);
            shader.setLocalMatrix(matrix);
        }
        paint.setShader(shader);
        paint.setAntiAlias(true);
        float radius = diameter / 2f;
        canvas.drawCircle(radius, radius, radius, paint);

        if (toReuse != null && !pool.put(toReuse)) {
            toReuse.recycle();
        }
        return result;
    }
}
```

下面我来对transform()方法中的逻辑做下简单的解释。首先第18行先算出原图宽度和高度中较小的值，因为对图片进行圆形化变换肯定要以较小的那个值作为直径来进行裁剪。第20-26行则和刚才一样，从Bitmap缓存池中尝试获取一个Bitmap对象来进行重用，如果没有可重用的Bitmap对象的话就创建一个。第28-41行是具体进行圆形化变换的部分，这里算出了画布的偏移值，并且根据刚才得到的直径算出半径来进行画圆。最后，尝试将复用的Bitmap对象重新放回到缓存池当中，并将圆形化变换后的Bitmap对象进行返回。

这样，一个自定义图片变换的功能就写好了，那么现在我们就来尝试使用一下它吧。使用方法非常简单，刚才已经介绍过了，就是把这个自定义图片变换的实例传入到transform()方法中即可，如下所示：

```
Glide.with(this)
     .load(url)
     .transform(new CircleCrop(this))
     .into(imageView);
```

现在我们重新运行一下程序，效果如下图所示。

![img](http://img.blog.csdn.net/20170823204448719?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

# 更多图片变换功能

虽说Glide的图片变换功能框架已经很强大了，使得我们可以轻松地自定义图片变换效果，但是如果每一种图片变换都要我们自己去写还是蛮吃力的。事实上，确实也没有必要完全靠自己去实现各种各样的图片变换效果，因为大多数的图片变换都是比较通用的，各个项目会用到的效果都差不多，我们每一个都自己去重新实现无异于重复造轮子。

也正是因此，网上出现了很多Glide的图片变换开源库，其中做的最出色的应该要数glide-transformations这个库了。它实现了很多通用的图片变换效果，如裁剪变换、颜色变换、模糊变换等等，使得我们可以非常轻松地进行各种各样的图片变换。

glide-transformations的项目主页地址是 <https://github.com/wasabeef/glide-transformations> 。

下面我们就来体验一下这个库的强大功能吧。首先需要将这个库引入到我们的项目当中，在app/build.gradle文件当中添加如下依赖：

```
dependencies {
    compile 'jp.wasabeef:glide-transformations:2.0.2'
}
```

现在如果我想对图片进行模糊化处理，那么就可以使用glide-transformations库中的BlurTransformation这个类，代码如下所示：

```
Glide.with(this)
     .load(url)
     .bitmapTransform(new BlurTransformation(this))
     .into(imageView);
```

注意这里我们调用的是bitmapTransform()方法而不是transform()方法，因为glide-transformations库都是专门针对静态图片变换来进行设计的。现在重新运行一下程度，效果如下图所示。

![img](http://img.blog.csdn.net/20170823205319689?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

没错，我们就这样轻松地实现模糊化的效果了。

接下来我们再试一下图片黑白化的效果，使用的是GrayscaleTransformation这个类，代码如下所示：

```
Glide.with(this)
     .load(url)
     .bitmapTransform(new GrayscaleTransformation(this))
     .into(imageView);
```

现在重新运行一下程度，效果如下图所示。

![img](http://img.blog.csdn.net/20170823205616112?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

而且我们还可以将多个图片变换效果组合在一起使用，比如同时执行模糊化和黑白化的变换：

```
Glide.with(this)
     .load(url)
     .bitmapTransform(new BlurTransformation(this), new GrayscaleTransformation(this))
     .into(imageView);
```

可以看到，同时执行多种图片变换的时候，只需要将它们都传入到bitmapTransform()方法中即可。现在重新运行一下程序，效果如下图所示。

![img](http://img.blog.csdn.net/20170823205854058?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

当然，这些只是glide-transformations库的一小部分功能而已，更多的图片变换效果你可以到它的GitHub项目主页去学习，所有变换的用法都是这么简单哦。