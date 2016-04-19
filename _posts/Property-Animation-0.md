---
title:  Android属性动画Property Animation系列(1)_概述
date: 2016-03-16 17:36:30
categories: Android
tags: 
     - Android
     - property animation
     - 属性动画
---


## 是什么

>**属性动画是一套通过改变对象属性来进行动画的框架。**

我们先来了解属性动画的下列特征（以及对应的调用方法）：

1. 动画持续时间（Duration）：动画的持续时间，默认为300ms。
  	`setDuration(long duration)`
  	
2. 时间插值(Time interpolation)：时间插值会将时间点作为参数计算对象的属性值。就是说它提供一个方法将时间的流逝转化为属性的变化（动画时间与属性值得函数、映射关系）。
  `setInterpolator(TimeInterpolator value)`
  
3. 重复次数与重复行为(Repeat count and behavior)：设置动画的重复次数，默认值为0，播放完就结束。如果重复次数大于0，就会在重复播放动画，并且会根据重复行为进行动画重播，可以从头开始播，也可以从尾开始倒着播放。
  `setRepeatCount(int value)和setRepeatMode(int value)`
  
4. 帧刷新延迟(Frame refresh delay)：帧刷新的频率，默认是每10ms刷新一帧。但是这个速度最终取决于系统的响应速度。
  `setFrameDelay(long frameDelay)`
  
 <!-- more -->
   
## 怎么工作

### 举一个属性动画工作的简单例子。

![线性动画示例](/images/animation/animation-linear.png)

假设有一个对象，它有一个属性x，这个对象进行一个持续40ms的线性动画。如图，随着时间t的变化，属性x值的也发生变化,而且这个变化是线性的(对应函数：x=t)。

x  | t (ms)
--------- | -------------
0  | 0
20 | 20
30 | 30
40 | 40

那么，如果这个属性x表示在屏幕中的横向坐标的话，那个这个动画会是这样的：在40ms内对象在屏幕上水平横向匀速移动（如图，对应的时间点和位置）。

### 再来一个稍微复杂一点的例子。

![非线性动画实例](/images/animation/animation-nonlinear.png)

还是一样的假设。但是这里例子中时间和属性x值的关系不是一个线性变化，而是一个先加速到中间然后从中间减速的变化。（AccelerateDecelerateInterpolator，有兴趣可以看这个插值器的实现）

### 如何实现上面例子的计算

![动画的计算方式](/images/animation/valueanimator.png)

上图是主要类（ValueAnimator）和其他类是如何一起工作的。

ValueAnimator这个类包含：

* duration、startPropertyValue、endPropertyValue： 动画持续时间、对象的属性的初始值、结束值。就是说在duration时间内动画的属性是从startPropertyValue变成endPropertyValue。
* TimeInterpolator ： 动画的时间差值器，提供一个函数变化时间。将时间的变化转变为属性变化。不过这个类不是直接进行时间到属性的变化，而是操作时间系数的（这个时间系数=当前时间/总时长，就是时间的百分比），开始的时候为0，结束的时候为1。时间差值器变化的值就变成属性值得系数，通过TypeEvaluator将系数变成属性的变化。
* TypeEvaluator ： 将属性系数转变为属性变化，计算出对应时间的属性值。

>**ValueAnimator将（由内部AnimationHandler进行的）时间变化通过TimeInterpolator、TypeEvaluator转变为属性变化，通过AnimatorUpdateListener通知给对象，对象再调用getAnimatedValue获取当前变化后属性。**

## API概要

* 属性动画系统的API在android.animation包中。
* 而插值器（interpolator）可以使用android.view.animation包下的插值器（因为视图动画系统定义了许多插值器，在属性动画系统中也是可以使用的）。

下列表格中介绍了属性动画系统的主要组件。

### 1）Animator类及其子类

**Animator类**是属性动画的超类，提供了创建动画的基本架构。通常我们不直接使用这个类，因为它只提供了基本功能（start，end、addListener等），因此要完全的支持属性动画就必须扩展这个类，下表列出了**Animator的子类**。

*表1.Animators*

