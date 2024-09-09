---
title: 【Android】Service
date: 2018-07-18 13:54 +0800
categories: [Android]
tags: [android, service]
---
* Service相当于一个**没有图形界面的Activity**，主要功能是为Activity程序提供一些必要的支持，例如mp3播放软件等。
* 声明Service类：

```java
public class MyService extends Service {
    @Override public void onCreate() { }
    @Override public int onStartCommand(Intent intent, int flags, int startId) { ... }
    @Override public void onDestroy() { }
    @Override public IBinder onBind(Intent intent) { return null; }
}
```

* Service也是组件，需要在AndroidManifest.xml表单文件中配置：

```xml
<service android:name=".MyService" />
```

* 启动Service：`startService(new Intent(MainActivity.this, MyService.class));`
* 停止Service：`stopService(new Intent(MainActivity.this, MyService.class));`
* 绑定Service：将前台Activity界面与后台Service绑定（生命周期相同，同生共死），则Activity退出时与它绑定的Service自动停止。使用ServiceConnection接口：

```java
private ServiceConnection serviceConnection = new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName componentName, IBinder iBinder) {
        try {
            System.out.println("### Service Connect Success = "
                    + iBinder.getInterfaceDescriptor());
        }
        catch (RemoteException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void onServiceDisconnected(ComponentName componentName) {
        System.out.println("### Service Failure.");
        // 断开连接，但Service仍在后台运行
    }
};
```

* 绑定：`bindService(new Intent(MainActivity.this, MyService.class), serviceConnection, Context.BIND_AUTO_CREATE);`
* 解除绑定：`unbindService(serviceConnection);`
* Android提供了很多种系统服务：`Object getSystemService(String name)`

| name | 返回的对象 | 说明 |
| --- | --- | --- |
| ACTIVITY_SERVICE | ActivityManager | 管理应用程序的系统状态 |
| ALARM_SERVICE | AlarmManager | 闹钟的服务 |
| CLIPBOARD_SERVICE | ClipboardManager | 剪贴板服务 |
| CONNECTIVITY_SERVICE | Connectivity | 网络连接的服务 |
| KEYGUARD_SERVICE | KeyguardManager | 键盘锁的服务 |
| LAYOUT_INFLATER_SERVICE | LayoutInflater | 取得xml里定义的view |
| LOCATION_SERVICE | LocationManager | 位置的服务，如GPS |
| NOTIFICATION_SERVICE | NotificationManager | 状态栏的服务 |
| POWER_SERVICE | PowerManger | 电源管理服务 |
| SEARCH_SERVICE | SearchManager | 搜索服务 |
| TELEPHONY_SERVICE | TeleponyManager | 电话服务 |
| VEBRATOR_SERVICE | Vebrator | 手机震动的服务 |
| WIFI_SERVICE | WifiManager | Wi-Fi服务 |
| WINDOW_SERVICE | WindowManager | 管理打开的窗口程序 |
