---
title: Android Studio 发布项目到jcenter库
date: 2016-04-06 14:59:13
tags:
    - 记录
---

转载请注明：http://www.qinglinyi.com/posts/publish-to-jcenter/

第一次发布项目到jcenter，虽然网上有很多教程了，但是过程还是比较曲折。不过最终还是找到简单的方式，使用com.novoda.bintray-release实现发布。



## 认识jcenter


我们经常在android studio项目中看到：

```
allprojects {
    repositories {
        jcenter()
    }
}
```

**那么jcenter到底是什么呢？**

我们可以将jcenter理解为**代码仓库**。如果我们在builde.gradle文件中设置例如：

```
compile 'com.google.code.gson:gson:2.3.1'
```

<!-- more -->

这时，Android Studio或者说Gradle会**自动**从jcenter**下载**	gson的jar包（实际上Maven packages，但是我们主要关心是jar或者aar），这样我们就可以在项目中使用gson了。不需要手动下载jar包然后导入到项目中了。 
  
  
我们在文件夹下面（.gradle/caches/modules-2/files-2.1，这个路径我电脑下的）找到这些由gradle下载的文件，像这样：

![gson文件](/images/ptj/gson_file.png)

不过我们一般不用关心这些，只要项目能自动导入依赖就好了。

如果你想关心这些文件在哪里也没关系。

![library属性](/images/ptj/lib_info_0.png)
![library属性](/images/ptj/lib_info_1.png)

（走偏了，回来。。）

**所以，我们就能大致明白了这个jcenter是干什么的。那么来确定一下吧：**

![about_jcenter](/images/ptj/about_jcenter.png)

详细信息在这里：<https://bintray.com/bintray/jcenter>

**主要就是说：**

> JCenter is the place to find and share popular Apache Maven packages for use by Maven, Gradle, Ivy, SBT, etc. 

 **提供Maven, Gradle, Ivy, SBT等查找和分享Apache Maven packages的地方**


顺便我们了解一下bintray <https://bintray.com/howbintrayworks>

![bintray](/images/ptj/how_bintray_work.png)

> Bintray, your platform for automated software distribution

**当然jcenter只是bintray下的一个maven packages repository**

bintray不只支持maven packages 还支持其他类型：

![bintray](/images/ptj/bintray_type.png)


**了解完jcenter我们就开始吧！**


## 账号注册

1. 要把项目发布到jcenter我们需要先注册bintray的账号，[https://bintray.com](https://bintray.com)，我是通过GitHub账号注册的。（这一步基本没什么问题） 
2. 然后获取API key （图）
![API key](/images/ptj/api_0.png)
![API key](/images/ptj/api_1.png)
![API key](/images/ptj/api_2.png)

**记下API key 发布的时候使用**


## 添加package

![package](/images/ptj/add_package_0.png)
![package](/images/ptj/add_package_1.png)
![package](/images/ptj/add_package_2.png)
![package](/images/ptj/add_package_3.png)

其中，**name** ：包名字
例如：
```
'com.android.tools.build:gradle:1.5.0'
```
gradle 就是名字。


## 配置项目

```
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.5.0'
        classpath 'com.novoda:bintray-release:0.3.4'
    }
}

allprojects {
    repositories {
        jcenter()
    }
}

ext {
    userOrg = 'qinglinyi'
    groupId = 'com.qinglinyi.arg'
    description = 'fragment arg'
    publishVersion = '1.0.0' 
    website = 'https://github.com/qinglinyi/FragmentArg'
    dryRun = 'false'
}

```

```
apply plugin: 'java'
apply plugin: 'com.novoda.bintray-release'


publish {
    artifactId = 'arg-api' // library的名字
    userOrg = rootProject.userOrg //用户所在组织
    groupId = rootProject.groupId // 包名
    publishVersion = rootProject.publishVersion // 版本
    description = rootProject.description // 描述
    website = rootProject.website 
    bintrayUser = rootProject.bintrayUser // 账户名
    bintrayKey = rootProject.bintrayKey // 就是API key
    dryRun = rootProject.dryRun

}
```

我是使用com.novoda:bintray-release来完成的，详细信息和各个字段的含义可以看这里：
<https://github.com/novoda/bintray-release>
<https://github.com/novoda/bintray-release/wiki/Configuration-of-the-publish-closure>

## 运行发布

点击Gradle的命令工具

![publish](/images/ptj/publish_0.png)

如果成功会在网站上看到：

![publish](/images/ptj/publish_1.png)

**证明发布成功了**

这样我们就能够使用了，但是这个只是在我们自己的仓库中，还没到jcenter。

```
allprojects {
    repositories {
        maven {
            url 'https://dl.bintray.com/qinglinyi/maven'
        }
        jcenter()

    }
}
```

这个地址你可以在网站上复制一下或者使用这个地址的中名字改成自己的就可以了。

复制在这里：

![use](/images/ptj/use_self.png)



## 添加到jcenter

最后我们将包添加到jcenter中，添加成功时候我们就不需要添加自己的maven地址了。

![to_jcenter](/images/ptj/add_to_jcenter_0.png)
![to_jcenter](/images/ptj/add_to_jcenter_1.png)

通过可能需要一些时间，注册查看通知。

**成功之后，是这样的**
![to_jcenter](/images/ptj/add_to_jcenter_2.png)


**大功告成！！**

详细的配置可以看我的Github项目<https://github.com/qinglinyi/FragmentArg>

## 补充

发布jcenter之后如果需要上传到Maven Center的话，需要修改maven仓库首页的GPG
![to_maven](/images/ptj/maven_add_gpg.png)

然后才能在这里同步到maven center

![to_maven](/images/ptj/maven_add.png)

**同时，提交到maven的时候bintray-release 插件有一个问题**

解决方案在这里<https://github.com/novoda/bintray-release/wiki/Add-support-for-syncing-to-maven-central>


**如果发布成功可以在这里检查**：

1. maven center <https://oss.sonatype.org/content/repositories/releases>
2. jcenter <http://jcenter.bintray.com>


## 参考

http://www.jianshu.com/p/499a086e3bab
http://www.cnblogs.com/qianxudetianxia/p/4322331.html



