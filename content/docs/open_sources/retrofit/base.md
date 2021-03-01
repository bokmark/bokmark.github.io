---
title: 'Retrofit - 大致流程源码解析'
date: 2020-12-02
weight: 1
---
# 通过实际例子来分析阅读源码

> Retrofit 的使用 可以看[这里](/docs/open_sources/retrofit/use)

## 栗子
我们先举一个例子。这是我们定义的

```Java
public interface GitHubService {
  @GET("users/{user}/repos")
  Call<List<Repo>> listRepos(@Path("user") String user);
}
```
这是我们常用的一个配置，添加了gson的转化器，和Rxjava的calladapter。


`Retrofit` 将会提供一个 `GitHubService` 的实现
```Java
Retrofit retrofit = new Retrofit.Builder()
    .addConverterFactory(GsonConverterFactory.create())
    .addCallAdapterFactory(RxJavaCallAdapterFactory.creat())
    .baseUrl("https://api.github.com/")
    .build();

GitHubService service = retrofit.create(GitHubService.class);
```

`GitHubService` 提供的每一个 `Call` 都将提供 同步或者异步的api请求去请求http
```Java
Call<List<Repo>> repos = service.listRepos("octocat");
```

> 到这里我就会疑惑retrofit自动生成的实例是什么样子的。我们点进去查看retrofit的create方法看一下

## 源码解析第一步-获取call
### create()

 `public <T> T create(final Class<T> service)`
```java
public <T> T create(final Class<T> service) {
    validateServiceInterface(service);
    return (T)
        Proxy.newProxyInstance(
            service.getClassLoader(),
            new Class<?>[] {service},
            new InvocationHandler() {
              private final Platform platform = Platform.get();
              private final Object[] emptyArgs = new Object[0];

              @Override
              public @Nullable Object invoke(Object proxy, Method method, @Nullable Object[] args)
                  throws Throwable {
                // If the method is a method from Object then defer to normal invocation.
                if (method.getDeclaringClass() == Object.class) {
                  return method.invoke(this, args);
                }
                args = args != null ? args : emptyArgs;
                return platform.isDefaultMethod(method)
                    ? platform.invokeDefaultMethod(method, service, proxy, args)
                    : loadServiceMethod(method).invoke(args);
              }
            });
  }
```
很明显这是一个动态代理。具体可以看[这里](/docs/java/other/object_proxy)
所谓动态代理就是指方法被调用的时候都会回调到`InvocationHandler`方法里面。

这段代码中的platform我们之后[再聊](/docs/open_sources/retrofit/platform)，先来看loadServiceMethod方法。

### ServiceMethod -> loadServiceMethod
```Java
ServiceMethod<?> loadServiceMethod(Method method) {
  // 做缓存，避免多次创建ServiceMethod，造成不必要的浪费。
  ServiceMethod<?> result = serviceMethodCache.get(method);
  if (result != null) return result;

  synchronized (serviceMethodCache) {
    result = serviceMethodCache.get(method);
    if (result == null) {
      // 别的代码都是 缓存代码，关键点在这里
      result = ServiceMethod.parseAnnotations(this, method);
      serviceMethodCache.put(method, result);
    }
  }
  return result;
}
```
这是一个DCL的缓存形式，所以，我们需要关注的是 `ServiceMethod.parseAnnotations`里面内容。

### ServiceMethod -> parseAnnotations
```java
abstract class ServiceMethod<T> {
  static <T> ServiceMethod<T> parseAnnotations(Retrofit retrofit, Method method) {
    RequestFactory requestFactory = RequestFactory.parseAnnotations(retrofit, method);

    Type returnType = method.getGenericReturnType();
    if (Utils.hasUnresolvableType(returnType)) {
      throw methodError(method,
          "Method return type must not include a type variable or wildcard: %s", returnType);
    }
    if (returnType == void.class) {
      throw methodError(method, "Service methods cannot return void.");
    }

    return HttpServiceMethod.parseAnnotations(retrofit, method, requestFactory);
  }

  abstract @Nullable T invoke(Object[] args);
}

```
这是一个虚类的定义，而 HttpServiceMethod 是他的一个实现类，我们先来看看HttpServiceMethod中parseAnnotations的实现

### HttpServiceMethod -> parseAnnotations

> 这里我们先不考虑kotlin中suspend 方法的实现逻辑，只考虑纯Java的实现

