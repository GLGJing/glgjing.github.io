---
layout: post
title: "Android 开发之 Notification 详解"
date: 2015-11-18 20:02:27 +0800
comments: true
categories: android, Notification
---

### 通知（Notification）的各种展现形式

#### 通知展现位置
1. 以图标的形式显示在通知区域中    
{% img /images/notify_1.png %}
2. 详细信息展示在抽屉式通知栏中    
{% img /images/notify_2.png %}
3. 以浮动形式通知    
{% img /images/notify_3.png %}
4. 在锁屏上展示通知    
{% img /images/notify_4.png %}

#### 通知展现样式
1. 标准样式    
{% img /images/notify_5.png %}
2. 扩展样式    
{% img /images/notify_6.png %}
3. 自定义样式    
{% img /images/notify_7.png %}

### 通知的基础用法

#### 创建通知
发送通知需要通过 `NotificationManager.notify()` 来实现，该函数的一个必要参数是 Notification 对象，Notification 对象描述了通知的具体内容，构建该对象需要利用 Notification.Builder 类，Notification.Builder 是 Android 3.0 (API 11) 引入的，为了兼容低版本，我们一般使用 Support V4 包提供的 NotificationCompat.Builder 来构建 Notification。具体代码如下：
<!-- more -->

```java
NotificationCompat.Builder builder = new NotificationCompat.Builder(context)

// 设置通知的基本信息：icon、标题、内容
builder.setSmallIcon(R.drawable.notification_icon)
builder.setContentTitle("My notification")
builder.setContentText("Hello World!");

// 设置通知的点击行为：这里启动一个 Activity
Intent intent = new Intent(this, ResultActivity.class);
PendingIntent pendingIntent = PendingIntent.getActivity(context, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT);
builder.setContentIntent(pendingIntent);

// 发送通知 id 需要在应用内唯一
NotificationManager notificationManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
notificationManager.notify(id, builder.build());
```

#### 更新通知
要想更新通知，需要利用 `NotificationManager.notify()` 的 id 参数，该 id 在应用内需要唯一。要想更新特定 id 的通知，只需要创建新的 Notification，并发出与之前所用 id 相同的 Notification。如果之前的通知仍然可见，则系统会根据新的 Notification 对象的内容更新该通知。相反，如果之前的通知已被清除，系统则会创建一个新通知。

#### 删除通知
删除通知可以有多种方式：
1. 通过 `NotificationCompat.Builder` 设置 `setAutoCancel(true)`，这样当用户点击通知后，通知自动删除。
2. 通过 `NotificationManager.cancel(id)` 方法，删除指定 id 的通知
3. 通过 `NotificationManager.cancelAll()` 方法，删除该应用的所有通知

### 通知的进阶用法

#### 浮动通知
Android 5.0（API 21）引入浮动通知的展现形式，想让通知能够以浮动形式展现，需要设置 Notification 的优先级为 PRIORITY_HIGH 或者 PRIORITY_MAX 并且使用铃声或振动，示例代码如下：

```
NotificationCompat.Builder builder = new NotificationCompat.Builder(context)

builder.setSmallIcon(R.drawable.notification_icon)
builder.setContentTitle("My notification")
builder.setContentText("Hello World!");
// 设置通知的优先级
builder.setPriority(NotificationCompat.PRIORITY_MAX);
Uri alarmSound = RingtoneManager.getDefaultUri(RingtoneManager.TYPE_NOTIFICATION);
// 设置通知的提示音
builder.setSound(alarmSound);

NotificationManager notificationManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
notificationManager.notify(id, builder.build());
```

#### 锁屏上展示通知
Android 5.0 通知现在还可显示在锁定屏幕上。用户可以通过“设置”选择是否将通知显示在锁定屏幕上，并且您可以指定您应用中的通知在锁定屏幕上是否可见。通过 setVisibility() 并指定以下值之一：    
1. VISIBILITY_PUBLIC 显示通知的完整内容。
2. VISIBILITY_SECRET 不会在锁定屏幕上显示此通知的任何部分。
3. VISIBILITY_PRIVATE 显示通知图标和内容标题等基本信息，但是隐藏通知的完整内容。     

设置 VISIBILITY_PRIVATE 后，您还可以提供其中隐藏了某些详细信息的替换版本通知内容。例如，短信 应用可能会显示一条通知，指出“您有 3 条新短信”，但是隐藏了短信内容和发件人。要提供此替换版本的通知，请先使用 NotificationCompat.Builder 创建替换通知。创建专用通知对象时，请通过 setPublicVersion() 方法为其附加替换通知。

#### 扩展样式通知
通过 Builder.setStyle() 函数可以为通知设置扩展样式，NotificationCompat 提供了三种扩展样式：BigPictureStyle, BigTextStyle, InboxStyle。

```
NotificationCompat.Builder builder = new NotificationCompat.Builder(context)

builder.setSmallIcon(R.drawable.notification_icon)
builder.setContentTitle("My notification")
builder.setContentText("Hello World!");

NotificationCompat.InboxStyle inboxStyle = new NotificationCompat.InboxStyle();
inboxStyle.setBigContentTitle("邮件标题：");
for (int i=0; i < 5; i++) {
    inboxStyle.addLine("邮件内容" + i);
}
builder.setStyle(inBoxStyle);
```

#### 自定义通知布局样式
通过 NotificationCompat.setContent() 可以设置自定义布局，该接口的参数为 RemoteViews 类型，通过 xml 构建出需要显示的 RemoteViews 然后调用 setContent 完成设置。自定义通知布局的可用高度取决于通知视图。普通视图布局限制为 64 dp，扩展视图布局限制为 256 dp。在 Android 4.1（API 16）以后，Notification 还支持大视图的通知，通过 Notification.bigContentView 设置对应的 RemoteViews 即可。

```
RemoteViews bigView;
RemoteViews smallView;

// 构建 bigView 和 smallView。
...

NotificationCompat.Builder builder = new NotificationCompat.Builder(context);

// 设置自定义 RemoteViews
builder.setContent(smallView).setSmallIcon(R.drawable.icon_notification);
Notification notification = builder.build();

// 如果系统版本 >= Android 4.1，设置大视图 RemoteViews
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN) {
  notification.bigContentView = bigView;
}
NotificationManager notificationManager = (NotificationManager) context.getSystemService(Context.NOTIFICATION_SERVICE);
notificationManager.notify(DAILY_PUSH_NOTIFICATION_ID, notification);
```
