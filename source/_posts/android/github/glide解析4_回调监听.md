title: Glide解析4_回调监听
date: 

categories: 
- android
- glide

tags: 
- github
- glide

---

> 本文出自[郭霖博客](http://blog.csdn.net/guolin_blog/article/details/70215985)

大家好，今天我们继续学习Glide。

在上一篇文章当中，我带着大家一起深入探究了Glide的缓存机制，我们不光掌握了Glide缓存的使用方法，还通过源码分析对缓存的工作原理进行了了解。虽说上篇文章和本篇文章的内容关系并不是很大，不过感兴趣的朋友还是可以去阅读一下 [Android图片加载框架最全解析（三），深入探究Glide的缓存机制](http://blog.csdn.net/guolin_blog/article/details/54895665) 。

今天是这个Glide系列的第四篇文章，我们又要选取一个新的功能模块开始学习了，那么就来研究一下Glide的回调和监听功能吧。今天的学习模式仍然是以基本用法和源码分析相结合的方式来进行的，当然，本文中的源码还是建在第二篇源码分析的基础之上，还没有看过这篇文章的朋友，建议先去阅读 [Android图片加载框架最全解析（二），从源码的角度理解Glide的执行流程](http://blog.csdn.net/guolin_blog/article/details/53939176) 。

# 回调的源码实现

作为一名Glide老手，相信大家对于Glide的基本用法已经非常熟练了。我们都知道，使用Glide在界面上加载并展示一张图片只需要一行代码：

```
Glide.with(this).load(url).into(imageView);
```

而在这一行代码的背后，Glide帮我们执行了成千上万行的逻辑。其实在第二篇文章当中，我们已经分析了这一行代码背后的完整执行流程，但是这里我准备再带着大家单独回顾一下回调这部分的源码，这将有助于我们今天这篇文章的学习。

首先来看一下into()方法，这里我们将ImageView的实例传入到into()方法当中，Glide将图片加载完成之后，图片就能显示到ImageView上了。这是怎么实现的呢？我们来看一下into()方法的源码：

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
            default:
                // Do nothing.
        }
    }
    return into(glide.buildImageViewTarget(view, transcodeClass));
}
```

可以看到，最后一行代码会调用glide.buildImageViewTarget()方法构建出一个Target对象，然后再把它传入到另一个接收Target参数的into()方法中。Target对象则是用来最终展示图片用的，如果我们跟进到glide.buildImageViewTarget()方法中，你会看到如下的源码：

```
public class ImageViewTargetFactory {

    @SuppressWarnings("unchecked")
    public <Z> Target<Z> buildTarget(ImageView view, Class<Z> clazz) {
        if (GlideDrawable.class.isAssignableFrom(clazz)) {
            return (Target<Z>) new GlideDrawableImageViewTarget(view);
        } else if (Bitmap.class.equals(clazz)) {
            return (Target<Z>) new BitmapImageViewTarget(view);
        } else if (Drawable.class.isAssignableFrom(clazz)) {
            return (Target<Z>) new DrawableImageViewTarget(view);
        } else {
            throw new IllegalArgumentException("Unhandled class: " + clazz
                    + ", try .as*(Class).transcode(ResourceTranscoder)");
        }
    }
}
```

buildTarget()方法会根据传入的class参数来构建不同的Target对象，如果你在使用Glide加载图片的时候调用了asBitmap()方法，那么这里就会构建出BitmapImageViewTarget对象，否则的话构建的都是GlideDrawableImageViewTarget对象。至于上述代码中的DrawableImageViewTarget对象，这个通常都是用不到的，我们可以暂时不用管它。

之后就会把这里构建出来的Target对象传入到GenericRequest当中，而Glide在图片加载完成之后又会回调GenericRequest的onResourceReady()方法，我们来看一下这部分源码：

```
public final class GenericRequest<A, T, Z, R> implements Request, SizeReadyCallback,
        ResourceCallback {

    private Target<R> target;
    ...

    private void onResourceReady(Resource<?> resource, R result) {
        boolean isFirstResource = isFirstReadyResource();
        status = Status.COMPLETE;
        this.resource = resource;
        if (requestListener == null || !requestListener.onResourceReady(result, model, target,
                loadedFromMemoryCache, isFirstResource)) {
            GlideAnimation<R> animation = animationFactory.build(loadedFromMemoryCache, isFirstResource);
            target.onResourceReady(result, animation);
        }
        notifyLoadSuccess();
    }
    ...
}
```

这里在第14行调用了target.onResourceReady()方法，而刚才我们已经知道，这里的target就是GlideDrawableImageViewTarget对象，那么我们再来看一下它的源码：

```
public class GlideDrawableImageViewTarget extends ImageViewTarget<GlideDrawable> {
    ...

