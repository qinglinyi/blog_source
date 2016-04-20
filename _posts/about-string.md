---
title: 关于Java String
date: 2016-04-18 16:03:53
categories: Java
tags: 
		- Java
		- String
---




***个人理解，或有错误，望指正。***

## 1. 概述


> The String class represents character strings. All string literals in Java programs, such as "abc", are implemented as instances of this class.
> 
> Strings are constant; their values cannot be changed after they are created. String buffers support mutable strings. Because String objects are immutable they can be shared.
String 类代表字符串。Java 程序中的所有字符串字面值（如 "abc" ）都是这个类的对象。
字符串是常量；它们的值在创建之后不能更改。字符串缓冲区支持可变的字符串。因为 String 对象是不可变的，所以可以共享。

**String不是基本数据类型，而是一种特殊的类。String代表的是不可变的字符序列，为不可变对象，一旦被创建，就不能修改它的值，对于已经存在的String对象的修改都是重新创建一个新的对象，然后把新的值保存进去，原来的对象并没有改变。**


## 2. 特征

String类具有这么几个特征：

1. 不是基本数据。
2. 不可变。一旦被创建，就不能修改它的值。
3. 修改方法并不会修改原对象，而是生成一个新的对象。
4. 可以被共享。

### 2.1 如何理解String的不可变

首先String是一个final类，不能被继承。
其次，String是使用一个char数组来存放字符的：

```java
private final char value[];
```
这个value也是final，不可变。这个变量只在String类的构造方法中初始化，之后不能改变（正常使用，非反射情况）。
因此，String一旦被创建就不能修改它的值。

### 2.2 如何看待String提供的修改方法

我们知道String提供一些方法例如：toUpperCase，toUpperCase等用来修改字符串。
其实这些方法并没有修改原来的对象，它们只是重新创建了一个对象。

```java
public String toLowerCase(Locale locale) {
   ...
   return new String(result, 0, len + resultOffset);
}
```

假如我们执行以下代码：

```java
public static void main(String[] args) {
   String a = "Hello World";
   a = a.toUpperCase();
   System.out.println(a);
}
```
执行之后，会生成两个对象"Hello World"和"HELLO WORLD"。而String对象的引用a的值由"Hello World"的地址变成"HELLO WORLD"的地址。而对象"Hello World"并没有改变。
至于这些对象在内存中的存储位置会在下面简单介绍，这里先不展开。


### 2.3 String对象不可变，但是可以通过反射需改值

虽然，String的value属性是一个final的属性，但是value是一个引用，它存储的其实是char数组对象的地址。那么，虽然value的引用不可变，但是这个数组对象可以改变。我们可以通过反射获取这个value指向的对象，然后修改这个对象。

```java
public static void main(String[] args) {
   String a = "Hello World";
   System.out.println(a.hashCode());
   try {
       //获取String类中的value字段
       Field  valueFieldOfString = String.class.getDeclaredField("value");
       //改变value属性的访问权限
       valueFieldOfString.setAccessible(true);
       //获取s对象上的value属性的值
       char[] value = (char[]) valueFieldOfString.get(a);
       //改变value所引用的数组中的第5个字符
       value[5] = '_';
       System.out.println("a = " + a);  //Hello_World
       System.out.println(a.hashCode());
   } catch (NoSuchFieldException | IllegalAccessException e) {
       e.printStackTrace();
   }
}
```
输出：a = Hello_World


### 2.4 如何实现共享

首先在看看class文件的结构：

![class结构](/images/about_string/about_string_class.gif)

> constant_pool 常量池
既然是池子，肯定是拿来放东西的，名字叫常量池，那肯定是放常量的，那哪些常量是可以放的。
1）放什么
常量池中主要存放两类内容：字面常量和符号引用。
字面常量主要包含文本字符串，被声明为final的常量等。
符号引用：因为java在编译的时候没有进行连接这一步，所有的引用都是在加载到虚拟机里动态连接的，这就要求class文件里存放这些信息。主要有以下三类常量：
 a)类和接口的全限定名
 b)字段的名称和描述符
 c)方法的名称和描述符

