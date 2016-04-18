---
title: Java反射--笔记
date: 2016-04-16 15:33:45
categories: Java
tags:
    - Java
    - 反射
    - 笔记
---

本文是《Java 核心技术 卷I》关于反射内容的读书笔记以及理解。

## 1. 是什么

> 反射机制：程序运行时，允许改变程序结构或变量类型的机制。Reflection，用在Java身上指的是我们可以于运行时加载、探知、使用编译期间完全未知的classes。

我的理解就是：反射就是在程序运行时能够动态操作Java代码的程序（或者叫机制）。

> 反射是一种功能强大且复杂的机制。使用它的主要是对象是工具构造者，而不是应用程序员。

## 2. 功能

1. 运行中分析类
2. 运行中构造任意类对象
3. 运行中操作对象的方法

## 3. Class

### 3.1 Class类介绍

书中是这样解释Class的：

> 在程序运行期间，Java运行时系统始终为所有的对象维护一个被称为运行时类型标识(*runtime type identication*)。这个信息保存着每个对象所属的类足迹。虚拟机利用运行时信息选择相应的方法执行。保存这些信息的类被称为Class。

我们再看看这个类的概述：

> Instances of the class Class represent classes and interfaces in a running Java application. An enum is a kind of class and an annotation is a kind of interface. Every array also belongs to a class that is reflected as a Class object that is shared by all arrays with the same element type and number of dimensions. The primitive Java types (boolean, byte, char, short, int, long, float, and double), and the keyword void are also represented as Class objects.
Class has no public constructor. Instead Class objects are constructed automatically by the Java Virtual Machine as classes are loaded and by calls to the defineClass method in the class loader.

意思是说：

Class类的实例代表Java程序运行时的类（classes，枚举也是一种类）和接口（interfaces，注解也是一种接口）。还可以用来代表enum、array、primitive Java types（boolean, byte, char, short, int, long, float, double）以及关键词void。

Class没有public的构造方法，它是在JVM加载类的时候自动实例化的。


### 3.2 获取Class对象

* 方式一：Object类的getClass()方法将会返回一个Class类实例

	```java
	Employee e;
	...
	Class cls = e.getClass();
	```
* 方式二：Class类的静态方法forName方法获取对应的Class类实例

  ```java
  String className = "java.util.Date";
  Class cls = Class.forName(className);
  ```
* 方式三：如果T是Java类，那么T.class代表匹配的类对象

  ```java
  Class cls1 = Date.class;
  Class cls2 = int.class;
  Class cls3 = Double[].class;
  ```
  
**注意：一个Class对象是用来表示一个类型（type），这个类型不一定都是类（class）。例如int不是一个类，但是int.class就是一个Class类型的对象。**


### 3.3 简单使用

虚拟机为每一个类型管理一个唯一Class对象（就是说一个type只对应着一个Class对象）。因此可以使用==运算符实现两个类对象的比较。例如：

```java
if(e.getClass() == Employee.class){...}
```

通过Class的newInstance()方法能快速的创建一个类实例：

```java
try {
    Date mDate = Date.class.newInstance();
    // 或者和forName联合使用
    String s = "java.util.Date";
    Object mDate2 = Class.forName(s).newInstance();
} catch (InstantiationException e) {
    e.printStackTrace();
} catch (IllegalAccessException e) {
    e.printStackTrace();
} catch (ClassNotFoundException e) {
    e.printStackTrace();
}
```
newInstance方法调用的是类的默认构造方法（无参构造方法）初始化创建的类对象，如果该类没有默认构造方法，将会抛出异常。如果需要使用带参的构造方法的话就不能这样调用了，需要使用Constructor类中的newInstance方法。

## 4.使用反射分析类

在java.lang.reflect包中有三个类Field、Method、Constructor分别用于描述普通类的域、方法和构造器。

```java
 Constructor[] constructors = cls.getConstructors();
 Method[] methods = cls.getMethods();
 Field[] fields = cls.getFields();
```

