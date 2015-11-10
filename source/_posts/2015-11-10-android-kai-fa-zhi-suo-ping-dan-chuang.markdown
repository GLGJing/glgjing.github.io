---
layout: post
title: "Android 开发之锁屏弹窗"
date: 2015-11-10 20:10:24 +0800
comments: true
categories: Android
---

## 尝试利用 WindowManager 添加浮窗的方式实现
想在锁屏上面实现弹窗，第一个想法就是利用 WindowManager 设置 Window 的 Flag，通过设置 Flag 的显示优先级来让窗口显示在锁屏的上面。接下来就是试验可能相关的 Window Type 属性，验证该方案是否可行。转载请注明出处：<http://glgjing.github.io/>      

在尝试各个 Window Type 属性之前需要明确各个 Type 所需要的权限，下面是 com.android.internal.policy.impl.PhoneWindowManager.checkAddPermission 的源码：

```java
public int checkAddPermission(WindowManager.LayoutParams attrs) {
    int type = attrs.type;
    
    if (type < WindowManager.LayoutParams.FIRST_SYSTEM_WINDOW
            || type > WindowManager.LayoutParams.LAST_SYSTEM_WINDOW) {
        return WindowManagerImpl.ADD_OKAY;
    }
    String permission = null;
    switch (type) {
        case TYPE_TOAST:
            // XXX right now the app process has complete control over
            // this...  should introduce a token to let the system
            // monitor/control what they are doing.
            break;
        case TYPE_INPUT_METHOD:
        case TYPE_WALLPAPER:
            // The window manager will check these.
            break;
        case TYPE_PHONE:
        case TYPE_PRIORITY_PHONE:
        case TYPE_SYSTEM_ALERT:
        case TYPE_SYSTEM_ERROR:
        case TYPE_SYSTEM_OVERLAY:
            permission = android.Manifest.permission.SYSTEM_ALERT_WINDOW;
            break;
        default:
            permission = android.Manifest.permission.INTERNAL_SYSTEM_WINDOW;
    }
    if (permission != null) {
        if (mContext.checkCallingOrSelfPermission(permission)
                != PackageManager.PERMISSION_GRANTED) {
            return WindowManagerImpl.ADD_PERMISSION_DENIED;
        }
    }
    return WindowManagerImpl.ADD_OKAY;
}
```

- 明显不适合的 Type：TYPE_TOAST, TYPE_INPUT_METHOD, TYPE_WALLPAPER。    
- 可能适合的 Type：TYPE_PHONE, TYPE_PRIORITY_PHONE, TYPE_SYSTEM_ALERT, TYPE_SYSTEM_ERROR, TYPE_SYSTEM_OVERLAY。     
- 其它类型的 Type：需要 android.Manifest.permission.INTERNAL_SYSTEM_WINDOW 权限，而申请该权限需要系统签名，所以我们是无法获取权限的。    
<!-- more -->
**TYPE_PHONE**

```java
/**
 * Window type: phone.  These are non-application windows providing
 * user interaction with the phone (in particular incoming calls).
 * These windows are normally placed above all applications, but behind
 * the status bar.
 * In multiuser systems shows on all users' windows.
 */
public static final int TYPE_PHONE              = FIRST_SYSTEM_WINDOW+2;
```

TYPE_PHONE 类型的窗口可以显示在其它 APP 的上面，但不能显示在锁屏的上面，所以 PASS。     

**TYPE_PRIORITY_PHONE**

```java
/**
 * Window type: priority phone UI, which needs to be displayed even if
 * the keyguard is active.  These windows must not take input
 * focus, or they will interfere with the keyguard.
 * In multiuser systems shows on all users' windows.
 */
public static final int TYPE_PRIORITY_PHONE     = FIRST_SYSTEM_WINDOW+7;
```

TYPE_PRIORITY_PHONE 类型的窗口可以显示在其它 APP 的上面，但不能显示在锁屏的上面，所以 PASS。而且实际的行为和注释并不相符，该类型的窗口是可以获取交互事件的，具体原因待查。     

