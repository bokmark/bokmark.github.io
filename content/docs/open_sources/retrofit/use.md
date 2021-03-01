---
title: '如何使用'
date: 2020-12-02
weight: 1
---
# Retrofit 如何使用，官方文档部分翻译。
**A type-safe HTTP client for Android and Java**

[官方文档](https://square.github.io/retrofit/)

## 介绍

retrofit 可以通过简单的注释将复杂的http 请求转化成java接口的亚子。
比如下面的：
```Java
public interface GitHubService {
  @GET("users/{user}/repos")
  Call<List<Repo>> listRepos(@Path("user") String user);
}
```

`Retrofit`通过`create`方法将会自动生成一个`GitHubService`接口的实现
```Java
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com/")
    .build();

GitHubService service = retrofit.create(GitHubService.class);
```

被创建出来的`GitHubService` 提供的每一个 `Call` 都将提供同步或者异步的方式去向远端服务器发送http请求
```Java
Call<List<Repo>> repos = service.listRepos("octocat");
```
我们使用注解来描述一个http请求，包括以下部分
- url以及请求参数，支持替换
- 类体自动转化成 request body
- 支持多模块的 request body 和文件上传

## api定义

在一个接口的方法上添加注解和参数上添加注解就已经足够描述一个接口将会被如何处理

### 请求方式
每一个方法都有一个 *HTTP* 的注解，用来表明请求方式和相对请求路径。有这些*HTTP*的请求方式: HTTP, GET, POST, PUT, PATCH, DELETE, OPTIONS 和 HEAD. url是定义在注解中的。
```
@GET("users/list")
```
你也可以同样定义请求参数到url中
```
@GET("users/list?sort=desc")
```

### URL
Retrofit支持请求URL可以动态的替换部分内容。需要被替换的部分必须被 *{* 和 *}* 包裹住。同时替换的参数必须添加*@Path* 注解。比如这样：

```
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId);
```
当然其他还是和正常的一样，比如可以添加 query

```
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId, @Query("sort") String sort);
```
如果你的请求有复杂的请求参数，那么你可以使用*@QueryMap* 。

```
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId, @QueryMap Map<String, String> options);
```

### 请求体
允许通过 *@Body* 注解，来讲一个类转化为请求体
```
@POST("users/new")
Call<User> createUser(@Body User user);
```
这个类，将会通过在retrofit实例化的时候传入的转化器转化为请求体，如果没有设置转化器，那么只允许设置requestbody。

### 表单和多模块表单
retrofit 也支持表单的提交
表单数据可以通过*@FormUrlEncoded* 标签来定，通过 *@Field* 标记一个key-value的键值对。
```
@FormUrlEncoded
@POST("user/edit")
Call<User> updateUser(@Field("first_name") String first, @Field("last_name") String last);
```
Multipart 请求添加 *@Multipart* 注解。每个part 通过 *@Part* 注解来标记。
```
@Multipart
@PUT("user/photo")
Call<User> updateUser(@Part("photo") RequestBody photo, @Part("description") RequestBody description);
```



### 请求头
你可以通过*@Headers* 注解来设置静态的请求头。
```
@Headers("Cache-Control: max-age=640000")
@GET("widget/list")
Call<List<Widget>> widgetList();
```
```
@Headers({
    "Accept: application/vnd.github.v3.full+json",
    "User-Agent: Retrofit-Sample-App"
})
@GET("users/{username}")
Call<User> getUser(@Path("username") String username);
```
**注意** 请求头们不会相互覆盖，所有同名的请求头都将会包含在请求中。

一个请求头可以动态的通过*@Header* 注解进行更新，只需要添加*@Header* 注解的参数到方法中。如果这个参数为空，那么这个请求头将会被忽略。
```
@GET("user")
Call<User> getUser(@Header("Authorization") String authorization)
```

与请求参数map相同，你也可以有请求头map
```
@GET("user")
Call<User> getUser(@HeaderMap Map<String, String> headers)
```
请求头可以在指定特定的拦截器中添加到每个请求中。 [interceptor](https://github.com/square/okhttp).

### 同步 VS. 异步
返回值 Call 的实例可以被同步或者异步的执行。每一个实例只能被调用一次，但是可以通过clone方法创建一个新的实例。
在Android平台上，回调将会在主线程上执行，在JVM上，会调将会发生在发起请求的那个线程上。

## Retrofit 的配置
Retrofit是一个用于将你的api转化成可执行方法的实例的类。默认情况下，Retrofit 将会给你一个合理的配置，但是你可以自定义。

### 转化器
默认情况下，Retrofit 只会将http的body转化成OkHttp中的ResponseBody，当然也只接受RequestBody作为@Body 注解的类型。

这有一些受欢迎的转化器可供读者使用。
- **Gson**: com.squareup.retrofit2:converter-gson
- **Jackson**: com.squareup.retrofit2:converter-jackson
- **Moshi**: com.squareup.retrofit2:converter-moshi
- **Protobuf**: com.squareup.retrofit2:converter-protobuf
- **Wire**: com.squareup.retrofit2:converter-wire
- **Simple XML**: com.squareup.retrofit2:converter-simplexml
- **JAXB**: com.squareup.retrofit2:converter-jaxb
- **Scalars (primitives, boxed, and String)**: com.squareup.retrofit2:converter-scalars

这是一个使用gson的例子。
使用GsonConverterFactory 创建了一个GitHubService接口的实例，其中用了gson来做序列化的工具。

```
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com/")
    .addConverterFactory(GsonConverterFactory.create())
    .build();

GitHubService service = retrofit.create(GitHubService.class);
```

## 引用方式
```
implementation 'com.squareup.retrofit2:retrofit:(insert latest version)'
```

## 混淆文件
```
# Retrofit does reflection on generic parameters. InnerClasses is required to use Signature and
# EnclosingMethod is required to use InnerClasses.
-keepattributes Signature, InnerClasses, EnclosingMethod

# Retrofit does reflection on method and parameter annotations.
-keepattributes RuntimeVisibleAnnotations, RuntimeVisibleParameterAnnotations

# Retain service method parameters when optimizing.
-keepclassmembers,allowshrinking,allowobfuscation interface * {
    @retrofit2.http.* <methods>;
}

# Ignore annotation used for build tooling.
-dontwarn org.codehaus.mojo.animal_sniffer.IgnoreJRERequirement

# Ignore JSR 305 annotations for embedding nullability information.
-dontwarn javax.annotation.**

# Guarded by a NoClassDefFoundError try/catch and only used when on the classpath.
-dontwarn kotlin.Unit

# Top-level functions that can only be used by Kotlin.
-dontwarn retrofit2.KotlinExtensions
-dontwarn retrofit2.KotlinExtensions$*

# With R8 full mode, it sees no subtypes of Retrofit interfaces since they are created with a Proxy
# and replaces all potential values with null. Explicitly keeping the interfaces prevents this.
-if interface * { @retrofit2.http.* <methods>; }
-keep,allowobfuscation interface <1>
```
