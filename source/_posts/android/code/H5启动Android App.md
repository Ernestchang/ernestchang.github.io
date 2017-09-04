---
title: H5启动Android App
date: 

categories: 
- android 
- code
tags:  
- android
- code
---

> 首先明晰，[微信](http://lib.csdn.net/base/wechat)里屏蔽了schema协议。除非你是微信的合作伙伴之类的，他们专门给你配置进白名单。否则，我们就没办法通过这个协议在微信中直接唤起app。
>
> 因此，我们需要先判断页面场景是否在微信中，如果在微信中，则会提示用户在浏览器中打开。

### H5中间接判断应用是否安装

> 这里的逻辑很简单，当没有成功打开app的时候，新页面不会弹出则页面逻辑继续进行；否则如果进入了新页面，则页面逻辑便终止了。所以我们可以另开一个延时的线程来判断这个事情

```
if(...){
document.location = '';
setTimeout(function(){
   //此处如果执行则表示没有app
},200);
}
```


### 通过H5唤起APP

> 编辑AndroidManifest.xml，主要是增加第二个<intent-filter>，launchapp用来标识schema，最好能保证手机系统唯一，那样就可以打开应用，而不是弹出一个选择框。可以附带自己的数据通过string传递到activity，比如完整url为 launchapp://?data=mydata

```
<activity    
     android:name="com.robert.MainActivity"    
     android:configChanges="orientation|keyboardHidden|navigation|screenSize"    
     android:screenOrientation="landscape"    
     android:theme="@android:style/Theme.NoTitleBar.Fullscreen" >    
     <intent-filter>    
         <action android:name="android.intent.action.MAIN" />    
         <category android:name="android.intent.category.LAUNCHER" />    
     </intent-filter>    
     <intent-filter>    
         <action android:name="android.intent.action.VIEW" />    
         <category android:name="android.intent.category.BROWSABLE" />    
         <category android:name="android.intent.category.DEFAULT"/>    
         <data android:scheme="launchapp" android:pathPrefix="/haha" />    
     </intent-filter>    
   </activity>   
```


  > 然后通过activity获得data数据：

   ``` 
     public void onCreate(Bundle savedInstanceState) {             
     	Uri uridata = this.getIntent().getData();             
     	String mydata = uridata.getQueryParameter("data");            
     }  
   ```


 ### 编写html页面 
 > 整个页面也许是某个app的详细介绍，这里只写出关键的js代码：

```
<activity  
     android:name="com.robert.MainActivity"    
     android:configChanges="orientation|keyboardHidden|navigation|screenSize"    
     android:screenOrientation="landscape"    
     android:theme="@android:style/Theme.NoTitleBar.Fullscreen" >    
     <intent-filter>    
         <action android:name="android.intent.action.MAIN" />    
         <category android:name="android.intent.category.LAUNCHER" />    
     </intent-filter>    
     <intent-filter>    
         <action android:name="android.intent.action.VIEW" />    
         <category android:name="android.intent.category.BROWSABLE" />    
         <category android:name="android.intent.category.DEFAULT"/>    
         <data android:scheme="launchapp" android:pathPrefix="/haha" />    
     </intent-filter>    
</activity>   
```

 > 下面代码可以达到这样一个目的，先请求 launchapp:// ,如果系统能处理，或者说已经安装了myapp表示的应用，那么就可以打开，另外，如果不能打开，直接刷新一下当前页面，等于是重置location。

```
function openApp() {    
       
           if (/android/i.test(navigator.userAgent)) {    
                var isrefresh = getUrlParam('refresh'); // 获得refresh参数    
                if(isrefresh == 1) {    
                    return    
                }    
                window.location.href = 'launchapp://haha?data=mydata';    
                window.setTimeout(function () {    
                        window.location.href += '&refresh=1' // 附加一个特殊参数，用来标识这次刷新不要再调用myapp:// 了    
                }, 500);    
            }    
       
   }  

```

###  整体代码

> 代码功能: 判断手机/平板是否安装app 如果安装 则调用app的scheme，传入url当作参数，来做后续操作 如果没有安装 则跳转到app store/google play 下载app


```
(function () {
    var openUrl = window.location.search;
    try {
        openUrl = openUrl.substring(1, openUrl.length);
    } catch (e) {}
    var isiOS = navigator.userAgent.match('iPad') || navigator.userAgent.match('iPhone') || navigator.userAgent.match(
        'iPod'),
        isAndroid = navigator.userAgent
            .match('Android'),
        isDesktop = !isiOS && !isAndroid;
    if (isiOS) {
        setTimeout(function () {
            window.location = "itms-apps://itunes.apple.com/app/[name]/[id]?mt=8";
        }, 25);
        window.location = "[scheme]://[host]?url=" + openUrl;
    } else if (isAndroid) {
        window.location = "intent://[host]/" + "url=" + openUrl + "#Intent;scheme=[scheme];package=[package_name];end";
    } else {
        window.location.href = openUrl;
    }
})();
```