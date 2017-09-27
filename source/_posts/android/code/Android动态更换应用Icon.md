---
title: Android动态更换应用Icon
date: 

categories: 
- android 
- code
tags:  
- android
- code
---

## 动态更换应用Icon

> From stackoverflow: [How to change an application icon programmatically in Android?](https://stackoverflow.com/questions/1103027/how-to-change-an-application-icon-programmatically-in-android) 

Try this, it works fine for me =)

1 . Modify your MainActivity section in AndroidManifest.xml, delete from it, line with MAIN category in intent-filter section

```
<activity android:name="ru.quickmessage.pa.MainActivity"
    android:configChanges="keyboardHidden|orientation"
    android:screenOrientation="portrait"
    android:label="@string/app_name"
    android:theme="@style/CustomTheme"
    android:launchMode="singleTask">
    <intent-filter>
         <action android:name="android.intent.action.MAIN" />
     ==> <category android:name="android.intent.category.LAUNCHER" /> <== Delete this line
    </intent-filter>
</activity>
```

2 . Create `<activity-alias>`, for each of your icons. Like this

```
<activity-alias android:label="@string/app_name" 
    android:icon="@drawable/icon" 
    android:name=".MainActivity-Red"
    android:enabled="false"
    android:targetActivity=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>   
</activity-alias>
```

3 . Set programmatically: set ENABLE attribute for the appropriate activity-alias

> Should use following code to disable all icons before enabling another one, otherwise it will add a new icon, instead of replacing it.

```
 getPackageManager().setComponentEnabledSetting(
        new ComponentName("ru.quickmessage.pa", "ru.quickmessage.pa.MainActivity-Red"), 
            PackageManager.COMPONENT_ENABLED_STATE_ENABLED, PackageManager.DONT_KILL_APP);
```

**Note, At least one must be enabled at all times.**



As Display:

### 原理1——activity-alias

在AndroidMainifest中，有两个属性：

```
// 决定应用程序最先启动的Activity
android.intent.action.MAIN 
// 决定应用程序是否显示在程序列表里
android.intent.category.LAUNCHER 
```

另外，还有一个activity-alias属性，这个属性可以用于创建多个不同的入口，相信做过系统Setting和Launcher开发的开发者在系统的源码中应该见过很多。

### 原理2——PM.setComponentEnabledSetting

PackageManager是一个大统领类，可以管理所有的系统组件，当然，如果Root了，你还可以管理其它App的所有组件，一些系统优化工具就是通过这个方式来禁用一些后台Service的。

使用方式异常简单：

```
private void enableComponent(ComponentName componentName) {
    mPm.setComponentEnabledSetting(componentName,
            PackageManager.COMPONENT_ENABLED_STATE_ENABLED,
            PackageManager.DONT_KILL_APP);
}

private void disableComponent(ComponentName componentName) {
    mPm.setComponentEnabledSetting(componentName,
            PackageManager.COMPONENT_ENABLED_STATE_DISABLED,
            PackageManager.DONT_KILL_APP);
}
```

根据PackageManager.COMPONENT_ENABLED_STATE_ENABLED和PackageManager.COMPONENT_ENABLED_STATE_DISABLED这两个标志量和对应的ComponentName，就可以控制一个组件的是否启用。

## 动态换Icon

有了上面的两个原理，来实现动态更换Icon就只剩下思路问题了。

首先，我们创建一个Activity，再创建一个带着默认图片的activity-alias，指向默认的Activity并带有正常的图片，再创建一个双12的activity-alias，指向默认的Activity并带有双12的图片……等等等。

```
<activity android:name=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>
    </intent-filter>
</activity>

<activity-alias
    android:name=".normal"
    android:enabled="true"
    android:icon="@drawable/normal"
    android:label="normal"
    android:targetActivity=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>

        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity-alias>

<activity-alias
    android:name=".Test12"
    android:enabled="false"
    android:icon="@drawable/s12"
    android:label="双12"
    android:targetActivity=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN"/>

        <category android:name="android.intent.category.LAUNCHER"/>
    </intent-filter>
</activity-alias>
```



```
public class MainActivity extends AppCompatActivity {

    private ComponentName mDefault;
    private ComponentName mDouble11;
    private ComponentName mDouble12;
    private PackageManager mPm;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        mDefault = new ComponentName(
                getBaseContext(),
                "com.xys.changeicon.normal");
        mDouble12 = new ComponentName(
                getBaseContext(),
                "com.xys.changeicon.Test12");
        mPm = getApplicationContext().getPackageManager();
    }

    public void changeIcon12(View view) {
        disableComponent(mDefault);
        enableComponent(mDouble12);
    }

    private void enableComponent(ComponentName componentName) {
        mPm.setComponentEnabledSetting(componentName,
                PackageManager.COMPONENT_ENABLED_STATE_ENABLED,
                PackageManager.DONT_KILL_APP);
    }

    private void disableComponent(ComponentName componentName) {
        mPm.setComponentEnabledSetting(componentName,
                PackageManager.COMPONENT_ENABLED_STATE_DISABLED,
                PackageManager.DONT_KILL_APP);
    }
}
```

OK了，禁用默认的Activity后，启用双12的activity-alias，结果不变还是指向了默认的Activity，但图标已经发生了改变。

> 根据ROM的不同，在禁用了组件之后，会等一会，Launcher会自动刷新图标。如果在更换图标期间，用户点击桌面图标，会暂时提示应用不存在。

