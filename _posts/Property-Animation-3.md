---
title: Android属性动画Property Animation系列(4)_AnimatorSet
date: 2016-03-24 10:10:47
categories: Android
tags:
     - Android
     - property animation
     - 属性动画
---

## 类概述

AnimatorSet是将一系列Animator对象按照指定的顺序进行播放的类。这些动画可以是同时播放，顺序播放，延迟播放。

主要了解以下几种方式：

1. playTogether() 两种参数形式：Animator... items和Collection<Animator> items。将这些Animator同时进行播放。
2. playSequentially() 和playTogether类似也是两种参数形式，Animator是顺序播放。
3. play(Animator) 返回一个Builder，这个Builder提供with,before方法设置这些Animator的顺序。

其实，playTogether和playSequentially也是使用play(Animator)和Builder的方式实现的。

<!-- more -->


## 使用（**[源码](https://github.com/qinglinyi/Animator)**）

**代码中**

先设置四个测试Animator
```
 animatorA = ObjectAnimator.ofFloat(mImageView, "translationX", 0f, 300f, 0f);// 位移
 animatorB = ObjectAnimator.ofFloat(mImageView, "scaleX", 1f, 1.5f, 1f);// 缩放
 animatorC = ObjectAnimator.ofFloat(mImageView, "rotationX", 0f, 90f, 0f);// 旋转
 animatorD = ObjectAnimator.ofFloat(mImageView, "alpha", 1f, 0f, 1f);// 透明度
```

同时播放playTogether

```
animatorSet.playTogether(animatorA, animatorB, animatorC, animatorD);
animatorSet.start();
```

顺序播放playSequentially

```
 animatorSet.playSequentially(animatorA, animatorB, animatorC, animatorD);
 animatorSet.start();
```

还试了一把Builder

```

// 尽量使用play.after,play.after..的形式如下

// 顺序 正 A B C D
// animatorSet.play(animatorA).before(animatorB);
// animatorSet.play(animatorB).before(animatorC);
// animatorSet.play(animatorC).before(animatorD);

// 顺序 反 D C B A
// animatorSet.play(animatorA).after(animatorB);
// animatorSet.play(animatorB).after(animatorC);
// animatorSet.play(animatorC).after(animatorD);


// 同时 使用with的时候连着设置都是同时播放
// animatorSet.play(animatorA).with(animatorB).with(animatorC).with(animatorD);

// 不要这样，因为这样BCD其实是没有直接关系的
//（没有理解为什么这样设计，提供了方法，但是使用的效果和预期的不一样）
// animatorSet.play(animatorA).before(animatorB).before(animatorC); // 这个时候吃掉B了，执行顺序是A->C

// animatorSet.play(animatorA).before(animatorB).before(animatorC).before(animatorD); 
// 还是吃掉B了，执行顺序是A->C&D

// A 依赖 B , B 依赖 C , C 依赖 A 报错了😢，当然文档说了，别这么干
// animatorSet.play(animatorA).after(animatorB);
// animatorSet.play(animatorB).after(animatorC);
// animatorSet.play(animatorC).after(animatorA);

// 好吧，也别这么搞
// animatorSet.playTogether(animatorA, animatorB, animatorC);
// animatorSet.playSequentially(animatorD, animatorSet);

//B--------
//         A---------
//         C---------
//              D--------

 animatorD.setStartDelay(2000);

 animatorSet.play(animatorB).before(animatorA);
 animatorSet.play(animatorA).with(animatorC);
 animatorSet.play(animatorD).after(animatorB);
 
 animatorSet.start();
 
```

1. playTogether和playSequentially很简单，将animator传入就可以了，当然也支持Collection作为参数。
2. Builder比较绕一点，但是可以灵活完成各种要求，注意一下官方建议的调用形式。
3. **使用代码设置AnimatorSet的时候，子Animator的Duration和TimeInterpolator是会被AnimatorSet覆盖的,并且这个Duration不是Set的总时长，是每个Animator的时长(和XML不一样)。**![Duration](/images/animation/animator_set1.png)![Duration和TimeInterpolator](/images/animation/animator_set0.png)

4. PivotX，PivotY的设置没有提供方法，可以自己手动设置。



