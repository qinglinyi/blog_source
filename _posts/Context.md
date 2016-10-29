---
title: Context的创建过程
date: 2016-10-28 10:30:17
categories: Android
tags:
		- Android
---

## 1.闲言碎语

一直想要系统的阅读Android源码，但是总是没能坚持，或许是打开方式不对。仔细想来，没必要一定从每个类的每个字段每个方法开始，没必要一定从底层开始，没必要一定从开机开始，没必要一定从启动开始。从一个点开始未必就不失为一个好的方式，用点连成线，用线连成面...。小范围跨度，低成本掌握，避免拖久必变，避免高坡度放弃，终能起到燎原之势！

牢骚一把。

进入主题，跟着[老罗](http://blog.csdn.net/luoshengyang/)练内功☞Context的创建过程。

ps:基于源码6.0

ps:原尊在[这里](http://blog.csdn.net/luoshengyang/article/details/8201936)，写的很详细。


## 2.创建过程

<img src="/images/context/Context.png" width = "1044" height = "629" alt="Context" />

### Step1: ActivityThread.performLaunchActivity

Activity组件的启动过程中，ActivityThread类的performLaunchActivity()方法会创建一个Activity实例，如下：

```java

private Activity performLaunchActivity(ActivityClientRecord r, Intent customIntent) {
	...
   ActivityInfo aInfo = r.activityInfo;
   ...
   ComponentName component = r.intent.getComponent();
   ...
   Activity activity = null;
   try {
       java.lang.ClassLoader cl = r.packageInfo.getClassLoader();
       activity = mInstrumentation.newActivity(
               cl, component.getClassName(), r.intent);
       ...
   } catch (Exception e) {
       ...
   }

   try {
       Application app = r.packageInfo.makeApplication(false, mInstrumentation);
       ...
       if (activity != null) {
           Context appContext = createBaseContextForActivity(r, activity);
           CharSequence title = r.activityInfo.loadLabel(appContext.getPackageManager());
           Configuration config = new Configuration(mCompatConfiguration);
           ...
           activity.attach(appContext, this, getInstrumentation(), r.token,
                   r.ident, app, r.intent, r.activityInfo, title, r.parent,
                   r.embeddedID, r.lastNonConfigurationInstances, config,
                   r.referrer, r.voiceInteractor);
           ...
           if (r.isPersistable()) {
               mInstrumentation.callActivityOnCreate(activity, r.state, r.persistentState);
           } else {
               mInstrumentation.callActivityOnCreate(activity, r.state);
           }
           ...
       }
       ...
   } catch (SuperNotCalledException e) {
       ...
    } catch (Exception e) {
       ...
   }
   return activity;
}

```

> 这个函数定义在文件frameworks/base/core/Java/android/app/ActivityThread.java中。


*这个方法中：*

* 通过mInstrumentation创建一个Activity实例。但是这个实例并没有初始化。初始化工作在activity.attach方法里进行。
* 通过方法createBaseContextForActivity()创建一个Context实例。这个方法中使用ContextImpl.createActivityContext方法生成ContextImpl实例，并将这个实例的outerContext设置为activity，将activity和这个ContextImpl实例关联，ContextImpl类里面的很多工作是通过这个outerContext完成，这样子也就是通过这个activity来完成。
* 通过activity.attach对activity进行初始化，将前面所创建的ContextImpl对象appContext以及Application对象app和Configuration对象config保存在它的内部。因为已经设置了Context，所以activity就可以访问它运行的上下文信息。activity-->appContext-->activity。
* 通过mInstrumentation的callActivityOnCreate方法会启动activity（调用OnCreate）。


### Step2: Instrumentation.newActivity

```java
public Activity newActivity(ClassLoader cl, String className,Intent intent)
		throws InstantiationException,IllegalAccessException,
		ClassNotFoundException {
	return (Activity)cl.loadClass(className).newInstance();
}
```
> 这个函数定义在文件frameworks/base/core/java/android/app/Instrumentation.java中。

在newActivity方法中，使用ClassLoader加载对应className的类，并通过newInstance实例化对象，得到的className的类（为Activity子类）的实例。


### Step3: new Activity

Activity子类实例化的时候，会调用系统提供的默认构造方法，同事调用父类的构造函数。但是，并没有执行实质的初始化工作，这个初始化工作要到attach中完成。

### Step4: ActivityThread.createBaseContextForActivity

```java
private Context createBaseContextForActivity(ActivityClientRecord r, final Activity activity) {
	...
	ContextImpl appContext = ContextImpl.createActivityContext(this, r.packageInfo, displayId, r.overrideConfig);
	appContext.setOuterContext(activity);
	Context baseContext = appContext;
	...
	baseContext = appContext.createDisplayContext(display);
	...
	return baseContext;
}

```
> 这个函数定义在文件frameworks/base/core/Java/android/app/ActivityThread.java中。

activity虽然创建，但是还需要设置Context。通过ActivityThread的createBaseContextForActivity创建一个ContextImpl的实例appContext。通过setOuterContext方法将activity设置为这个实例的OuterContext，同时将这个实例设置给Activity（见Step8）。这样，将Context和Activity进行关联，Activity有个成员mBase（来自ContextWrapper）指向appContext，ContextImpl有个成员mOuterContext指向activity。

```
activity.mBase --> appContext
appContext.mOuterContext --> activity
```

### Step5: createActivityContext

```java
static ContextImpl createActivityContext(ActivityThread mainThread,
		LoadedApk packageInfo, int displayId, Configuration overrideConfiguration) {
		
	if (packageInfo == null) throw new IllegalArgumentException("packageInfo");
	
	return new ContextImpl(null, mainThread, packageInfo, null, null, false,
		null, overrideConfiguration, displayId);
}
```

```java
private ContextImpl(ContextImpl container, ActivityThread mainThread,
	LoadedApk packageInfo, IBinder activityToken, UserHandle user, boolean
	restricted, Display display, Configuration overrideConfiguration, 
	int createDisplayWithId) {
	mOuterContext = this;
	...
}
```
> 这个函数定义在文件frameworks/base/core/Java/android/app/ContextImpl.java中。

ContextImpl在初始化的时候，mOuterContext指向的是自己，只有通过setOuterContext之后才指向activity(见下面Step6)。

### Step6: setOuterContext

```java
class ContextImpl extends Context {
	...
	private Context mOuterContext;
	...
	final void setOuterContext(Context context) {
		mOuterContext = context;
	}
}
```
> 这个函数定义在文件frameworks/base/core/Java/android/app/ContextImpl.java中。

如此，mOuterContext指向Activity。

### Step7: Activity attach

```java
final void attach(Context context, ActivityThread aThread,
	Instrumentation instr, IBinder token, int ident,
	Application application, Intent intent, ActivityInfo info,
	CharSequence title, Activity parent, String id,
	NonConfigurationInstances lastNonConfigurationInstances,
	Configuration config, String referrer, IVoiceInteractor voiceInteractor){
	attachBaseContext(context);
	mFragments.attachHost(null /*parent*/);

	mWindow = new PhoneWindow(this);
	mWindow.setCallback(this);
	mWindow.setOnWindowDismissedCallback(this);
	mWindow.getLayoutInflater().setPrivateFactory(this);
	if (info.softInputMode != WindowManager.LayoutParams.SOFT_INPUT_STATE_UNSPECIFIED) {
		mWindow.setSoftInputMode(info.softInputMode);
	}
	...
	mApplication = application;
	...
	mWindow.setWindowManager(
		(WindowManager)context.getSystemService(Context.WINDOW_SERVICE),
		mToken, mComponent.flattenToString(),
		(info.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0);
 	...
 	mWindowManager = mWindow.getWindowManager();
 	mCurrentConfig = config;
}
```
> 这个函数定义在文件frameworks/base/core/java/android/app/Activity.java中。

*这个方法：*

* 用attachBaseContext方法保存了context。
* 创建了一个PhoneWindow实例赋值给mWindow，并setCallback，setOnWindowDismissedCallback。
* 用mApplication保存application。
* 设置mWindow的WindowManager。
* 用mWindowManager保存这个WindowManager。
* 用mCurrentConfig保存config

*其中：*

> 这个PhoneWindow是用来描述当前正在启动的应用程序窗口的。这个应用程序窗口在运行的过程中，会接收到一些事件，例如，键盘、触摸屏事件等，这些事件需要转发给与它所关联的Activity组件处理，这个转发操作是通过一个Window.Callback接口来实现的。由于Activity类实现了Window.Callback接口，因此，函数就可以将当前正在启动的Activity组件所实现的一个Window.Callback接口设置到前面创建的一个PhoneWindow里面去，这是通过调用Window类的成员函数setCallback来实现的。

ps:Window应属另一个点了，静候佳音...

### Step8:  ContextThemeWrapper.attachBaseConext

```java
@Override
protected void attachBaseContext(Context newBase) {
	super.attachBaseContext(newBase);
}
```
> 这个函数定义在文件frameworks/base/core/java/android/view/ContextThemeWrapper.java中。

### Step 9: ContextWrapper.attachBaseConext

```java
public class ContextWrapper extends Context {
	Context mBase;
	...
	protected void attachBaseContext(Context base) {
		if (mBase != null) {
			throw new IllegalStateException("Base context already set");
		}
		mBase = base;
	}
}
```

> 这个函数定义在文件frameworks/base/core/java/android/content/ContextWrapper.java 中。

### Step 10: Instrumentation.callActivityOnCreate

```java
public void callActivityOnCreate(Activity activity, Bundle icicle,
		PersistableBundle persistentState) {
	prePerformCreate(activity);
	activity.performCreate(icicle, persistentState);
	postPerformCreate(activity);
}
```
> 这个函数定义在文件frameworks/base/core/java/android/app/Instrumentation.java中。

### Step 11: Activity.performCreate

```java
final void performCreate(Bundle icicle, PersistableBundle persistentState) {
	onCreate(icicle, persistentState);
	mActivityTransitionState.readState(icicle);
	performCreateCommon();
}
```
> 这个函数定义在文件frameworks/base/core/java/android/app/Activity.java中。

### Step 12: Activity.onCreate

```java
public void onCreate(@Nullable Bundle savedInstanceState,
		@Nullable PersistableBundle persistentState) {
	onCreate(savedInstanceState);
}
```
> 这个函数定义在文件frameworks/base/core/java/android/app/Activity.java中。

### Step 13: Activity.onCreate

我们所熟悉的onCreate方法。（略）

## 3.类关系

通过上面我们调用过程我们知道ContextImpl和Activity的关系，下面我们在看看它们的类关系：

<img src="/images/context/Context_class.png" width = "733" height = "448" alt="Context_class" />


>  这个类图在设计模式里面就可以称为装饰模式。Activity组件通过其父类ContextWrapper的成员变量mBase来引用了一个ContextImpl对象，这样，Activity组件以后就可以通过这个ContextImpl对象来执行一些具体的操作。同时，ContextImpl类又通过自己的成员变量mOuterContext来引用了与它关联的一个Activity组件，这样，ContextImpl类也可以将一些操作转发给Activity组件来处理。