以上是来自[《最简class文件格式分析(一) class文件结构(1)》](http://blog.chinaunix.net/uid-21718047-id-3177289.html)关于class文件常量池的介绍。

就是说字面常量是存放在class文件的常量池中。

然后看一下JVM的结构（这是一张HotspotJVM的结构图，详细信息可以点击[这里](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html)）

![jvm结构](/images/about_string/about_string_jvm.png)

> 在Java虚拟机中，关于被装载类型的信息存储在一个逻辑上被称为方法区(Method Area)的内存中，当虚拟机装载某个类型时，它使用类装载器定位相应的class文件，然后读入这个class文件——一个线性二进制数据流——然后将它传输到虚拟机中。紧接着虚拟机提取其中的类型信息，并将这些信息存储到方法区(Method Area)。该类型中的类变量(static变量)同样也是存储在方法区(Method Area)中。 
JVM加载class后，会在方法区中为它们开辟了空间-运行常量池，用以存放类，方法，接口等中的常量，也包括字符串常量。
就是说String常量在类加载之后会存储在方法区的**运行常量池**中。
理解一下这个过程：java文件在编译成class文件的时候字面常量（例如String s1 = "abc";中的abc）会存放在class文件的常量池中，JVM在加载这个class的时候将这个这些字面常量加载到方法区的运行常量池中（当然不止字面常量）。
（注：关于JVM结构和class文件的结构大家可以看看参考中的文章，以后再详细介绍😭）

通过例子说明：

```java
public static void main(String[] args) {
	String s1 = "abc";   
	String s2 = "abc";  
}
```
使用这种声明方式的话，JVM加载类的时候会将class文件常量池的常量（abc）加载到运行常量池。当JVM执行String s1 = "abc"的语句的时候，会查找运行常量池，返回这个对象的引用。同样的s2也是返回这个引用。从而导致s1和s2同时指向abc对象的内存地址，实现了字符串共享。
举个例子看看是不是这样子的：

```java
public static void main(String[] args) {
   String a = "Hello World";
   String b = "Hello World";
   System.out.println("原始 a = " + a);
   System.out.println("原始 b = " + b);
   try {
       //获取String类中的value字段
       Field valueFieldOfString = String.class.getDeclaredField("value");
       //改变value属性的访问权限
       valueFieldOfString.setAccessible(true);
       //获取s对象上的value属性的值
       char[] value = (char[]) valueFieldOfString.get(a);
       //改变value所引用的数组中的第5个字符
       value[5] = '_';
       System.out.println("a = " + a);  //Hello_World
       System.out.println("b = " + b);  //Hello_World
   } catch (NoSuchFieldException | IllegalAccessException e) {
       e.printStackTrace();
   }
}
```

运行结果是：

```
原始 a = Hello World
原始 b = Hello World
a = Hello_World
b = Hello_World
```

说明a和b确实指向相同的String对象地址。

**因此注意：如果使用这个方式声明的String变量，那么这个String对象会存在内存中的常量池中。并且不一定会创建String对象，有可能是共享的常量池中已经存在的对象。**


## 3. 字符串拼接

String允许是用+号拼接两个字符串，例如：

```java
public static void main(String[] args) {
        String s1 = "Hello";
        String s2 = s1 + " World";
        String s3 = "Hello" + " World";

        System.out.println(s1 + s2 + s3);
}
```
这个要注意，（使用jdk1.8）编译的时候变量s2会使用StringBuilder拼接，而变量s3会自动拼接成 "Hello World"。

我们可以通过javap -c 命令查看字节码实现。（可以参考[这里](http://www.jb51.net/article/48380.htm)）

![字节码图](/images/about_string/about_string_class_code.png)


## 4. 方法

### 4.1 子串 substring

String类的substring方法可以从一个较大的字符串提取出一个子串：

```java
String greeting = "Hello";
String s = greeting.substring(0,3);
```
创建了一个有字符“Hel”组成的字符串。
substring方法第一个参数表示开始的位置（包括），第二个参数表示结束位置（不包括）。这样容易计算长度3-0=3。

然后我们看看这个方法的源码：

```java
public String substring(int beginIndex, int endIndex) {
   if (beginIndex < 0) {
       throw new StringIndexOutOfBoundsException(beginIndex);
   }
   if (endIndex > value.length) {
       throw new StringIndexOutOfBoundsException(endIndex);
   }
   int subLen = endIndex - beginIndex;
   if (subLen < 0) {
       throw new StringIndexOutOfBoundsException(subLen);
   }
   return ((beginIndex == 0) && (endIndex == value.length)) ? this
           : new String(value, beginIndex, subLen);
}
```
先判断边界是否正确，然后通过构造器String(char value[], int offset, int count)生成String对象。
构造器是这样的：

```java
public String(char value[], int offset, int count) {
        if (offset < 0) {
            throw new StringIndexOutOfBoundsException(offset);
        }
        if (count < 0) {
            throw new StringIndexOutOfBoundsException(count);
        }
        // Note: offset or count might be near -1>>>1.
        if (offset > value.length - count) {
            throw new StringIndexOutOfBoundsException(offset + count);
        }
        this.value = Arrays.copyOfRange(value, offset, offset+count);
}
```

其实就是复制了一份char数组。
因此，这个方法也是新建了一个String对象。**String方法并不会修改原对象，而是生成一个新的对象**

### 4.2 String.intern()

String对象调用intern方法后，可以让JVM检查运行常量池（字符串池），如果常量池中存在与这个相等的对象就返回常量池中的对象的引用（不是该对象）；如果没有这个对象，就将本对象放入常量池并返回常量池中这个对象的引用。我的理解是：这个是时候可能有两个对象，但是最后返回的是常量池中对象的引用。为什么说是可能两个呢，举例说一下：

```java
public static void main(String[] args) {
   String s1 = new String("Hello！！");
   String s2 = s1.intern();
   System.out.println(s1==s2);

   String s3 = "World！！";
   String s4 = s3.intern();
   System.out.println(s3==s4);
}
```

s1是堆中“Hello！！”对象的引用，s2是s1调用intern方法之后在常量池检查之后返回的引用（来自常量池）。所以这个时候是两个对象。
s3是常量池的中"World！！"对象引用，s4是s3调用intern方法之后在常量池检查之后返回的引用，其实是同一个对象，这个时候就一个对象。


## 5. 与StringBuffer StringBuilder

三者的区别是:

1. String 不可变字符串。
2. StringBuffer StringBuilder 可变字符串。
3. StringBuffer 线程安全的可变字符串，它的append方法才有synchronized关键字修饰，因此是线程安全的。
4. StringBuffer StringBuilder 都继承自AbstractStringBuilder。
5. String StringBuilder StringBuffer都实现了CharSequence接口。

最后，扩展一下AbstractStringBuilder这个类。AbstractStringBuilder实现Appendable, CharSequence，使用char数组保存字符串，提供很多append用来连接字符串，在append的时候都会先判断一下char数字的容量是否够，如果不够就进行扩容，然后将新字符串放入数组中。
我们可以看一下其中的一个方法：

```java
public AbstractStringBuilder append(String str) {
   if (str == null)
       return appendNull();
   int len = str.length();
   ensureCapacityInternal(count + len);
   str.getChars(0, len, value, count);
   count += len;
   return this;
}
```
```java
/**
* This method has the same contract as ensureCapacity, but is
* never synchronized.
*/
private void ensureCapacityInternal(int minimumCapacity) {
   // overflow-conscious code
   if (minimumCapacity - value.length > 0)
       expandCapacity(minimumCapacity);
}

/**
* This implements the expansion semantics of ensureCapacity with no
* size check or synchronization.
*/
void expandCapacity(int minimumCapacity) {
   int newCapacity = value.length * 2 + 2;
   if (newCapacity - minimumCapacity < 0)
       newCapacity = minimumCapacity;
   if (newCapacity < 0) {
       if (minimumCapacity < 0) // overflow
           throw new OutOfMemoryError();
       newCapacity = Integer.MAX_VALUE;
   }
   value = Arrays.copyOf(value, newCapacity);
}
```

每次扩容的时候都是扩大2倍+2，如果比目标长度还小的话就使用目标长度。
这就是为什么在使用+拼接字符串的时候，编译器会将+的字符串转化为StringBuilder。因为这样子会减少创建String对象，减少开销。



## 6. 参考

[1.JDK源码分析之String篇](http://www.cnblogs.com/huntfor/p/3909059.html?utm_source=tuicool&utm_medium=referral)
[2.深入理解Java：String](http://www.cnblogs.com/ITtangtang/p/3976820.html)
[3.JVM学习笔记-方法区（Method Area](http://denverj.iteye.com/blog/1209506)
[4.触摸java常量池](http://www.cnblogs.com/iyangyuan/p/4631696.html)
[5. 最简class文件格式分析(一) class文件结构(1)](http://blog.chinaunix.net/uid-21718047-id-3177289.html)
<https://commons.apache.org/proper/commons-bcel/manual.html>





