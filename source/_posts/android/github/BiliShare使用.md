# BiliShare
[ ![Download](https://api.bintray.com/packages/jungerr/maven/biliShare/images/download.svg)](https://bintray.com/jungerr/maven/biliShare/_latestVersion)

支持分享到微博、QQ聊天、QQ空间、微信聊天、微信朋友圈，系统分享等。

QQ交流群：397462257

## 预览
 - 下载demo [bilibili][1]
 - screenshot

 ![screenshot][2]

## 使用姿势

### 配置

 - 在build.gradle里添加依赖.
 "biliShare"是核心库(必需)，"biliShare-util"是分享的选择器(非必需)，如上截图所示.

```

allprojects {
    repositories {
        jcenter()
        maven { url "https://dl.bintray.com/thelasterstar/maven/"}
    }
}

dependencies {
    compile 'com.jungly.socialize:biliShare:0.1.0701' //必需
    compile 'com.jungly.socialize:biliShare-util:0.1.0701@aar' //非必需
}
```

 - 配置QQ分享，在AndroidManifest文件里添加如下配置，注意在scheme里添加你的appId。

```
<activity
    android:name="com.tencent.tauth.AuthActivity"
    android:launchMode="singleTask"
    android:noHistory="true">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <data android:scheme="tencent你的AppId" />
   </intent-filter>
</activity>
```

 - 配置微信分享，在{root package}/wxapi/下添加WXEntryActivity，并且配置到AndroidManifest文件里。

```java
public class WXEntryActivity extends BaseWXEntryActivity {
    @Override
    protected String getAppId() {
        return .....;
    }
}
```
```xml
<activity
    android:name=".wxapi.WXEntryActivity"
    android:configChanges="keyboardHidden|orientation|screenSize"
    android:exported="true"
    android:screenOrientation="portrait"
    android:theme="@android:style/Theme.Translucent.NoTitleBar"/>
```

### 使用

 - 示例代码

 ```java
 BiliShareConfiguration configuration = new BiliShareConfiguration.Builder(context)
                .sina(appKey, redirectUrl, scope) //配置新浪
                .qq(appId) //配置qq
                .weixin(appId) //配置微信
                .imageDownloader(new ShareFrescoImageDownloader()) //图片下载器
                .build();

    //global client全局共用，也可以用BiliShare.get(name)获取一个特定的client，以便业务隔离。
    BiliShare shareClient = BiliShare.global();
    shareClient.config(configuration); //config只需要配置一次

    shareClient.share(context, socializeMedia, shareParam, shareListener);
 ```

 - 具体参考/sample/src/main/java/com/bilibili/socialize/sample/MainActivity.class

## 版本
|版本|时间|变更|
| :-- | :-- | :-- |
|0.1.0701|2017-05-05|1，升级微博SDK至2.0.3；<br/>2，升级QQ SDK至5788；<br/>3，升级微信SDK至1.1.6；<br/>4，去除BiliShare的onActivityResult()。<br/>5，解决若干bug。|
|0.1.06|2017-04-24|支持多BiliShare实例，方便多业务隔离|
|......|||


License
---

    Copyright 2015-2017 Bilibili

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.


  [1]: http://wsdownload.hdslb.net/app/BiliPlayer3.apk
  [2]: http://7qnau5.com1.z0.glb.clouddn.com/Screenshot_2016-04-26-00-13-35.png?imageView2/1/w/360/h/640
