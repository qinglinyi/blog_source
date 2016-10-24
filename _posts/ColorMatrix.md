---
title: ColorMatrix
date: 2016-09-28 09:56:11
tags:
		- Android
---

## 一、简介

ColorMatrix是一个4x5的矩阵，是用于转换Bitmap的颜色和透明度的组件。这个矩阵可以通过一个简单的数组设置，如下：

```
[ a, b, c, d, e,
  f, g, h, i, j,
  k, l, m, n, o,
  p, q, r, s, t ]
```

当用于color [R, G, B, A]的时候，结果color将变成：

```
   R’ = a*R + b*G + c*B + d*A + e;
   G’ = f*R + g*G + h*B + i*A + j;
   B’ = k*R + l*G + m*B + n*A + o;
   A’ = p*R + q*G + r*B + s*A + t;
```

并且得到的结果 color [R’, G’, B’, A’]的每一项的取值范围都是0~255。

如下一个例子ColorMatrix是倒置每一项颜色：

```
   [ -1, 0, 0, 0, 255,
     0, -1, 0, 0, 255,
     0, 0, -1, 0, 255,
     0, 0, 0, 1, 0 ]
```


## 二、主要方法

1. 用于YUV和RGB转换的方法：setRGB2YUV()和setYUV2RGB()
2. 用于设置饱和度的方法：setSaturation(float sat)，可以设置图片为黑白
3. 用于调整颜色比例的方法：setScale(float rScale, float gScale, float bScale, float aScale)，可以调整R、G、B、A各项的比例
4. 用于设置围绕一个颜色进行旋转（Set the rotation on a color axis by the specified values）：setRotate(int axis, float degrees)

   1）*axis=0 Red*、*axis=1 Green*、*axis=2 Blue* 
   2）*degrees* 角度 0-360

## 三、栗子

<https://github.com/qinglinyi/ColorMatrixDemo>

