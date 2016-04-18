---
title: Android属性动画Property Animation系列(2)_VauleAnimator
date: 2016-03-17 11:59:55
categories: Android
tags:
     - Android
     - property animation
     - 属性动画
---


## 类概述

这个类提供一个简单的运行动画的时间机制，这个机制会计算动画值并且将这个值设置给目标对象。

它有一个供所有动画使用的简单的定时器。并且它运行一个自定义的Handler来保证属性变化是发生在UI线程。

ValueAnimator 默认使用非线性的时间插值器--AccelerateDecelerateInterpolator ，AccelerateDecelerateInterpolator会加速进入和减少退出动画。可以通过setInterpolator方法替换默认的插值器。

 <!-- more -->
 
## 使用

使用有两种方式：代码和xml资源的形式来完成。

### 代码：

```
   private void beginCodeValueAnimator() {
        if (mCodeValueAnimator == null) {
            mCodeValueAnimator = ValueAnimator.ofFloat(0f, 800f);
            // 使用监听器监听动画属性值变化，更新到目标对象
            mCodeValueAnimator.addUpdateListener(
                    new ValueAnimator.AnimatorUpdateListener() {
                        @Override
                        public void onAnimationUpdate(ValueAnimator animation) {
                            float i = (float) animation.getAnimatedValue();
                            mCodeLine.setmX(i);
                        }
                    });
            mCodeValueAnimator.setDuration(1000);
        } else {
            mCodeValueAnimator.end();
        }
        // 根据CheckBox设置Evaluator
        mCodeValueAnimator.setEvaluator(mCheckBox.isChecked()
                ? new MyEvaluator()
                : new FloatEvaluator());
        mCodeValueAnimator.start();
    }
```
```
    // 自定义Evaluator，将结果扩大1.3倍
    public class MyEvaluator implements TypeEvaluator<Float> {

        @Override
        public Float evaluate(float fraction, Float startValue, Float endValue) {
            return startValue + 1.3f * fraction * (endValue - startValue);
        }

    }
```
代码生成ValueAnimator并且设置监听器，更新属性。也可以通过setEvaluator设置自定义的TypeEvaluator，实现自己计算属性值。


### xml完成：

```
<animator xmlns:android="http://schemas.android.com/apk/res/android"
          android:duration="1000"
          android:valueFrom="1"
          android:valueTo="800"
          android:valueType="floatType"/>
```

```
private void beginXmlValueAnimator() {
        if (mXmlValueAnimator == null) {
            mXmlValueAnimator = (ValueAnimator)  
                AnimatorInflater.loadAnimator(this, R.animator.value_animator);        
                mXmlValueAnimator.addUpdateListener(
                new ValueAnimator.AnimatorUpdateListener() {
                @Override
                public void onAnimationUpdate(ValueAnimator animation) {
                    float i = (float) animation.getAnimatedValue();
                    mXmlLine.setmX(i);
                }
            });
        } else {
            mXmlValueAnimator.end();
        }
        //设置插值器TimeInterpolator
        mXmlValueAnimator.setInterpolator(mInterpolatorCheckBox.isChecked()
                ? new BounceInterpolator()
                : null);
        mXmlValueAnimator.start();
 }
```
在布局文件中设置ValueAnimator，在代码中使用AnimatorInflater.loadAnimator加载。使用setInterpolator设置自己的插值器。

ps:mXmlLine是自定义的View提供一个属性mX,通过方法setmX更新mX。

看个效果图吧：

![ValueAnimator动画实例](/images/animation/simple_value_animator.gif)



## PropertyValuesHolder完成ValueAnimator


先来看看实现方式吧：
```
private void beginPropertyValuesHolderValueAnimator() {
        if (mPropertyValuesHolderValueAnimator == null) {
              PropertyValuesHolder holder0 = 
                    PropertyValuesHolder.ofFloat("A", 0f,800f, 600f);
            PropertyValuesHolder holder1 = 
                   PropertyValuesHolder.ofInt("B", RED, CYAN);
            holder1.setEvaluator(new ArgbEvaluator());
            mPropertyValuesHolderValueAnimator =
                   ValueAnimator.ofPropertyValuesHolder(holder0, holder1);
            mPropertyValuesHolderValueAnimator.setDuration(1000);
            mPropertyValuesHolderValueAnimator.addUpdateListener(
               new ValueAnimator.AnimatorUpdateListener() {
                 @Override
                 public void onAnimationUpdate(ValueAnimator animation) {
                    float x = (float) animation.getAnimatedValue("A");
                    int color = (int) animation.getAnimatedValue("B");
                    mPropertyValuesHolderLine.updateView(x, color);
                 }
            });
        } else {
            mPropertyValuesHolderValueAnimator.end();
        }
        mPropertyValuesHolderValueAnimator.start();

 }
```