**TYPE_SYSTEM_ALERT**

```java
/**
 * Window type: system window, such as low power alert. These windows
 * are always on top of application windows.
 * In multiuser systems shows only on the owning user's window.
 */
public static final int TYPE_SYSTEM_ALERT       = FIRST_SYSTEM_WINDOW+3;
```

TYPE_SYSTEM_ALERT 类型的窗口可以显示在其它 APP 的上面，但不能显示在锁屏的上面，所以 PASS。      

**TYPE_SYSTEM_OVERLAY**

```java
/**
 * Window type: system overlay windows, which need to be displayed
 * on top of everything else.  These windows must not take input
 * focus, or they will interfere with the keyguard.
 * In multiuser systems shows only on the owning user's window.
 */
public static final int TYPE_SYSTEM_OVERLAY     = FIRST_SYSTEM_WINDOW+6;
```

TYPE_SYSTEM_OVERLAY 类型的窗口可以显示在所有其它窗口的上面，包括锁屏，而且不会影响它下面窗口的交互事件响应，但是该属性窗口不能获得焦点，无法进行交互（如果该窗口可以获取焦点，那么就可以用来抓取用户的锁屏密码，出于安全考虑，系统是不会允许的），所以只能用来简单的展示内容，如果需要交互的锁屏弹窗，那么该属性 PASS。      

**TYPE_SYSTEM_ERROR**

```java
/**
 * Window type: internal system error windows, appear on top of
 * everything they can.
 * In multiuser systems shows only on the owning user's window.
 */
public static final int TYPE_SYSTEM_ERROR       = FIRST_SYSTEM_WINDOW+10;
```

在原生 ROM 5.1 下试验是可以显示出来的，但根据注释来看（appear on top of everything they can）不是在所有情况下都可以显示在锁屏上面的，<font color="#B72712">而且像 MIUI 和 Flyme 等 ROM 默认是屏蔽浮窗权限的，考虑到这点，利用 WindowManager 添加浮窗的方式实现锁屏弹窗的方案基本 PASS。</font>


## 使用 Activity 的方式实现
#### 首先需要对 Activity 进行如下设置

```java
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);  
    final Window win = getWindow();  
    win.addFlags(WindowManager.LayoutParams.FLAG_SHOW_WHEN_LOCKED  
            | WindowManager.LayoutParams.FLAG_DISMISS_KEYGUARD  
            | WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON  
            | WindowManager.LayoutParams.FLAG_TURN_SCREEN_ON);  
}
```

其中最主要也是必须要设置的就是：FLAG_SHOW_WHEN_LOCKED，顾名思义就是锁屏下显示该 Activity。而其它几个 Flag 包括：解锁、保持屏幕常亮、点亮屏幕可以根据具体的需求选择设置。    

#### 在 AndroidManifest.xml 中声明 Activity
同样该 Activity 也需要在 AndroidManifest.xml 中声明，声明时需注意添加 `android:excludeFromRecents="true"` 属性，是为了将该 Activity 从最近任务列表中去除，否则用户会觉得很奇怪。还有因为这个 Activity 会整个盖在锁屏上面，而且就算设置成背景透明，锁屏界面也不会显示在下面（系统主要是出于安全考虑），所以需要考虑下该 Activity 的背景，这里为了显示不要太突兀将主题设为壁纸。

```java
<activity android:name=".LockScreenActivity"  
          android:launchMode="singleInstance"  
          android:excludeFromRecents="true"  
          android:theme="@android:style/Theme.Wallpaper.NoTitleBar"/>  
```

#### 启动 Activity
由于该 Activity 是为了在锁屏的情况下显示的，所以启动 Activity 时不要忘了判断手机是否处于锁屏状态，可以通过下面这种方式判断锁屏状态：

```java
KeyguardManager km = (KeyguardManager) context.getSystemService(Context.KEYGUARD_SERVICE);
if (km.inKeyguardRestrictedInputMode()) {
    // 处于锁屏状态
}
```