<u>**Class类中的getFields()、getMethods()、getConstructors()方法将返回类提供的public的域、方法和构造器，并且包括超类的共有成员。getDeclaredField()、getDeclaredMethods()、getDeclaredConstructors()能够获取全部的域、方法、构造器，包括私有的和所保护的成员，但是不包括超类的成员。**</u>

```java
 Constructor[] constructors = cls.getDeclaredConstructors();
 Method[] methods = cls.getDeclaredMethods();
 Field[] fields = cls.getDeclaredFields();
```

同时：

1. 可以使用Class类以及Field、Method、Constructor都可以使用getName()获取对应的名称；
2. 使用getModifiers()方法可以获取修饰符；
3. Method和Constructor使用getParameterTypes()可以获取方法和构造器的参数类型； 
4. Field使用getType()方法可以获取域所属类型的Class对象，即能获取域的类型。
5. Method使用getReturnType()获取方法返回值类型。
6. 对于修饰符可以使用Modifier类的静态方法toString方法获取对应的名称。Modifier还提供了例如isPublic、isPrivate或者isFinal方法来判断方法或者构造器是否是public、private或者final。

打印一个类的信息：

```java
private static void printClass(Class cls) {

        String name = cls.getName();
        Class supCls = cls.getSuperclass();
        printModifiers(cls.getModifiers());
        System.out.print("class " + name);
        if (supCls != null && supCls != Object.class) {
            System.out.printf(" extends " + supCls.getName());
        }
        System.out.println(" {");
        System.out.println();
        printConstructors(cls);
        System.out.println();
        printMethods(cls);
        System.out.println();
        printFields(cls);
        System.out.println("}");

    }

    private static void printConstructors(Class cls) {
        // 获取构造方法
        Constructor[] constructors = cls.getDeclaredConstructors();
        for (Constructor constructor : constructors) {
            printModifiers(constructor.getModifiers());
            System.out.print(constructor.getName());
            // 获取方法参数
            printParameters(constructor.getParameterTypes());
        }
    }


    private static void printMethods(Class cls) {
        Method[] methods = cls.getDeclaredMethods();
        for (Method method : methods) {
            printModifiers(method.getModifiers());
            Class returnCls = method.getReturnType();
            System.out.print(returnCls.getName() + " ");
            System.out.print(method.getName());
            printParameters(method.getParameterTypes());
        }
    }

    private static void printModifiers(int mod) {
        String modifier = Modifier.toString(mod);
        if (modifier.length() > 0) {
            System.out.print(modifier + " ");
        }
    }

    private static void printFields(Class cls) {
        Field[] fields = cls.getDeclaredFields();
        for (Field field : fields) {
            printModifiers(field.getModifiers());
            Class type = field.getType();
            System.out.print(type.getName() + " ");
            System.out.println(field.getName());
        }
    }

    private static void printParameters(Class[] parameters) {
        System.out.print("(");
        for (int i = 0; i < parameters.length; i++) {
            if (i > 0) {
                System.out.print(", ");
            }
            System.out.print(parameters[i].getName());
        }
        System.out.println(");");
    }

```java

打印一下Double.class：

```java
    public static void main(String[] args) {
        printClass(Double.class);
    }
```

结果：

```java
public final class java.lang.Double extends java.lang.Number {

public java.lang.Double(double);
public java.lang.Double(java.lang.String);

public boolean equals(java.lang.Object);
public static java.lang.String toString(double);
public java.lang.String toString();
public int hashCode();
public static int hashCode(double);
public static double min(double, double);
public static double max(double, double);
public static native long doubleToRawLongBits(double);
public static long doubleToLongBits(double);
public static native double longBitsToDouble(long);
public volatile int compareTo(java.lang.Object);
public int compareTo(java.lang.Double);
public byte byteValue();
public short shortValue();
public int intValue();
public long longValue();
public float floatValue();
public double doubleValue();
public static java.lang.Double valueOf(java.lang.String);
public static java.lang.Double valueOf(double);
public static java.lang.String toHexString(double);
public static int compare(double, double);
public static boolean isNaN(double);
public boolean isNaN();
public static boolean isInfinite(double);
public boolean isInfinite();
public static boolean isFinite(double);
public static double sum(double, double);
public static double parseDouble(java.lang.String);

public static final double POSITIVE_INFINITY
public static final double NEGATIVE_INFINITY
public static final double NaN
public static final double MAX_VALUE
public static final double MIN_NORMAL
public static final double MIN_VALUE
public static final int MAX_EXPONENT
public static final int MIN_EXPONENT
public static final int SIZE
public static final int BYTES
public static final java.lang.Class TYPE
private final double value
private static final long serialVersionUID
}
```