```java
  static <ResponseT, ReturnT> HttpServiceMethod<ResponseT, ReturnT> parseAnnotations(
      Retrofit retrofit, Method method, RequestFactory requestFactory) {
    boolean isKotlinSuspendFunction = requestFactory.isKotlinSuspendFunction;
    boolean continuationWantsResponse = false;
    boolean continuationBodyNullable = false;

    // 1.获取当前方法的所有注解
    Annotation[] annotations = method.getAnnotations();
    Type adapterType;
    if (isKotlinSuspendFunction) {
      /**/
    } else {
      adapterType = method.getGenericReturnType();
    }

    // 2.根据方法的返回值，创建一个一个call的适配器，其中包含了
    CallAdapter<ResponseT, ReturnT> callAdapter =
        createCallAdapter(retrofit, method, adapterType, annotations);
    Type responseType = callAdapter.responseType();
    if (responseType == okhttp3.Response.class) {
      throw methodError(method, "'"
          + getRawType(responseType).getName()
          + "' is not a valid response body type. Did you mean ResponseBody?");
    }
    if (responseType == Response.class) {
      throw methodError(method, "Response must include generic type (e.g., Response<String>)");
    }
    // TODO support Unit for Kotlin?
    if (requestFactory.httpMethod.equals("HEAD") && !Void.class.equals(responseType)) {
      throw methodError(method, "HEAD method must use Void as response type.");
    }

    // 3.创建一个返回值的转化器
    Converter<ResponseBody, ResponseT> responseConverter =
        createResponseConverter(retrofit, method, responseType);

    // 4.创建一个CallAdapted 的类
    okhttp3.Call.Factory callFactory = retrofit.callFactory;
    if (!isKotlinSuspendFunction) {
      return new CallAdapted<>(requestFactory, callFactory, responseConverter, callAdapter);
    /**/
  }
```

这里介绍了基础的注解的解析方式。大致分为以下几步
- 根据method的返回值创建calladapter，从这里我们看到retrofit在这里定义了两条规则
  - 不支持返回值是 单纯的 `okhttp3.Response` 可以是ResponseBody 也可以是`okhttp3.Response<T>`
  - 如果这个method的请求是HEAD的那么必须是void的返回值。*从这里的注解也可以看到这里不支持kotlin的Unit*

- 根据返回类型创建responseConverter，将返回值转化为需要的类型
- 获取callFactory,赋值地方在这里。其实就是传入了OkHttpClient
  ```java
  public Builder client(OkHttpClient client) {
    return callFactory(checkNotNull(client, "client == null"));
  }
  public Builder callFactory(okhttp3.Call.Factory factory) {
    this.callFactory = checkNotNull(factory, "factory == null");
    return this;
  }
  ```

- 创建一个内部实现类并返回，我们在不考虑kotin的情况下查看CallAdapted的实现

### CallAdapted
```java
static final class CallAdapted<ResponseT, ReturnT> extends HttpServiceMethod<ResponseT, ReturnT> {
  private final CallAdapter<ResponseT, ReturnT> callAdapter;

  CallAdapted(
      RequestFactory requestFactory,
      okhttp3.Call.Factory callFactory,
      Converter<ResponseBody, ResponseT> responseConverter,
      CallAdapter<ResponseT, ReturnT> callAdapter) {
    super(requestFactory, callFactory, responseConverter);
    this.callAdapter = callAdapter;
  }

  @Override
  protected ReturnT adapt(Call<ResponseT> call, Object[] args) {
    return callAdapter.adapt(call);
  }
}
```
这里很简单主要是调用基本所有的逻辑都是在HttpServiceMethod中实现的。

**回到第一步的`create`**
```Java
loadServiceMethod(method).invoke(args)
```
我们来看invoke的实现也就是`HttpServiceMethod`的`invoke`的实现

### HttpServiceMethod -> invoke

```Java
@Override
final @Nullable ReturnT invoke(Object[] args) {
  Call<ResponseT> call = new OkHttpCall<>(requestFactory, args, callFactory, responseConverter);
  return adapt(call, args);
}

protected abstract @Nullable ReturnT adapt(Call<ResponseT> call, Object[] args);
```
这理顺这里面的逻辑就是`CallAdapter.adapt(OkHttpCall)`。

