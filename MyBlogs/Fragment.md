# Fragment

***官方说明***：Fragment表示应用界面中可重复使用的一部分。Fragment 定义和管理自己的布局，具有自己的生命周期，并且可以处理自己的输入事件。Fragment 不能独立存在，而是必须由 Activity 或另一个 Fragment 托管。Fragment 的视图层次结构会成为宿主的视图层次结构的一部分，或附加到宿主的视图层次结构。

## 一. Fragment

### 生命周期状态

Fragment必须是依存与Activity而存在的，因此Activity的生命周期会直接影响到Fragment的生命周期。Fragment状态与Activity类似，也存在如下4种状态：

- 运行：当前Fmgment位于前台，用户可见，可以获得焦点。
- 暂停：其他Activity位于前台，该Fragment依然可见，只是不能获得焦点。
- 停止：该Fragment不可见，失去焦点。
- 销毁：该Fragment被完全删除，或该Fragment所在的Activity被结束。

### 生命周期方法

Created

- onAttach() ：当Fragment与Activity发生关联时调用。
- onCreate()：创建Fragment时被回调。
- onCreateView()：每次创建、绘制该Fragment的View组件时回调该方法，Fragment将会显示该方法返回的View 组件。
- onViewCreated()
- onActivityCreated()：当 Fragment 所在的Activity被启动完成后回调该方法。

Started

- onStart()：启动 Fragment 时被回调，此时Fragment可见。

Resumed

- onResume()：恢复 Fragment 时被回调，获取焦点时回调。

Paused

- onPause()：暂停 Fragment 时被回调，失去焦点时回调。

Stoped

- onStop()：停止 Fragment 时被回调，Fragment不可见时回调。

Destroyed

- onDestroyView()：销毁与Fragment有关的视图，但未与Activity解除绑定。
- onDestroy()：销毁 Fragment 时被回调。
- onDetach()：与onAttach相对应，当Fragment与Activity关联被取消时调用。

### 生命周期变化

- 正常创建：onAttach() —> onCreate() —> onCreateView() —> onActivityCreated() —> onStart() —> onResume()

- 按下Home键回到桌面、锁屏：onPause() —> onStop() 

- 从桌面返回Fragment、解锁：onStart() —> onResume()

- 切换到其他Fragment：onPause() —> onStop() —> onDestroyView()

- 切换回本身的Fragment：onCreateView() —> onActivityCreated() —> onStart() —> onResume()

- 按下Back键退出：onPause() —> onStop() —> onDestroyView() —> onDestroy() —> onDetach()

## 二. Fragment的基础用法

### 1. 初始化Fragment

创建一个AFragment类继承自Fragment:

```java
public class AFragment extends Fragment {
    private TextView textView;
    private View root;

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        if (root == null){
            root = inflater.inflate(R.layout.a_fragment, container, false);
        }
        return root;
    }

    @Override
    public void onViewCreated(View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        textView = view.findViewById(R.id.textview);
    }
}
```

R.layout.a_fragment文件如下:

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.coordinatorlayout.widget.CoordinatorLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:id="@+id/textview"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:text="AFragment"
        android:textSize="50sp"/>

</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

### 2. 使用

MainActivity如下:

```java
public class MainActivity extends AppCompatActivity {
    private AFragment afragment;
    private BFragment bfragment;
    private Button button;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_container);

        afragment = new AFragment();
        afragment = new BFragment();
        
        button = findViewById(R.id.button);
        getFragmentManager().beginTransaction().add(R.id.fl, afragment).commit();

        button.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                getFragmentManager().beginTransaction().replace(R.id.fl, bfragment).commit();
            }
        });

    }
}
```

R.layout.activity_main文件如下:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".ContainerActivity">

    <Button
        android:id="@+id/button"
        android:layout_width="match_parent"
        android:layout_height="50dp"
        android:text="change" />

    <FrameLayout
        android:id="@+id/fl"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_below="@id/button_f"/>
</RelativeLayout>
```

## 三. Fragment的复用

使用时，afragment和bfragment都必须提前初始化（new xxx（）），且afragment已add至R.id.fl

```java
  switchFragment(R.id.fl, afragment, bfragment);
```

```java
 public void switchFragment(int contentRes, Fragment from, Fragment to) {
        FragmentTransaction transaction = getSupportFragmentManager().beginTransaction();
        if (!to.isAdded()) {	// 先判断是否被add过
            transaction.hide(from).add(contentRes, to).commit(); // 隐藏当前的fragment，add下一个到Activity中
        } else {
            transaction.hide(from).show(to).commit(); // 隐藏当前的fragment，显示下一个
        }
    }
```

