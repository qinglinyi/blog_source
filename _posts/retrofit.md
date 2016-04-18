
---
title: Retrofit2 源码解析
date: 2016-03-08 12:22:29
categories: Android
tags:
	- Retrofit2
	- Retrofit
	- Android
---

转载请注明：<http://www.qinglinyi.com/posts/retrofit/>

本文是关于Retrofit2源码解析的，主要介绍了Retrofit 2.0API的基本原理、主要类以及接口还有Retrofit是怎么工作的。

## 认识Retrofit2

***  A type-safe HTTP client for Android and Java ***
意思就是说Retrofit是一个Android和Java的类型安全的Http客户端/请求工具。
对于Retrofit的详细介绍我们可以通过[官网](http://square.github.io/retrofit/)了解。 

## Retrofit2 使用
  
  ```
  public interface GitHubService {
  	@GET("users/{user}/repos")
  	Call<List<Repo>> listRepos(@Path("user") String user);
  }
  
  Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com")
    .build();

  GitHubService service = retrofit.create(GitHubService.class);
  
  Call<List<Repo>> repos = service.listRepos("octocat");
  
  repos.enqueue(new Callback<List<Repo>>() {
    @Override
    public void onResponse(Response<List<Repo>> response) {
      // do something  
    }
    @Override
    public void onFailure(Throwable t) {
      // do something
    }
 });
 
 ```
 
 或者这样：
 
 ```
  public interface GitHubService {
  	@GET("users/{user}/repos")
  	Observable<List<Repo>> listRepos(@Path("user") String user);
  }
  
  Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com")
    .addConverterFactory(GsonConverterFactory.create())
    .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
    .build();

  GitHubService service = retrofit.create(GitHubService.class);
  
  service.listRepos("octocat")
                .observeOn(AndroidSchedulers.mainThread())
                .subscribeOn(Schedulers.io())
                .subscribe();
 
 ```
** `注意：`** `详细的使用，大家可以查看其它教程。`


<!-- more -->

## Retrofit2是怎么工作的

如上面的使用中我们可以看到：

1. 通过Retrofit.Builder()创建Retrofit。
2. 调用Retrofit的create()方法将生成接口GitHubService的实例。
3. 调用GitHubService的listRepos()方法返`Call<List<Repo>>`。
4. 执行`Call<List<Repo>>`的enqueue方法在回调中处理返回的`List<Repo>`。

## Retrofit2的主要成员

那么在了解Retrofit2到底是怎么工作的之前我们先认识一下Retrofit2的主要成员：

1. Call(接口)--向服务器发送请求并返回响应的调用
2. CallAdapter(接口)--Call的适配器，用来包装转换Call
3. CallBack(接口)--顾名思义Call的回调,Call执行时的回调
4. Converter(接口)--数据转换器，将一个对象转化另外一个对象
5. CallAdapter.Factory(接口)--CallAdapter的工厂，通过get方法获取CallAdapter
6. Converter.Factory(抽象类) -- 数据转换器Converter的工厂
   * responseBodyConverter -- 将服务器返回的数据转化ResponseBody。可以理解为数据解析的转换器
   * requestBodyConverter -- 将GitHubService.listRepos()中的Body，Part和PartMap注解转换为RequestBody(OkHttp3)，以便http请求的时候使用。
   * stringConverter -- 将Field，FieldMap 值，Header，Path，Query,和QueryMap值转化为String，以便http请求的时候使用。
   
7. MethodHandler -- 处理、执行GitHubService方法的类
8. RequestFactory -- 创建OkHttp请求的Request
9. RequestFactoryParser -- 解析GitHubService.listRepos()方法的注解和参数，生成RequestFactory。（会用到requestBodyConverter，stringConverter）
10. OkHttpCall -- 实现Call接口，获取传入的Call（代理Call，通过Retrofit.callFactory生成的）执行请求，获取数据并使用responseConverter进行解析。


## 创建Retrofit


通过Retrofit.Builder创建Retrofit，先看看Retrofit.Builder都有什么。

Retrofit.Builder：

```
  	 private okhttp3.Call.Factory callFactory; // okhttp3 的网络请求Call
    private BaseUrl baseUrl; // 网络请求地址的基础部分（公共部分），Retrofit会自动拼接成真正的请求地址
    private List<Converter.Factory> converterFactories = new ArrayList<>();
    private List<CallAdapter.Factory> adapterFactories = new ArrayList<>();
    private Executor callbackExecutor; // 回调的Executor
    private boolean validateEagerly;// 若为真，会提交调用eagerlyValidateMethods方法，提前加载MethodHandler
```


```
    public Retrofit build() {
      if (baseUrl == null) {
        throw new IllegalStateException("Base URL required.");
      }

      okhttp3.Call.Factory callFactory = this.callFactory;
      if (callFactory == null) {
        callFactory = new OkHttpClient();
      }

      // Make a defensive copy of the adapters and add the default Call adapter.
      List<CallAdapter.Factory> adapterFactories = new ArrayList<>(this.adapterFactories);
      adapterFactories.add(Platform.get().defaultCallAdapterFactory(callbackExecutor));

      // Make a defensive copy of the converters.
      List<Converter.Factory> converterFactories = new ArrayList<>(this.converterFactories);

      return new Retrofit(callFactory, baseUrl, converterFactories, adapterFactories,
          callbackExecutor, validateEagerly);
    }

```

通过build的方法，我可以知道Retrofit.Builder提供一个默认的OkHttpClient，一个默认的CallAdapterFactory。
默认Platform：

```
  CallAdapter.Factory defaultCallAdapterFactory(Executor callbackExecutor) {
    if (callbackExecutor != null) {
      return new ExecutorCallAdapterFactory(callbackExecutor);
    }
    return DefaultCallAdapter.FACTORY;
  }
```
Platform.get().defaultCallAdapterFactory(callbackExecutor)返回默认的CallAdapterFactory有两种情况：

1. callbackExecutor 为空的时候返回的是DefaultCallAdapter.FACTORY，这个DefaultCallAdapter.FACTORY返回一个DefaultCallAdapter，DefaultCallAdapter很简单，没做什么处理。
2. callbackExecutor不为空的时候返回ExecutorCallAdapterFactory，这个Factory返回的CallAdapter是ExecutorCallbackCall，我们看看这个ExecutorCallbackCall的enqueue方法：

```
@Override public void enqueue(final Callback<T> callback) {
      delegate.enqueue(new Callback<T>() {
        @Override public void onResponse(final Response<T> response) {
          callbackExecutor.execute(new Runnable() {
            @Override public void run() {
              if (delegate.isCanceled()) {
                // Emulate OkHttp's behavior of throwing/delivering an IOException on cancelation
                callback.onFailure(new IOException("Canceled"));
              } else {
                callback.onResponse(response);
              }
            }
          });
        }

        @Override public void onFailure(final Throwable t) {
          callbackExecutor.execute(new Runnable() {
            @Override public void run() {
              callback.onFailure(t);
            }
          });
        }
      });
    }
```
也很简单，就是修饰了一下CallBack，用callbackExecutor执行真正的回调，这样回调可以在线程中执行并使用callbackExecutor管理。

如果是Android的话：

```
 static class Android extends Platform {
    @Override CallAdapter.Factory defaultCallAdapterFactory(Executor callbackExecutor) {
      if (callbackExecutor == null) {
        callbackExecutor = new MainThreadExecutor();
      }
      return new ExecutorCallAdapterFactory(callbackExecutor);
    }

    static class MainThreadExecutor implements Executor {
      private final Handler handler = new Handler(Looper.getMainLooper());

      @Override public void execute(Runnable r) {
        handler.post(r);
      }
    }
  }
```
也就是说如果在Android上使用的话，如果callbackExecutor为空会直接使用handler将回调返回到主线程（MainThread）。

当然，如果需要自己配置请求Client的话可以设置callFactory。

	        HttpLoggingInterceptor logger = new HttpLoggingInterceptor();
            logger.setLevel(HttpLoggingInterceptor.Level.BODY);
            OkHttpClient client = new OkHttpClient
                    .Builder()
                    .addInterceptor(logger).build();
            Retrofit retrofit = new Retrofit.Builder()
                    .baseUrl("https://api.github.com")
                    .client(client)
                    .build();


使用自己的OkHttpClient，并为这个OkHttpClient设置一个日志工具。通过这种方式设置，整个项目可以共用一个OkHttpClient。
    
        
              
                  
                  
## 动态代理

```
 GitHubService service = retrofit.create(GitHubService.class);
```

create方法传入一个接口GitHubService，然后动态代理GitHubService。

```
 public <T> T create(final Class<T> service) {
    Utils.validateServiceInterface(service);
    if (validateEagerly) {
      eagerlyValidateMethods(service);
    }
    return (T) Proxy.newProxyInstance(service.getClassLoader(), new Class<?>[] { service },
        new InvocationHandler() {
          private final Platform platform = Platform.get();

          @Override public Object invoke(Object proxy, Method method, Object... args)
              throws Throwable {
            // If the method is a method from Object then defer to normal invocation.
            if (method.getDeclaringClass() == Object.class) {
              return method.invoke(this, args);
            }
            if (platform.isDefaultMethod(method)) {
              return platform.invokeDefaultMethod(method, service, proxy, args);
            }
            return loadMethodHandler(method).invoke(args);
          }
        });
  }
```
主要是loadMethodHandler(method).invoke(args)。也就是说在调用service.listRepos方法的时候，会通过loadMethodHandler(method).invoke(args)来执行。那么看看loadMethodHandler做了什么：

```
 MethodHandler loadMethodHandler(Method method) {
    MethodHandler handler;
    synchronized (methodHandlerCache) {
      handler = methodHandlerCache.get(method);
      if (handler == null) {
        handler = MethodHandler.create(this, method);
        methodHandlerCache.put(method, handler);
      }
    }
    return handler;
  }
```

创建MethodHandler并使用一个Map-methodHandlerCache来自缓存MethodHandler。看来剩下的事交给MethodHandler了。

## MethodHandler

```
  static MethodHandler create(Retrofit retrofit, Method method) {
    CallAdapter<?> callAdapter = createCallAdapter(method, retrofit);
    Type responseType = callAdapter.responseType();
    if (responseType == Response.class || responseType == okhttp3.Response.class) {
      throw Utils.methodError(method, "'"
          + Utils.getRawType(responseType).getName()
          + "' is not a valid response body type. Did you mean ResponseBody?");
    }
    Converter<ResponseBody, ?> responseConverter =
        createResponseConverter(method, retrofit, responseType);
    RequestFactory requestFactory = RequestFactoryParser.parse(method, responseType, retrofit);
    return new MethodHandler(retrofit.callFactory(), requestFactory, callAdapter,
        responseConverter);
  }
```

```
  Object invoke(Object... args) {
    return callAdapter.adapt(
        new OkHttpCall<>(callFactory, requestFactory, args, responseConverter));
  }
```

1. invoke()方法，调用callAdapter.adapt的方法并传入一个OkHttpCall。如果是默认的CallAdapter的话返回的就是一个OkHttpCall或者ExecutorCallAdapterFactory.ExecutorCallbackCall，也就是说listRepos("user")方法就返回Call<List<Repo>> 。所以说方法返回什么要看CallAdapter.adapt，如果要返回Observable<List<Repo>>就需要RxJavaCallAdapterFactory.create()的CallAdapter。
2. 在介绍的主要成员的时候说过OkHttpCall，执行请求的类，那么执行请求之前呢。那就要看create方法，这个方法中拿到：

	* CallAdapter--通过Retrofit类的callAdapter方法获得。
	* 拿到ResponseConverter--通过Retrofit类的createResponseConverter方法获得。
	* 获得RequestFactory--通过RequestFactoryParser.parse方法获得。这个parse方法会解析method也就是listRepos("user")方法的注解和参数，返回一个RequestFactory。
	
	 
		```
   static RequestFactory parse(Method method, Type responseType, Retrofit retrofit) {
    	RequestFactoryParser parser = new RequestFactoryParser(method);
    	parser.parseMethodAnnotations(responseType);
    	parser.parseParameters(retrofit);
    	return parser.toRequestFactory(retrofit.baseUrl());
   }
  ```
 `注：具体的解析就不展开了，主要是将注解和参数的值获取放入RequestAction，RequestFactory会根据这些RequestAction和基本参数构建OkHttp使用的Request。`
3. 最后这些callFactory, requestFactory, responseConverter将传入OkHttpCall


## OkHttpCall

执行Http请求返回数据并解析成实体，在回调中返回。

* 请求:通过CallFactory(OkHttpClient)的newCall方法传入一个Request（通过requestFactory.create(args)创建）执行。

	```
	okhttp3.Call rawCall = createRawCall();
	```
	```
	private okhttp3.Call createRawCall() throws IOException {
   	 okhttp3.Call call = callFactory.newCall(requestFactory.create(args));
    	if (call == null) {
     	 throw new NullPointerException("Call.Factory returned null.");
    	}
     return call;
  }
	```
	

* 解析:通过responseConverter.convert()方法解析

	```
	Response<T> parseResponse(okhttp3.Response rawResponse) throws IOException {
    ...
    try {
      T body = responseConverter.convert(catchingBody);
      return Response.success(body, rawResponse);
    } catch (RuntimeException e) {
      // If the underlying source threw an exception, propagate that rather than indicating it was
      // a runtime exception.
      catchingBody.throwIfCaught();
      throw e;
    }
  }
	```

## 总结

1. Retrofit就是一个Http请求工具。
2. Retrofit使用接口代替Java类。
3. Retrofit通过动态代理，用MethodHandler完成接口方法。
4. Retrofit的MethodHandler通过RequestFactoryParser.parse解析，获得接口方法的参数和注解的值，传入到OkHttpCall，OkHttpCall生成okhttp3.Call完成Http请求并使用Converter解析数据回调。
5. Retrofit通过工厂设置CallAdapter和Converter，CallAdapter包装转换Call，Converter转换（解析）服务器返回的数据、接口方法的注解参数。










