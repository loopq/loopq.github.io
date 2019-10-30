---
layout:     post
title:      "记一次解决CoordinatorLayout 内部嵌套滑动的问题"
subtitle:   " \"coordinatorLayout with appbar and recyclerView\""
date:       2019/10/28 10:39:30
author:     "loopq"
header-img: "img/post_bg_train.jpg"
catalog: true
tags:
    - Android
---

### 问题视频

![bug视频](https://tva1.sinaimg.cn/large/006y8mN6ly1g8g25gbm69g30k00zku14.gif)

### 问题简述

这是一个普通的CoordinatorLayout + AppbarLayout + RecyclerView 布局，你可以看到Appbar有一个**Scroll**按钮，当我点击按钮的时候**RecyclerView**会强制定位到第20位，也就是 **ScrollToPosition**方法。也就是这个时候，问题来了，上方的AppbarLayout即使没有滑动到最顶部也不能滑动了。

布局代码：

```xml
<androidx.coordinatorlayout.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/root_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fitsSystemWindows="true">

    <com.google.android.material.appbar.AppBarLayout
        android:id="@+id/appbar_layout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:layout_behavior="com.google.android.material.appbar.AppBarLayout$Behavior">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:minHeight="48dp"
            android:orientation="vertical"
            app:layout_scrollFlags="scroll|exitUntilCollapsed">

            <TextView
                android:id="@+id/text_view"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@string/text"
                android:textColor="@android:color/white" />

            <Button
                android:id="@+id/btn_scroll"
                android:layout_width="90dp"
                android:layout_height="48dp"
                android:text="scroll"
                android:textSize="14sp" />
        </LinearLayout>

    </com.google.android.material.appbar.AppBarLayout>

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recycler_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layoutManager="androidx.recyclerview.widget.LinearLayoutManager"
        app:layout_behavior="com.google.android.material.appbar.AppBarLayout$ScrollingViewBehavior" />

</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

### 问题解决思路

- 首先这个问题是在我们项目中出现的，然后我自己实现了一个Demo复现了这个问题。界面效果类似于豆瓣的影片详情页，我们实现的思路和豆瓣几乎一模一样，但是豆瓣的底部RecyclerView强制定位之后AppbarLayout依然可以滚动,所以这就是很纳闷的地方。于是我去看了一下豆瓣的UI布局，发现和我们的布局基本一致，除了豆瓣的Appbar里面是套了一层RecyclerView。当时看的时候觉得「**嗯，人家AppbarLayout里面的东西比较多，用一个RecyclerView也很正常」**，后面才想到难怪别人没有遇到这个问题，遇到问题凡事多想想总没坏处。
- 然后去看了CoordinatorLayout内部是如何工作的，RecylerView如何联动AppbarLayout，看到了**NestedScrollParent2**和**NestedScrollChild2**用来处理滚动的逻辑，于是我掉头回去看了看AppbarLayout的源码，发现是继承了LinearLayout，并且不具备滑动的特性，它能滑动是依赖的**RecyclerView**和**CoordinatorLayout**，中间也顺便翻到了RecyclerView的源码，RecyclerView内部持有**NestedScrollChildHelper**用来处理滑动，回过头想到我的布局代码和豆瓣的唯一区别就是AppbarLayout里面是否能嵌套滑动（豆瓣appbar里面有RecyclerView），于是我在AppbarLayout里面套了一层**NestedScrollView**，所有问题迎刃而解。
- 中间走了很多弯路，Google也搜索了很多，基本上没有类似的情况。这时候就只能自己耐心看源码，理解工作机制才能解决问题。虽然最终就加了几行代码，但是中间的过程却是很有意义的。