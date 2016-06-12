---
title: Logger源码解析
date: 2016-06-07 11:03:59
categories: Android
tags: 
		- Android
		- Log
		- 源码解析
---

## 概述

介绍一个Logger的源码（版本：1.12）。

## 简介

Logger是android.util.Log日志类的封装，但是*更美观、更简单、更强大*。
使用介绍、详细内容移步[GitHub](https://github.com/orhanobut/logger)。

Logger能干什么：

* 线程信息
* 类信息
* 方法信息
* 美观的Json内容
* 美观的换行"\n"
* 转跳到源码

简单使用

```
Logger
  .init(YOUR_TAG)                 // default PRETTYLOGGER or use just init()
  .methodCount(3)                 // default 2
  .hideThreadInfo()               // default shown
  .logLevel(LogLevel.NONE)        // default LogLevel.FULL
  .methodOffset(2)                // default 0
  .logTool(new AndroidLogTool()); // custom log tool, optional
}
```


## 类与接口

各类和接口的关系如下图：

![类关系图](/images/logger/Logger_class.png)

### 1.Logger类

提供一个静态Printer,处理打印。相当于对printer类的包装。

![Logger结构](/images/logger/Logger_methods.png)

1）init初始化
2）clear清除
3）t 设置打印tag
4）执行打印的i,w,e等

这些方法都是调用成员变量printer来执行，所以说Logger其实就是Printer的包装。

### 2.Printer接口

真正执行处理打印的接口，LoggerPrinter是其子类。

### 3.LoggerPrinter类

Printer的子类，Logger使用它进行处理、美化、包装打印信息调用它的各种方法例如init、clear、t、i、w等。

### 4.Settings类

主要用来设置打印的属性：

```
  private int methodCount = 2;//打印方法的方法数
  private boolean showThreadInfo = true;//是否显示线程信息
  private int methodOffset = 0;// 打印方法Offset，跳过几个方法开始打印方法
  private LogTool logTool;// 真正执行打印的类
  private LogLevel logLevel = LogLevel.FULL;// 打印的级别

```

### 5.LogLevel枚举

打印的级别的枚举

```
public enum LogLevel {
  // 打印所以信息
  FULL,
  // 不打印信息
  NONE 
}
```

### 6.LogTool接口

LogTool是真正用来打印的，在Printer中调用LogTool执行打印。可以通过Settings类设置自定义的LogTool。默认使用AndroidLogTool。

### 7.AndroidLogTool类

实现LogTool接口，使用系统的Log执行打印。


## 源码解析

Logger的重点是Printer，所以我们来解析一下LoggerPrinter类。

### 1）init

设置默认的Tag和设置Settings，该方法返回一个Settings实例。再看一下使用：

```
Logger
  .init(YOUR_TAG)                 // default PRETTYLOGGER or use just init()
  .methodCount(3)                 // default 2
  .hideThreadInfo()               // default shown
  .logLevel(LogLevel.NONE)        // default LogLevel.FULL
  .methodOffset(2)                // default 0
  .logTool(new AndroidLogTool()); // custom log tool, optional
}
```

### 2）t

设置tag和打印的方法数，不过这个设置只有一次使用有效。并且使用ThreadLocal来保证线程安全。
使用：

```
Logger.t("mytag").d("hello");
```

### 3）打印

我们看一下任意一个打印方法：

```
  @Override public void d(String message, Object... args) {
    log(DEBUG, message, args);
  }
```
所以的打印方法都是使用log方法，只是打印的logType不同。

我们来看看log方法：

![log方法](/images/logger/Logger_log.png)

然后我们将他简化成一个示意图：

![log方法示意图](/images/logger/Logger_log_info.png)

除去打印边框和分割线，主要就是***logHeaderContent***、***logContent***

```
 private void logHeaderContent(int logType, String tag, int methodCount) {
 	 // 获取当前堆栈跟踪，能够获取调用的打印的类和方法路劲
    StackTraceElement[] trace = Thread.currentThread().getStackTrace();
    // 打印线程信息
    if (settings.isShowThreadInfo()) {
      logChunk(logType, tag, HORIZONTAL_DOUBLE_LINE + " Thread: " + Thread.currentThread().getName());
      logDivider(logType, tag);
    }
    String level = "";
    
	  // 获取需要打印的方法在数组中的位置
    int stackOffset = getStackOffset(trace) + settings.getMethodOffset();

    //corresponding method count with the current stack may exceeds the stack trace. Trims the count
    if (methodCount + stackOffset > trace.length) {
      methodCount = trace.length - stackOffset - 1;
    }

    for (int i = methodCount; i > 0; i--) {
      int stackIndex = i + stackOffset;
      if (stackIndex >= trace.length) {
        continue;
      }
      StringBuilder builder = new StringBuilder();
      builder.append("║ ")
          .append(level)
          .append(getSimpleClassName(trace[stackIndex].getClassName()))
          .append(".")
          .append(trace[stackIndex].getMethodName())
          .append(" ")
          .append(" (")
          .append(trace[stackIndex].getFileName())
          .append(":")
          .append(trace[stackIndex].getLineNumber())
          .append(")");
      level += "   ";
      logChunk(logType, tag, builder.toString());
    }
  }
```

```
  private void logContent(int logType, String tag, String chunk) {
    String[] lines = chunk.split(System.getProperty("line.separator"));
    for (String line : lines) {
      logChunk(logType, tag, HORIZONTAL_DOUBLE_LINE + " " + line);
    }
  }
```


不管是哪个打印方法，都是调用***logChunk***方法

```
private void logChunk(int logType, String tag, String chunk) {
      ...
        settings.getLogTool().e(finalTag, chunk);
      ...
    }
  }
```

然后就调用LogTool的打印方法。

看看LogTool：

![log方法示意图](/images/logger/Logger_tool.png)

调用android.util.Log的方法。

