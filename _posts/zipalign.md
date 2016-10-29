---
title: zipalign
date: 2016-10-24 14:26:07
tags:
	  - Android
---

## 前言

zipalign是Android构建过程的一个环节，本文主要是翻译官网关于zipalign的介绍，并且配图。

## zipalign

<https://developer.android.com/studio/command-line/zipalign.html>

zipalign是一个归档对齐工具，它为Android应用程序（APK）文件提供重要的优化。 目的是确保所有未压缩数据以相对于文件开头的特定对齐开始。 具体来说，它会使APK中的所有未压缩数据（例如图像或raw）在4字节边界上对齐。 这允许使用mmap（）直接访问所有部分，即使它们包含具有对齐限制的二进制数据。 其优点是减少运行应用程序时消耗的RAM量。


此工具应始终用于在将APK文件分发给最终用户之前对齐。 Android构建工具可以为您处理此问题。 Android Studio会自动对齐您的APK。

> 注意：您必须在应用程序构建过程中的两个特定点之一使用zipalign，具体取决于您使用的应用程序签名工具：
> 
 1. 如果您使用apksigner，zipalign只能在签署APK文件之前执行。 如果您使用apksigner签署APK，并对APK进行进一步更改，则其签名将失效。
 2. 如果你使用jarsigner，zipalign只能在APK文件签名后才能执行。

<img src="/images/zipalign/zipalign.png" width = "661" height = "527" alt="apksigner" />


通过更改zip本地文件头部分中“额外”字段的大小进行调整。 “额外”字段中的现有数据可能会被此过程更改。
有关如何在构建应用程序时使用zipalign的更多信息，请阅读签名您的应用程序。

## 使用

对齐 infile.apk 并保存为 outfile.apk:

```
zipalign [-f] [-v] <alignment> infile.apk outfile.apk
```

确认existing.apk的对齐方式：

```
zipalign -c -v <alignment> existing.apk
```
alignment是一个定义字节对齐边界的整数。 这必须总是4（提供32位对齐），否则它什么也不做。


标示:

 *  -f : 覆盖输出的 outfile.zip
 *  -v : 详细输出
 *  -p : outfile.zip应该对infile.zip中的所有共享对象文件使用相同的页面对齐
 *  -c : 检查给定的文件是否对齐


使用

```
zipalign -c -v 4 destination.apk 
```

检查结果:

 ```
       50 META-INF/MANIFEST.MF (OK - compressed)
     467 META-INF/APP.SF (OK - compressed)
     985 META-INF/APP.RSA (OK - compressed)
    2142 res/drawable/ic_launcher.png (BAD - 2)
    4391 res/layout/main.xml (OK - compressed)
    4729 AndroidManifest.xml (OK - compressed)
    5348 resources.arsc (OK)
    6477 classes.dex (OK - compressed)
Verification FAILED
 ```
```
    49 AndroidManifest.xml (OK - compressed)
     876 META-INF/CERT.RSA (OK - compressed)
    1977 META-INF/CERT.SF (OK - compressed)
   12956 META-INF/MANIFEST.MF (OK - compressed)
   23893 classes.dex (OK - compressed)
 ...
 1035004 res/mipmap-hdpi-v4/ic_launcher.png (OK)
 1038196 res/mipmap-mdpi-v4/ic_launcher.png (OK)
 1040224 res/mipmap-xhdpi-v4/ic_launcher.png (OK)
...
 1201748 resources.arsc (OK)
Verification succesful
```

## 配图

看了[Android构建过程分析](http://mp.weixin.qq.com/s?__biz=MzI1NjEwMTM4OA==&mid=2651232113&idx=1&sn=02f413999ab0865e23d272e69b9e6196&scene=1&srcid=0831gT4p6M0NFG5HTTeRHTUC#wechat_redirect)感觉不错，不过配图不是太好看，自己结合[官网的介绍](https://developer.android.com/studio/build/index.html)配了一个图。但是由于暂时不准备单独写一篇分析的文章，便放到了此处了。

不考虑Dependencies(Library Modules/AAR Libraries/JAR Libraries)，或者说将Dependencies理解成已经和Application Module进行marge了。

<img src="/images/zipalign/Build Process.png" width = "803" height = "1368" alt="Build Process" />

官网的构建配图是这样的：

<img src="/images/zipalign/build-process_2x-2.png" width = "475" height = "534" alt="Build Process" />



