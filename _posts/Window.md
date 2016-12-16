---
title: Window
date: 2016-10-31 15:35:46
categories: Android
tags:
		- Android
---

本文简单介绍Window的创建过程。基于源码6.0，主要参考[这里](http://blog.csdn.net/luoshengyang/article/details/8223770)。


## 1.创建过程

<img src="/images/window/Window.png" width = "816" height = "612" alt="Window" />

### Step1:  Activity.attach

```java
final void attach(Context context, ActivityThread aThread,
       Instrumentation instr, IBinder token, int ident,
       Application application, Intent intent, ActivityInfo info,
       CharSequence title, Activity parent, String id,
       NonConfigurationInstances lastNonConfigurationInstances,
       Configuration config, String referrer, IVoiceInteractor voiceInteractor) {
   ...
   mWindow = new PhoneWindow(this);
   mWindow.setCallback(this);
   mWindow.setOnWindowDismissedCallback(this);
   mWindow.getLayoutInflater().setPrivateFactory(this);
   if (info.softInputMode != WindowManager.LayoutParams.SOFT_INPUT_STATE_UNSPECIFIED) {
       mWindow.setSoftInputMode(info.softInputMode);
   }
   if (info.uiOptions != 0) {
       mWindow.setUiOptions(info.uiOptions);
   }
  ...
   mWindow.setWindowManager(
           (WindowManager)context.getSystemService(Context.WINDOW_SERVICE),
           mToken, mComponent.flattenToString(),
           (info.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0);
   if (mParent != null) {
       mWindow.setContainer(mParent.getWindow());
   }
   mWindowManager = mWindow.getWindowManager();
   ...
}

```

>这个函数定义在文件frameworks/base/core/Java/android/app/Activity.java中。

在attach方法中，创建PhoneWindow对象并且保存在Activity类的成员变量mWindow中。然后方法setCallback、setOnWindowDismissedCallback、setSoftInputMode和setWindowManager来设置窗口回调接口、窗口消失回调接口、软键盘输入区域的显示模式和窗口管理器。

### Step2: PhoneWindow.new

```java 
public class PhoneWindow extends Window implements MenuBuilder.Callback {
	...
	private LayoutInflater mLayoutInflater;
	...
	private DecorView mDecor;
	private ViewGroup mContentParent;
	...
	public PhoneWindow(Context context) {
		super(context);
		mLayoutInflater = LayoutInflater.from(context);
	}
	...
}
```
> 这个函数定义在文件frameworks/base/core/java/com/android/internal/policy/PhoneWindow.java中。

> 调用LayoutInflater的静态成员函数from创建一个LayoutInflater实例，并且保存在成员变量mLayoutInflater中。这样，PhoneWindow类以后就可以通过成员变量mLayoutInflater来创建应用程序窗口的视图，这个视图使用类型为DecorView的成员变量mDecor来描述。PhoneWindow类还有另外一个类型为ViewGroup的成员变量mContentParent，用来描述一个视图容器，这个容器存放的就是成员变量mDecor所描述的视图的内容，不过这个容器也有可能指向的是mDecor本身。

Window的构造很简单，只是初始化一个Context。如下：

```java
public abstract class Window {
	...
	private final Context mContext;
	...
	private int mFeatures;
	...
	public Window(Context context) {
		mContext = context;
		mFeatures = mLocalFeatures = getDefaultFeatures(context);
	}
	...
}
```
> 这个函数定义在frameworks/base/core/java/android/view/Window.java中。

这个Context来自Activity，这样Window可以访问Activity的相关资源。使用getDefaultFeatures
方法设置默认的Features，这些Features将在requestFeature()中使用到，用来设置Window的一些现实特征，如下：

```java
/** Flag for the "options panel" feature.  This is enabled by default. */
public static final int FEATURE_OPTIONS_PANEL = 0;
/** Flag for the "no title" feature, turning off the title at the top
*  of the screen. */
public static final int FEATURE_NO_TITLE = 1;
/** Flag for the progress indicator feature */
public static final int FEATURE_PROGRESS = 2;
/** Flag for having an icon on the left side of the title bar */
public static final int FEATURE_LEFT_ICON = 3;
/** Flag for having an icon on the right side of the title bar */
public static final int FEATURE_RIGHT_ICON = 4;
/** Flag for indeterminate progress */
public static final int FEATURE_INDETERMINATE_PROGRESS = 5;
/** Flag for the context menu.  This is enabled by default. */
public static final int FEATURE_CONTEXT_MENU = 6;
/** Flag for custom title. You cannot combine this feature with other title features. */
public static final int FEATURE_CUSTOM_TITLE = 7;
/**
* Flag for enabling the Action Bar.
* This is enabled by default for some devices. The Action Bar
* replaces the title bar and provides an alternate location
* for an on-screen menu button on some devices.
*/
public static final int FEATURE_ACTION_BAR = 8;
/**
* Flag for requesting an Action Bar that overlays window content.
* Normally an Action Bar will sit in the space above window content, but if this
* feature is requested along with {@link #FEATURE_ACTION_BAR} it will be layered over
* the window content itself. This is useful if you would like your app to have more control
* over how the Action Bar is displayed, such as letting application content scroll beneath
* an Action Bar with a transparent background or otherwise displaying a transparent/translucent
* Action Bar over application content.
*
* <p>This mode is especially useful with {@link View#SYSTEM_UI_FLAG_FULLSCREEN
* View.SYSTEM_UI_FLAG_FULLSCREEN}, which allows you to seamlessly hide the
* action bar in conjunction with other screen decorations.
*
* <p>As of {@link android.os.Build.VERSION_CODES#JELLY_BEAN}, when an
* ActionBar is in this mode it will adjust the insets provided to
* {@link View#fitSystemWindows(android.graphics.Rect) View.fitSystemWindows(Rect)}
* to include the content covered by the action bar, so you can do layout within
* that space.
*/
public static final int FEATURE_ACTION_BAR_OVERLAY = 9;
/**
* Flag for specifying the behavior of action modes when an Action Bar is not present.
* If overlay is enabled, the action mode UI will be allowed to cover existing window content.
*/
public static final int FEATURE_ACTION_MODE_OVERLAY = 10;
/**
* Flag for requesting a decoration-free window that is dismissed by swiping from the left.
*/
public static final int FEATURE_SWIPE_TO_DISMISS = 11;
/**
* Flag for requesting that window content changes should be animated using a
* TransitionManager.
*
* <p>The TransitionManager is set using
* {@link #setTransitionManager(android.transition.TransitionManager)}. If none is set,
* a default TransitionManager will be used.</p>
*
* @see #setContentView
*/
public static final int FEATURE_CONTENT_TRANSITIONS = 12;

/**
* Enables Activities to run Activity Transitions either through sending or receiving
* ActivityOptions bundle created with
* {@link android.app.ActivityOptions#makeSceneTransitionAnimation(android.app.Activity,
* android.util.Pair[])} or {@link android.app.ActivityOptions#makeSceneTransitionAnimation(
* android.app.Activity, View, String)}.
*/
public static final int FEATURE_ACTIVITY_TRANSITIONS = 13;
```

### Step3: Window.setCallback

```java
public abstract class Window {
	...
	private Callback mCallback;
	...
	public void setCallback(Callback callback) {
		mCallback = callback;
	}
	...
}
```
>这个函数定义在文件frameworks/base/core/java/android/view/Window.java中。

设置Callback，这个方法在PhoneWindow的父类Window中，用来拦截键盘时间和其他触摸操作等。这个Callback在Activity中实现。

### Step4: Window.setOnWindowDismissedCallback

```java
public abstract class Window {
	...
	private OnWindowDismissedCallback mOnWindowDismissedCallback;
	...
	public final void setOnWindowDismissedCallback(OnWindowDismissedCallback dcb) {
		mOnWindowDismissedCallback = dcb;
	}
	...
}
```

>这个函数定义在文件frameworks/base/core/java/android/view/Window.java中。

这个OnWindowDismissedCallback会在Window gone的时候调用，这个回调也是在Activity中实现。


### Step5: Window.setSoftInputMode

```java
public abstract class Window {
	...
	private boolean mHasSoftInputMode = false;
	...
	public void setSoftInputMode(int mode) {
		final WindowManager.LayoutParams attrs = getAttributes();
		if (mode != WindowManager.LayoutParams.SOFT_INPUT_STATE_UNSPECIFIED) {
		  attrs.softInputMode = mode;
		  mHasSoftInputMode = true;
		} else {
		  mHasSoftInputMode = false;
		}
		dispatchWindowAttributesChanged(attrs);
	}
	...
}
```
>这个函数定义在文件frameworks/base/core/java/android/view/Window.java中。

setSoftInputMode用来设置应用程序窗口的软键盘输入区域的显示模式。取值如下：

```java
...
/**
* Visibility state for {@link #softInputMode}: no state has been specified.
*/
public static final int SOFT_INPUT_STATE_UNSPECIFIED = 0;

/**
* Visibility state for {@link #softInputMode}: please don't change the state of
* the soft input area.
*/
public static final int SOFT_INPUT_STATE_UNCHANGED = 1;

/**
* Visibility state for {@link #softInputMode}: please hide any soft input
* area when normally appropriate (when the user is navigating
* forward to your window).
*/
public static final int SOFT_INPUT_STATE_HIDDEN = 2;

/**
* Visibility state for {@link #softInputMode}: please always hide any
* soft input area when this window receives focus.
*/
public static final int SOFT_INPUT_STATE_ALWAYS_HIDDEN = 3;

/**
* Visibility state for {@link #softInputMode}: please show the soft
* input area when normally appropriate (when the user is navigating
* forward to your window).
*/
public static final int SOFT_INPUT_STATE_VISIBLE = 4;

/**
* Visibility state for {@link #softInputMode}: please always make the
* soft input area visible when this window receives input focus.
*/
public static final int SOFT_INPUT_STATE_ALWAYS_VISIBLE = 5;
...
/** Adjustment option for {@link #softInputMode}: nothing specified.
* The system will try to pick one or
* the other depending on the contents of the window.
*/
public static final int SOFT_INPUT_ADJUST_UNSPECIFIED = 0x00;

/** Adjustment option for {@link #softInputMode}: set to allow the
* window to be resized when an input
* method is shown, so that its contents are not covered by the input
* method.  This can <em>not</em> be combined with
* {@link #SOFT_INPUT_ADJUST_PAN}; if
* neither of these are set, then the system will try to pick one or
* the other depending on the contents of the window. If the window's
* layout parameter flags include {@link #FLAG_FULLSCREEN}, this
* value for {@link #softInputMode} will be ignored; the window will
* not resize, but will stay fullscreen.
*/
public static final int SOFT_INPUT_ADJUST_RESIZE = 0x10;

/** Adjustment option for {@link #softInputMode}: set to have a window
* pan when an input method is
* shown, so it doesn't need to deal with resizing but just panned
* by the framework to ensure the current input focus is visible.  This
* can <em>not</em> be combined with {@link #SOFT_INPUT_ADJUST_RESIZE}; if
* neither of these are set, then the system will try to pick one or
* the other depending on the contents of the window.
*/
public static final int SOFT_INPUT_ADJUST_PAN = 0x20;

/** Adjustment option for {@link #softInputMode}: set to have a window
* not adjust for a shown input method.  The window will not be resized,
* and it will not be panned to make its focus visible.
*/
public static final int SOFT_INPUT_ADJUST_NOTHING = 0x30;
...
/**
* Desired operating mode for any soft input area.  May be any combination
* of:
*
* <ul>
* <li> One of the visibility states
* {@link #SOFT_INPUT_STATE_UNSPECIFIED}, {@link #SOFT_INPUT_STATE_UNCHANGED},
* {@link #SOFT_INPUT_STATE_HIDDEN}, {@link #SOFT_INPUT_STATE_ALWAYS_VISIBLE}, or
* {@link #SOFT_INPUT_STATE_VISIBLE}.
* <li> One of the adjustment options
* {@link #SOFT_INPUT_ADJUST_UNSPECIFIED},
* {@link #SOFT_INPUT_ADJUST_RESIZE}, or
* {@link #SOFT_INPUT_ADJUST_PAN}.
* </ul>
*
*
* <p>This flag can be controlled in your theme through the
* {@link android.R.attr#windowSoftInputMode} attribute.</p>
*/
public int softInputMode;
```

> 1. SOFT_INPUT_STATE_UNSPECIFIED：没有指定软键盘输入区域的显示状态。
> 2. SOFT_INPUT_STATE_UNCHANGED：不要改变软键盘输入区域的显示状态。
> 3. SOFT_INPUT_STATE_HIDDEN：在合适的时候隐藏软键盘输入区域，例如，当用户导航到当前窗口时。
> 4. SOFT_INPUT_STATE_ALWAYS_HIDDEN：当窗口获得焦点时，总是隐藏软键盘输入区域。
> 5. SOFT_INPUT_STATE_VISIBLE：在合适的时候显示软键盘输入区域，例如，当用户导航到当前窗口时。
> 6. SOFT_INPUT_STATE_ALWAYS_VISIBLE：当窗口获得焦点时，总是显示软键盘输入区域。

- - -

- [ ]  等待详细介绍

- - -



### Step5: Window.setWindowManager

```java
public abstract class Window {
	...
	private WindowManager mWindowManager;
	private IBinder mAppToken;
	private String mAppName;
	private boolean mHardwareAccelerated;
	...
	public void setWindowManager(WindowManager wm, IBinder appToken, 
		String appName,boolean hardwareAccelerated) {
		mAppToken = appToken;
		mAppName = appName;
		mHardwareAccelerated = hardwareAccelerated
		 || SystemProperties.getBoolean(PROPERTY_HARDWARE_UI, false);
		if (wm == null) {
			wm = (WindowManager)mContext.getSystemService(Context.WINDOW_SERVICE);
		}
		mWindowManager = ((WindowManagerImpl)wm).createLocalWindowManager(this);
	}
}
```
>这个函数定义在文件frameworks/base/core/java/android/view/Window.java中。

从Activity的attach方法setWindowManager,传进来一个默认的WindowManager(WindowManagerImpl，来自SystemService
)，调用这个WindowManager的createLocalWindowManager方法生成新的WindowManager对象保存到Window的成员mWindowManager。

并且在Activity中也使用成员mWindowManager保存这个Window生成的WindowManger（见Step1）。

>参数appToken用来描述当前正在处理的窗口是与哪一个Activity组件关联的，它是一个Binder代理对象，引用了在ActivityManagerService这一侧所创建的一个类型为ActivityRecord的Binder本地对象。
>参数appName用来描述当前正在处理的窗口所关联的Activity组件的名称，这个名称会被保存在Window类的成员变量mAppName中。

### Step6: WindowManagerImpl.createLocalWindowManager

```java
public final class WindowManagerImpl implements WindowManager {
	...
	private final WindowManagerGlobal mGlobal = WindowManagerGlobal.getInstance();
	private final Display mDisplay;
	private final Window mParentWindow;
	...

	private WindowManagerImpl(Display display, Window parentWindow) {
		mDisplay = display;
		mParentWindow = parentWindow;
	}

	public WindowManagerImpl createLocalWindowManager(Window parentWindow) {
		return new WindowManagerImpl(mDisplay, parentWindow);
	}
	...
}
```
>这个函数定义在文件frameworks/base/core/java/android/view/WindowManagerImpl.java中。

WindowManagerGlobal是提供与上下文无关的系统窗口管理，只用于内部实现全局函数。

WindowManagerGlobal是一个单例，并且是窗口管理的真正现实类，不过在一般情况下，我们不直接使用使用它，而是通过已经绑定了上下文的WindowManager（WindowManagerImpl）来管理窗口。

所以，理所当然，WindowManager的实现类WindowManagerImpl里面需要有WindowManagerGlobal的实例，来真正处理窗口管理。

Display是用来描述全局的屏幕属性的，提供逻辑显示区域大小、密度的相关信息。

成员mParentWindow就是前面PhoneWindow实例，在创建WindowManager的时候传入的（见Step5）。


## 2.类关系

<img src="/images/window/Window2.png" width = "748" height = "603" alt="Window" />