    @Override
    public void onResourceReady(GlideDrawable resource, GlideAnimation<? super GlideDrawable> animation) {
        if (!resource.isAnimated()) {
            float viewRatio = view.getWidth() / (float) view.getHeight();
            float drawableRatio = resource.getIntrinsicWidth() / (float) resource.getIntrinsicHeight();
            if (Math.abs(viewRatio - 1f) <= SQUARE_RATIO_MARGIN
                    && Math.abs(drawableRatio - 1f) <= SQUARE_RATIO_MARGIN) {
                resource = new SquaringDrawable(resource, view.getWidth());
            }
        }
        super.onResourceReady(resource, animation);
        this.resource = resource;
        resource.setLoopCount(maxLoopCount);
        resource.start();
    }

    @Override
    protected void setResource(GlideDrawable resource) {
        view.setImageDrawable(resource);
    }

    ...
}
```

可以看到，这里在onResourceReady()方法中处理了图片展示，还有GIF播放的逻辑，那么一张图片也就显示出来了，这也就是Glide回调的基本实现原理。

好的，那么原理就先分析到这儿，接下来我们就来看一下在回调和监听方面还有哪些知识是可以扩展的。

# into()方法

使用了这么久的Glide，我们都知道into()方法中是可以传入ImageView的。那么into()方法还可以传入别的参数吗？我可以让Glide加载出来的图片不显示到ImageView上吗？答案是肯定的，这就需要用到自定义Target功能。

其实通过上面的分析，我们已经知道了，into()方法还有一个接收Target参数的重载。即使我们传入的参数是ImageView，Glide也会在内部自动构建一个Target对象。而如果我们能够掌握自定义Target技术的话，就可以更加随心所欲地控制Glide的回调了。

我们先来看一下Glide中Target的继承结构图吧，如下所示：

![img](http://img.blog.csdn.net/20170614222823388?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

可以看到，Target的继承结构还是相当复杂的，实现Target接口的子类非常多。不过你不用被这么多的子类所吓到，这些大多数都是Glide已经实现好的具备完整功能的Target子类，如果我们要进行自定义的话，通常只需要在两种Target的基础上去自定义就可以了，一种是SimpleTarget，一种是ViewTarget。

接下来我就分别以这两种Target来举例，学习一下自定义Target的功能。

首先来看SimpleTarget，顾名思义，它是一种极为简单的Target，我们使用它可以将Glide加载出来的图片对象获取到，而不是像之前那样只能将图片在ImageView上显示出来。

那么下面我们来看一下SimpleTarget的用法示例吧，其实非常简单：

```
SimpleTarget<GlideDrawable> simpleTarget = new SimpleTarget<GlideDrawable>() {
    @Override
    public void onResourceReady(GlideDrawable resource, GlideAnimation glideAnimation) {
        imageView.setImageDrawable(resource);
    }
};

public void loadImage(View view) {
    String url = "http://cn.bing.com/az/hprichbg/rb/TOAD_ZH-CN7336795473_1920x1080.jpg";
    Glide.with(this)
         .load(url)
         .into(simpleTarget);
}
```

怎么样？不愧是SimpleTarget吧，短短几行代码就搞了。这里我们创建了一个SimpleTarget的实例，并且指定它的泛型是GlideDrawable，然后重写了onResourceReady()方法。在onResourceReady()方法中，我们就可以获取到Glide加载出来的图片对象了，也就是方法参数中传过来的GlideDrawable对象。有了这个对象之后你可以使用它进行任意的逻辑操作，这里我只是简单地把它显示到了ImageView上。

SimpleTarget的实现创建好了，那么只需要在加载图片的时候将它传入到into()方法中就可以了，现在运行一下程序，效果如下图所示。

![img](http://img.blog.csdn.net/20170615212142394)

虽然目前这个效果和直接在into()方法中传入ImageView并没有什么区别，但是我们已经拿到了图片对象的实例，然后就可以随意做更多的事情了。

当然，SimpleTarget中的泛型并不一定只能是GlideDrawable，如果你能确定你正在加载的是一张静态图而不是GIF图的话，我们还能直接拿到这张图的Bitmap对象，如下所示：

```
SimpleTarget<Bitmap> simpleTarget = new SimpleTarget<Bitmap>() {
    @Override
    public void onResourceReady(Bitmap resource, GlideAnimation glideAnimation) {
        imageView.setImageBitmap(resource);
    }
};

