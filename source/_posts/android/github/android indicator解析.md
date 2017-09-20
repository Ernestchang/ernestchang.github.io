title: android indicator解析
date: 

categories: 
- android
- github

tags: 
- github
- anslysis
- indicator

---

### 解析对象

- [BGAIndicator-Android](https://github.com/bingoogolapple/BGAIndicator-Android)  滑动指示器


- [PagerSlidingTabStrip](https://github.com/ta893115871/PagerSlidingTabStrip) Android-导航栏特效，新闻类APP(仿iOS 版的网易，今日头条效果)，主要是导航栏字体大小和颜色的渐变特效

- [ViewPagerIndicator](https://github.com/LuckyJayce/ViewPagerIndicator) Indicator 取代 tabhost，实现网易顶部tab，新浪微博主页底部tab，引导页，无限轮播banner等效果，高度自定义tab和特效

- [ColorTrackTabLayout](https://github.com/yewei02538/ColorTrackTabLayout) 一个自适应Tab宽度，可以滑动文字逐渐变色的TabLayout

- [ColorTrackView](https://github.com/hongyangAndroid/ColorTrackView) 字体或者图片可以逐渐染色和逐渐褪色的动画效果

  ​



#### [BGAIndicator-Android](https://github.com/bingoogolapple/BGAIndicator-Android)  

支持Indicator和ViewPager宽度不相等时也能正确运行的滑动指示器

- 效果图

![Image of BGAFixedIndicatorDemo](http://bingoshare.u.qiniudn.com/BGAFixedIndicatorDemo.gif)

- 自定义属性说明

```
<declare-styleable name="BGAIndicator">
    <attr name="indicator_textColor" format="color|reference" />
    <attr name="indicator_textSizeNormal" format="dimension|reference" />
    <attr name="indicator_textSizeSelected" format="dimension|reference" />
    <attr name="indicator_triangleHorizontalMargin" format="dimension|reference" />
    <attr name="indicator_triangleColor" format="color|reference" />
    <attr name="indicator_triangleHeight" format="dimension|reference" />
    <attr name="indicator_hasDivider" format="boolean" />
    <attr name="indicator_dividerColor" format="color|reference" />
    <attr name="indicator_dividerWidth" format="dimension|reference" />
    <attr name="indicator_dividerVerticalMargin" format="dimension|reference" />
</declare-styleable>
```

- 功能
  - 主要实现单屏viewpager滑动指示器展示
  - 不支持指示器超过一屏的滑动，（包裹HorizontalScrollView后，跟随页面切换，滑动指示器的联动亦是问题）

- 实现

  - 继承自LinearLayout

    ```
    public class BGAFixedIndicator extends LinearLayout implements View.OnClickListener, View.OnFocusChangeListener {
        private static final String TAG = BGAFixedIndicator.class.getSimpleName();
    ```

  - 监听ViewPager滑动事件，实现滑动指示器绘制

    ```
    // 初始化选项卡
    public void initData(int currentTab, ViewPager viewPager) {
        removeAllViews();

        mViewPager = viewPager;
        mTabCount = mViewPager.getAdapter().getCount();

        mViewPager.addOnPageChangeListener(new ViewPager.SimpleOnPageChangeListener() {
            @Override
            public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
                mTriangleLeftX = (int) (mItemWidth * (position + positionOffset));
                postInvalidate();
            }

            @Override
            public void onPageSelected(int position) {
                setCurrentTab(position);
            }
        });

        initTab(currentTab);
        postInvalidate();
    }
    ```

  - 初始化添加tabs

    ```
    private void initTab(int currentTab) {
        for (int index = 0; index < mTabCount; index++) {
            View tabIndicator = mInflater.inflate(R.layout.view_indicator, this, false);
            tabIndicator.setId(BSSEEID + index);
            tabIndicator.setOnClickListener(this);

            TextView titleTv = (TextView) tabIndicator.findViewById(R.id.tv_indicator_title);
            if (mTextColor != null) {
                titleTv.setTextColor(mTextColor);
            }
            titleTv.setTextSize(TypedValue.COMPLEX_UNIT_PX, mTextSizeNormal);
            titleTv.setText(mViewPager.getAdapter().getPageTitle(index));

            LayoutParams tabLp = new LayoutParams(0, LayoutParams.MATCH_PARENT, 1);
            tabLp.gravity = Gravity.CENTER;
            tabIndicator.setLayoutParams(tabLp);
            // 防止currentTab为0时，第一个tab文字颜色没变化
            if (index == 0) {
                resetTab(tabIndicator, true);
            }
            this.addView(tabIndicator);

            if (index != mTabCount - 1 && mHasDivider) {
                LayoutParams dividerLp = new LayoutParams(mDividerWidth, LayoutParams.MATCH_PARENT);
                dividerLp.setMargins(0, mDividerVerticalMargin, 0, mDividerVerticalMargin);
                View vLine = new View(getContext());
                vLine.setBackgroundColor(mDividerColor);
                vLine.setLayoutParams(dividerLp);
                this.addView(vLine);
            }
        }
        setCurrentTab(currentTab);
    }
    ```

  - 切换tabs

    ```
    private void setCurrentTab(int index) {
        if (mCurrentTabIndex != index && index > -1 && index < mTabCount) {
            View oldTab = findViewById(BSSEEID + mCurrentTabIndex);
            resetTab(oldTab, false);

            mCurrentTabIndex = index;
            View newTab = findViewById(BSSEEID + mCurrentTabIndex);
            resetTab(newTab, true);

            if (mViewPager.getCurrentItem() != mCurrentTabIndex) {
                mViewPager.setCurrentItem(mCurrentTabIndex, false);
            }
            postInvalidate();
        }
    }

    private void resetTab(View tab, boolean isSelected) {
        TextView tv = (TextView) tab.findViewById(R.id.tv_indicator_title);
        tv.setTextSize(TypedValue.COMPLEX_UNIT_PX, isSelected ? mTextSizeSelected : mTextSizeNormal);
        tab.setSelected(isSelected);
        tab.setPressed(isSelected);
    }
    ```

  - 绘制滑动下划线

    ```
    // 注意：必须要设置背景后，该方法才会被调用
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        mPath.rewind();
        float left_x = mTriangleHorizontalMargin + mTriangleLeftX;
        float right_x = mTriangleLeftX + mItemWidth - mTriangleHorizontalMargin;
        float top_y = getHeight() - mTriangleHeight;
        float bottom_y = getHeight();

        mPath.moveTo(left_x, top_y);
        mPath.lineTo(right_x, top_y);
        mPath.lineTo(right_x, bottom_y);
        mPath.lineTo(left_x, bottom_y);
        mPath.close();

        canvas.drawPath(mPath, mPaintFooterTriangle);
    }

    @Override
    protected void onSizeChanged(int w, int h, int oldw, int oldh) {
        super.onSizeChanged(w, h, oldw, oldh);
        mItemWidth = getWidth();
        if (mTabCount != 0) {
            mItemWidth = getWidth() / mTabCount;
        }
    }
    ```

  - 监听切换页面

    ```
    @Override
    public void onClick(View v) {
        int currentTabIndex = v.getId() - BSSEEID;
        setCurrentTab(currentTabIndex);
    }

    // 当滑动指示器重新获取焦点时（比如从上一页返回当前页），立即初始化当前选中状态
    @Override
    public void onFocusChange(View v, boolean hasFocus) {
        if (v == this && hasFocus) {
            findViewById(BSSEEID + mCurrentTabIndex).requestFocus();
            return;
        } else if (hasFocus) {
            for (int i = 0; i < mTabCount; i++) {
                if (getChildAt(i) == v) {
                    setCurrentTab(i);
                    return;
                }
            }
        }
    }
    ```



#### [PagerSlidingTabStrip](https://github.com/ta893115871/PagerSlidingTabStrip) 

Android-导航栏特效，主要是导航栏字体大小和颜色的渐变特效
可以是固定的几个,可也可以是水平滚动
-  效果图
   <img src="http://img.blog.csdn.net/20160110124059659?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center"/>

-  自定义属性说明

```
   app:pstsIndicatorColor  指示器的颜色
   app:pstsIndicatorHeight 指示器的高度
   app:pstsUnderlineColor 底部线的颜色
   app:pstsUnderlineHeight 底部线的高度
   app:pstsDividerColor 分割线的颜色
   app:pstsDividerPaddingTopBottom 分割线的上下间距
   app:pstsTabPaddingLeftRight 文本的左右间距
   app:pstsTextSelectedColor TAB选中的颜色
   app:pstsScrollOffset
   app:pstsTabBackground 每一个TAB的背景
   <!--该属性表示里面的TAB是否均分整个PagerSlidingTabStrip控件的宽,true是,false不均分,从左到右排列,默认false-->
   app:pstsShouldExpand
   app:pstsTextAllCaps 所有的小写英文文本自动大写 ,默认是true,默认大写
   <!--缩放的最大值,0.3表示放大后最大是原来的0.3倍,默认为0.3-->
   app:pstsScaleZoomMax
   android:textColor="@color/color_45c01a" 正常状态的文字颜色
   android:textSize="16sp" 正常状态的文字的大小
   app:pstsSmoothScrollWhenClickTab="false" 当点击tab时内容区域Viewpager是否是左右滑动,默认是true
```

```
<com.gxz.PagerSlidingTabStrip
    android:id="@+id/tabs"
    android:layout_width="match_parent"
    android:layout_height="40dp"
    android:textColor="@color/color_45c01a"
    android:textSize="16sp"
    app:pstsDividerColor="@android:color/transparent"
    app:pstsIndicatorColor="@color/accent_material_light"
    app:pstsIndicatorHeight="5dp"
    app:pstsShouldExpand="false"
    app:pstsTextSelectedColor="@color/accent_material_light"
    app:pstsUnderlineColor="@color/colorAccent" />
```

- 功能
  - 基本实现类似头条滑动指示器，可多屏滑动展示

- 实现

   - 继承HorizonalScrollView

   ```
   public class PagerSlidingTabStrip extends HorizontalScrollView {
       public interface IconTabProvider {
           int getPageIconResId(int position);
       }
   ```
   - 监听viewpager滑动，绘制指示器

   ```
   private class PageListener implements OnPageChangeListener {
       private int oldPosition = 0;

       @Override
       public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
           currentPosition = position;
           currentPositionOffset = positionOffset;

           if (tabsContainer != null && tabsContainer.getChildAt(position) != null) {
               scrollToChild(position, (int) (positionOffset * tabsContainer.getChildAt(position).getWidth()));
           }

           invalidate();

           if (delegatePageListener != null) {
               delegatePageListener.onPageScrolled(position, positionOffset, positionOffsetPixels);
           }

           if (mState == State.IDLE && positionOffset > 0) {
               oldPage = pager.getCurrentItem();
               mState = position == oldPage ? State.GOING_RIGHT : State.GOING_LEFT;
           }
           boolean goingRight = position == oldPage;
           if (mState == State.GOING_RIGHT && !goingRight)
               mState = State.GOING_LEFT;
           else if (mState == State.GOING_LEFT && goingRight)
               mState = State.GOING_RIGHT;


           float effectOffset = isSmall(positionOffset) ? 0 : positionOffset;


           View mLeft = tabsContainer.getChildAt(position);
           View mRight = tabsContainer.getChildAt(position + 1);


           if (effectOffset == 0) {
               mState = State.IDLE;
           }

           if (mFadeEnabled)
               animateFadeScale(mLeft, mRight, effectOffset, position);

       }

       @Override
       public void onPageScrollStateChanged(int state) {
           if (state == ViewPager.SCROLL_STATE_IDLE) {
               scrollToChild(pager.getCurrentItem(), 0);
               mFadeEnabled = true;
           }
           if (delegatePageListener != null) {
               delegatePageListener.onPageScrollStateChanged(state);
           }
       }

       @Override
       public void onPageSelected(int position) {
           selectedPosition = position;

           //set old view statue
           ViewHelper.setAlpha(tabViews.get(oldPosition).get("normal"), 1);
           ViewHelper.setAlpha(tabViews.get(oldPosition).get("selected"), 0);
           View v_old = tabsContainer.getChildAt(oldPosition);
           ViewHelper.setPivotX(v_old, v_old.getMeasuredWidth() * 0.5f);
           ViewHelper.setPivotY(v_old, v_old.getMeasuredHeight() * 0.5f);
           ViewHelper.setScaleX(v_old, 1f);
           ViewHelper.setScaleY(v_old, 1f);

           //set new view statue
           ViewHelper.setAlpha(tabViews.get(position).get("normal"), 0);
           ViewHelper.setAlpha(tabViews.get(position).get("selected"), 1);
           View v_new = tabsContainer.getChildAt(position);
           ViewHelper.setPivotX(v_new, v_new.getMeasuredWidth() * 0.5f);
           ViewHelper.setPivotY(v_new, v_new.getMeasuredHeight() * 0.5f);
           ViewHelper.setScaleX(v_new, 1 + zoomMax);
           ViewHelper.setScaleY(v_new, 1 + zoomMax);

           if (delegatePageListener != null) {
               delegatePageListener.onPageSelected(position);
           }
           oldPosition = selectedPosition;
       }
   }
   ```

#### [ColorTrackTabLayout](https://github.com/yewei02538/ColorTrackTabLayout) 
一个自适应Tab宽度，可以滑动文字逐渐变色的`TabLayout`,该控件继承自TabLayout做了一些扩展。

- 效果
  ![](https://raw.githubusercontent.com/yewei02538/ColorTrackTabLayout/master/screenshot/screenshot.gif)

- 自定义属性

```
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_today_news_acitivty"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context="me.weyye.colortracktablayout.TodayNewsAcitivty">
    
    <me.weyye.library.colortrackview.ColorTrackTabLayout
        android:id="@+id/tab"
        android:layout_width="match_parent"
        android:layout_height="40dp"
        android:background="#ffffff"
        app:tabIndicatorColor="@color/red"
        app:tabMode="scrollable"
        app:tabSelectedTextColor="@color/red"
        app:tabTextAppearance="@style/TabStyle"
        app:tabTextColor="@color/black"/>
    
    <android.support.v4.view.ViewPager
        android:id="@+id/viewPager"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
</LinearLayout>
```
- 使用

        titles = new String[]{"推荐", "视频", "社会"};
        final List<Fragment> fragments = new ArrayList<>();
        for (int i = 0; i < titles.length; i++) {
            fragments.add(MyFragment.newInstance());
        }
    
        mViewPager.setAdapter(new FragmentPagerAdapter(getSupportFragmentManager()) {
            @Override
            public Fragment getItem(int position) {
                return fragments.get(position);
            }
    
            @Override
            public int getCount() {
                return titles.length;
            }
    
            @Override
            public CharSequence getPageTitle(int position) {
                return titles[position];
            }
        });
      //隐藏指示器
      mTab.setSelectedTabIndicatorHeight(0);
      //设置每个Tab的内边距
      mTab.setTabPaddingLeftAndRight(20, 20);
      mTab.setupWithViewPager(mViewPager);
- 实现

  继承自TabLayout

  ```
  public class ColorTrackTabLayout extends TabLayout {
      private int mTabTextSize;
      private int mTabSelectedTextColor;
      private int mTabTextColor;
  ```

  既然是随着页面的滑动文字颜色渐变那么肯定少不了ViewPager的页面监听，这个在我们调用`setupWithViewPager`的时候TabLayout就已经添加监听。

```
    private void setupWithViewPager(@Nullable final ViewPager viewPager, boolean autoRefresh,
            boolean implicitSetup) {
        ....

        if (viewPager != null) {
            mViewPager = viewPager;

            // Add our custom OnPageChangeListener to the ViewPager
            if (mPageChangeListener == null) {
                //添加了滑动监听
                mPageChangeListener = new TabLayoutOnPageChangeListener(this);
            }
            mPageChangeListener.reset();
            viewPager.addOnPageChangeListener(mPageChangeListener);

            // Now we'll add a tab selected listener to set ViewPager's current item
            mCurrentVpSelectedListener = new ViewPagerOnTabSelectedListener(viewPager);
            addOnTabSelectedListener(mCurrentVpSelectedListener);

            final PagerAdapter adapter = viewPager.getAdapter();
            if (adapter != null) {
                // Now we'll populate ourselves from the pager adapter, adding an observer if
                // autoRefresh is enabled
                setPagerAdapter(adapter, autoRefresh);
            }

            // Add a listener so that we're notified of any adapter changes
            if (mAdapterChangeListener == null) {
                mAdapterChangeListener = new AdapterChangeListener();
            }
            mAdapterChangeListener.setAutoRefresh(autoRefresh);
            viewPager.addOnAdapterChangeListener(mAdapterChangeListener);

            // Now update the scroll position to match the ViewPager's current item
            setScrollPosition(viewPager.getCurrentItem(), 0f, true);
        } else {
            // We've been given a null ViewPager so we need to clear out the internal state,
            // listeners and observers
            mViewPager = null;
            setPagerAdapter(null, false);
        }
        ....

    }
```

​	创建了mPageChangeListener并添加了监听

`mPageChangeListener = new TabLayoutOnPageChangeListener(this);`，所以我们必须要重写`setupWithViewPager`删除掉原来的监听，换成我们自己的监听

```
    @Override
    public void setupWithViewPager(@Nullable ViewPager viewPager, boolean autoRefresh) {
        super.setupWithViewPager(viewPager, autoRefresh);
        try {
            //通过反射找到mPageChangeListener
            Field field = TabLayout.class.getDeclaredField("mPageChangeListener");
            field.setAccessible(true);
            TabLayoutOnPageChangeListener listener = (TabLayoutOnPageChangeListener) field.get(this);
            if (listener != null) {
                //删除自带监听
                viewPager.removeOnPageChangeListener(listener);
                mPageChangeListenter = new ColorTrackTabLayoutOnPageChangeListener(this);
                mPageChangeListenter.reset();
                viewPager.addOnPageChangeListener(mPageChangeListenter);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }


    }
```

​	还需要做的一点就是把TabLayout每一个的Tab布局替换成我们的。怎么替换呢？重写`addTab`，在添加的时候改成我们的布局

```
    @Override
    public void addTab(@NonNull Tab tab, int position, boolean setSelected) {
        ColorTrackView colorTrackView = new ColorTrackView(getContext());
        colorTrackView.setProgress(setSelected ? 1 : 0);
        colorTrackView.setText(tab.getText() + "");
        colorTrackView.setTextSize(mTabTextSize);
        colorTrackView.setTag(position);
        colorTrackView.setTextChangeColor(mTabSelectedTextColor);
        colorTrackView.setTextOriginColor(mTabTextColor);
        LinearLayout.LayoutParams layoutParams = new LinearLayout.LayoutParams(LinearLayout.LayoutParams.WRAP_CONTENT, LinearLayout.LayoutParams.WRAP_CONTENT);
        colorTrackView.setLayoutParams(layoutParams);
        tab.setCustomView(colorTrackView);

        super.addTab(tab, position, setSelected);
        if (position == 0) {
            //默认选中第一个
            setSelectedView(position);
        }

        setTabWidth(position, colorTrackView);
    }
```

​	其关键就是将我们的布局利用setCustomView方法来设置上去。























