---
title: 初识Java字节码
date: 2016-12-14 10:12:55
categories: Java
tags:
		- Java
---


## 1. Class 文件结构


Class文件是一组以8字节为基础单位的二进制流，各个数据项目严格按照顺序紧凑排列在Class文件中，中间没有分隔符。如果遇到需要占用超过8个字节的数据项目时，会按照高位在前（big-endian order，大端序）的方式分割成若干个8位字节进行存储。

Class文件格式采用一种类似于C语言结构体的伪结构来存储，这种伪结构中只有两种数据类型：无符号数和表。

无符号数就是u1、u2、u4、u8来分别代表1个、2个、4个、8个字节。无符号数可以用来描述数字、索引引用、数量值或者按照UTF-8编码构成字符串值。

表是由多个无符号数或其他表作为数据项构成的复合数据类型，以“_info”结尾。在表开始位置，通常会使用一个前置的容量计数器，因为表通常要描述数量不定的多个数据。

下图（表）就是我们来看看Class文件的结构：

<img src="/images/bytecode/Class_file.png" width = "809" height = "867" alt="Class文件结构" />

然后，我们通过一个简单例子来认识一下Class文件结构。


## 2. 简单例子


java代码如下：

```java
package com.qinglinyi.demo;


public class ByteCodeSample {

    private String msg = "hello world";

    public void say() {
        System.out.println(msg);
    }
}
```

文件位置：

<img src="/images/bytecode/java_package.png" width = "600" alt="java文件位置" />

编译之后生成ByteCodeSample.class，使用sublime打开可以看到这些二进制流：

<img src="/images/bytecode/class_byte_code.png" width="300" height = "300" alt="class文件二进制流" />

使用javap之后：

```java

  Last modified Nov 16, 2016; size 585 bytes
  MD5 checksum 05f97704a322e75b8e3635f42ddf83c8
  Compiled from "ByteCodeSample.java"
public class com.qinglinyi.demo.ByteCodeSample
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #7.#20         // java/lang/Object."<init>":()V
   #2 = String             #21            // hello world
   #3 = Fieldref           #6.#22         // com/qinglinyi/demo/ByteCodeSample.msg:Ljava/lang/String;
   #4 = Fieldref           #23.#24        // java/lang/System.out:Ljava/io/PrintStream;
   #5 = Methodref          #25.#26        // java/io/PrintStream.println:(Ljava/lang/String;)V
   #6 = Class              #27            // com/qinglinyi/demo/ByteCodeSample
   #7 = Class              #28            // java/lang/Object
   #8 = Utf8               msg
   #9 = Utf8               Ljava/lang/String;
  #10 = Utf8               <init>
  #11 = Utf8               ()V
  #12 = Utf8               Code
  #13 = Utf8               LineNumberTable
  #14 = Utf8               LocalVariableTable
  #15 = Utf8               this
  #16 = Utf8               Lcom/qinglinyi/demo/ByteCodeSample;
  #17 = Utf8               say
  #18 = Utf8               SourceFile
  #19 = Utf8               ByteCodeSample.java
  #20 = NameAndType        #10:#11        // "<init>":()V
  #21 = Utf8               hello world
  #22 = NameAndType        #8:#9          // msg:Ljava/lang/String;
  #23 = Class              #29            // java/lang/System
  #24 = NameAndType        #30:#31        // out:Ljava/io/PrintStream;
  #25 = Class              #32            // java/io/PrintStream
  #26 = NameAndType        #33:#34        // println:(Ljava/lang/String;)V
  #27 = Utf8               com/qinglinyi/demo/ByteCodeSample
  #28 = Utf8               java/lang/Object
  #29 = Utf8               java/lang/System
  #30 = Utf8               out
  #31 = Utf8               Ljava/io/PrintStream;
  #32 = Utf8               java/io/PrintStream
  #33 = Utf8               println
  #34 = Utf8               (Ljava/lang/String;)V
{
  public com.qinglinyi.demo.ByteCodeSample();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: ldc           #2                  // String hello world
         7: putfield      #3                  // Field msg:Ljava/lang/String;
        10: return
      LineNumberTable:
        line 4: 0
        line 6: 4
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      11     0  this   Lcom/qinglinyi/demo/ByteCodeSample;

  public void say();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #4                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: aload_0
         4: getfield      #3                  // Field msg:Ljava/lang/String;
         7: invokevirtual #5                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        10: return
      LineNumberTable:
        line 9: 0
        line 10: 10
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      11     0  this   Lcom/qinglinyi/demo/ByteCodeSample;
}
SourceFile: "ByteCodeSample.java"
```