public void loadImage(View view) {
    String url = "http://cn.bing.com/az/hprichbg/rb/TOAD_ZH-CN7336795473_1920x1080.jpg";
    Glide.with(this)
         .load(url)
         .asBitmap()
         .into(simpleTarget);
}
```

可以看到，这里我们将SimpleTarget的泛型指定成Bitmap，然后在加载图片的时候调用了asBitmap()方法强制指定这是一张静态图，这样就能在onResourceReady()方法中获取到这张图的Bitmap对象了。

好了，SimpleTarget的用法就是这么简单，接下来我们学习一下ViewTarget的用法。

事实上，从刚才的继承结构图上就能看出，Glide在内部自动帮我们创建的GlideDrawableImageViewTarget就是ViewTarget的子类。只不过GlideDrawableImageViewTarget被限定只能作用在ImageView上，而ViewTarget的功能更加广泛，它可以作用在任意的View上。

这里我们还是通过一个例子来演示一下吧，比如我创建了一个自定义布局MyLayout，如下所示：

```
public class MyLayout extends LinearLayout {

    private ViewTarget<MyLayout, GlideDrawable> viewTarget;

    public MyLayout(Context context, AttributeSet attrs) {
        super(context, attrs);
        viewTarget = new ViewTarget<MyLayout, GlideDrawable>(this) {
            @Override
            public void onResourceReady(GlideDrawable resource, GlideAnimation glideAnimation) {
                MyLayout myLayout = getView();
                myLayout.setImageAsBackground(resource);
            }
        };
    }

    public ViewTarget<MyLayout, GlideDrawable> getTarget() {
        return viewTarget;
    }

    public void setImageAsBackground(GlideDrawable resource) {
        setBackground(resource);
    }

}
```

在MyLayout的构造函数中，我们创建了一个ViewTarget的实例，并将Mylayout当前的实例this传了进去。ViewTarget中需要指定两个泛型，一个是View的类型，一个图片的类型（GlideDrawable或Bitmap）。然后在onResourceReady()方法中，我们就可以通过getView()方法获取到MyLayout的实例，并调用它的任意接口了。比如说这里我们调用了setImageAsBackground()方法来将加载出来的图片作为MyLayout布局的背景图。

接下来看一下怎么使用这个Target吧，由于MyLayout中已经提供了getTarget()接口，我们只需要在加载图片的地方这样写就可以了：

```
public class MainActivity extends AppCompatActivity {

    MyLayout myLayout;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        myLayout = (MyLayout) findViewById(R.id.background);
    }

    public void loadImage(View view) {
        String url = "http://cn.bing.com/az/hprichbg/rb/TOAD_ZH-CN7336795473_1920x1080.jpg";
        Glide.with(this)
             .load(url)
             .into(myLayout.getTarget());
    }

}
```

就是这么简单，在into()方法中传入myLayout.getTarget()即可。现在重新运行一下程序，效果如下图所示。

![img](http://img.blog.csdn.net/20170615225048126)

好的，关于自定义Target的功能我们就介绍这么多，这些虽说都是自定义Target最基本的用法，但掌握了这些用法之后，你就能应对各种各样复杂的逻辑了。

# preload()方法

Glide加载图片虽说非常智能，它会自动判断该图片是否已经有缓存了，如果有的话就直接从缓存中读取，没有的话再从网络去下载。但是如果我希望提前对图片进行一个预加载，等真正需要加载图片的时候就直接从缓存中读取，不想再等待慢长的网络加载时间了，这该怎么办呢？

对于很多Glide新手来说这确实是一个烦恼的问题，因为在没有学习本篇文章之前，into()方法中必须传入一个ImageView呀，而传了ImageView之后图片就显示出来了，这还怎么预加载呢？

不过在学习了本篇文章之后，相信你已经能够想到解决方案了。因为into()方法中除了传入ImageView之后还可以传入Target对象，如果我们在Target对象的onResourceReady()方法中做一个空实现，也就是不做任何逻辑处理，那么图片自然也就显示不出来了，而Glide的缓存机制却仍然还会正常工作，这样不就实现预加载功能了吗？

没错，上述的做法完全可以实现预加载功能，不过有没有感觉这种实现方式有点笨笨的。事实上，Glide专门给我们提供了预加载的接口，也就是preload()方法，我们只需要直接使用就可以了。

preload()方法有两个方法重载，一个不带参数，表示将会加载图片的原始尺寸，另一个可以通过参数指定加载图片的宽和高。

preload()方法的用法也非常简单，直接使用它来替换into()方法即可，如下所示：

```
Glide.with(this)
     .load(url)
     .diskCacheStrategy(DiskCacheStrategy.SOURCE)
     .preload();