CallAdapter由我们在创建retrofit的时候传入
```Java
Retrofit retrofit = new Retrofit.Builder()
    .addConverterFactory(GsonConverterFactory.create())
    .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
    .baseUrl("https://api.github.com/")
    .build();
```
当然我们也可以看下默认的实现`DefaultCallAdapterFactory`
```Java

final class DefaultCallAdapterFactory extends CallAdapter.Factory {
  private final @Nullable Executor callbackExecutor;

  DefaultCallAdapterFactory(@Nullable Executor callbackExecutor) {
    this.callbackExecutor = callbackExecutor;
  }

  @Override
  public @Nullable CallAdapter<?, ?> get(
      Type returnType, Annotation[] annotations, Retrofit retrofit) {
    if (getRawType(returnType) != Call.class) {
      return null;
    }
    if (!(returnType instanceof ParameterizedType)) {
      throw new IllegalArgumentException(
          "Call return type must be parameterized as Call<Foo> or Call<? extends Foo>");
    }
    final Type responseType = Utils.getParameterUpperBound(0, (ParameterizedType) returnType);

    final Executor executor =
        Utils.isAnnotationPresent(annotations, SkipCallbackExecutor.class)
            ? null
            : callbackExecutor;

    return new CallAdapter<Object, Call<?>>() {
      @Override
      public Type responseType() {
        return responseType;
      }

      @Override
      public Call<Object> adapt(Call<Object> call) {
        return executor == null ? call : new ExecutorCallbackCall<>(executor, call);
      }
    };
  }

  static final class ExecutorCallbackCall<T> implements Call<T> {

    /**/
  }
}
```
其中的重点是`public CallAdapter<?, ?> get(Type,Annotation[],Retrofit)`具体代码我们就不讲解了，主要就是做一些检查，然后返回一个能返回`ExecutorCallbackCall`的callAdapter

### ExecutorCallbackCall
```Java
static final class ExecutorCallbackCall<T> implements Call<T> {
    final Executor callbackExecutor;
    final Call<T> delegate;

    ExecutorCallbackCall(Executor callbackExecutor, Call<T> delegate) {
      this.callbackExecutor = callbackExecutor;
      this.delegate = delegate;
    }

    @Override
    public void enqueue(final Callback<T> callback) {
      Objects.requireNonNull(callback, "callback == null");

      delegate.enqueue(
          new Callback<T>() {
            @Override
            public void onResponse(Call<T> call, final Response<T> response) {
              callbackExecutor.execute(
                  () -> {
                    if (delegate.isCanceled()) {
                      // Emulate OkHttp's behavior of throwing/delivering an IOException on
                      // cancellation.
                      callback.onFailure(ExecutorCallbackCall.this, new IOException("Canceled"));
                    } else {
                      callback.onResponse(ExecutorCallbackCall.this, response);
                    }
                  });
            }

            @Override
            public void onFailure(Call<T> call, final Throwable t) {
              callbackExecutor.execute(() -> callback.onFailure(ExecutorCallbackCall.this, t));
            }
          });
    }

    /**/
    @SuppressWarnings("CloneDoesntCallSuperClone") // Performing deep clone.
    @Override
    public Call<T> clone() {
      return new ExecutorCallbackCall<>(callbackExecutor, delegate.clone());
    }
    /**/
  }

```
这里面的delegate 就是之前我们说的`okHttpCall`。从`enqueue`方法就能看出，其实这个CallAdapter的默认实现就是用线程池来执行`okHttpCall`的`enqueue`


## 第一步小结

### 流程图
返回call的流程

{{< mermaid >}}
sequenceDiagram
    Proxy->>ServiceMethod: loadServiceMethod()
    ServiceMethod->>ServiceMethod: parseAnnotations()
    ServiceMethod->>HttpServiceMethod: parseAnnotations()
    HttpServiceMethod->>+CallAdapter:createCallAdapter()
    CallAdapter-->>-HttpServiceMethod:createCallAdapter()
    HttpServiceMethod->>+Converter:createResponseConverter()
    Converter-->>-HttpServiceMethod:createResponseConverter()
    HttpServiceMethod->>CallAdapted:new CallAdapted()
    CallAdapted->>ServiceMethod:return
    CallAdapted->>ExecutorCallbackCall:enqueue
    ExecutorCallbackCall->>Proxy:delegate
{{</ mermaid >}}

### 描述
主要通过解析注解，将我们传入的各个adapter注入到okhttpcall中。

## 源码解析第二步 enqueue

### OkHttpCall -> enqueue
```java

@Override
public void enqueue(final Callback<T> callback) {
  Objects.requireNonNull(callback, "callback == null");

  okhttp3.Call call;
  Throwable failure;
  /**/

  call.enqueue(
      new okhttp3.Callback() {
        @Override
        public void onResponse(okhttp3.Call call, okhttp3.Response rawResponse) {
          Response<T> response;
          try {
            response = parseResponse(rawResponse);
          } catch (Throwable e) {
            throwIfFatal(e);
            callFailure(e);
            return;
          }

          try {
            callback.onResponse(OkHttpCall.this, response);
          } catch (Throwable t) {
            throwIfFatal(t);
            t.printStackTrace(); // TODO this is not great
          }
        }

        @Override
        public void onFailure(okhttp3.Call call, IOException e) {
          callFailure(e);
        }

        private void callFailure(Throwable e) {
          try {
            callback.onFailure(OkHttpCall.this, e);
          } catch (Throwable t) {
            throwIfFatal(t);
            t.printStackTrace(); // TODO this is not great
          }
        }
      });
}
```
这里的重点当然是`response = parseResponse(rawResponse);`
rawResponse:okHttpCall传入一个request创建出一个rawresponse。
你可以认为下面的方法中rawResponse 里面只有request相关内容以及一个经过请求之后返回的内容

