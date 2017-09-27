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

### God say

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



## 动态更换Icon

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



## 获取组件当前状态

### 状态说明

**'packageManager.getComponentEnabledSetting'**：

- PackageManager.COMPONENT_ENABLED_STATE_DEFAULT

> 组件默认状态，未调用'mPm.setComponentEnabledSetting()'设置组件，则组件为此状态。如以上代码，mDefalut, mDouble12刚开始均为此状态。

- PackageManager.COMPONENT_ENABLED_STATE_ENABLED

> 组件可用状态，调用mPm.setComponentEnabledSetting()'设置组件开启，则为此状态。

- PackageManager.COMPONENT_ENABLED_STATE_DISABLED

> 组件不可用状态，调用mPm.setComponentEnabledSetting()'设置组件关闭，则为此状态。



### 获取当前组件状态

如果设置一个app的启动MainActivity为COMPONENT_ENABLED_STATE_DISABLED状态，则不会再launcher的程序图标中发现该app

 ```
		PackageManager packageManager = getPackageManager();
        ComponentName componentName = new ComponentName(this, StartActivity.class);
        int res = packageManager.getComponentEnabledSetting(componentName);
  	
        if (res == PackageManager.COMPONENT_ENABLED_STATE_DEFAULT
                || res == PackageManager.COMPONENT_ENABLED_STATE_ENABLED) {
            // 隐藏应用图标
            packageManager.setComponentEnabledSetting(componentName, PackageManager.COMPONENT_ENABLED_STATE_DISABLED,
                    PackageManager.DONT_KILL_APP);
        } else {
            // 显示应用图标
            packageManager.setComponentEnabledSetting(componentName, PackageManager.COMPONENT_ENABLED_STATE_DEFAULT,
                    PackageManager.DONT_KILL_APP);
        }
 ```



 ### 动态更换Icon完善

1. 设置两个`<activity-alias`，`SplashActivity_Normal`默认`android:enabled="true"`启用，`SplashActivity_Festival` 默认关闭,  初始化默认组件状态均为`PackageManager.COMPONENT_ENABLED_STATE_DEFAULT`.
2. `isShowFestivalIcons`后台控制，默认`false`。而`SplashActivity_Festival`默认为`PackageManager.COMPONENT_ENABLED_STATE_DEFAULT`，故而不调用更换Icon代码。
3. 当`isShowFestivalIcons`为true, 调用更换Icon代码，设置`SplashActivity_Normal` 关闭，`SplashActivity_Festival`启用。
4.  当`isShowFestivalIcons`为false, 调用更换Icon代码，设置`SplashActivity_Normal` 启用，`SplashActivity_Festival`关闭。

```
		
        if (isShowFestivalIcons) {

            if (!CommonUtil.isComponentEnable(CommonUtil.LAUCH_COMPONENT_FESTIVAL)) {
            	// 更换Icon
                CommonUtil.disableComponent(CommonUtil.LAUCH_COMPONENT_NORMAL);
                CommonUtil.enableComponent(CommonUtil.LAUCH_COMPONENT_FESTIVAL);
            }
            
            // to show festival icons

        } else {
			
            if (CommonUtil.isComponentEnable(CommonUtil.LAUCH_COMPONENT_FESTIVAL)) {
            	// 更换Icon
                CommonUtil.disableComponent(CommonUtil.LAUCH_COMPONENT_FESTIVAL);
                CommonUtil.enableComponent(CommonUtil.LAUCH_COMPONENT_NORMAL);
            }
            
             // to show normal icons
             
        }
        
 public  boolean isComponentEnable(String componentName) {
    	int isEnabled = getPackageManager().getComponentEnabledSetting(new ComponentName(getPackageName(), componentName));
        return isEnabled == PackageManager.COMPONENT_ENABLED_STATE_ENABLED;
    }
    
    
```