```

需要注意的是，我们如果使用了preload()方法，最好要将diskCacheStrategy的缓存策略指定成DiskCacheStrategy.SOURCE。因为preload()方法默认是预加载的原始图片大小，而into()方法则默认会根据ImageView控件的大小来动态决定加载图片的大小。因此，如果不将diskCacheStrategy的缓存策略指定成DiskCacheStrategy.SOURCE的话，很容易会造成我们在预加载完成之后再使用into()方法加载图片，却仍然还是要从网络上去请求图片这种现象。

调用了预加载之后，我们以后想再去加载这张图片就会非常快了，因为Glide会直接从缓存当中去读取图片并显示出来，代码如下所示：

```
Glide.with(this)
     .load(url)
     .diskCacheStrategy(DiskCacheStrategy.SOURCE)
     .into(imageView);
```

注意，这里我们仍然需要使用diskCacheStrategy()方法将硬盘缓存策略指定成DiskCacheStrategy.SOURCE，以保证Glide一定会去读取刚才预加载的图片缓存。

preload()方法的用法大概就是这么简单，但是仅仅会使用显然层次有些太低了，下面我们就满足一下好奇心，看看它的源码是如何实现的。

和into()方法一样，preload()方法也是在GenericRequestBuilder类当中的，代码如下所示：

```
public class GenericRequestBuilder<ModelType, DataType, ResourceType, TranscodeType> implements Cloneable {
    ...

    public Target<TranscodeType> preload(int width, int height) {
        final PreloadTarget<TranscodeType> target = PreloadTarget.obtain(width, height);
        return into(target);
    }

    public Target<TranscodeType> preload() {
        return preload(Target.SIZE_ORIGINAL, Target.SIZE_ORIGINAL);
    }

    ...
}
```

正如刚才所说，preload()方法有两个方法重载，你可以调用带参数的preload()方法来明确指定图片的宽和高，也可以调用不带参数的preload()方法，它会在内部自动将图片的宽和高都指定成Target.SIZE_ORIGINAL，也就是图片的原始尺寸。

然后我们可以看到，这里在第5行调用了PreloadTarget.obtain()方法获取一个PreloadTarget的实例，并把它传入到了into()方法当中。从刚才的继承结构图中可以看出，PreloadTarget是SimpleTarget的子类，因此它是可以直接传入到into()方法中的。

那么现在的问题就是，PreloadTarget具体的实现到底是什么样子的了，我们看一下它的源码，如下所示：

```
public final class PreloadTarget<Z> extends SimpleTarget<Z> {

    public static <Z> PreloadTarget<Z> obtain(int width, int height) {
        return new PreloadTarget<Z>(width, height);
    }

    private PreloadTarget(int width, int height) {
        super(width, height);
    }

