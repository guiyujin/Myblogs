# Android性能优化
## 启动优化
数据加载，内容显示
冷启动：需要执行Application，Application为**进程**做数据准备；
热启动：不需要执行Application；

#### 多进程启动初始化

在MyApplication的onCreate方法里

```java
    public void onCreate() {
        super.onCreate();
        
        if (isMainProcess(BuildConfig.APPLICATION_ID)){
            //只需要在主进程初始化的操作
        }else {
            //其余各进程初始化的操作
        }

    }
```

判断方法

```java
    protected boolean isMainProcess(String mainApplicationPackageName){
        String processName = getProcessName(this);
        if (processName != null){
            if (processName.equals(mainApplicationPackageName)){
                return true;
            }
        }
        return false;
    }

    protected String getProcessName(Context context){
        ActivityManager activityManager = (ActivityManager) context.getSystemService(Context.ACTIVITY_SERVICE);
        List<ActivityManager.RunningAppProcessInfo> appProcessInfos = activityManager.getRunningAppProcesses();
        if (appProcessInfos == null){
            return null;
        }
        for (ActivityManager.RunningAppProcessInfo info: appProcessInfos){
            if (info.pid == android.os.Process.myPid()){
                if (info.processName != null){
                    return info.processName;
                }
            }
        }
        return null;
    }
```

#### 统一启动入口

使用LauncherActivity，进行分发跳转（Scheme、通知、Shortcut、桌面）。

在AndroidManifest.xml内

```xml
        <activity
            android:name=".activity.LauncherActivity"
            android:theme="@style/Launch">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
            <intent-filter>
                <data android:scheme="myscheme"/>
                <action android:name="android.intent.action.VIEW"/>
                <category android:name="android.intent.category.DEFAULT"/>
                <category android:name="android.intent.category.BROWSABLE"/>
            </intent-filter>
        </activity>
```

#### 黑白屏

在styles.xml内

```xml
    <style name="Launch" >
        <item name="android:windowFullscreen">true</item>
        <item name="android:windowBackground">@drawable/launcher_layout</item>
        <item name="android:windowNoTitle">true</item>
        <item name="windowActionBar">false</item>
        <item name="windowNoTitle">true</item>
    </style>
```

launcher_layout.xml如下

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">

    <item >
        <!-- 整体的背景颜色 -->
        <color android:color="@color/MiColorPrimary"/>
    </item>

    <item

        android:width="300dp"
        android:height="100dp"
        android:top="50dp"
        android:gravity="center_horizontal"
        >

        <!-- 图标和图标位置 -->
        <bitmap
            android:src="@drawable/title_text" />
    </item>

</layer-list>
```

有时候需要以下语句隐藏启动页

```java
getWindow().getDecorView().setBackground(new ColorDrawable(Color.TRANSPARENT));
```

#### 懒加载

仅加载首个可见Fragment

## 内存优化

#### 内存抖动（OOM）

根本原因：内存频繁的分配和回收

影响：卡顿

出现情况：字符串拼接

#### 内存泄漏

根本原因：该回收的对象无法回收

##### 可达性分析算法

通过一系列的名为“GC Root”的对象作为起点，从这些节点向下搜索，搜索所走过的路径称为引用链(Reference Chain)，当一个对象到GC Root没有任何引用链相连时，则该对象不可达，该对象是不可使用的，垃圾收集器将回收其所占的内存。

  在java语言中，可作为GCRoot的对象包括以下几种：

-   java虚拟机栈(栈帧中的本地变量表)中的引用的对象。 

-   方法区中的类静态属性引用的对象。 

-   方法区中的常量引用的对象。 

-   本地方法栈中JNI（Native 本地）方法的引用对象。


##### 四大引用

1. 强引用
    直接new出的对象：String str = new String("ABC");
    1.1 介绍
    普遍使用的引用对象，但是即便内存不足时也不会通过回收机制来回收而导致程序奔溃
    1.2 特点
    强引用可以直接访问目标对象
    强引用所指的对象在任何时候都不会被回收，即便jvm抛出oom
    强引用可能会导致内存泄漏
    
2. 软引用
    SoftReference：软引用–>当虚拟机内存不足时，将会回收它指向的对象；需要获取对象时，可以调用get方法。
    软引用对象不会很快被jvm回收，jvm 会根据当前堆的使用情况来判断何时回收，当堆的使用频率接近阀值时才会被回收
    
3. 弱引用
    3.1 介绍
    WeakReference： 弱引用–>随时可能会被垃圾回收器回收，不一定要等到虚拟机内存不足时才强制回收。要获取对象时，同样可以调用get方法。
    3.2 特点
    如果当前引用只具备弱引用，那么在垃圾回收时，不管内存是否够用都会被回收，但垃圾回收机制优先级低，一般不会被很快发现弱引用对象。
    3.3 使用场景
    避免内存泄漏  handler
    
4. 虚引用 PhantomReference
     虚引用是所有引用类型中最弱的一个。一个持有虚引用的对象，和没有引用几乎是一样的，随时都可能被垃圾回收器回收。当试图通过虚引用的get()方法取得强引用时，总是会失败。并且，虚引用必须和引用队列一起使用，它的作用在于跟踪垃圾回收过程。 当垃圾回收器准备回收一个对象时，如果发现它还有虚引用，就会在垃圾回收后，销毁这个对象，奖这个虚引用加入引用队列。Android实际开发中不用。

 **弱引用和软引用区别**
    弱引用与软引用的根本区别在于：只具有弱引用的对象拥有更短暂的生命周期，可能随时被回收。而只具有软引用的对象只有当内存不够的时候才被回收，在内存足够的时候，通常不被回收。

**使用软引用或者弱引用防止内存泄漏**
   在Android应用的开发中，为了防止内存溢出，在处理一些占用内存大而且声明周期较长的对象时候，可以尽量应用软引用和弱引用技术。
   软引用、弱引用都非常适合来保存那些可有可无的缓存数据。如果这样做，当系统内存不足时，这些缓存数据会被回收，不会导致内存溢出。而当内存资源充足时，这些缓存数据又可以存在相当长的时间。

**到底什么时候使用软引用，什么时候使用弱引用呢？**
   个人认为，如果只是想避免OutOfMemory异常的发生，则可以使用软引用。如果对于应用的性能更在意，想尽快回收一些占用内存比较大的对象，则可以使用弱引用。
   还有就是可以根据对象是否经常使用来判断。如果该对象可能会经常使用的，就尽量用软引用。如果该对象不被使用的可能性更大些，就可以用弱引用。
   另外，和弱引用功能类似的是WeakHashMap。WeakHashMap对于一个给定的键，其映射的存在并不阻止垃圾回收器对该键的回收，回收以后，其条目从映射中有效地移除。WeakHashMap使用ReferenceQueue实现的这种机制。



## APK瘦身

图片压缩

国际化

动态so库

移除无用资源

## 电量优化

- Alarm Manager wakeup 唤醒过多
- 频繁使用局部唤醒锁
- 后台网络使用量过高
- 后台 WiFi scans 过多