对这个二进制进行分析，找出对应的数据项：

![class文件二进制流介绍](/images/bytecode/class_byte_code_info_1.png)

![class文件二进制流介绍](/images/bytecode/class_byte_code_info_2.png)

接着，我们逐项分析


## 3. 魔数（u4: magic）

```java
ca fe ba be
```

用来标记这个文件是否是一个Class文件。


## 4. 版本（u2: minor_version，u2: major_version）

```java
00 00 00 34
```

次版本号（minor_version）= 0（十进制）
主版本号（major_version）= 52（十进制）

所以这个Class文件的格式版本号为52.0


## 5. 常量池计数器 （u2: constant_pool_count）

```java
00 23
```

标记常量池表的数量，值等于constant_pool表中的成员数加1。本例中值为23（对应的十进制为35）

## 6. 常量池（cp_info: constant_pool）

所有的常量池项都具有如下通用格式: 

```json

cp_info {
	u1 tag;
	u1 info[]; 
} 

```

常量池中，每个cp_info项的格式必须相同，它们都以一个表示cp_info类型的单字节 “tag”项开头。后面 info[]项的内容tag由的类型所决定。每个tag项必须跟随2个或更多的字节，这些字节用于给定这个常量的信息，附加字节的信息格式由tag的值决定。 

|类型|值| 
|:-----------:|:-------------:|
| CONSTANT_Class | 7  |
| CONSTANT_Fieldref | 9 |
| CONSTANT_Methodref | 10 |
| CONSTANT_InterfaceMethodref | 11 |
| CONSTANT_String | 8 |
| CONSTANT_Integer | 3 |
| CONSTANT_Float | 4 |
| CONSTANT_Long | 5 |
| CONSTANT_Double | 6 |
| CONSTANT_NameAndType | 12 |
| CONSTANT_Utf8 | 1 |
| CONSTANT_MethodHandle | 15 |
| CONSTANT_MethodType | 16 |
| CONSTANT_InvokeDynamic | 18 |


![class常量池](/images/bytecode/constant_pool.png)


我们来看两个：

### 6.1 CONSTANT_Methodref例子

```java
0a 00 07 00 14
```

u1:tag => 0a => CONSTANT_Methodref(10)

CONSTANT_Methodref结构为：

```
CONSTANT_Methodref_info {
  u1 tag;
  u2 class_index;
  u2 name_and_type_index;
}
```

那么对应的：

u2:class_index => 00 07 => 指向常量表弟7项类型为CONSTANT_Class的常量 => java/lang/Object

u2:name_and_type_index => 00 14 => 指向常量表弟20项类型为CONSTANT_NameAndType的常量 => "<init>":()V


### 6.2 CONSTANT_Utf8例子

```java
01 00 12 4c 6a 61 76 61 2f 6c 61 6e 67 2f 53 74 72 69 6e 67 3b
```

u1:tag => 01 => CONSTANT_Utf8(1)

CONSTANT_Utf8的结构为

```
CONSTANT_Utf8_info {
    u1 tag;
    u2 length;
    u1 bytes[length];
}
```


u2:length => 00 12 => 长度为18 

u2:bytes[] => 4c 6a 61 76 61 2f 6c 61 6e 67 2f 53 74 72 69 6e 67 3b => UTF-8缩写编码表示的字符串，128个US-ASCII字符只需一个字节编码（Unicode范围由U+0000至U+007F）=> Ljava/lang/String;

其他更多类型查看[官网](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html)。


## 7. 访问标志（u2:access_flags）

```java
00 21
```

|标记名|值| 描述 |
|:-----------:|:-------------:|:-------------:|
| ACC_PUBLIC     | 0x0001    | 是否为public类型 |
| ACC_FINAL      | 0x0010    | 是否被申明为final，只有类可设置 |
| ACC_SUPER      | 0x0020    | 是否允许使用invokespecial字节码指令的新语义，JDK 1.0.2之后编译出来的类这个标志都必须为真 |
| ACC_INTERFACE  | 0x0200    | 标识接口 |
| ACC_ABSTRACT   | 0x0400    | 是否为abstract，对于接口和抽象类来说，此标志为真，其他类值为假的 |
| ACC_SYNTHETIC  | 0x1000    | 标识这个类并非由用户代码产生的 |
| ACC_ANNOTATION | 0x2000    | 标识这是一个注解 |
| ACC_ENUM       | 0x4000    | 标识这是一个枚举 |

