---
title: 属性动画系列之ViewPropertyAnimator
date: 2016-03-29 15:55:30
categories: Android
tags:
     - Android
     - property animation
     - 属性动画
---

## 写在开始

转载请注明：http://www.qinglinyi.com/posts/ViewPropertyAnimator/

属性动画动画系列从开始到现在，其他类例如ValueAnimator、ObjectAnimator还有AnimatorSet等的介绍挺多的，但是关于ViewPropertyAnimator这个类的介绍似乎比较少，所以本文详细的介绍一下这个类（主要是这个类的内部实现）。

<!-- more -->

## 类概述

ViewPropertyAnimator这个类能够对View对象进行自动添加和优化属性动画（用来便捷方式处理一系列View对象的属性动画的类）。

这个类提供了链式调用进行多个属性同时变化的动画，更加简洁方便我们操作组合动画（多个属性同时进行变化）。

这个类可以为同时动画提供更好的性能。这个类在动画进行的时候只对多个属性的变化进行一次invalidate调用，而不是对变化每个属性进行调用（n个ObjectAnimator就会进行n次属性变化，就有n次invalidate）。当然这个类的调用方式更方便，调用对应属性方法传一个属性值就可以自动实现动画。每个属性方法都有两种调用形式，例如 alpha(float value) 和alphaBy(float value)，前者是变化到多少，后者是变化多少。

这个类没用提供公开的构造方法，通过调用view的animate()获取引用。

上面说的比较拗口，**总结**起来就是：

1. 这个类操作View对象的。
2. 提供链式调用设置多个属性动画，这些动画同时进行的。
3. 更好的性能，多个属性动画是一次同时变化，只执行一次UI刷新。
4. 每个属性提供两种类型方法设置。
5. 这个类只能通过View的animate()获取引用进行通话设置。


## 使用

```java
	imageView.animate()
           .setDuration(4000)
           .rotationY(45f)
           .translationX(imageView.getWidth())
           .alpha(0f);
```

如上，使用很方便，对imageView设置三个属性的变化，同时进行动画。

**等价**的ObjectAnimator形式是这样的：

```java
 new AnimatorSet().playTogether(
       ObjectAnimator.ofFloat(imageView, "translationX", imageView.getWidth()),
       ObjectAnimator.ofFloat(imageView, "alpha", 0),
       ObjectAnimator.ofFloat(imageView, "rotationY", 45f)
 );
```
前者明显**更简洁**。

ViewPropertyAnimator使用很简单，那么来看看它是怎么实现的。

## 内部类

1. NameValuesHolder  标记每个属性值对应的开始值和变化值。
2. PropertyBundle    这个类包含的全局动画的属性集合的信息。包含一个NameValueHolder列表和mPropertyMask，mPropertyMask是标记这个动画都有什么属性变化。通过这个类我们可以知道都有那么属性需要变化，变化的值是什么。
3. AnimatorEventListener  操作各种Animator事件的工具类。这个类实现了Animator.AnimatorListener, ValueAnimator.AnimatorUpdateListener接口。我们只需要关心这些事件的**结束事件**（在动画结束之后用来清除animator map的事件）和**更新事件**（计算view对象的当前属性值，下面还会介绍）。


## 重要属性

**ArrayList<NameValuesHolder> mPendingAnimations** : 要进行动画的属性值（NameValueHolder）列表。

![mPendingAnimations](/images/animation/view_property_animator_0.png)

**Runnable mAnimationStarter** : 用来执行动画的Runnable。它会执行startAnimation方法，启动动画。

![mAnimationStarter](/images/animation/view_property_animator_1.png)

**HashMap<Animator, PropertyBundle> mAnimatorMap** : Animator到PropertyBundle的Map。这个Map放到是正在执行的动画和对应的PropertyBundle，这个PropertyBundle包含这这次动画的所有进行动画的属性的信息。在AnimatorEventListener的更新方法中会通过这个map获取相应的属性，然后更新属性值，达到动画效果。这个Map同时还保证当前动画的属性变化是唯一的，不会存在两个Animator在操作同一个属性。

![mAnimatorMap](/images/animation/view_property_animator_2.png)


## 重要方法


**animatePropertyBy(int constantName, float startValue, float byValue)方法**

```java
    private void animatePropertyBy(int constantName, float startValue, float byValue) {
        // 移除正在进行动画的对应的这个属性
        if (mAnimatorMap.size() > 0) {
            Animator animatorToCancel = null;
            Set<Animator> animatorSet = mAnimatorMap.keySet();
            for (Animator runningAnim : animatorSet) {
                PropertyBundle bundle = mAnimatorMap.get(runningAnim);
                if (bundle.cancel(constantName)) {// 移除对应属性
                    if (bundle.mPropertyMask == NONE) {// 判断还有其他属性没
                        animatorToCancel = runningAnim;
                        break;
                    }
                }
            }
            if (animatorToCancel != null) {
                animatorToCancel.cancel();
            }
        }
			// 创建这个属性的NameValuesHolder对象，放到mPendingAnimations列表中
        NameValuesHolder nameValuePair = new NameValuesHolder(constantName, startValue, byValue);
        mPendingAnimations.add(nameValuePair);
        //  执行动画
        mView.removeCallbacks(mAnimationStarter);
        mView.postOnAnimation(mAnimationStarter);
    }
```

