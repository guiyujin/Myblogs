# Activity

**Activity**是一个应用程序组件，提供一个屏幕，用户可以用来交互为了完成某项任务

## APP启动流程
1. 点击桌面App图标，Launcher进程采用Binder IPC向system_server进程发起startActivity请求；
2. system_server进程接收到请求后，向zygote进程发送创建进程的请求；
3. Zygote进程fork出新的子进程，即App进程；
4. App进程，通过Binder IPC向sytem_server进程发起attachApplication请求；
5. system_server进程在收到请求后，进行一系列准备工作后，再通过binder IPC向App进程发送scheduleLaunchActivity请求；
6. App进程的binder线程（ApplicationThread）在收到请求后，通过handler向主线程发送LAUNCH_ACTIVITY消息；
7. 主线程在收到Message后，通过发射机制创建目标Activity，并回调Activity.onCreate()等方法。
8. 到此，App便正式启动，开始进入Activity生命周期，执行完onCreate/onStart/onResume方法，UI渲染结束后便可以看到App的主界面。

## 生命周期状态

- Running：当前显示在屏幕的Activity位于Activity任务栈的栈顶，用户可见并且可操作

- Paused：当前状态可见，但是界面焦点已经失去，此Activity无法与用户交互

- Stopped：用户不可见也不可操作，可能被覆盖或者在后台，此时的Activity有可能被系统回收

- Killed：界面被销毁，等待被系统回收

## 生命周期方法

- onCreate：在系统首次创建 Activity 时触发，Activity 会在创建后进入“已创建”状态。在 onCreate()方法中，您需执行基本应用启动逻辑，该逻辑在 Activity 的整个生命周期中只应发生一次。这个方法会初始化setContentLayout()方法（屏幕绘制）

- onStart：onCreate()方法完成后，此时activity进入了onStart（）方法，当前activity是用户可见状态，但是还不能交互，在此可做一些动画的初始化操作

- onResume：onStart()后activity进入onResume方法，当前activity状态属于运行状态，此时的activity可见、可交互

- onPause：系统将此方法视为用户将要离开您的 Activity 的第一个标志；此方法表示 Activity 不再位于前台。使用onPause()方法暂停或调整当Activity 处于“已暂停”状态时不应继续（或应有节制地继续）的操作，以及您希望很快恢复的操作。通常用于确认对于持久性的数据保存更改，动画的停止以及任何其他可能消耗cpu的内容，必须迅速的执行所需的操作，该方法执行后，下一个Activity才能开始执行，该方法执行后应该执行onStop()方法

- onStop：当Activity对与用户不在可见的时候调用，可能是被另一个Activity覆盖，或者退回到桌面，在onStop方法下系统内存紧张时，有可能会被系统回收

- onDestory：在Activity被销毁前调用，这是Activity收到的最后调用，当Activity结束或者被系统销毁Activity实例的时候，会被调动该方法

- onRestart：在Activity被停止后再次启动的时候调用，比如从桌面回到应用中时，然后调用onStart方法()

## 生命周期变化
* 正常启动：onCreate、onStart、onResume

* 按下Home回到桌面、锁屏：onPause、onStop

* 从桌面返回应用、解锁：onRestart、onStart、onResume

* 按下Back键退出：onPause、onStop、onDestroy

* 横竖屏切换：onPause、onStop、onDestroy、onCreate、onStart、onResume

## 启动模式
* standard：每次启动Activity都会创建实例

* singleTop：启动时，若在栈顶则直接复用；在栈中则重新创建

* singleTask：在栈中则直接复用，并弹出其上所有实例

* singleInstance：Acyivity单独位于一个任务栈

## 数据传递
在MainActivity中发送数据

```java
Intent intent = new Intent(MainActivity.this, SecondActivity.class);
Bundle bundle = new Bundle();
bundle.putString("key", "data");
intent.putExtras(bundle);
startActivityForResult(intent, 1);
```
在SecondActivity中接收并回传数据
```java
Bundle bundle  = getIntent().getExtras();
String data = bundle.getString("key");
Toast.makeText(ContainerActivity.this,data, Toast.LENGTH_SHORT).show();

Intent intent = new Intent();
intent.putExtra("key", "Result");
setResult(2, intent);
finish();
```
在MainActivity中接收数据
```java
@Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);
         if (requestCode == 1 && resultCode == 2){
            Toast.makeText(MainActivity.this, data.getStringExtra("key"), Toast.LENGTH_SHORT).show();
        }
    }
```

## 其余
。。。。。。