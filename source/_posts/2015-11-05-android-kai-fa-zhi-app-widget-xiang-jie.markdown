---
layout: post
title: "Android 开发之 App Widget 详解"
date: 2015-11-05 13:08:53 +0800
comments: true
categories: android, widget
---
### 简介
App Widget：应用程序窗口小部件，微型的应用程序视图，它可以被嵌入到其它应用程序中，比如桌面，并接收周期性的更新。你可以通过一个 App Widget Provider 来发布一个 Widget。可以容纳 Widget 的应用叫做 App Widget Host，详细参考[App Widgets| Android Developers](http://developer.android.com/intl/zh-cn/guide/topics/appwidgets/index.html)，转载请注明出处：<http://glgjing.github.io/>。

### 创建一个 App Widget 的主要步骤
1. 在 AndroidManifest 中声明 App Widget
2. 在 xml 目录定义 App Widget 的初始化 xml 文件
3. 实现 Widget 具体布局的 Layout xml。
4. 继承 AppWidgetProvider 类，实现具体的 Widget 业务逻辑。

### 在 AndroidManifest 中声明 App Widget

```xml
<receiver android:name=".sample.MyWidgetProvider" >
    <intent-filter>
        <action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
    </intent-filter>
    <meta-data android:name="android.appwidget.provider"
               android:resource="@xml/appwidget_provider"/>
</receiver>

```
<!-- more -->
`<receiver>`的 android:name 属性声明的就是 Widget 所用的 AppWidgetProvider 类，并且`<intent-filter>`中必须要包含 APPWIDGET_UPDATE 这个 `<action>`，所有 Widget 的 broadcast 都是通过这个 filter 来接收的。

`<meta-data>` 声明了 Widget 的 AppWidgetProviderInfo 对应的资源 xml 的位置，用的是 xml 目录下的 appwidget_provider.xml。这里需要简单介绍下 AppWidgetProviderInfo 类，该类是用来描述 Widget 的 meta 信息，包括 Widget 的 xml 布局文件、刷新频率、最小宽高等等，而这些信息正是通过上述 xml 的`<appwidget-provider>` 标签来描述的。

### 在 xml 目录定义 App Widget 的初始化 xml 文件
前面所说的用来描述 AppWidgetProviderInfo 的 xml 定义如下：

```xml
<?xml version="1.0" encoding="utf-8"?>

<appwidget-provider 
	xmlns:android="http://schemas.android.com/apk/res/android"
    android:minWidth="200dp"
    android:minHeight="50dp"
    android:updatePeriodMillis="86400000"
    android:previewImage="@drawable/app_icon"
    android:initialLayout="@layout/widget"
    android:configure="com.example.android.ExampleAppWidgetConfigure"
    android:resizeMode="horizontal|vertical"
    android:widgetCategory="home_screen|keyguard"/>
```

**minWidth** & **minHeight**：定义了 Widget 的最小宽高，当 minWidth 和 minHeight 不是桌面 cell 的整数倍时，Widget 的宽高会被阔至与其最接近的 cells 大小。Google 官方给出了一个大致估算 minWidth & minHeight 的公式，根据 Widget 所占的 cell 数量来计算宽高：`70 × n − 30`，n 是所占的 cell 数量。    
**updatePeriodMillis**：定义了 Widget 的刷新频率，也就是 App Widget Framework 多久请求一次 AppWidgetProvider 的 onUpdate() 回调函数。该时间间隔并不保证精确，出于节约用户电量的考虑，Android 系统默认最小更新周期是 30 分钟，也就是说：如果您的程序需要实时更新数据，设置这个更新周期是 2 秒，那么您的程序是不会每隔 2 秒就收到更新通知的，而是要等到 30 分钟以上才可以，要想实时的更新 Widget，一般可以采用 Service 和 AlarmManager 对 Widget 进行更新。     
**previewImage**：当用户选择添加 Widget 时的预览图片。如果该属性没有定义，则展示 application 的 launcher icon。<font color="#B72712">该属性是在 3.0 以后引入的。</font>      
**initialLayout**：Widget 的布局 Layout 文件。      
**configure**：定义了用户在添加 Widget 时弹出的配置页面的 Activity，用户可以在此进行 Widget 的一些配置，该 Activity 是可选的，如果不需要可以不进行声明。     
**resizeMode**：Widget 在水平和垂直方向是否可以调整大小，值可以为：horizontal（水平方向可以调整大小），vertical（垂直方向可以调整大小），none（不可以调整大小），也可以 horizontal|vertical 组合表示水平和垂直方向均可以调整大小。      
**widgetCategory**：表示 Widget 可以显示的位置，包括 home_screen（桌面），keyguard（锁屏），keyguard 属性需要 5.0 或以上 Android 版本才可以。      
其它更多详细属性可以参考 [AppWidgetProviderInfo](http://developer.android.com/intl/zh-cn/reference/android/appwidget/AppWidgetProviderInfo.html)。

### 继承 AppWidgetProvider 类
AppWidgetProvider 继承自 BroadcastReceiver，内部逻辑非常简单，就是在 onReceive() 中处理 Widget 相关的广播事件（ACTION_APPWIDGET_UPDATE, ACTION_APPWIDGET_DELETED, ACTION_APPWIDGET_ENABLED, ACTION_APPWIDGET_DISABLED, ACTION_APPWIDGET_OPTIONS_CHANGED）分发到各个回调函数中（onUpdate(), onDeleted(), onEnabled(), onDisabled, onAppWidgetOptionsChanged()）。      

**onUpdate()：**是最重要的回调函数，根据 updatePeriodMillis 定义的定期刷新操作会调用该函数，此外当用户添加 Widget 时
也会调用该函数，可以在这里进行必要的初始化操作。但如果在`<appwidget-provider>`中声明了 android:configure 的 Activity，在用户添加 Widget 时，不会调用 onUpdate()，需要由 configure Activity 去负责去调用 AppWidgetManager.updateAppWidget() 完成 Widget 更新，后续的定时更新还是会继续调用 onUpdate() 的。      
**onDeleted()：**当 Widget 被删除时调用该方法。     
**onEnabled()：**当 Widget 第一次被添加时调用，例如用户添加了两个你的 Widget，那么只有在添加第一个 Widget 时该方法会被调用。所以该方法比较适合执行你所有 Widgets 只需进行一次的操作。     
**onDisabled()：**与 onEnabled 恰好相反，当你的最后一个 Widget 被删除时调用该方法，所以这里用来清理之前在 onEnabled() 中进行的操作。      
**onAppWidgetOptionsChanged()：**当 Widget 第一次被添加或者大小发生变化时调用该方法，可以在此控制 Widget 元素的显示和隐藏。     

示例代码：

```java
public class ExampleAppWidgetProvider extends AppWidgetProvider {

    public void onUpdate(Context context, AppWidgetManager appWidgetManager, int[] appWidgetIds) {
        final int N = appWidgetIds.length;

        // Perform this loop procedure for each App Widget that belongs to this provider
        for (int i=0; i<N; i++) {
            int appWidgetId = appWidgetIds[i];

            // Create an Intent to launch ExampleActivity
            Intent intent = new Intent(context, ExampleActivity.class);
            PendingIntent pendingIntent = PendingIntent.getActivity(context, 0, intent, 0);

            // Get the layout for the App Widget and attach an on-click listener
            // to the button
            RemoteViews views = new RemoteViews(context.getPackageName(), R.layout.appwidget_provider_layout);
            views.setOnClickPendingIntent(R.id.button, pendingIntent);

            // Tell the AppWidgetManager to perform an update on the current app widget
            appWidgetManager.updateAppWidget(appWidgetId, views);
        }
    }
}
```

### 创建 App Widget Configuration Activity
如果你的 Widget 需要用户配置一些选项，你可以为你的 Widget 创建 Configuration Activity，当用户添加 Widget 时会自动弹出该 Activity。Configuration Activity 和普通 Activity 一样需要在 Manifest 中声明，但是需要额外声明一个 intent-filter: APPWIDGET_CONFIGURE，例如：

```xml
<activity android:name=".ExampleAppWidgetConfigure">
    <intent-filter>
        <action android:name="android.appwidget.action.APPWIDGET_CONFIGURE"/>
    </intent-filter>
</activity>
```

同时还需要在上述的 appwidget-provider 中声明：

```xml
<appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
    ...
    android:configure="com.example.android.ExampleAppWidgetConfigure" 
    ... >
</appwidget-provider>
```

有两点需要注意的是：      

1. Activity 必须返回带 EXTRA_APPWIDGET_ID 的 result。
2. 声明Configuration Activity 后 onUpdate() 在 Widget 添加时不会被调用，Activity 负责调用 AppWidgetManager.updateAppWidget() 完成 Widget 更新。

```java
Intent intent = getIntent();
Bundle extras = intent.getExtras();
if (extras != null) {
    mAppWidgetId = extras.getInt(
        AppWidgetManager.EXTRA_APPWIDGET_ID, 
        AppWidgetManager.INVALID_APPWIDGET_ID);
}

AppWidgetManager appWidgetManager = AppWidgetManager.getInstance(context);
RemoteViews views = new RemoteViews(context.getPackageName(),R.layout.example_appwidget);
appWidgetManager.updateAppWidget(mAppWidgetId, views);

Intent resultValue = new Intent();
resultValue.putExtra(AppWidgetManager.EXTRA_APPWIDGET_ID, mAppWidgetId);
setResult(RESULT_OK, resultValue);
finish();
```


