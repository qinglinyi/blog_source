---
title: Android属性动画Property Animation系列(3)_ObjectAnimator
date: 2016-03-22 11:11:25
categories: Android
tags:
     - Android
     - property animation
     - 属性动画
---


## 类概述

ObjectAnimator类是ValueAnimator类的子类，它提供了直接绑定对象的方法，能够自动更新对象的属性，不要再实现ValueAnimator.AnimatorUpdateListener接口。

使用注意：

1. 绑定的对象需要提供属性的set方法，以便ObjectAnimator自动设置属性值。
2. 如果你只给values...传入一个参数的时候，需要属性的get方法。
3. 根据属性的情况，需要调用invalidate()方法更新View。可以在onAnimationUpdate()回调方法中调用，也可以在set方法中调用。

<!-- more -->


## 使用

ObjectAnimator的使用基本和ValueAnimator类似。在这里就简单的实现一下：

*代码*

```
private void codeBegin() {
        ObjectAnimator objectAnimator;
        if (codeLine.getTag() != null 
            && codeLine.getTag() instanceof ObjectAnimator) {
            objectAnimator = (ObjectAnimator) codeLine.getTag();
        } else {
         // objectAnimator = ObjectAnimator.ofFloat(codeLine, "myX", 0f, 800f);
         // 使用这个方式的时候，注意需要同时具有set和get方法
            objectAnimator = ObjectAnimator.ofFloat(codeLine, "myX", 800f);
            objectAnimator.setDuration(4000);
            objectAnimator.setInterpolator(new AccelerateDecelerateInterpolator());
            codeLine.setTag(objectAnimator);
        }
        objectAnimator.end();
        objectAnimator.start();
    }
```

*xml*

```
<?xml version="1.0" encoding="utf-8"?>
<objectAnimator xmlns:android="http://schemas.android.com/apk/res/android"
                android:duration="4000"
                android:interpolator="@android:interpolator/accelerate_decelerate"
                android:repeatCount="1"
                android:valueFrom="0"
                android:valueTo="800"
                android:repeatMode="restart"
                android:propertyName="myX"
                android:valueType="floatType"/>
```



其他方式的使用可参考ValueAnimator。**代码在[这里](https://github.com/qinglinyi/Animator)**


## 总结

1. ObjectAnimator是ValueAnimator的子类，绑定对象直接变化属性进行动画。
2. 基本使用类似ValueAnimator，但是注意对象属性的set和get方法以及invalidate()更新UI。