```
mImageView.setPivotX(mImageView.getWidth() / 2);
mImageView.setPivotY(mImageView.getHeight() / 2);
```


**XML**

相对应的XML实现就比较麻烦了

顺序执行

```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
     android:ordering="sequentially">
    <!-- 官网显示支持propertyValuesHolder和keyframe，但是我测试还是不行。所以只能用这个笨方法了-->
    <!-- ps:测试机 4.4.4和5.0 -->
    <objectAnimator
        android:duration="2000"
        android:interpolator="@android:interpolator/linear"
        android:propertyName="translationX"
        android:valueFrom="0"
        android:valueTo="300"
        android:valueType="floatType"/>

    <objectAnimator
        android:duration="2000"
        android:interpolator="@android:interpolator/linear"
        android:propertyName="translationX"
        android:valueFrom="300"
        android:valueTo="0"
        android:valueType="floatType"/>

    <objectAnimator
        android:startOffset="2000"
        android:duration="2000"
        android:interpolator="@android:interpolator/linear"
        android:propertyName="scaleX"
        android:valueFrom="1"
        android:valueTo="1.5"
        android:valueType="floatType"/>

    <objectAnimator
        android:duration="2000"
        android:interpolator="@android:interpolator/linear"
        android:propertyName="scaleX"
        android:valueFrom="1.5"
        android:valueTo="1"
        android:valueType="floatType"/>

    <objectAnimator
        android:duration="2000"
        android:interpolator="@android:interpolator/linear"
        android:propertyName="rotationX"
        android:valueFrom="0"
        android:valueTo="90"
        android:valueType="floatType"/>

    <objectAnimator
        android:duration="2000"
        android:interpolator="@android:interpolator/linear"
        android:propertyName="rotationX"
        android:valueFrom="90"
        android:valueTo="0"
        android:valueType="floatType"/>

    <objectAnimator
        android:duration="2000"
        android:interpolator="@android:interpolator/linear"
        android:propertyName="alpha"
        android:valueFrom="1"
        android:valueTo="0"
        android:valueType="floatType"/>

    <objectAnimator
        android:duration="2000"
        android:interpolator="@android:interpolator/linear"
        android:propertyName="alpha"
        android:valueFrom="0"
        android:valueTo="1"
        android:valueType="floatType"/>

</set>
```


同时执行

```
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
     android:ordering="sequentially">
    <set android:ordering="together">
        <objectAnimator
            android:duration="2000"
            android:interpolator="@android:interpolator/linear"
            android:propertyName="translationX"
            android:valueFrom="0"
            android:valueTo="300"
            android:valueType="floatType"/>
        <objectAnimator
            android:duration="2000"
            android:interpolator="@android:interpolator/linear"
            android:propertyName="scaleX"
            android:valueFrom="1"
            android:valueTo="1.5"
            android:valueType="floatType"/>
        <objectAnimator
            android:duration="2000"
            android:interpolator="@android:interpolator/linear"
            android:propertyName="rotationX"
            android:valueFrom="0"
            android:valueTo="90"
            android:valueType="floatType"/>
        <objectAnimator
            android:duration="2000"
            android:interpolator="@android:interpolator/linear"
            android:propertyName="alpha"
            android:valueFrom="1"
            android:valueTo="0"
            android:valueType="floatType"/>
    </set>

    <set android:ordering="together">

        <objectAnimator
            android:duration="2000"
            android:interpolator="@android:interpolator/linear"
            android:propertyName="translationX"
            android:valueFrom="300"
            android:valueTo="0"
            android:valueType="floatType"/>


        <objectAnimator
            android:duration="2000"
            android:interpolator="@android:interpolator/linear"
            android:propertyName="scaleX"
            android:valueFrom="1.5"
            android:valueTo="1"
            android:valueType="floatType"/>


        <objectAnimator
            android:duration="2000"
            android:interpolator="@android:interpolator/linear"
            android:propertyName="rotationX"
            android:valueFrom="90"
            android:valueTo="0"
            android:valueType="floatType"/>


        <objectAnimator
            android:duration="2000"
            android:interpolator="@android:interpolator/linear"
            android:propertyName="alpha"
            android:valueFrom="0"
            android:valueTo="1"
            android:valueType="floatType"/>
    </set>
</set>
```




