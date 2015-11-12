---
layout: post
title: "Android 开发之实时更新 App Widget"
date: 2015-11-12 16:14:50 +0800
comments: true
categories: android, widget
---

[Android 开发之 App Widget 详解](http://glgjing.github.io/blog/2015/11/05/android-kai-fa-zhi-app-widget-xiang-jie/)中说到 updatePeriodMills 定义了 Widget 的刷新频率，但是出于节约用户电量的考虑，Android 系统默认最小更新周期是 30 分钟，也就是说：如果您的程序需要实时更新数据，设置这个更新周期是 2 秒，那么您的程序是不会每隔 2 秒就收到更新通知的，而是要等到 30 分钟以上才可以，要想实时的更新 Widget，一般可以采用 Service 和 AlarmManager 对 Widget 进行更新。

### 利用 Service 更新 Widget
在 onUpdate() 方法中启动 Service：

```java
public class MyWidgetProvider extends AppWidgetProvider {

  @Override
  public void onUpdate(Context context, AppWidgetManager appWidgetManager, int[] appWidgetIds) {
    context.startService(new Intent(context, WidgetService.class));
  }
}
```

在 Service 中对 Widget 进行更新，这里 Service 利用 AlarmManager 每隔一段时间进行自启，防止 Service 被系统 Kill 掉后无法对 Widget 进行更新，转载请注明出处：<http://glgjing.github.io/>。
<!-- more -->

```java
public class WidgetService extends Service {
  private static final int ALARM_DURATION  = 5 * 60 * 1000; // service 自启间隔
  private static final int UPDATE_DURATION = 10 * 1000;     // Widget 更新间隔
  private static final int UPDATE_MESSAGE  = 1000;

  private UpdateHandler updateHandler; // 更新 Widget 的 Handler

  @Override
  public IBinder onBind(Intent intent) {
    return null;
  }

  @Override
  public int onStartCommand(Intent intent, int flags, int startId) {
    // 每个 ALARM_DURATION 自启一次
    AlarmManager manager = (AlarmManager) getSystemService(Context.ALARM_SERVICE);
    Intent alarmIntent = new Intent(getBaseContext(), WidgetService.class);
    PendingIntent pendingIntent = PendingIntent.getService(getBaseContext(), 0,
      alarmIntent, PendingIntent.FLAG_UPDATE_CURRENT);

    manager.set(AlarmManager.ELAPSED_REALTIME_WAKEUP,
      SystemClock.elapsedRealtime() + ALARM_DURATION, pendingIntent);

    return START_STICKY;
  }

  @Override
  public void onCreate() {
    super.onCreate();

    Message message = updateHandler.obtainMessage();
    message.what = UPDATE_MESSAGE;
    updateHandler = new UpdateHandler();
    updateHandler.sendMessageDelayed(message, UPDATE_DURATION);
  }

  private void updateWidget() {
    // 更新 Widget
    RemoteViews remoteViews = new RemoteViews(getApplicationContext().getPackageName(), R.layout.widget);
    AppWidgetManager appWidgetManager = AppWidgetManager.getInstance(context);
    appWidgetManager.updateAppWidget(new ComponentName(context, MyWidgetProvider.class), remoteViews);

    // 发送下次更新的消息
    Message message = updateHandler.obtainMessage();
    message.what = UPDATE_MESSAGE;
    updateHandler.sendMessageDelayed(message, UPDATE_DURATION);
  }

  protected final class UpdateHandler extends Handler {

    @Override
    public void handleMessage(Message msg) {
      switch (msg.what) {
        case UPDATE_MESSAGE:
          updateWidget();
          break;
        default:
          break;
      }
    }
  }
}
```

### Service + AlarmManager 更新 Widget
上面是利用 Service 的内部消息循环更新 Widget，也可以利用 AlarmManager 来定时触发更新。在 onUpdate() 中启动 Alarm，通过 AlarmManager 来循环启动 Service，剩下的原理基本就是一样的了。记得在 onDisabled() 取消掉 Alarm。

```java
public class MyWidgetProvider extends AppWidgetProvider {
  private static final int UPDATE_DURATION = 10 * 1000; // Widget 更新间隔

  private PendingIntent pendingIntent = null; 

  @Override
  public void onUpdate(Context context, AppWidgetManager appWidgetManager, int[] appWidgetIds) {
    AlarmManager manager = (AlarmManager) getSystemService(Context.ALARM_SERVICE);
    Intent alarmIntent = new Intent(getBaseContext(), WidgetService.class);
    if (pendingIntent == null) {
      pendingIntent = PendingIntent.getService(getBaseContext(), 0,
        alarmIntent, PendingIntent.FLAG_UPDATE_CURRENT);
    }

    manager.setRepeating(AlarmManager.ELAPSED_REALTIME_WAKEUP,
      SystemClock.elapsedRealtime(), UPDATE_DURATION, pendingIntent);
  }

  @Override  
  public void onDisabled(Context context) {
    AlarmManager manager = (AlarmManager) context.getSystemService(Context.ALARM_SERVICE);  
    manager.cancel(pendingIntent);
  }
}
```

MyService 源码：

```java
public class MyService extends Service  
{
  @Override  
  public int onStartCommand(Intent intent, int flags, int startId) {  
    buildUpdate();  
    return super.onStartCommand(intent, flags, startId);  
  }  
  
  private void buildUpdate() {  
    RemoteViews view = new RemoteViews(getPackageName(), R.layout.widget);  
  
    RemoteViews remoteViews = new RemoteViews(getApplicationContext().getPackageName(), R.layout.widget);
    AppWidgetManager appWidgetManager = AppWidgetManager.getInstance(context);
    appWidgetManager.updateAppWidget(new ComponentName(context, MyWidgetProvider.class), remoteViews); 
  }
}
```

### 添加自定义 View

更新 Widget 是通过 RemoteViews 实现的，而 RemoteViews 支持的 View 有限，详细参考[这里](http://developer.android.com/intl/zh-cn/guide/topics/appwidgets/index.html#CreatingLayout)，如果想要在 Widget 中使用自定义 View，可以通过一下方式实现：

```java
RemoteViews remoteViews = new RemoteViews(context.getPackageName(), R.layout.widget);

MyCustomView customView = new MyCustomView(context);
customView.measure(width, height);
customView.layout(0, 0, width, height);
Bitmap bitmap = Bitmap.createBitmap(width, height, Bitmap.Config.ARGB_8888);
customView.draw(new Canvas(bitmap));
remoteViews.setImageViewBitmap(R.id.bitmap, bitmap);
```

实际上就是将自定义 View 在 Bitmap 上绘制，然后通过 ImageView 进行展现。

### 处理点击事件
RemoteViews 可以设置 setOnClickPendingIntent，通过 PendingIntent 来处理点击事件：

```java
// 设置 button 事件为启动一个 Activity
Intent intent1 = new Intent("open_widget_activity");
PendingIntent pendingIntent1 = PendingIntent.getActivity(context, 0, intent1, 0);
remoteViews.setOnClickPendingIntent(R.id.button1, pendingIntent1);

// 设置 button 事件为发送一个广播
Intent intent2 = new Intent("send_broadcast");
PendingIntent pendingIntent2 = PendingIntent.getBroadcast(context, 0, intent2, 0);
remoteViews.setOnClickPendingIntent(R.id.button2, pendingIntent2);
```

然后需要处理事件的 Activity 或者 Receiver 接受对应的 Intent 即可。
