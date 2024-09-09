---
title: 【Android】Fragment
date: 2020-02-17 17:28 +0800
categories: [Android]
tags: [android, fragment]
---
* 官方文档：<https://developer.android.google.cn/guide/components/fragments.html?hl=zh_cn>
* 参考：
  * <https://www.cnblogs.com/DreamRecorder/p/8961987.html>
  * <https://blog.csdn.net/qq_39494252/article/details/80368582>
* 片段Fragment表示界面的一部分，可以在一个Activity中组合多个Fragment，一个Fragment也可以被多个Activity复用
* Fragment必须嵌入在Activity中，不能独立存在，有自己的生命周期，并且受宿主Activity生命周期的影响，可以在Activity中动态添加和删除Fragment

1.定义Fragment，实现onCreateView()方法返回布局：

```java
public class MyFragment extends Fragment {

    public MyFragment() {}

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        return inflater.inflate(R.layout.fragment_my, container, false);
    }
 
}
```

2.将Fragment添加到Activity中

* 方法一：在xml中使用`<fragment>`标签
* 方法二：在代码中动态添加

首先在Activity的xml中添加一个用于放置Fragment的布局（常用FrameLayout）：

```xml
<FrameLayout
    android:id="@+id/content"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

在Activity代码中创建Fragment并添加到指定的布局：

```java
MyFragment fragment = new MyFragment();
FragmentManager fragmentManager = getFragmentManager();
fragmentManager.beginTransaction()
        .add(R.id.content, fragment)
        .commit();
```

除add()外，FragmentTransaction还有remove(), replace()等方法可用于管理Fragment

注意：
* (1) 最后一定要调用commit()提交
* (2) commit() **不是立即执行**，因此不能在commit之后立即访问Fragment对象的成员（此时还未初始化），应使用setArguments()（见5）
* (3) 另见[Fragment生命周期]({% post_url 2020-07-01-android-fragment-lifecycle %})

3.在Fragment中获取需要的控件

可在onCreateView()方法中通过根视图获取：

```java
View rootView = inflater.inflate(R.layout.fragment_my, container, false);
TextView textView = rootView.findViewById(R.id.textView);
return rootView;
```

4.在Fragment中获取资源：getContext().getResources()

5.向Fragment传递参数

**一定不要通过构造器来传递参数，而要使用setArguments()方法！**

* Fragment类默认构造器的官方文档：

> It is strongly recommended that subclasses do not have other constructors with parameters, since these constructors will not be called when the fragment is re-instantiated; instead, arguments can be supplied by the caller with setArguments and later retrieved by the Fragment with getArguments.

即Fragment在恢复状态时（例如从其他Activity切换回Fragment所属的Activity）会自动被系统重新实例化，此时只会调用无参构造函数，因此无法通过构造函数传入数据

* 在Activity中创建Fragment时使用setArguments()传入参数：

```java
MyFragment fragment = new MyFragment();
Bundle bundle = new Bundle();
bundle.putInt("arg", 1);
fragment.setArguments(bundle);
getFragmentManager().beginTransaction()
        .add(R.id.content, fragment)
        .commit();
```

* 在Fragment的onCreateView()方法中使用getArguments()方法获取参数：

```java
Bundle bundle = getArguments();
if (bundle != null)
    arg = bundle.getInt("arg");
```

* 使用setArguments()传入的参数在Fragment重新实例化时还可以获取到，不需要重新传入

6.嵌套Fragment

在一个Fragment中嵌套另一个Fragment和在Activity中嵌套Fragment基本一样，只是Fragment使用getChildFragmentManager()方法来获取FragmentManager

7.Activity重建时Fragment重叠的问题
* 参考：
  * <https://blog.csdn.net/songmingzhan/article/details/84452610>
  * <https://www.jianshu.com/p/8bf76160f606>
* 解决方法：在Activity的onCreate()方法或Fragment的onCreateView()方法中判断savedInstanceState是否为null，如果是null则表示首次创建，需要新建Fragment对象；否则使用getFragmentManager().findFragmentById()或getChildFragmentManager().findFragmentById()获取Fragment对象（此时不需再次设置参数）

典型代码：

```java
if (savedInstanceState == null) {
    fragment = new MyFragment();
    // 设置参数
    getFragmentManager().beginTransaction()
            .add(R.id.content, fragment)
            .commit();
}
else
    fragment = (MyFragment) getFragmentManager().findFragmentById(R.id.content);
```


8.保存和恢复状态
* 因为Activity恢复等原因重建时通过getFragmentManager().findFragmentById()获取到的不是之前的Fragment对象（已经被销毁），而是系统自动重新实例化的一个对象，但系统无法恢复Fragment中自定义的状态数据，需要通过覆盖onSaveInstanceState()和onActivityCreated()（没有onRestoreInstanceState()）方法自行保存和恢复数据
* 见[保存和恢复状态]({% post_url 2020-02-18-android-saving-states %})