使用ValueAnimator.ofPropertyValuesHolder传入PropertyValuesHolder，其实和直接使用ofFloat基本一样，我们可以先看看一眼源码：

![ValueAnimator的ofFloat](/images/animation/vaule_animator_0.png)
![ValueAnimator的ofFloat](/images/animation/vaule_animator_1.png)

可以看出两者实现是基本一样的，只是ofFloat只是使用一个PropertyValuesHolder，但是ValueAnimator.ofPropertyValuesHolder可以使用多个PropertyValuesHolder。如上面代码中实现的。

还要说一下PropertyValuesHolder.ofFloat方法，第一个参数是propertyName。但是如果是我们自己监听动画属性变化的话，propertyName只要传入相同的key就行，如上面代码中我使用“A”,"B"。VauleAnimator使用一个Map存放propertyName-PropertyValuesHolder。

上面说到ValueAnimator.ofPropertyValuesHolder可以使用多个PropertyValuesHolder，代码中使用A变化x值，使用B变化颜色。

看看效果：

![ValueAnimator动画实例2](/images/animation/value_animator1.gif)

在线长度变化的时候，颜色也变化了，这就达到了两个属性同时变化的效果。



## Keyframe完成ValueAnimator


```
private void beginKeyFrameValueAnimator() {

        if (mKeyFrameValueAnimator == null) {

            Keyframe keyframe0 = Keyframe.ofFloat(0f, 0f);
            Keyframe keyframe1 = Keyframe.ofFloat(.5f, 800f);
            Keyframe keyframe2 = Keyframe.ofFloat(1f, 600f);

            PropertyValuesHolder holder0 = PropertyValuesHolder.ofKeyframe("A", 
                keyframe0, keyframe1,keyframe2);
            PropertyValuesHolder holder1 = PropertyValuesHolder.ofInt("B",
                RED, CYAN);
            holder1.setEvaluator(new ArgbEvaluator());
            mKeyFrameValueAnimator = ValueAnimator.ofPropertyValuesHolder(holder0, 
               holder1);
            mKeyFrameValueAnimator.setDuration(1000);
            mKeyFrameValueAnimator.addUpdateListener(
              new ValueAnimator.AnimatorUpdateListener() {
                @Override
                public void onAnimationUpdate(ValueAnimator animation) {
                    float x = (float) animation.getAnimatedValue("A");
                    int color = (int) animation.getAnimatedValue("B");
                    mKeyFrameCodeLine.updateView(x, color);
                }
            });
        } else {
            mKeyFrameValueAnimator.end();
        }
        mKeyFrameValueAnimator.start();
}
```

这个实现等价于PropertyValuesHolder完成ValueAnimator。但是如果我们修改一个

```
Keyframe keyframe0 = Keyframe.ofFloat(0f, 0f);
Keyframe keyframe1 = Keyframe.ofFloat(.8f, 800f);// 这里变化一下 0.5 --> 0.8
Keyframe keyframe2 = Keyframe.ofFloat(1f, 600f);
```

那么x到达800的时间点就发生了变化，速度也发生了变化。

Keyframe.ofFloat(.8f, 800f)指定这个关键帧的时间变化系数和属性值的关系，在系数为0.8的时候，x的值为800。

所以要指定特殊的关键帧还是要使用这种形式进行设置。

先来总结一下ValueAnimator，在来看看不同插值器、不同关键帧的有趣效果。



## ValueAnimator总结

1. 可以使用代码
2. 可以使用XML资源文件
3. 可以使用PropertyValuesHolder指定多个属性变化
4. 可以使用Keyframe指定关键帧

## 来看个有趣效果


![ValueAnimator动画实例2](/images/animation/value_animator2.gif)

从上到下分别是A，B，C，D，左上角是时间，其中:

1. A和B，C和D 插值器不同，关键帧相同
2. A和C，B和D 关键帧不同，插值器不同


## 代码

**代码在[这里](https://github.com/qinglinyi/Animator)**