    @Override
    public void onResourceReady(Z resource, GlideAnimation<? super Z> glideAnimation) {
        Glide.clear(this);
    }
}
```

PreloadTarget的源码非常简单，obtain()方法中就是new了一个PreloadTarget的实例而已，而onResourceReady()方法中也没做什么事情，只是调用了Glide.clear()方法。

这里的Glide.clear()并不是清空缓存的意思，而是表示加载已完成，释放资源的意思，因此不用在这里产生疑惑。

其实PreloadTarget的思想和我们刚才提到设计思路是一样的，就是什么都不做就可以了。因为图片加载完成之后只将它缓存而不去显示它，那不就相当于预加载了嘛。

preload()方法不管是在用法方面还是源码实现方面都还是非常简单的，那么关于这个方法我们就学到这里。

# downloadOnly()方法

一直以来，我们使用Glide都是为了将图片显示到界面上。虽然我们知道Glide会在图片的加载过程中对图片进行缓存，但是缓存文件到底是存在哪里的，以及如何去直接访问这些缓存文件？我们都还不知道。

其实Glide将图片加载接口设计成这样也是希望我们使用起来更加的方便，不用过多去考虑底层的实现细节。但如果我现在就是想要去访问图片的缓存文件该怎么办呢？这就需要用到downloadOnly()方法了。

和preload()方法类似，downloadOnly()方法也是可以替换into()方法的，不过downloadOnly()方法的用法明显要比preload()方法复杂不少。顾名思义，downloadOnly()方法表示只会下载图片，而不会对图片进行加载。当图片下载完成之后，我们可以得到图片的存储路径，以便后续进行操作。

那么首先我们还是先来看下基本用法。downloadOnly()方法是定义在DrawableTypeRequest类当中的，它有两个方法重载，一个接收图片的宽度和高度，另一个接收一个泛型对象，如下所示：

- downloadOnly(int width, int height)
- downloadOnly(Y target)

这两个方法各自有各自的应用场景，其中downloadOnly(int width, int height)是用于在子线程中下载图片的，而downloadOnly(Y target)是用于在主线程中下载图片的。

那么我们先来看downloadOnly(int width, int height)的用法。当调用了downloadOnly(int width, int height)方法后会立即返回一个FutureTarget对象，然后Glide会在后台开始下载图片文件。接下来我们调用FutureTarget的get()方法就可以去获取下载好的图片文件了，如果此时图片还没有下载完，那么get()方法就会阻塞住，一直等到图片下载完成才会有值返回。

下面我们通过一个例子来演示一下吧，代码如下所示：

```
public void downloadImage(View view) {
    new Thread(new Runnable() {
        @Override
        public void run() {
            try {
                String url = "http://cn.bing.com/az/hprichbg/rb/TOAD_ZH-CN7336795473_1920x1080.jpg";
                final Context context = getApplicationContext();
                FutureTarget<File> target = Glide.with(context)
                                                 .load(url)
                                                 .downloadOnly(Target.SIZE_ORIGINAL, Target.SIZE_ORIGINAL);
                final File imageFile = target.get();
                runOnUiThread(new Runnable() {
                    @Override
                    public void run() {
                        Toast.makeText(context, imageFile.getPath(), Toast.LENGTH_LONG).show();
                    }
                });
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
    }).start();
}
```

这段代码稍微有一点点长，我带着大家解读一下。首先刚才说了，downloadOnly(int width, int height)方法必须要用在子线程当中，因此这里的第一步就是new了一个Thread。在子线程当中，我们先获取了一个Application Context，这个时候不能再用Activity作为Context了，因为会有Activity销毁了但子线程还没执行完这种可能出现。

接下来就是Glide的基本用法，只不过将into()方法替换成了downloadOnly()方法。downloadOnly()方法会返回一个FutureTarget对象，这个时候其实Glide已经开始在后台下载图片了，我们随时都可以调用FutureTarget的get()方法来获取下载的图片文件，只不过如果图片还没下载好线程会暂时阻塞住，等下载完成了才会把图片的File对象返回。

最后，我们使用runOnUiThread()切回到主线程，然后使用Toast将下载好的图片文件路径显示出来。

现在重新运行一下代码，效果如下图所示。

![img](http://img.blog.csdn.net/20170616223448840)

这样我们就能清晰地看出来图片完整的缓存路径是什么了。

之后我们可以使用如下代码去加载这张图片，图片就会立即显示出来，而不用再去网络上请求了：

```
public void loadImage(View view) {
    String url = "http://cn.bing.com/az/hprichbg/rb/TOAD_ZH-CN7336795473_1920x1080.jpg";
    Glide.with(this)
            .load(url)
            .diskCacheStrategy(DiskCacheStrategy.SOURCE)
            .into(imageView);
}
```

需要注意的是，这里必须将硬盘缓存策略指定成DiskCacheStrategy.SOURCE或者DiskCacheStrategy.ALL，否则Glide将无法使用我们刚才下载好的图片缓存文件。

现在重新运行一下代码，效果如下图所示。

![img](http://img.blog.csdn.net/20170616223618749)

可以看到，图片的加载和显示是非常快的，因为Glide直接使用的是刚才下载好的缓存文件。

那么这个downloadOnly(int width, int height)方法的工作原理到底是什么样的呢？我们来简单快速地看一下它的源码吧。

首先在DrawableTypeRequest类当中可以找到定义这个方法的地方，如下所示：

```
public class DrawableTypeRequest<ModelType> extends DrawableRequestBuilder<ModelType>
        implements DownloadOptions {
    ...

    public FutureTarget<File> downloadOnly(int width, int height) {
        return getDownloadOnlyRequest().downloadOnly(width, height);
    }

    private GenericTranscodeRequest<ModelType, InputStream, File> getDownloadOnlyRequest() {
        return optionsApplier.apply(new GenericTranscodeRequest<ModelType, InputStream, File>(
            File.class, this, streamModelLoader, InputStream.class, File.class, optionsApplier));
    }
}
```

这里会先调用getDownloadOnlyRequest()方法得到一个GenericTranscodeRequest对象，然后再调用它的downloadOnly()方法，代码如下所示：

```
public class GenericTranscodeRequest<ModelType, DataType, ResourceType>
    implements DownloadOptions {
    ...

    public FutureTarget<File> downloadOnly(int width, int height) {
        return getDownloadOnlyRequest().into(width, height);
    }

    private GenericRequestBuilder<ModelType, DataType, File, File> getDownloadOnlyRequest() {
        ResourceTranscoder<File, File> transcoder = UnitTranscoder.get();
        DataLoadProvider<DataType, File> dataLoadProvider = glide.buildDataProvider(dataClass, File.class);
        FixedLoadProvider<ModelType, DataType, File, File> fixedLoadProvider =
            new FixedLoadProvider<ModelType, DataType, File, File>(modelLoader, transcoder, dataLoadProvider);
        return optionsApplier.apply(
                new GenericRequestBuilder<ModelType, DataType, File, File>(fixedLoadProvider,
                File.class, this))
                .priority(Priority.LOW)
                .diskCacheStrategy(DiskCacheStrategy.SOURCE)
                .skipMemoryCache(true);
    }
}
```

这里又是调用了一个getDownloadOnlyRequest()方法来构建了一个图片下载的请求，getDownloadOnlyRequest()方法会返回一个GenericRequestBuilder对象，接着调用它的into(width, height)方法，我们继续跟进去瞧一瞧：

```
public FutureTarget<TranscodeType> into(int width, int height) {
    final RequestFutureTarget<ModelType, TranscodeType> target =
            new RequestFutureTarget<ModelType, TranscodeType>(glide.getMainHandler(), width, height);
    glide.getMainHandler().post(new Runnable() {
        @Override
        public void run() {
            if (!target.isCancelled()) {
                into(target);
            }
        }
    });
    return target;
}
```

可以看到，这里首先是new出了一个RequestFutureTarget对象，RequestFutureTarget也是Target的子类之一。然后通过Handler将线程切回到主线程当中，再将这个RequestFutureTarget传入到into()方法当中。

那么也就是说，其实这里就是调用了接收Target参数的into()方法，然后Glide就开始执行正常的图片加载逻辑了。那么现在剩下的问题就是，这个RequestFutureTarget中到底处理了些什么逻辑？我们打开它的源码看一看：

```
public class RequestFutureTarget<T, R> implements FutureTarget<R>, Runnable {
    ...

    @Override
    public R get() throws InterruptedException, ExecutionException {
        try {
            return doGet(null);
        } catch (TimeoutException e) {
            throw new AssertionError(e);
        }
    }

    @Override
    public R get(long time, TimeUnit timeUnit) throws InterruptedException, ExecutionException, 
        TimeoutException {
        return doGet(timeUnit.toMillis(time));
    }

    @Override
    public void getSize(SizeReadyCallback cb) {
        cb.onSizeReady(width, height);
    }

    @Override
    public synchronized void onLoadFailed(Exception e, Drawable errorDrawable) {
        exceptionReceived = true;
        this.exception = e;
        waiter.notifyAll(this);
    }

    @Override
    public synchronized void onResourceReady(R resource, GlideAnimation<? super R> glideAnimation) {
        resultReceived = true;
        this.resource = resource;
        waiter.notifyAll(this);
    }

    private synchronized R doGet(Long timeoutMillis) throws ExecutionException, InterruptedException, 
        TimeoutException {
        if (assertBackgroundThread) {
            Util.assertBackgroundThread();
        }

        if (isCancelled) {
            throw new CancellationException();
        } else if (exceptionReceived) {
            throw new ExecutionException(exception);
        } else if (resultReceived) {
            return resource;
        }

        if (timeoutMillis == null) {
            waiter.waitForTimeout(this, 0);
        } else if (timeoutMillis > 0) {
            waiter.waitForTimeout(this, timeoutMillis);
        }

        if (Thread.interrupted()) {
            throw new InterruptedException();
        } else if (exceptionReceived) {
            throw new ExecutionException(exception);
        } else if (isCancelled) {
            throw new CancellationException();
        } else if (!resultReceived) {
            throw new TimeoutException();
        }

        return resource;
    }

    static class Waiter {

        public void waitForTimeout(Object toWaitOn, long timeoutMillis) throws InterruptedException {
            toWaitOn.wait(timeoutMillis);
        }

        public void notifyAll(Object toNotify) {
            toNotify.notifyAll();
        }
    }

    ...
}
```

这里我对RequestFutureTarget的源码做了一些精简，我们只看最主要的逻辑就可以了。

刚才我们已经学习过了downloadOnly()方法的基本用法，在调用了downloadOnly()方法之后，再调用FutureTarget的get()方法，就能获取到下载的图片文件了。而downloadOnly()方法返回的FutureTarget对象其实就是这个RequestFutureTarget，因此我们直接来看它的get()方法就行了。

RequestFutureTarget的get()方法中又调用了一个doGet()方法，而doGet()方法才是真正处理具体逻辑的地方。首先在doGet()方法中会判断当前是否是在子线程当中，如果不是的话会直接抛出一个异常。然后下面会判断下载是否已取消、或者已失败，如果是已取消或者已失败的话都会直接抛出一个异常。接下来会根据resultReceived这个变量来判断下载是否已完成，如果这个变量为true的话，就直接把结果进行返回。

那么如果下载还没有完成呢？我们继续往下看，接下来就进入到一个wait()当中，把当前线程给阻塞住，从而阻止代码继续往下执行。这也是为什么downloadOnly(int width, int height)方法要求必须在子线程当中使用，因为它会对当前线程进行阻塞，如果在主线程当中使用的话，那么就会让主线程卡死，从而用户无法进行任何其他操作。

那么现在线程被阻塞住了，什么时候才能恢复呢？答案在onResourceReady()方法中。可以看到，onResourceReady()方法中只有三行代码，第一行把resultReceived赋值成true，说明图片文件已经下载好了，这样下次再调用get()方法时就不会再阻塞线程，而是可以直接将结果返回。第二行把下载好的图片文件赋值到一个全局的resource变量上面，这样doGet()方法就也可以访问到它。第三行notifyAll一下，通知所有wait的线程取消阻塞，这个时候图片文件已经下载好了，因此doGet()方法也就可以返回结果了。

好的，这就是downloadOnly(int width, int height)方法的基本用法和实现原理，那么下面我们来看一下downloadOnly(Y target)方法。

回想一下，其实downloadOnly(int width, int height)方法必须使用在子线程当中，最主要还是因为它在内部帮我们自动创建了一个RequestFutureTarget，是这个RequestFutureTarget要求必须在子线程当中执行。而downloadOnly(Y target)方法则要求我们传入一个自己创建的Target，因此就不受RequestFutureTarget的限制了。

但是downloadOnly(Y target)方法的用法也会相对更复杂一些，因为我们又要自己创建一个Target了，而且这次必须直接去实现最顶层的Target接口，比之前的SimpleTarget和ViewTarget都要复杂不少。

那么下面我们就来实现一个最简单的DownloadImageTarget吧，注意Target接口的泛型必须指定成File对象，这是downloadOnly(Y target)方法要求的，代码如下所示：

```
public class DownloadImageTarget implements Target<File> {

    private static final String TAG = "DownloadImageTarget";

    @Override
    public void onStart() {
    }

    @Override
    public void onStop() {
    }

    @Override
    public void onDestroy() {
    }

    @Override
    public void onLoadStarted(Drawable placeholder) {
    }

    @Override
    public void onLoadFailed(Exception e, Drawable errorDrawable) {
    }

    @Override
    public void onResourceReady(File resource, GlideAnimation<? super File> glideAnimation) {
        Log.d(TAG, resource.getPath());
    }

    @Override
    public void onLoadCleared(Drawable placeholder) {
    }

    @Override
    public void getSize(SizeReadyCallback cb) {
        cb.onSizeReady(Target.SIZE_ORIGINAL, Target.SIZE_ORIGINAL);
    }

    @Override
    public void setRequest(Request request) {
    }

    @Override
    public Request getRequest() {
        return null;
    }
}
```

由于是要直接实现Target接口，因此需要重写的方法非常多。这些方法大多是数Glide加载图片生命周期的一些回调，我们可以不用管它们，其中只有两个方法是必须实现的，一个是getSize()方法，一个是onResourceReady()方法。

在第二篇Glide源码解析的时候，我带着大家一起分析过，Glide在开始加载图片之前会先计算图片的大小，然后回调到onSizeReady()方法当中，之后才会开始执行图片加载。而这里，计算图片大小的任务就交给我们了。只不过这是一个最简单的Target实现，我在getSize()方法中就直接回调了Target.SIZE_ORIGINAL，表示图片的原始尺寸。

然后onResourceReady()方法我们就非常熟悉了，图片下载完成之后就会回调到这里，我在这个方法中只是打印了一下下载的图片文件的路径。

这样一个最简单的DownloadImageTarget就定义好了，使用它也非常的简单，我们不用再考虑什么线程的问题了，而是直接把它的实例传入downloadOnly(Y target)方法中即可，如下所示：

```
public void downloadImage(View view) {
    String url = "http://cn.bing.com/az/hprichbg/rb/TOAD_ZH-CN7336795473_1920x1080.jpg";
    Glide.with(this)
            .load(url)
            .downloadOnly(new DownloadImageTarget());
}
```

现在重新运行一下代码并点击Download Image按钮，然后观察控制台日志的输出，结果如下图所示。

![img](http://img.blog.csdn.net/20170617112709246?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvZ3VvbGluX2Jsb2c=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这样我们就使用了downloadOnly(Y target)方法同样获取到下载的图片文件的缓存路径了。

好的，那么关于downloadOnly()方法我们就学到这里。

# listener()方法

今天学习的内容已经够多了，下面我们就以一个简单的知识点结尾吧，Glide回调与监听的最后一部分——listener()方法。

其实listener()方法的作用非常普遍，它可以用来监听Glide加载图片的状态。举个例子，比如说我们刚才使用了preload()方法来对图片进行预加载，但是我怎样确定预加载有没有完成呢？还有如果Glide加载图片失败了，我该怎样调试错误的原因呢？答案都在listener()方法当中。

首先来看下listener()方法的基本用法吧，不同于刚才几个方法都是要替换into()方法的，listener()是结合into()方法一起使用的，当然也可以结合preload()方法一起使用。最基本的用法如下所示：

```
public void loadImage(View view) {
    String url = "http://cn.bing.com/az/hprichbg/rb/TOAD_ZH-CN7336795473_1920x1080.jpg";
    Glide.with(this)
            .load(url)
            .listener(new RequestListener<String, GlideDrawable>() {
                @Override
                public boolean onException(Exception e, String model, Target<GlideDrawable> target,
                    boolean isFirstResource) {
                    return false;
                }

                @Override
                public boolean onResourceReady(GlideDrawable resource, String model,
                    Target<GlideDrawable> target, boolean isFromMemoryCache, boolean isFirstResource) {
                    return false;
                }
            })
            .into(imageView);
}
```

这里我们在into()方法之前串接了一个listener()方法，然后实现了一个RequestListener的实例。其中RequestListener需要实现两个方法，一个onResourceReady()方法，一个onException()方法。从方法名上就可以看出来了，当图片加载完成的时候就会回调onResourceReady()方法，而当图片加载失败的时候就会回调onException()方法，onException()方法中会将失败的Exception参数传进来，这样我们就可以定位具体失败的原因了。

没错，listener()方法就是这么简单。不过还有一点需要处理，onResourceReady()方法和onException()方法都有一个布尔值的返回值，返回false就表示这个事件没有被处理，还会继续向下传递，返回true就表示这个事件已经被处理掉了，从而不会再继续向下传递。举个简单点的例子，如果我们在RequestListener的onResourceReady()方法中返回了true，那么就不会再回调Target的onResourceReady()方法了。

关于listener()方法的用法就讲这么多，不过还是老规矩，我们再来看一下它的源码是怎么实现的吧。

首先，listener()方法是定义在GenericRequestBuilder类当中的，而我们传入到listener()方法中的实例则会赋值到一个requestListener变量当中，如下所示：

```
public class GenericRequestBuilder<ModelType, DataType, ResourceType, TranscodeType> implements Cloneable {

    private RequestListener<? super ModelType, TranscodeType> requestListener;
    ...

    public GenericRequestBuilder<ModelType, DataType, ResourceType, TranscodeType> listener(
            RequestListener<? super ModelType, TranscodeType> requestListener) {
        this.requestListener = requestListener;
        return this;
    }

    ...
}
```

接下来在构建GenericRequest的时候这个变量也会被一起传进去，最后在图片加载完成的时候，我们会看到如下逻辑：

```
public final class GenericRequest<A, T, Z, R> implements Request, SizeReadyCallback,
        ResourceCallback {

    private RequestListener<? super A, R> requestListener;
    ...

    private void onResourceReady(Resource<?> resource, R result) {
        boolean isFirstResource = isFirstReadyResource();
        status = Status.COMPLETE;
        this.resource = resource;
        if (requestListener == null || !requestListener.onResourceReady(result, model, target,
                loadedFromMemoryCache, isFirstResource)) {
            GlideAnimation<R> animation = animationFactory.build(loadedFromMemoryCache, isFirstResource);
            target.onResourceReady(result, animation);
        }
        notifyLoadSuccess();
    }
    ...
}
```

可以看到，这里在第11行会先回调requestListener的onResourceReady()方法，只有当这个onResourceReady()方法返回false的时候，才会继续调用Target的onResourceReady()方法，这也就是listener()方法的实现原理。

另外一个onException()方法的实现机制也是一模一样的，代码同样是在GenericRequest类中，如下所示：

```
public final class GenericRequest<A, T, Z, R> implements Request, SizeReadyCallback,
        ResourceCallback {
    ...

    @Override
    public void onException(Exception e) {
        status = Status.FAILED;
        if (requestListener == null || 
                !requestListener.onException(e, model, target, isFirstReadyResource())) {
            setErrorPlaceholder(e);
        }
    }

    ...
}
```

可以看到，这里会在第9行回调requestListener的onException()方法，只有在onException()方法返回false的情况下才会继续调用setErrorPlaceholder()方法。也就是说，如果我们在onException()方法中返回了true，那么Glide请求中使用error(int resourceId)方法设置的异常占位图就失效了。

这样我们也就将listener()方法的全部实现原理都分析完了。

好了，关于Glide回调与监听方面的内容今天就讲到这里，这一篇文章的内容非常充实，希望大家都能好好掌握。下一篇文章当中，我会继续带着大家深入分析Glide的其他功能模块，讲一讲图片变换方面的知识，敬请期待。