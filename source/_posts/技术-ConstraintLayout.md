---
title: 从需求探索ConstraintLayout
date: 2018-12-01 09:25:00
author: ThisMJ
img: /medias/images/2018-12-01.jpeg 
mathjax: false
categories: Android
tags:
  - Layout
---

## 前言
对于前端开发，画界面这种事情是必不可少的，并且能够直观地看到代码到视觉的转换，我觉得是一件很有意思的事情。当然，界面是由很多子元素组成的，我们需要选取不同的布局方式，把这些子元素按照设计稿的要求排列起来。首先简明扼要地概括一下Android中几种常用的布局方式：

* 帧布局（FrameLayout）：子控件按照先后顺序叠在一起。
* 线性布局（LinearLayout）：子控件水平方向排成一行或者垂直方向排成一列。
* 相对布局（RelativeLayout）：子控件的位置相对于同层的其它子控件或者父控件排列。

以上三种布局方式，在Android开发中基本算是家常便饭了，它们术业有专攻，各司其职。也正因为如此，如果碰到一个稍微复杂点的界面，需要它们相互配合，一层一层地嵌套起来。那么有没有一种布局方式能够足够灵活，使所有的子元素都能平铺开来而没有任何嵌套呢？

<!--![](http://5b0988e595225.cdn.sohucs.com/images/20171018/6b61446f7ccf448d893b4ff1dc031f77.jpeg)-->

ConstraintLayout，了解一下？那么，我们根据相关的需求来探索一下，约束布局到底能做什么！

---

## 边的相对性
首先，最基本的，两个Button A和B，A需要放在父控件的左上角，B需要放在A的右边。用约束布局来描述的话就是：在水平方向，A的左边约束在父控件的左边，A的上边约束在父控件的上边；B的左边约束在A的右边，B的上边约束在父控件的上边。听起来更复杂些，但确是合理的，因为仅仅只是某个方向的约束并不能确定一个子控件的具体位置，所以对于约束布局来说，每一个子控件必须在水平以及垂直方向至少施加一个约束条件。

![](http://thismj.nos-eastchina1.126.net/image/post_constraints_relative_positioning_example.png)
```xml
<android.support.constraint.ConstraintLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:ignore="HardcodedText">

    <Button
        android:id="@+id/a"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginStart="8dp"
        android:layout_marginTop="8dp"
        android:text="A"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <Button
        android:id="@+id/b"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginStart="8dp"
        android:layout_marginTop="8dp"
        android:text="B"
        app:layout_constraintStart_toEndOf="@id/a"
        app:layout_constraintTop_toTopOf="parent" />

</android.support.constraint.ConstraintLayout>
```

按照个人理解，RelativeLayout的定义倾向于整体，比如Button B在Button A的右边，但是ConstraintLayout更倾向于部分，比如Button B的左边约束在Button A的右边，当然，实际上原理跟结果是一样的。

![](http://thismj.nos-eastchina1.126.net/image/post_constraints_relative_positioning.png)

---

## 如何居中
相对定位是有了，那么如何让子控件居中呢？这种需求基本算是家常便饭了。我们知道，不管是帧布局、线性布局还是相对布局，都有layout_gravity的属性，但是约束布局是没有直接属性可以让一个控件居中的，它以一种更加通用的方式来实现居中，看看代码就清楚了：
```xml
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:ignore="HardcodedText">

    <Button
        android:id="@+id/a"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="A"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</android.support.constraint.ConstraintLayout>
```

![](http://thismj.nos-eastchina1.126.net/image/post_constraints_mediate.png)

为什么说这种方式更通用，因为layout_gravity只能相对于父控件居中，而约束布局居中的方式，可以让一个控件相对于其它任何控件居中，并且不会有布局嵌套的情况。如下图所示：

![](http://thismj.nos-eastchina1.126.net/image/post_constraints_mediate_2.png)

---

## 居中偏移
看来约束布局对居中的需求是游刃有余啊，但是对于有美感追求的设计师来说，可能认为标准的居中会过于死板，他们希望你能把控件放在中间再往上偏移10%的位置，或者放在黄金分割比的位置。那么，约束布局要如何做到呢？

对约束布局来说，当一个控件设置了居中之后，可以使用Bias相关属性，Bias的默认值是0.5，代表控件居中摆放，可以通过改变这个值来让控件在水平或者竖直方向上做偏移。比如把Button A放置在父控件中心往上偏移10%的位置：

![](http://thismj.nos-eastchina1.126.net/image/post_constraints_bias_example.png)

可以通过设置竖直方向的Bias值来实现：
```xml
<android.support.constraint.ConstraintLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:ignore="HardcodedText">

    <Button
        android:id="@+id/a"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="A"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_bias="0.4" />

</android.support.constraint.ConstraintLayout>
```

---

## 中心的相对性
需求不会永远都是中规中矩的，比如：以父控件为中心，摆个时钟造型出来。这样的需求普通的布局就没法实现了，可能需要我们自己去动态计算角度跟距离进行绘制，略显麻烦，但是对约束布局来说，它都给你实现好了：

```xml
<android.support.constraint.ConstraintLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:ignore="HardcodedText">

    <View
        android:id="@+id/a"
        android:layout_width="10dp"
        android:layout_height="10dp"
        android:background="@drawable/circle_accent"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="12"
        android:textColor="@android:color/white"
        app:layout_constraintCircle="@id/a"
        app:layout_constraintCircleAngle="0"
        app:layout_constraintCircleRadius="100dp" />

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="1"
        android:textColor="@android:color/white"
        app:layout_constraintCircle="@id/a"
        app:layout_constraintCircleAngle="30"
        app:layout_constraintCircleRadius="100dp" />
    
    ...(省略一些)
    
    <View
        android:layout_width="1dp"
        android:layout_height="80dp"
        android:background="@color/colorAccent"
        android:rotation="0"
        android:transformPivotX="1dp"
        android:transformPivotY="80dp"
        app:layout_constraintCircle="@id/a"
        app:layout_constraintCircleAngle="0"
        app:layout_constraintCircleRadius="40dp" />

    <View
        android:layout_width="2dp"
        android:layout_height="60dp"
        android:background="@color/colorAccent"
        android:rotation="60"
        android:transformPivotX="1dp"
        android:transformPivotY="60dp"
        app:layout_constraintCircle="@id/a"
        app:layout_constraintCircleAngle="0"
        app:layout_constraintCircleRadius="30dp" />

</android.support.constraint.ConstraintLayout>
```

这种布局方式称之为中心定位，效果如下图：

![](http://thismj.nos-eastchina1.126.net/image/post_constraints_circle_position_example.png)

---

## 外边距的调整
继续探究，看下图，Button A、B、C、D、E、F相对摆放，分别标记A、C、E为GONE，观察此时B、D、F的行为：

![](http://thismj.nos-eastchina1.126.net/image/post_constraints_visibility_behavior.gif)

当有子控件被标记为GONE的时候，界面会重新布局，此时其它子控件的位置会相应的发生改变。我们可以把标记为GONE的子控件看成一个点，它对其它子控件的约束依然是成立的（但是自身的Margin属性都会被忽略），所以当A消失的时候，B会往左边移动。想一下，如果当A消失的时候，我们给B设置一个不同的Margin值，那么B的行为就跟D、F一样发生了变化。如果使用相对布局，我们可以在代码中通过LayoutParam动态设置Margin值来实现，但是约束布局就不一样了，它提供了这些属性。值得注意的是，这些属性在做一些简单的布局动画的时候相当有用。

---

## 百分比布局
Android的屏幕适配是个大问题，所以平常写布局的时候比较倾向于通过比例去指定控件的宽高，之前Google出过一个百分比的库，现在去看已经被标记为deprecated了，而替代品就是约束布局。一个简单的需求，Button A在屏幕的中间，宽是屏幕的一半，高是屏幕的1/8，在实现这个需求之前，我们需要先了解一下约束布局一般如何指定layout_width和layout_height：
* 使用固定尺寸，例如123dp
* 使用WRAP_CONTENT，子控件计算自身尺寸
* 使用0dp，相当于"MATCH_CONSTRAINT"，在约束范围内所能占据的最大尺寸

我们着重关注下0dp，以前我们对一个控件使用0dp的时候，一般是跟weight属性配合使用的，但是约束布局里面，它有着其他的含义，如下图，我们把A的左边约束在父控件的左边，A的右边约束在父控件的右边。图a，指定A的宽度为WRAP_CONTENT；图b，指定A的宽度为0dp；图c，指定A的宽度为0dp，并在其左边设置一定的margin值。

![](http://thismj.nos-eastchina1.126.net/image/post_constraints_dimension.png)

MATCH_PARENT也是可以正常使用的，但是在约束布局里面推荐使用0dp然后约束在父控件两边来实现同等效果（如上图b）。

回到刚才的需求，Button A在屏幕的中间，宽是屏幕的一半，高是屏幕的1/8，我们可以这么实现（宽跟高必须设置为0dp）：

```xml
<android.support.constraint.ConstraintLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:ignore="HardcodedText">

    <Button
        android:id="@+id/a"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:text="A"
        app:layout_constraintWidth_percent="0.5"
        app:layout_constraintHeight_percent="0.125"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintTop_toTopOf="parent" />
        
</android.support.constraint.ConstraintLayout>
```

![](http://thismj.nos-eastchina1.126.net/image/post_constraints_dimension_percent.png)

---

## 固定宽高比
再来个需求，Button A在屏幕中心往上偏移20%的位置，并且宽高比固定为1:1，很显然，在约束布局中，这也是很容易就能实现的：

```xml
<android.support.constraint.ConstraintLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:ignore="HardcodedText">

    <Button
        android:id="@+id/a"
        android:layout_width="wrap_content"
        android:layout_height="0dp"
        android:text="A"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintDimensionRatio="1:1"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_bias="0.3" />

</android.support.constraint.ConstraintLayout>
```

![](http://thismj.nos-eastchina1.126.net/image/post_constraints_dimension_ratio.png)

如上所示，要固定控件的宽高比，我们需要先设置宽或者高为0dp，被设置为0dp一边的尺寸，会根据ratio值以及另一边的尺寸进行计算得出。当然，如果宽跟高都设置为0dp的话，也是可以的，不过需要在ratio值前面增加"W"或者"H"来指定那一边会被约束。举个例子，比如我们实现一个首页的banner，宽度要跟屏幕相等，并且宽高比固定为16:9，因为宽必须跟屏幕相等，所以我们需要指定高被约束，即在ratio值前面加上"H"，如下：

```xml
<android.support.constraint.ConstraintLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:ignore="HardcodedText">

    <android.support.v4.view.ViewPager
        android:id="@+id/a"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:background="@color/colorAccent"
        app:layout_constraintDimensionRatio="H,16:9"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</android.support.constraint.ConstraintLayout>
```
![](http://thismj.nos-eastchina1.126.net/image/post_constraints_dimension_ratio_2.png)

---

## Chains
再继续探索一下，我们上面所讨论的都是单个子控件的情况，但是时常会有这样的需求，比如，让Button A、B、C整体相对于父控件水平居中。传统的三把斧布局方式要实现这个需求那是相当简单，比如可以把A、B、C放进D里面，然后再把D居中于父控件即可；或者可以使用线性布局的gravity属性。但是，约束布局没有gravity属性，并且也不可能使用布局嵌套，那么它要怎么做到？

想让A、B、C整体居中于父控件，得先把它们看成一个整体，在约束布局中，我们可以把A、B、C看成是一个控件组，称之为链（Chains），根据需求，我们实现一条水平链：

```xml
<android.support.constraint.ConstraintLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:ignore="HardcodedText">

    <Button
        android:id="@+id/a"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="A"
        app:layout_constraintEnd_toStartOf="@+id/b"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintHorizontal_chainStyle="packed"/>

    <Button
        android:id="@+id/b"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="B"
        app:layout_constraintEnd_toStartOf="@+id/c"
        app:layout_constraintStart_toEndOf="@+id/a" />

    <Button
        android:id="@+id/c"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="C"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@+id/b" />

</android.support.constraint.ConstraintLayout>
```

![](http://thismj.nos-eastchina1.126.net/image/post_constraints_chains_packed.png)

控件链的第一个控件，称之为链头，如上面代码所示，我们可以在链头上设定chainStyle属性，来实现各种不同的效果，上图为了实现A、B、C整体居中的需求，使用的chainStyle是packed，在使用packed的时候，我们当然也可以在链头上使用Bias属性，实现中心偏移的需求：

![](http://thismj.nos-eastchina1.126.net/image/post_constraints_chains_styles.png)

实际上，chainStyle并没有weighted这个属性值，但是我们可以实现类似于线性布局weight属性的效果：

```xml
<android.support.constraint.ConstraintLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:ignore="HardcodedText">

    <Button
        android:id="@+id/a"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="A"
        app:layout_constraintEnd_toStartOf="@+id/b"
        app:layout_constraintHorizontal_chainStyle="packed"
        app:layout_constraintHorizontal_weight="3.0"
        app:layout_constraintStart_toStartOf="parent" />

    <Button
        android:id="@+id/b"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="B"
        app:layout_constraintEnd_toStartOf="@+id/c"
        app:layout_constraintHorizontal_weight="2.0"
        app:layout_constraintStart_toEndOf="@+id/a" />

    <Button
        android:id="@+id/c"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:text="C"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_weight="1.0"
        app:layout_constraintStart_toEndOf="@+id/b" />

</android.support.constraint.ConstraintLayout>
```

![](http://thismj.nos-eastchina1.126.net/image/post_constraints_%20chains_weighted.png)

---

## Barrier
因为约束布局讲究得是所有的子控件都平铺开来，而没有任何布局嵌套，所以对一些存在整体相对性的需求时，只能另辟蹊径。再看一个例子，如下图所示，上面有两个文本控件，内容是根据后台接口动态变化的，文本区域的下方，我们需要放一个Button：

![](http://thismj.nos-eastchina1.126.net/image/post_constraints_barrier_1.png)

现在我们开始开发，根据设计稿，分分钟就把布局写好了：

```xml
<android.support.constraint.ConstraintLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="10dp"
    tools:ignore="HardcodedText">

    <TextView
        android:id="@+id/a"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:maxWidth="200dp"
        android:text="左边的文字......"
        android:textColor="@android:color/white"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/b"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_marginStart="10dp"
        android:text="右边的文字......"
        android:textColor="@android:color/white"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@+id/a"
        app:layout_constraintTop_toTopOf="parent" />

    <Button
        android:id="@+id/c"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="10dp"
        android:background="@drawable/rect_round_accent"
        android:text="按钮"
        android:textColor="@android:color/white"
        android:textColorHint="@color/app_white_70"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/a" />

</android.support.constraint.ConstraintLayout>
```

得到的效果也跟设计稿相差无几，于是需求顺利完成了，但是过了一段时间之后，有人过来给你报了个bug，并提供了截图作为证据，如下：

![](http://thismj.nos-eastchina1.126.net/image/post_constraints_barrier_2.png)

通过分析发现，之前写布局的时候，因为设计稿里面左边文本控件看起来内容比较多，所以把Button约束在了左边文本控件的下面，但是这两个文本控件的内容不是固定的，公司的产品又需要支持多语言，当内容变化或者翻译成英文的时候，反而右边的文本控件内容比左边多了，这就导致了UI错误叠加的问题。那么这个问题约束布局如何解？

约束布局为了应对跟整体挂钩的需求，定义了多个虚拟的辅助类，这些虚拟的控件仅仅起到辅助布局的作用，并不占据任何空间。Barrier就是其中一员，它可以指定多个控件id，然后在某个方向施加一个屏障，这样其它的控件就可以在这个屏障上施加约束条件了。针对上面的需求，我们修改布局文件如下：

```xml
<android.support.constraint.ConstraintLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="10dp"
    tools:ignore="HardcodedText">

    <TextView
        android:id="@+id/a"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:maxWidth="200dp"
        android:textColor="@android:color/white"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <TextView
        android:id="@+id/b"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:textColor="@android:color/white"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@+id/a"
        app:layout_constraintTop_toTopOf="parent" />

    <android.support.constraint.Barrier
        android:id="@+id/barrier"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:barrierDirection="bottom"
        app:constraint_referenced_ids="a,b" />

    <Button
        android:id="@+id/c"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="10dp"
        android:background="@drawable/rect_round_accent"
        android:text="button"
        android:textColor="@android:color/white"
        android:textColorHint="@color/app_white_70"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/barrier" />

</android.support.constraint.ConstraintLayout>
```

效果如下：

![](http://thismj.nos-eastchina1.126.net/image/post_constraints_barrier_3.png)

---

## Guideline
辅助控件Guideline，跟Barrier比较相似，它们的区别在于，Barrier需要指定多个控件，它的位置会根据这些控件尺寸的改变而改变。但是Guideline的位置是固定的，固定的位置可以是具体的尺寸也可以是相对于父控件的百分比，我们可以在水平或者竖直方向设置Guideline。比如，在父控件30%的地方设置一个水平方向的Guideline，然后Button A、B、C分别约束在这个Guideline的上下两侧：

```xml
<android.support.constraint.ConstraintLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:ignore="HardcodedText">

    <android.support.constraint.Guideline
        android:id="@+id/guideline"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        app:layout_constraintGuide_percent="0.3" />

    <Button
        android:id="@+id/a"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="A"
        app:layout_constraintBottom_toTopOf="@id/guideline"
        app:layout_constraintStart_toStartOf="parent" />

    <Button
        android:id="@+id/b"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="B"
        app:layout_constraintStart_toEndOf="@+id/a"
        app:layout_constraintTop_toTopOf="@id/guideline" />

    <Button
        android:id="@+id/c"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="C"
        app:layout_constraintBottom_toTopOf="@id/guideline"
        app:layout_constraintStart_toEndOf="@id/b" />

</android.support.constraint.ConstraintLayout>
```
![](http://thismj.nos-eastchina1.126.net/image/post_constraints_guideline.png)

Guideline的使用场景也是相当多的，举个例子，一般我们设置margin都是固定dp值，但是如果想以父控件尺寸的百分比作为margin值，Guideline就能发挥作用了。

---

## Group
想一下，约束布局如果想让多个子控件同时标记为GONE或者VISIBLE，因为没有布局嵌套了，是否要一个一个地操作，那未免也太蠢了，所以就有了Group这个辅助控件，它可以指定多个子控件，来整体控制它们的可见性，举例如下：

```xml
<android.support.constraint.ConstraintLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:ignore="HardcodedText">

    <android.support.constraint.Guideline
        android:id="@+id/guideline"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        app:layout_constraintGuide_percent="0.3" />

    <Button
        android:id="@+id/a"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="A"
        app:layout_constraintBottom_toTopOf="@id/guideline"
        app:layout_constraintStart_toStartOf="parent" />

    <Button
        android:id="@+id/b"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="B"
        app:layout_constraintStart_toEndOf="@+id/a"
        app:layout_constraintTop_toTopOf="@id/guideline" />

    <Button
        android:id="@+id/c"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="C"
        app:layout_constraintBottom_toTopOf="@id/guideline"
        app:layout_constraintStart_toEndOf="@id/b" />

    <android.support.constraint.Group
        android:id="@+id/group"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:visibility="gone"
        app:constraint_referenced_ids="b,c" />

</android.support.constraint.ConstraintLayout>
```

![](http://thismj.nos-eastchina1.126.net/image/post_constraints_group.png)

---

## Placeholder
另外一个辅助控件Placeholder，它相当于一个占位符，可以通过指定一个其它的控件id来显示具体的内容，一个非常简单的例子：

```xml
<android.support.constraint.ConstraintLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:ignore="HardcodedText, ContentDescription">

    <Button
        android:id="@+id/a"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="A" />

    <android.support.constraint.Placeholder
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:content="@id/a"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent" />

</android.support.constraint.ConstraintLayout>
```

在约束布局中，一个控件必须在水平和竖直方向至少各施加一个约束，但是很显然上面的Button A并没有施加任何约束。我们在下面的Placehoder辅助控件中指定了Button A的id，看看预览图：

![](http://thismj.nos-eastchina1.126.net/image/post_constraints_placeholder.png)

可以看出，Button A的位置显示在了Placeholder的位置上，很容易理解，但是我们需要思考一下，这个辅助控件的使用场景。首先，因为Placeholder的特性，我们可以把它应用在布局模版上面，是的，代码有template，布局也可以有template，比如我们写一个首页的布局模版：

```xml
<merge xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:parentTag="android.support.constraint.ConstraintLayout">

    <android.support.constraint.Placeholder
        android:id="@+id/template_content"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:content="@id/content"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <android.support.constraint.Placeholder
        android:id="@+id/template_tab_1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="10dp"
        app:content="@+id/tab_1"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toStartOf="@id/template_tab_2"
        app:layout_constraintStart_toStartOf="parent" />

    <android.support.constraint.Placeholder
        android:id="@+id/template_tab_2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="10dp"
        app:content="@+id/tab_2"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toStartOf="@id/template_tab_3"
        app:layout_constraintStart_toEndOf="@id/template_tab_1" />

    <android.support.constraint.Placeholder
        android:id="@+id/template_tab_3"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="10dp"
        app:content="@+id/tab_3"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toStartOf="@+id/template_tab_4"
        app:layout_constraintStart_toEndOf="@+id/template_tab_2" />

    <android.support.constraint.Placeholder
        android:id="@+id/template_tab_4"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginBottom="10dp"
        app:content="@id/tab_4"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toEndOf="@+id/template_tab_3" />

</merge>
```

布局模版写好了，我们可以把它放进公共库里面，如果要新开发一个首页，那么我们就可以使用模版了，模版里面所有控件的位置都已经固定，我们需要做的就是往里面添东西，举个例子：

```xml
<android.support.constraint.ConstraintLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:ignore="HardcodedText,ContentDescription">
    
    <include layout="@layout/template_constraint_placeholder" />

    <TextView
        android:id="@+id/content"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:gravity="center_vertical"
        android:text="Content"
        android:textSize="20sp" />

    <ImageView
        android:id="@+id/tab_1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@drawable/tree" />

    <ImageView
        android:id="@+id/tab_2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@drawable/camping_tent" />

    <ImageView
        android:id="@+id/tab_3"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@drawable/boat" />

    <ImageView
        android:id="@+id/tab_4"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@drawable/television" />

</android.support.constraint.ConstraintLayout>
```

我们首先把模版include进来，然后根据所设置的id，把所有控件添加进去就可以了，此时不用再为这些控件设置约束条件，因为模版里面已经封装好了，效果：

![](http://thismj.nos-eastchina1.126.net/image/post_constraints_placeholder_template.png)

Placeholder是否可以用在页面加载占位图上面？

---

## 总结
综上所述，ConstraintLayout基本集合了常用布局的所有特性，针对复杂的布局，可以最大限度地减少嵌套问题。








