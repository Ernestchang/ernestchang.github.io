---
title: Android桌面图标实现
date: 

categories: 
- android 
- code
tags:  
- android
- code
---

### 基于[ShortcutBadger](https://github.com/leolin310148/ShortcutBadger)实现

> The ShortcutBadger makes your Android App show the count of unread messages as a badge on your App shortcut!

 ![img](http://upload-images.jianshu.io/upload_images/759172-2b36d5b0302a4a12.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


### ShortcutBadger使用

- 主要实现badge生成和消除功能。

- 一般badge会配合Notifaction使用，故而在生成Notifaction的地方，调用badge生成方法。

- 而当App从后台返回前台时，调用badge清除方法。

### 测试运行效果
- 测试可行厂商

  - 华为
  - 三星

- 需调试厂商

  - [小米](http://dev.xiaomi.com/doc/p=3904/index.html)
    - 小米MIUI 6.0开始，发送通知自动生成桌面角标，且默认桌面角标数badge会根据通知ID个数自动累加，点击应用badge自动消失。同时，其提供开放API，可自定义badge个数。
    - 针对小米，在调用端分开处理。

- 未开通桌面图标厂商

  - 魅族

  - 中兴

  - 酷派

  - OPPO部分应用开通

### 最终代码

- 生成Notifaction和Badge时：

  ```
    /**
       * 生成通知和桌面角标
       */
      private void setFlowNotifaction(Context context, Intent resultIntent, String title, String msg) {

          mBadge++;

          // 小米miui6.0以后根据通知自动生成badge,根据通知次数自动累加。调用ShortcutBadger会自动生成空通知，需调用以下修改badge
          if (Build.MANUFACTURER.equalsIgnoreCase("Xiaomi") && Build.VERSION.SDK_INT > Build.VERSION_CODES.JELLY_BEAN) {
              tryNewMiuiBadge(context, resultIntent, mBadge, title, msg, R.drawable.icon_app);
          } else {
              showNotifacitonBadge(context, resultIntent, mBadge, title, msg, R.drawable.icon_app);
          }
      }
      
      private void showNotifacitonBadge(Context context, Intent resultIntent, int badge, String title, String msg, int drawIcon) {
          PendingIntent pendingIntent = PendingIntent.getActivity(context,
                  0, resultIntent, PendingIntent.FLAG_UPDATE_CURRENT);
          NotificationCompat.Builder mBuilder =
                  new NotificationCompat.Builder(context)
                          .setSmallIcon(drawIcon)
                          .setContentTitle(title)
                          .setContentText(msg)
                          .setTicker(msg);
  //        mBuilder.setFullScreenIntent(pendingIntent, true);
          mBuilder.setContentIntent(pendingIntent);
          mBuilder.setAutoCancel(true);


          // 设置通知的优先级
          mBuilder.setPriority(NotificationCompat.PRIORITY_HIGH);
  //        Uri alarmSound = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION);
  //        // 设置通知的提示音
  //        mBuilder.setSound(alarmSound);
          mBuilder.setDefaults(Notification.DEFAULT_ALL);


          int mNotificationId = 10086;
          NotificationManager mNotifyMgr = (NotificationManager) context
                  .getSystemService(Context.NOTIFICATION_SERVICE);
          mNotifyMgr.notify(mNotificationId, mBuilder.build());
          // 生成桌面角标
          ShortcutBadger.applyCount(JApplication.getInstance(), badge);
      }

      @TargetApi(Build.VERSION_CODES.JELLY_BEAN)
      private void tryNewMiuiBadge(Context context, Intent resultIntent, int badgeCount, String title, String msg, int drawIcon) {

          PendingIntent pendingIntent = PendingIntent.getActivity(context,
                  0, resultIntent, PendingIntent.FLAG_UPDATE_CURRENT);
          NotificationCompat.Builder mBuilder =
                  new NotificationCompat.Builder(context)
                          .setSmallIcon(drawIcon)
                          .setContentTitle(title)
                          .setContentText(msg)
                          .setTicker(msg);
  //        mBuilder.setFullScreenIntent(pendingIntent, true);
          mBuilder.setContentIntent(pendingIntent);
          mBuilder.setAutoCancel(true);


          // 设置通知的优先级
          mBuilder.setPriority(NotificationCompat.PRIORITY_HIGH);
  //        Uri alarmSound = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION);
  //        // 设置通知的提示音
  //        mBuilder.setSound(alarmSound);
          mBuilder.setDefaults(Notification.DEFAULT_ALL);


  //        int mNotificationId = 10086;
          NotificationManager mNotifyMgr = (NotificationManager) context
                  .getSystemService(Context.NOTIFICATION_SERVICE);
          Notification notification = mBuilder.build();

          try {
              Field field = notification.getClass().getDeclaredField("extraNotification");
              Object extraNotification = field.get(notification);
              Method method = extraNotification.getClass().getDeclaredMethod("setMessageCount", int.class);
              method.invoke(extraNotification, badgeCount);

          } catch (Exception e) {
              e.printStackTrace();
          }

          mNotifyMgr.notify(0, notification);
      }

  ```

  ​

- 消除Badge时：

  > App从后台返回前台都需要清除badge，故而现在Activity基类中做统一检测处理

  ```
     @Override
      protected void onResume() {
          super.onResume();
          MobclickAgent.onResume(this);

          // 小米miui6.0后，会自动清除badge，调用ShortcutBadger会自动生成空通知
          if (mBadge > 0) {
              if (Build.MANUFACTURER.equalsIgnoreCase("Xiaomi") && Build.VERSION.SDK_INT > Build.VERSION_CODES.JELLY_BEAN) {
                  mBadge = 0;

              } else {
                  mBadge = 0;
                  ShortcutBadger.applyCount(this, 0);
              }
          }
      }
  ```