类/接口  | 描述
--------- | -------------
	ValueAnimator  |  用于计算处理动画属性值的主要类。通过这个类可以设置动画持续时间、对象的属性的初始值、结束值，动画是否重复播放和重播模式，时间插值器等。这个类只能处理属性值的变化，不能将这个属性变化绑定到目标对象。但是它提供监听器来监听这种变化。
ObjectAnimator | ValueAnimator类的一个子类，能够将属性的变化绑定到对象，就是能够将属性值的变化及时设置到对象。我们大部分时候直接使用这个类来完成属性动画，因为这个类使目标对象的属性动画变的更加简单。但是，我们有时候也会直接使用ValueAnimator，因为ObjectAnimator的使用会有一些限制，例如，需要为目标对象设置特定的acessor方法。
AnimatorSet | 将多个动画关联运行的类，可以设置各个动画播放的顺序、播放时间点等。

### 2）Evaluators

Evaluator（计算器？）是为属性动画系统计算对象的属性值的。它将Animator类提供的时间系数、动画初始值、结束值计算出属性的当前值（当然其中还经过了插值器的转换）。属性动画系统提供了以下Evaluator：


*表2.Evaluators*

类/接口  | 描述
---------      | -------------
IntEvaluator   | 用以计算int类型属性值的默认的Evaluator。
FloatEvaluator | 用以计算float类型属性值的默认的Evaluator。
ArgbEvaluator  | 用以计算color类型属性值（十六进制表示）的默认的Evaluator。
TypeEvaluator  | 这是个接口，前面这些Evaluator都实现了这个接口，所以可以通过实现这个接口来自定义自己的Evaluator。

### 3）时间插值器

时间插值器定义了动画时间与属性值之间的映射关系，就是从时间到属性值的函数，通过它能将时间变化转变成属性变化。表3是android.view.animation包下的插值器的介绍，你也可以通过实现接口TimeInterpolator来自定义自己的插值器。

*表3.Interpolators*

类/接口  | 描述
---------                        | -------------
AccelerateDecelerateInterpolator | 先加速后减速的插值器
AccelerateInterpolator           | 加速插值器
AnticipateInterpolator           | 先向后，然后向前抛出（抛物运动）
AnticipateOvershootInterpolator  | 先向后，向前抛出并超过目标值，然后最终返回到目标值
BounceInterpolator               | 在结束时反弹
CycleInterpolator                | 用指定的循环数，重复播放动画
DecelerateInterpolator           | 减速插值器
LinearInterpolator               | 线性插值器
OvershootInterpolator            | 向前抛出，并超过目标值，然后再返回
TimeInterpolator                 | 插值器接口，通过实现它可以自定义自己的插值器

直观的介绍可以在这里进行[传送](http://www.cnblogs.com/mengdd/p/3346003.html)

## 补充   与View Animation系统的区别

View Animation系统只提供了View对象的动画能力，对于非View对象如果想要实现动画的话需要自己实现。并且View Animation只提供了View对象的一些动画，例如缩放、位移，但是并没有提供View关于background color的动画。就是说View Animation的使用范围比较窄，使用它能完成的工作比较少，只能做一些关于View对象的缩放、位移等。

另外View Animation还有一个局限就是它只是修改View的绘制的位置，并没有修改View本身。例如，将一个button移动位置，button的显示位置会马上变化，但是它的点击接受的位置还是原来的位置，没有变化，需要我们自己实现逻辑进行处理。

使用属性动画（property animation）系统就不会有这些限制了，你可以通过改变任何对象（View和非View）的属性来进行动画，并且这个对象（的属性）确实被修改了。并且在移除动画的时候属性动画系统更加健壮。你可以进行属性的动画例如颜色、位置、尺寸，也可以对部分动画设置自己的插值器，还可以同时进行多个动画。

然而，View Animation系统具有启动的时间少和代码量少的优点。如果View Animation能够满足你的需求后者已经使用它完成工作了，那就没有必要使用属性动画了。同时在一些情况下，有需求的话也是可以使用两者同时完成动画的。


## 参考

1. <http://developer.android.com/intl/zh-cn/guide/topics/graphics/prop-animation.html#object-animator>
2. <http://www.jcodecraeer.com/a/anzhuokaifa/developer/2013/0312/1006.html>