## 5.使用反射分析对象

前面我们获取了对象（或者说类）的域名称和类型。那么如何获取域的值呢？

使用Field的get(obj)方法获取:

```java
User mUser = new User("Jim", 15);// name和age
Class cls = mUser.getClass();
try {
     Field nameField = cls.getDeclaredField("name");
     Object name = nameField.get(mUser);
} catch (NoSuchFieldException | IllegalAccessException e) {
     e.printStackTrace();
}
```

实际上，如果name是private的话是会抛出java.lang.IllegalAccessException异常的。因为，反射机制的默认行为受限于Java的访问限制。这个时候需要使用setAccessible(true)覆盖访问控制。

```java
User mUser = new User("Jim", 15);
Class cls = mUser.getClass();
try {
  Field nameField = cls.getDeclaredField("name");
  nameField.setAccessible(true);
  Object name = nameField.get(mUser);
  System.out.println(name);
} catch (NoSuchFieldException | IllegalAccessException e) {
  e.printStackTrace();
}
```

当然，如果是获取基本类型的话可以使用Field的对应方法，例如：

```java
Field ageField = cls.getDeclaredField("age");
ageField.setAccessible(true);
int age = ageField.getInt(mUser);
```
setAccessible是AccessibleObject类的方法，AccessibleObject是Field、Method、Constructor       的超类。


## 6.使用反射操作数组

java.lang.reflect包下的Array类使用操作数组的，可以使用Array的静态方法newInstance构建新的数组，使用getLength获取数组长度，使用get、getXXX获取数组数组自定位置的内容，使用set、setXXX设置内容。

```java
int ary[] = (int[]) Array.newInstance(int.class, 5);
for (int i = 0; i < Array.getLength(ary); i++) {
  Array.set(ary, i, i * i);
  // Array.setInt(ary, i, i * i);
}
```


## 7.使用反射操作方法

> Recall that you can inspect a field of an object with the get method of the Field class. Similarly, the Method class has an invoke method that lets you call the method that is wrapped in the current Method object. The signature for the invoke method isObject invoke(Object obj, Object... args)The first parameter is the implicit parameter, and the remaining objects provide the explicit parameters.For a static method, the first parameter is ignored—you can set it to null.

Method类提供了一个invoke方法用来包装当前对象的方法：
Object invoke(Object obj, Object... args)
第一个参数是隐藏参数，表示调用方法的对象；其他参数为显示参数，用以表示方法的参数。如果是静态方法，第一个参数为null。

假如User类有一个方法：

```java
private int calculate(int a, int b) {
   return a + b;
}
```
我们可以通过反射这样调用：

```java
try {
  User demo = new User();
  Method m = demo.getClass().getDeclaredMethod("calculate", int.class, int.class);
  int value = (int) m.invoke(demo, 1, 2);
  System.out.println("反射计算:" + value);
} catch (NoSuchMethodException | InvocationTargetException | IllegalAccessException e) {
  e.printStackTrace();
}
```

对于静态方法，我们假如调用Math的sqrt(double)方法：

```java
try {
  Method sqrtMethod = Math.class.getDeclaredMethod("sqrt", double.class);
  double value = (double) sqrtMethod.invoke(null, 4);
  System.out.printf("%.1f",value);
} catch (NoSuchMethodException | InvocationTargetException | IllegalAccessException e) {
  e.printStackTrace();
}
```

**注意：**

1. 使用invoke方法获取的值是Object的。
2. 使用invoke反射会比直接调用方法明显慢一些。

**所以：仅在必要的时候才使用Method对象。**


          

