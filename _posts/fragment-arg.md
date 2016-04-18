---
title: 使用注解操作Fragment Argument
date: 2016-04-15 11:53:36
categories: Android
tags:
    - Android
    - Fragment
---

最近做了一个操作Fragment的Argument的简便方式，使用注解在编译时生成操作Argument的代码，在Fragment注入获取代码，完成对Argument的操作。

这样的轮子已经有了[这个](https://github.com/sockeqwe/fragmentargs)，再造的同时，顺便学习一下编译时注解，另外可以自定义自己的需求。

项目地址在这里<https://github.com/qinglinyi/FragmentArg>

## 使用

```java
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.0.0'
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.4'
    }
}
```

```java
apply plugin: 'com.neenbedankt.android-apt'
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.qinglinyi.arg:arg-api:0.9.0'
    apt 'com.qinglinyi.arg:arg-compiler:0.9.0'
}

apt {
    arguments {
        // FragmentBuilder的包
        argPackageName "com.qinglinyi.arg.sample"
    }
}
```
**Fragment**

```java
@UseArg
public class MyFragment extends Fragment {

    @Arg
    String name;

    @Arg
    int age;

    @Arg
    ArrayList<String> interests;

    @Arg
    String[] friendNames;

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ArgInjector.inject(this);
    }
    ...
```


**Activity**两种方式

```java
 mFragment0 = FragmentBuilder.builder(new MyFragment())
                .age(15)
                .name("张三")
                .friendNames(new String[]{"李四", "王五"})
                .interests(list)
                .build();
```

```java
 mFragment1 = new MyFragmentBuilder().age(15)
                .name("李四")
                .friendNames(new String[]{"张三", "王五"})
                .interests(list)
                .build();
```



