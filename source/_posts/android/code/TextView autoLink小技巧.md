title: TextView autoLink小技巧

date: 

categories: 
- android
- code

tags: 
- android
- textview
- analysis
- autoLink

---



本篇主要参照 :[http://www.jianshu.com/p/d3bef8449960](http://www.jianshu.com/p/d3bef8449960) 

### 去掉TextView中autoLink的下划线

所以要解决这个问题，就只有再设置一个没有下划线的Spannable对象。

首先，继承一个CharacterStyle或其已实现的子类，重写updateDrawState方法，代码如下：

```

import android.text.TextPaint;
import android.text.style.UnderlineSpan;

/**
 * 无下划线的Span
 */
public class NoUnderlineSpan extends UnderlineSpan {

    @Override
    public void updateDrawState(TextPaint ds) {
        ds.setColor(ds.linkColor);
        ds.setUnderlineText(false);
    }
}
```

然后在textview设置了内容之后，调用以下的代码，设置一个span：

```
NoUnderlineSpan mNoUnderlineSpan = new NoUnderlineSpan();
if (textview.getText() instanceof Spannable) {
    Spannable s = (Spannable) textview.getText();
    s.setSpan(mNoUnderlineSpan, 0, s.length(), Spanned.SPAN_MARK_MARK);
}

```

 

### TextView autoLink详细解析设置

autoLink属性，在TextView的xml里加一句`android:autoLink="web"`，TextView就可以自动的把文字内的网址识别出来并加上了点击跳转到浏览器的效果，简单粗暴有效。

但会有以下四个问题

1. 这超链接网址字体颜色和TextView设置的字体颜色根本不一致啊
2. 这个自带的下划线好烦人，不想要
3. 我想点击网址跳转到我自己应用内的WebView打开而不是用手机的浏览器
4. 和TextView长按事件有冲突，每次onLongClick后都会带出一发超链接网址的onClick，在onLongClick里返回什么都没用。



带着4个问题去看TextView的源码，看能否解决这些问题
因为是在xml里写的属性，总要在构造方法里被取出来

```
public TextView(Context context, @Nullable AttributeSet attrs, int defStyleAttr, int defStyleRes) {
        super(context, attrs, defStyleAttr, defStyleRes);

        //跳过一堆没用的...

        a = theme.obtainStyledAttributes(
                    attrs, com.android.internal.R.styleable.TextView, defStyleAttr, defStyleRes);

        int n = a.getIndexCount();
        for (int i = 0; i < n; i++) {
            int attr = a.getIndex(i);

            switch (attr) {
        //没用的...
            case com.android.internal.R.styleable.TextView_autoLink:
                mAutoLinkMask = a.getInt(attr, 0);
                break;
        //跳过一堆没用的...
            }
        }
}
```

不要在意那些跳过的，的确都没啥用，我们找到了一个mAutoLinkMask的成员变量
随便搜一下，我们就在setText里发现了他

```
 private void setText(CharSequence text, BufferType type,boolean notifyBefore, int oldlen) {
            //前略...
            if (Linkify.addLinks(s2, mAutoLinkMask)) {
                text = s2;
                type = (type == BufferType.EDITABLE) ? BufferType.EDITABLE : BufferType.SPANNABLE;

                mText = text;

                if (mLinksClickable && !textCanBeSelected()) {
                    setMovementMethod(LinkMovementMethod.getInstance());
                }
            }
            //后略...
}
```

可以很清楚的看到，代码跳到`Linkify.addLinks(s2, mAutoLinkMask)`这里去了
那么`Linkify`这个类是做什么的呢？API是这么解释的

```
Linkify take a piece of text and a regular expression and turns all of the 
regex matches in the text into clickable links.
```

大致意思是 Linkify 会使用正则表达式将一篇文本中可以被该正则匹配出的部分变为能点击的链接。
有点不明觉厉是不是，请原谅我小学英语水平.........
从API的字面意思也可以看出，我们已经找到核心了，接着点进去看。

```
public static final boolean addLinks(@NonNull Spannable text, @LinkifyMask int mask) {
        if (mask == 0) {
            return false;
        }

        URLSpan[] old = text.getSpans(0, text.length(), URLSpan.class);

        for (int i = old.length - 1; i >= 0; i--) {
            text.removeSpan(old[i]);
        }

        ArrayList<LinkSpec> links = new ArrayList<LinkSpec>();

        if ((mask & WEB_URLS) != 0) {
            gatherLinks(links, text, Patterns.AUTOLINK_WEB_URL,
                new String[] { "http://", "https://", "rtsp://" },
                sUrlMatchFilter, null);
        }

        if ((mask & EMAIL_ADDRESSES) != 0) {
            gatherLinks(links, text, Patterns.AUTOLINK_EMAIL_ADDRESS,
                new String[] { "mailto:" },
                null, null);
        }

        if ((mask & PHONE_NUMBERS) != 0) {
            gatherTelLinks(links, text);
        }

        if ((mask & MAP_ADDRESSES) != 0) {
            gatherMapLinks(links, text);
        }

        pruneOverlaps(links);

        if (links.size() == 0) {
            return false;
        }

        for (LinkSpec link: links) {
            applyLink(link.url, link.start, link.end, text);
        }

        return true;
    }
```

方法代码不长，也很容易理解，可以发现，先把传进来的text里所有的URLSpan类型的span拿出来，全部移除掉，其实看到这我们就应该清楚了，下面肯定是按照自己的逻辑作出新的URLSpan加进去。
实际上也正是这样，下面的几个if就是在做这项工作，可以看到第一个if就是用来匹配网址的，gatherLinks这个方法从名字上看就能看出是在收集Links，然后统一放到links这个ArrayList中，至于LinkSpec，他是Linkify这个java文件中写的另外一个类，长这样

```
class LinkSpec {
    String url;
    int start;
    int end;
}
```

最后的applyLink就是setSpan操作了，我们点进去印证一下

```
    private static final void applyLink(String url, int start, int end, Spannable text) {
        URLSpan span = new URLSpan(url);

        text.setSpan(span, start, end, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE);
    }
```

果然如此，到此我们其实已经摸清楚了autoLink的工作机制，其实就是利用自己的正则匹配出文本中的网址，然后创建URLSpan  setSpan而已。

我们回过头来看一下一开始的四个问题，应该能感觉到，解决问题的方式都藏在URLSpan里

打开URLSpan看一眼

```
public class URLSpan extends ClickableSpan implements ParcelableSpan {

    private final String mURL;

    public URLSpan(String url) {
        mURL = url;
    }

    public URLSpan(Parcel src) {
        mURL = src.readString();
    }

    public int getSpanTypeId() {
        return getSpanTypeIdInternal();
    }

    /** @hide */
    public int getSpanTypeIdInternal() {
        return TextUtils.URL_SPAN;
    }

    public int describeContents() {
        return 0;
    }

    public void writeToParcel(Parcel dest, int flags) {
        writeToParcelInternal(dest, flags);
    }

    /** @hide */
    public void writeToParcelInternal(Parcel dest, int flags) {
        dest.writeString(mURL);
    }

    public String getURL() {
        return mURL;
    }

    @Override
    public void onClick(View widget) {
        Uri uri = Uri.parse(getURL());
        Context context = widget.getContext();
        Intent intent = new Intent(Intent.ACTION_VIEW, uri);
        intent.putExtra(Browser.EXTRA_APPLICATION_ID, context.getPackageName());
        try {
            context.startActivity(intent);
        } catch (ActivityNotFoundException e) {
            Log.w("URLSpan", "Actvity was not found for intent, " + intent.toString());
        }
    }
}
```

忽略掉那几个get方法和Parcelable的方法，就只剩个`onClick(View widget)`，可以看到，onClick里是用Intent打开了外部浏览器，那么问题3的根源找到了，怎么解决呢？我们自己定义一个URLSpan子类重写`onClick(View widget)`换成我们自己的逻辑不就行了，然后再学着Linkify一样把原本Linkify加好的URLSpan换成我们的。
不着急先动手，先接着看下去，看看其他三个问题根源在哪？
点进URLSpan的父类ClickableSpan看看

```
public abstract class ClickableSpan extends CharacterStyle implements UpdateAppearance {

    public abstract void onClick(View widget);

    @Override
    public void updateDrawState(TextPaint ds) {
        ds.setColor(ds.linkColor);
        ds.setUnderlineText(true);
    }
}
```

好吧，问题1和2的根源也找到了，`updateDrawState(TextPaint ds)`方法中TextPaint 都提供给我们了，那还不是想怎么搞就怎么搞么，依然靠自定义URLSpan子类重写updateDrawState解决。

然后？问题4呢？好像没看到和问题4相关的代码

I select die

回过来想一下也是啊，长按点击事件最后都是TextView的事，关这什么事呢，既然是事件，那就去TextView的onTouchEvent瞄一眼吧

```
 public boolean onTouchEvent(MotionEvent event) {
        final int action = event.getActionMasked();
        if (mEditor != null) {
            mEditor.onTouchEvent(event);

            if (mEditor.mSelectionModifierCursorController != null &&
                    mEditor.mSelectionModifierCursorController.isDragAcceleratorActive()) {
                return true;
            }
        }

        final boolean superResult = super.onTouchEvent(event);

        if (mEditor != null && mEditor.mDiscardNextActionUp && action == MotionEvent.ACTION_UP) {
            mEditor.mDiscardNextActionUp = false;

            if (mEditor.mIsInsertionActionModeStartPending) {
                mEditor.startInsertionActionMode();
                mEditor.mIsInsertionActionModeStartPending = false;
            }
            return superResult;
        }

        final boolean touchIsFinished = (action == MotionEvent.ACTION_UP) &&
                (mEditor == null || !mEditor.mIgnoreActionUpEvent) && isFocused();

         if ((mMovement != null || onCheckIsTextEditor()) && isEnabled()
                && mText instanceof Spannable && mLayout != null) {
            boolean handled = false;

            if (mMovement != null) {
                handled |= mMovement.onTouchEvent(this, (Spannable) mText, event);
            }

            final boolean textIsSelectable = isTextSelectable();
            if (touchIsFinished && mLinksClickable && mAutoLinkMask != 0 && textIsSelectable) {
                ClickableSpan[] links = ((Spannable) mText).getSpans(getSelectionStart(),
                        getSelectionEnd(), ClickableSpan.class);

                if (links.length > 0) {
                    links[0].onClick(this);
                    handled = true;
                }
            }

            if (touchIsFinished && (isTextEditable() || textIsSelectable)) {
                final InputMethodManager imm = InputMethodManager.peekInstance();
                viewClicked(imm);
                if (!textIsSelectable && mEditor.mShowSoftInputOnFocus) {
                    handled |= imm != null && imm.showSoftInput(this, 0);
                }

                mEditor.onTouchUpEvent(event);

                handled = true;
            }

            if (handled) {
                return true;
            }
        }

        return superResult;
    }
```

TextView的onTouchEvent代码并不多，也都不难理解，首先如果mEditor != null会将touch事件交给mEditor处理，这个mEditor其实是和EditText有关系的，没有使用EditText这里应该是不会被创建的。
我们跳过mEditor有关的代码，接下来发现交给了super.onTouchEvent(event)父类去处理，而LongClick就是在这里被处理的，至于父类怎么处理的LongClick我们先不去研究，先找到UrlSpan的click是什么时候被调用的。
继续往下看，可以发现一段比较关键的代码

```
if ((mMovement != null || onCheckIsTextEditor()) && isEnabled()
                && mText instanceof Spannable && mLayout != null) {
            boolean handled = false;
            if (mMovement != null) {
                handled |= mMovement.onTouchEvent(this, (Spannable) mText, event);
            }
            final boolean textIsSelectable = isTextSelectable();
            if (touchIsFinished && mLinksClickable && mAutoLinkMask != 0 && textIsSelectable) {
                ClickableSpan[] links = ((Spannable) mText).getSpans(getSelectionStart(),
                        getSelectionEnd(), ClickableSpan.class);

                if (links.length > 0) {
                    links[0].onClick(this);
                    handled = true;
                }
            }
}
```

这里会判断mMovement != null，如果不为空，会调用mMovement的onTouchEvent，那么这个mMovement又是个什么，点进去可以看到

```
public interface MovementMethod {
    public void initialize(TextView widget, Spannable text);
    public boolean onKeyDown(TextView widget, Spannable text, int keyCode, KeyEvent event);
    public boolean onKeyUp(TextView widget, Spannable text, int keyCode, KeyEvent event);
    public boolean onKeyOther(TextView view, Spannable text, KeyEvent event);

    public void onTakeFocus(TextView widget, Spannable text, int direction);
    public boolean onTrackballEvent(TextView widget, Spannable text, MotionEvent event);
    public boolean onTouchEvent(TextView widget, Spannable text, MotionEvent event);
    public boolean onGenericMotionEvent(TextView widget, Spannable text, MotionEvent event);
    public boolean canSelectArbitrarily();
}
```

这是一个接口，有一系列关于事件的方法，我觉得大致可以理解成为TextView提供事件响应服务的类，既然是一个接口，肯定是有实现类来提供具体功能的，那么和我们URLSpan又有什么关系？
不知道大家有没有注意到，其实在我们刚开始寻找mAutoLinkMask找到setText方法里已经看到MovementMethod了

```
if (Linkify.addLinks(s2, mAutoLinkMask)) {
     text = s2;
     type = (type == BufferType.EDITABLE) ? BufferType.EDITABLE : BufferType.SPANNABLE;
      mText = text;
      if (mLinksClickable && !textCanBeSelected()) {
           setMovementMethod(LinkMovementMethod.getInstance());
      }
}
```

从这段代码来看，当Linkify.addLinks(s2, mAutoLinkMask)之后，如果mLinksClickable 为true和textCanBeSelected()为false的情况下，会给mMovement赋值，值就是MovementMethod 的实现类LinkMovementMethod，我们先不管LinkMovementMethod里面到底是什么，先看这个if条件能不能成立
mLinksClickable在TextView中可以找到

```
    private boolean mLinksClickable = true;
```

本身就是true，而其唯一一次变动是在构造方法中

```
   case com.android.internal.R.styleable.TextView_linksClickable:
        mLinksClickable = a.getBoolean(attr, true);
        break;
```

显然，这里值一定是true的
那么textCanBeSelected()呢？点开这个方法来看

```
    boolean textCanBeSelected() {
        if (mMovement == null || !mMovement.canSelectArbitrarily()) return false;
        return isTextEditable() ||
                (isTextSelectable() && mText instanceof Spannable && isEnabled());
    }
```

先看第一句，`if (mMovement == null || !mMovement.canSelectArbitrarily()) return false;`这是又给绕回来了，当mMovement == null时就return false，那此时mMovement到底是不是null呢，答案其实是不确定的，因为在TextView中除了setText，还有一些地方有mMovement 赋值的操作，比如

```
public void setTextIsSelectable(boolean selectable) {
        if (!selectable && mEditor == null) return; // false is default value with no edit data

        createEditorIfNeeded();
        if (mEditor.mTextIsSelectable == selectable) return;

        mEditor.mTextIsSelectable = selectable;
        setFocusableInTouchMode(selectable);
        setFocusable(selectable);
        setClickable(selectable);
        setLongClickable(selectable);

        // mInputType should already be EditorInfo.TYPE_NULL and mInput should be null

        setMovementMethod(selectable ? ArrowKeyMovementMethod.getInstance() : null);
        setText(mText, selectable ? BufferType.SPANNABLE : BufferType.NORMAL);

        // Called by setText above, but safer in case of future code changes
        mEditor.prepareCursorControllers();
    }
```

如果我们设置了TextView的文本可以被选择，那么mMovement在这里就会被赋值为ArrowKeyMovementMethod的实例，mMovement就不为null了，那mMovement.canSelectArbitrarily()呢，这个简单，看看ArrowKeyMovementMethod的实现会发现

```
    @Override
    public boolean canSelectArbitrarily() {
        return true;
    }
```

是一定为true的，那么这种情况下这个if判断就不成立，textCanBeSelected会return true
那么我们再回到

```
      if (mLinksClickable && !textCanBeSelected()) {
           setMovementMethod(LinkMovementMethod.getInstance());
      }
```

也就是说这里mMovement会不会被赋值成LinkMovementMethod的实例还不一定呢。

那么我们现在假设两种情况，第一种是mMovement被赋值为LinkMovementMethod的实例，第二种是mMovement被赋值为ArrowKeyMovementMethod的实例。
无论是哪种，代码都会走到MovementMethod的onTouchEvent中，我们到各自的实现类里去看一看。

先看第一种LinkMovementMethod的onTouchEvent方法

```
public boolean onTouchEvent(TextView widget, Spannable buffer,
                                MotionEvent event) {
        int action = event.getAction();

        if (action == MotionEvent.ACTION_UP ||
            action == MotionEvent.ACTION_DOWN) {
            int x = (int) event.getX();
            int y = (int) event.getY();

            x -= widget.getTotalPaddingLeft();
            y -= widget.getTotalPaddingTop();

            x += widget.getScrollX();
            y += widget.getScrollY();

            Layout layout = widget.getLayout();
            int line = layout.getLineForVertical(y);
            int off = layout.getOffsetForHorizontal(line, x);

            ClickableSpan[] link = buffer.getSpans(off, off, ClickableSpan.class);

            if (link.length != 0) {
                if (action == MotionEvent.ACTION_UP) {
                    link[0].onClick(widget);
                } else if (action == MotionEvent.ACTION_DOWN) {
                    Selection.setSelection(buffer,
                                           buffer.getSpanStart(link[0]),
                                           buffer.getSpanEnd(link[0]));
                }

                return true;
            } else {
                Selection.removeSelection(buffer);
            }
        }

        return super.onTouchEvent(widget, buffer, event);
    }
```

我们直接跳过x、y和Layout的代码，立刻就可以看到这里根据off取出了所有的ClickableSpan，并调用了ClickableSpan的onClick方法，而URLSpan是继承自ClickableSpan的，它就是一个ClickableSpan，这里还有一点要注意的，就是

```
                if (action == MotionEvent.ACTION_UP) {
                    link[0].onClick(widget);
```

它是在 MotionEvent.ACTION_UP的时候才被调用的！
它是在 MotionEvent.ACTION_UP的时候才被调用的！
它是在 MotionEvent.ACTION_UP的时候才被调用的！

关键的事情说三遍！

为什么这个这么关键呢，因为我们找到了一开始提的问题4的根源了，这里其实和View的onLongClick根本没什么合作互动嘛，只要是在手指抬起事件传递过来一定会被调用！

看到这我隐隐的觉得ArrowKeyMovementMethod不用看了，它的onTouchEvent一定和URLSpan没什么毛线关系了，但是标题上都写了从源码的角度理解，装了这么大一个B了，必须得严谨，严谨！

事实证明，的确是毛线关系都没有.....代码就不贴了，好长，真的没关系，各位老爷们不信可以自己去看。

wo cong wei jian guo you ru ci ren zhen cheng shi zhi ren

那如果我们设置了TextView的IsSelectable，mMovement在这里是ArrowKeyMovementMethod的实例，岂不是autoLink点击事件就失效了，其实并不是这样，让我们坐时光机再回到TextView的onTouchEvent方法（绕来绕去你晕不晕！）

```
            if (mMovement != null) {
                handled |= mMovement.onTouchEvent(this, (Spannable) mText, event);
            }
            final boolean textIsSelectable = isTextSelectable();
            if (touchIsFinished && mLinksClickable && mAutoLinkMask != 0 && textIsSelectable) {
                ClickableSpan[] links = ((Spannable) mText).getSpans(getSelectionStart(),
                        getSelectionEnd(), ClickableSpan.class);

                if (links.length > 0) {
                    links[0].onClick(this);
                    handled = true;
                }
             }
```

可以发现，谷歌霸霸其实已经帮我们处理好这种情况了，当textIsSelectable为true的时候，会直接走下面的if把所有的ClickableSpan取出来执行onClick，赞美谷歌霸霸。
这里依然有一点要注意，就是if判断里有一句touchIsFinished，来看一下它是怎么被赋值的

```
      final boolean touchIsFinished = (action == MotionEvent.ACTION_UP) &&
                (mEditor == null || !mEditor.mIgnoreActionUpEvent) && isFocused();
```

恩，一眼就可以看出，又是 MotionEvent.ACTION_UP ，我觉得我已经不用说什么了....

到了这，我们其实已经摸清楚了URLSpan的onClick是怎么被调用的了，也摸清楚了和长按冲突的原因，总结一句话就是无论有没有长按，当MotionEvent.ACTION_UP传入TextView时，一定会触发autoLink的点击事件。

知道了原因，解决起来就简单多了，有一个比较简单的方法就是可以在onLongClick中给TextView设置一个Tag，然后在URLSpan的onClick中取出这个Tag，只要Tag存在，就return，不再执行URLSpan的onClick，这和解决问题1、2、3的思路一样，都需要我们自定义一个URLSpan的子类替代掉原本的URLSpan。

好，下面我们就动手写代码去解决这4个问题

先写一个自己的AutolinkSpan

```
public class AutolinkSpan extends URLSpan {

    public AutolinkSpan(String url) {
        super(url);
    }

    @Override
    public void onClick(View widget) {
        if (widget.getTag(R.id.long_click) != null) {
            widget.setTag(R.id.long_click, null);
            return;
        }
        Intent i = new Intent(widget.getContext(),WebViewActivity.class);
        i.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        widget.getContext().startActivity(i);
    }

    @Override
    public void updateDrawState(TextPaint ds) {
        ds.setColor(ds.getColor());
        ds.setUnderlineText(false);
    }
}
```

当然updateDrawState方法里那两句话其实是两句废话，和不写是一样的........

TextView onLongClick的改动

```
textview.setOnLongClickListener(new OnLongClickListener() {
     @Override
     public boolean onLongClick(View v) {
          textview.setTag(R.id.long_click,true);
          return true;
     }
});
```

最后，是将Linkify添加的URLSpan全部换成我们自己的AutolinkSpan，代码如下：

```
 Spannable spannable = (Spannable) textview.getText();
        URLSpan[] spans = spannable.getSpans(0, spannable.length(), URLSpan.class);
        for (int i = 0; i < spans.length; i++) {
            String url = spans[i].getURL();
            int index = spannable.toString().indexOf(url);
            int end = index + url.length();
            if (index == -1) {
                if (url.contains("http://")) {
                    url = url.replace("http://", "");
                } else if (url.contains("https://")) {
                    url = url.replace("https://", "");
                } else if (url.contains("rtsp://")) {
                    url = url.replace("rtsp://", "");
                }
                index = spannable.toString().indexOf(url);
                end = index + url.length();
            }
            if (index != -1) {
                spannable.removeSpan(spans[i]);
                spannable.setSpan(new AutolinkSpan(spans[i].getURL()), index
                        , end, Spanned.SPAN_INCLUSIVE_INCLUSIVE);
            }
        }
```

一定要在setText之后调用，因为从之前的分析可以知道URLSpan是在setText时才被生成设置的。
至于代码中好几个if的url.contains是因为URLSpan.getURL()拿到的url网址一定是`http://`开头，比如我们在setText的时候传的字符串是 `www.google.com`，URLSpan.getURL()拿到后会变为`http://www.google.com`，这个时候再去`indexOf("http://www.google.com")`肯定会得到-1。至于这是怎么回事，其实很简单，在Linkify中，有一个makeUrl方法，就是它干的好事！代码就不贴了，有兴趣的朋友可以自己去看。