### OkHttpCall -> parseResponse

```Java
Response<T> parseResponse(okhttp3.Response rawResponse) throws IOException {
  ResponseBody rawBody = rawResponse.body();

  // Remove the body's source (the only stateful object) so we can pass the response along.
  rawResponse =
      rawResponse
          .newBuilder()
          .body(new NoContentResponseBody(rawBody.contentType(), rawBody.contentLength()))
          .build();

  int code = rawResponse.code();
  if (code < 200 || code >= 300) {
    try {
      // Buffer the entire body to avoid future I/O.
      ResponseBody bufferedBody = Utils.buffer(rawBody);
      return Response.error(bufferedBody, rawResponse);
    } finally {
      rawBody.close();
    }
  }

  if (code == 204 || code == 205) {
    rawBody.close();
    return Response.success(null, rawResponse);
  }

  ExceptionCatchingResponseBody catchingBody = new ExceptionCatchingResponseBody(rawBody);
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
总结一下
- 获取contenttype和长度创建responsebody
- 根据httpcode的值进行处理
- 如果code为200则使用responseConverter对结果进行转化并返回

### responseConverter -> convert
responseConverter 由创建okhttpcall的时候传入。
看这里就是传入`Converter`的地方

```Java
//retrofit -> Build
public Retrofit build() {
  /**/

  // Make a defensive copy of the converters.
  List<Converter.Factory> converterFactories =
      new ArrayList<>(
          1 + this.converterFactories.size() + platform.defaultConverterFactoriesSize());

  converterFactories.add(new BuiltInConverters());
  converterFactories.addAll(this.converterFactories);
  //根据是否用java8编译添加OptionalConverterFactory
  converterFactories.addAll(platform.defaultConverterFactories());

  return new Retrofit(
      callFactory,
      baseUrl,
      unmodifiableList(converterFactories),
      unmodifiableList(callAdapterFactories),
      callbackExecutor,
      validateEagerly);
}
```
如果是json的返回值我们一般添加`GsonConverterFactory`

### GsonConverterFactory
```Java
public final class GsonConverterFactory extends Converter.Factory {
  /**
   * Create an instance using a default {@link Gson} instance for conversion. Encoding to JSON and
   * decoding from JSON (when no charset is specified by a header) will use UTF-8.
   */
  public static GsonConverterFactory create() {
    return create(new Gson());
  }

  /**
   * Create an instance using {@code gson} for conversion. Encoding to JSON and decoding from JSON
   * (when no charset is specified by a header) will use UTF-8.
   */
  @SuppressWarnings("ConstantConditions") // Guarding public API nullability.
  public static GsonConverterFactory create(Gson gson) {
    if (gson == null) throw new NullPointerException("gson == null");
    return new GsonConverterFactory(gson);
  }

  private final Gson gson;

  private GsonConverterFactory(Gson gson) {
    this.gson = gson;
  }

  @Override
  public Converter<ResponseBody, ?> responseBodyConverter(
      Type type, Annotation[] annotations, Retrofit retrofit) {
    TypeAdapter<?> adapter = gson.getAdapter(TypeToken.get(type));
    return new GsonResponseBodyConverter<>(gson, adapter);
  }

  @Override
  public Converter<?, RequestBody> requestBodyConverter(
      Type type,
      Annotation[] parameterAnnotations,
      Annotation[] methodAnnotations,
      Retrofit retrofit) {
    TypeAdapter<?> adapter = gson.getAdapter(TypeToken.get(type));
    return new GsonRequestBodyConverter<>(gson, adapter);
  }
}
```
我们前面提到的`responseConverter.convert(catchingBody);` 就是 `GsonResponseBodyConverter`
#### GsonResponseBodyConverter
```java

final class GsonResponseBodyConverter<T> implements Converter<ResponseBody, T> {
  private final Gson gson;
  private final TypeAdapter<T> adapter;

  GsonResponseBodyConverter(Gson gson, TypeAdapter<T> adapter) {
    this.gson = gson;
    this.adapter = adapter;
  }

  @Override
  public T convert(ResponseBody value) throws IOException {
    JsonReader jsonReader = gson.newJsonReader(value.charStream());
    try {
      T result = adapter.read(jsonReader);
      if (jsonReader.peek() != JsonToken.END_DOCUMENT) {
        throw new JsonIOException("JSON document was not fully consumed.");
      }
      return result;
    } finally {
      value.close();
    }
  }
}
```
这里就是soeasy 利用了gson的解析功能呢。