这个方法**主要完成**三件事：

1. 如果当前已经存在正在运行的Animator的话，并且这个Animator包含这个属性的话，将这个Animator的这个属性移除，移除完之后如果这个Animator没有属性了就结束它。这个操作是保证View对象的一个属性只有一个动画在操作它。

2. 将属性和属性值变化装到NameValuesHolder类中，然后放到mPendingAnimations列表中。以便在之后放到mAnimatorMap中。

3. 取消并且重新启动mAnimationStarter就是启动动画，并且保证链式设置的属性是同时进行的。


**startAnimation方法**

```java
    private void startAnimation() {
        if (mRTBackend != null && mRTBackend.startAnimation(this)) {
            return;
        }
        mView.setHasTransientState(true);
        // 创建简单的animator,变化值从0到1
        ValueAnimator animator = ValueAnimator.ofFloat(1.0f);
        // 拷贝mPendingAnimations并且清除原有的mPendingAnimations
        ArrayList<NameValuesHolder> nameValueList =
                (ArrayList<NameValuesHolder>) mPendingAnimations.clone();
        mPendingAnimations.clear();
        // 迭代拷贝之后的mPendingAnimations获取属性值标记propertyMask
        int propertyMask = 0;
        int propertyCount = nameValueList.size();
        for (int i = 0; i < propertyCount; ++i) {
            NameValuesHolder nameValuesHolder = nameValueList.get(i);
            propertyMask |= nameValuesHolder.mNameConstant;
        }
        // 将这个animator放入到mAnimatorMap。
        mAnimatorMap.put(animator, new PropertyBundle(propertyMask, nameValueList));
        ...
        // 设置监听器器
        animator.addUpdateListener(mAnimatorEventListener);
        animator.addListener(mAnimatorEventListener);
        if (mStartDelaySet) {
            animator.setStartDelay(mStartDelay);
        }
        if (mDurationSet) {
            animator.setDuration(mDuration);
        }
        if (mInterpolatorSet) {
            animator.setInterpolator(mInterpolator);
        }
        // 启动动画
        animator.start();
    }
```
在animatePropertyBy方法中执行mView.postOnAnimation(mAnimationStarter)会触发startAnimation方法，startAnimation方法**主要**做了这些事情：

1. 创建简单的Animator,变化值从0到1，设置监听器mAnimatorEventListener。
2. 拷贝mPendingAnimations这个列表，并计算属性值标记propertyMask，生成PropertyBundle对象。
3. 使用mAnimatorMap保存这个Animator和对应的PropertyBundle对象。以便在animatePropertyBy方法和Animator监听器mAnimatorEventListener中使用。
4. 设置监听器mAnimatorEventListener并且启动动画。


**mAnimatorEventListener的onAnimationUpdate方法**

```java
        public void onAnimationUpdate(ValueAnimator animation) {
        PropertyBundle propertyBundle = mAnimatorMap.get(animation);
            ...
            ArrayList<NameValuesHolder> valueList = propertyBundle.mNameValuesHolder;
            if (valueList != null) {
                int count = valueList.size();
                // 迭代所有的属性，计算变化之后的属性值，将属性值设置给对应的属性
                for (int i = 0; i < count; ++i) {
                    NameValuesHolder values = valueList.get(i);
                    float value = values.mFromValue + fraction * values.mDeltaValue;
                    if (values.mNameConstant == ALPHA) {
                        alphaHandled = mView.setAlphaNoInvalidation(value);
                    } else {
                        setValue(values.mNameConstant, value);
                    }
                }
            }
            ...
            if (alphaHandled) {
                mView.invalidate(true);
            } else {
                mView.invalidateViewProperty(false, false);
            }
            ...
        }
```

这个方法完成了属性值的设置。通过mAnimatorMap获取对应的所有属性，计算变化之后的属性，然后设置给对应的属性，再进行更行UI。


## 对照使用串联ViewPropertyAnimator

我们再次回过来看看我们前面是怎么使用的。

```java
	imageView.animate()
           .setDuration(4000)
           .rotationY(45f)
           .translationX(imageView.getWidth())
           .alpha(0f);
```

通过imageView.animate()获取ViewPropertyAnimator类对象。调用rotationY、translationX、alpha设置属性动画。

我们看看alpha方法的**实现**：

```
    public ViewPropertyAnimator alpha(float value) {
        animateProperty(ALPHA, value);
        return this;
    }
    
    private void animateProperty(int constantName, float toValue) {
        float fromValue = getValue(constantName);
        float deltaValue = toValue - fromValue;
        animatePropertyBy(constantName, fromValue, deltaValue);
    }
```

最终调用了**animatePropertyBy(int constantName, float startValue, float byValue)方法**

这样我们就可以将这个ViewPropertyAnimator类的使用**串起来了**：

1. 通过imageView.animate()获取ViewPropertyAnimator对象。
2. 调用alpha、rotationY等方法，返回当前ViewPropertyAnimator对象，可以继续调用。
3. alpha等方法会调用animatePropertyBy(int constantName, float startValue, float byValue)方法。
4. animatePropertyBy方法启动mAnimationStarter，调用startAnimation，开始动画。
5. 在动画的监听器里面设置所有属性的变化值，并刷新。