我们的例子是一个最简单的类，并且编译器在1.0.2之后所以 ACC_PUBLIC ACC_SUPER 为真 即
0x0001 | 0x0020 所以我们的结果是0021


## 8. 类索引（u2: this_class）

用来确定这个类的全限定名。

本例中：

```java
00 06
```

这个索引指向常量池index为6类型为Class的常量，即：

```java
#6 = Class              #27            // com/qinglinyi/demo/ByteCodeSample
```

## 9. 父类索引（u2: super_class）

用来确定这个类父类的全限定名。

本例中：
```java
00 07
```

同类索引指向：

```java
#7 = Class              #28            // java/lang/Object
```

## 10. 接口计数器（u2: interfaces_count）

用来标记接口（索引）的数量

本例中：

```java
00 00
```

我们的例子没有接口，所以数量为0

## 11. 接口索引集合（u2: interfaces）

用来描述该类实现哪些接口

interfaces_count个u2，每个u2指向常量池，标记一个接口。

本例中没有接口。


## 12. 字段计数器（u2: fields_count）

用来标记字段数量

本例中：

```java
00 01
```

我们只有一个字段

## 13. 字段表集合（field_info: fields[fields_count]）

有fields_count个字段表field_info，用来表示字段。

本例中：

<img src="/images/bytecode/field_info.png" width = "800" alt="字段表集合" />

```java
00 02 00 08 00 09 00 00
```

因为只有一个字段，所以就一个field_info

字段表的结构是这样的：

```java
field_info {
    u2             access_flags;
    u2             name_index;
    u2             descriptor_index;
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```

也就是这样：

<img src="/images/bytecode/field_info2.png" width = "800" alt="字段表集合" />


### 13.1 u2: access_flag

字段的访问权限类型有这些：

|类型|值|
|-----------|:-------------:|
| ACC_PUBLIC	      | 0x0001  
| ACC_PRIVATE	   | 0x0002
| ACC_PROTECTED	   | 0x0004
| ACC_STATIC	    | 0x0008
| ACC_FINAL	      | 0x0010
| ACC_VOLATILE	   | 0x0040
| ACC_TRANSIENT	   | 0x0080
| ACC_SYNTHETIC	   | 0x1000
| ACC_ENUM	       | 0x4000

本例中：

```java
00 02
```	
   
所以我们只有一个ACC_PUBLIC


### 13.2 u2: name_index

字段名索引，可以获取字段名

本例中：

```java
00 08
```

对应的常量池：

```java
#8 = Utf8               msg
```
说明我们的字段名字是msg

### 13.3 u2: descriptor_index

描述符索引

本例中：

```java
00 09
```

对应的常量池：

```java
#9 = Utf8               Ljava/lang/String;
```

说明我们这个字段是String类型

### 13.4 u2: attributes_count

属性数量

本例中：

```java
00 00
```
数量为0，无属性。

### 13.5 attribute_info attributes[attributes_count]

attributes_count个attribute_info，对应着attributes_count个属性，本例中数量为0。


## 14. 方法计数器（u2: methods_count）

用来标记方法数量

本例中：

```java
00 02
```

我们只有2个方法


## 15. 方法表集合（method_info: methods[methods_count]）

有methods_count个字段表method_info，用来表示方法。

结构如下：

```java
method_info {
    u2             access_flags;
    u2             name_index;
    u2             descriptor_index;
    u2             attributes_count;
    attribute_info attributes[attributes_count];
}
```

本例中：

第一个方法：

```java
  00 0100 0a00 0b00 0100 0c00 0000 3900
0200 0100 0000 0b2a b700 012a 1202 b500
03b1 0000 0002 000d 0000 000a 0002 0000
0004 0004 0006 000e 0000 000c 0001 0000
000b 000f 0010 0000
```

第二个方法：

```java
                    0001 0011 000b 0001
000c 0000 0039 0002 0001 0000 000b b200
042a b400 03b6 0005 b100 0000 0200 0d00
0000 0a00 0200 0000 0900 0a00 0a00 0e00
0000 0c00 0100 0000 0b00 0f00 1000 00
```

![方法表](/images/bytecode/methodinfo.png)

先看看第一个方法：

```java
  public com.qinglinyi.demo.ByteCodeSample();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: ldc           #2                  // String hello world
         7: putfield      #3                  // Field msg:Ljava/lang/String;
        10: return
      LineNumberTable:
        line 4: 0
        line 6: 4
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      11     0  this   Lcom/qinglinyi/demo/ByteCodeSample;
```

