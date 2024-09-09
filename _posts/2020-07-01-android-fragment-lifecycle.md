---
title: 【Android】Fragment生命周期
date: 2020-07-01 14:42 +0800
categories: [Android]
tags: [android, fragment]
---
* Fragment有和Activity类似的生命周期，且受其所属的Activity的生命周期影响：

![Fragment生命周期](/assets/images/android-fragment-lifecycle/Fragment生命周期.png)

![Activity状态与Fragment回调](/assets/images/android-fragment-lifecycle/Activity状态与Fragment回调.png)

* 常用的生命周期：
  * onAttach()：在Fragment已与Activity关联时进行调用
  * onCreate()：系统会在创建Fragment时调用此方法
  * onCreateView()：系统会在Fragment首次绘制其界面时调用此方法
  * onActivityCreated()：当Activity的onCreate()方法已返回时进行调用
  * onViewStateRestored()：（该方法在官方文档的图中未显示）当Fragment的视图层次结构的状态均已恢复时调用，在onActivityCreated()之后、onStart()之前调用（此时可以通过savedInstanceState参数是否为null判断初次创建还是重建，且子Fragment的onCreateView()已调用）
  * onStart()：当Fragment对用户可见时调用
* 由于FragmentTransaction的commit()方法不是立即执行的，因此不能在Fragment的onCreateView()方法中访问子Fragment在onCreateView()方法中初始化的数据（例如视图元素）
* 假设FragmentA包含了FragmentB和FragmentC，且FragmentB在FragmentC之前创建，则初次创建时各回调函数的调用顺序如下：

```
FragmentA.onCreateView
FragmentA.onActivityCreated
FragmentB.onCreateView
FragmentB.onActivityCreated
FragmentB.onViewStateRestored
FragmentC.onCreateView
FragmentC.onActivityCreated
FragmentC.onViewStateRestored
FragmentA.onViewStateRestored
FragmentA.onStart
FragmentB.onStart
FragmentC.onStart
```