### 15.1 u2: access_flag

和字段表访问标志类似

方法访问标志如下表：

| 标记名 | 值 | 描述 |
| :-----------: | :-------------: | :-------------: |
| ACC_PUBLIC	         |  0x0001 | | 
| ACC_PRIVATE	         |  0x0002 | |  
| ACC_PROTECTED	         |  0x0004 | |  
| ACC_STATIC	         |  0x0008 | |  
| ACC_FINAL	         |  0x0010 | |  
| ACC_SYNCHRONIZED	 |  0x0020 | |  
| ACC_BRIDGE	         |  0x0040 | 是否由编译器产生的桥接方法 | 
| ACC_VARARGS	         |  0x0080 | 是否接受不定参数 |
| ACC_NATIVE	         |  0x0100 | |  
| ACC_ABSTRACT	         |  0x0400 | |  
| ACC_STRICT	         |  0x0800 | |  
| ACC_SYNTHETIC	         |  0x1000 | 方法是否由编译器自动产生 |


本例中：

```java
  00 01
```
该方法为public方法。

### 15.2 u2: name_index

方法名索引，本例中：

```java
  00 0b
```

对应的方法名：

```java
#10 = Utf8               <init>
```

### 15.3 u2: descriptor_index

描述符索引，本例中：

```java
  00 0a
```

对应的方法名：

```java
#11 = Utf8               ()V
```

### 15.4 u2: attributes_count

属性计数器，本例中：

```java
  00 01
```
1个属性

### 15.5 attribute_info attributes[attributes_count]

属性表集合

属性结构如下：

```java
attribute_info {
    u2 attribute_name_index;
    u4 attribute_length;
    u1 info[attribute_length];
}
```
本例中：

```java
                      00 0c00 0000 3900
0200 0100 0000 0b2a b700 012a 1202 b500
03b1 0000 0002 000d 0000 000a 0002 0000
0004 0004 0006 000e 0000 000c 0001 0000
000b 000f 0010 0000
```

#### 15.5.1 u2: attribute_name_index

```java
00 0c
```

指向

```java
#12 = Utf8               Code
```

code类型：

```java
Code_attribute {
    u2 attribute_name_index;
    u4 attribute_length;
    u2 max_stack;
    u2 max_locals;
    u4 code_length;
    u1 code[code_length];
    u2 exception_table_length;
    {   u2 start_pc;
        u2 end_pc;
        u2 handler_pc;
        u2 catch_type;
    } exception_table[exception_table_length];
    u2 attributes_count;
    attribute_info attributes[attributes_count];
}
```

#### 15.5.2 u4: attribute_length

```java
00 00 00 39
```
属性数量为57

#### 15.5.3 other

```java
                                     00
0200 0100 0000 0b2a b700 012a 1202 b500
03b1 0000 0002 000d 0000 000a 0002 0000
0004 0004 0006 000e 0000 000c 0001 0000
000b 000f 0010 0000
```

```java
Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: aload_0
         5: ldc           #2                  // String hello world
         7: putfield      #3                  // Field msg:Ljava/lang/String;
        10: return
      LineNumberTable:
        line 4: 0
        line 6: 4
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0      11     0  this   Lcom/qinglinyi/demo/ByteCodeSample;
```

## 16. 属性计数器（u2: attributes_count）

类属性数量，本例中：

```java
00 01
```
一个属性

## 17. 属性表集合（attribute_info: attributes[attributes_count]）

类属性集合，本例只有一个：

```java
00 12 00 00 00 02 00 13
```
0012对应

```java
#18 = Utf8               SourceFile
```

这是一个SourceFile类型的属性，结构如下：

```java
SourceFile_attribute {
    u2 attribute_name_index;
    u4 attribute_length;
    u2 sourcefile_index;
}
```
attribute_length:00 00 00 02为2，属性长度为2（SourceFile的长度就是2）。
sourcefile_index:00 13为13（十进制19）对应：

```java
#19 = Utf8               ByteCodeSample.java
```

*ps: 属性还有其他类型，暂略。。*

*ps: 更多详细内容，请看参考*

## 18. 参考

1.[学会阅读Java字节码](http://blog.csdn.net/dc_726/article/details/7944154)
2.《深入理解Java虚拟机：JVM高级特性与最佳实践 第2版》
3.[java文档](https://docs.oracle.com/javase/specs/jvms/se7/html/jvms-4.html#jvms-4.7.10)


