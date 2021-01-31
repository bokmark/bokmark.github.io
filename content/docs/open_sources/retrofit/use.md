---
title: 'Retrofit - 原理'
date: 2020-12-02
weight: 1
---

> Retrofit 的使用 可以看这里(/docs/open_sources/retrofit/use)

## 介绍

retrofit 将你的htppapi 转化成java接口的样子
```Java
public interface GitHubService {
  @GET("users/{user}/repos")
  Call<List<Repo>> listRepos(@Path("user") String user);
}
```

`Retrofit` 将会提供一个 `GitHubService` 的实现
```Java
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com/")
    .build();

GitHubService service = retrofit.create(GitHubService.class);
```

`GitHubService` 提供的每一个 `Call` 都将提供 同步或者异步的api请求去请求http
```Java
Call<List<Repo>> repos = service.listRepos("octocat");
```
我们使用注解来描述一个http请求

## api定义

在一个接口的方法上添加注解和参数上添加注解就已经足够描述一个接口将会被如何处理

#### REQUEST METHOD
Every method must have an HTTP annotation that provides the request method and relative URL. There are eight built-in annotations: HTTP, GET, POST, PUT, PATCH, DELETE, OPTIONS and HEAD. The relative URL of the resource is specified in the annotation.
```
@GET("users/list")
```
You can also specify query parameters in the URL.
```
@GET("users/list?sort=desc")
```

#### URL MANIPULATION
A request URL can be updated dynamically using replacement blocks and parameters on the method. A replacement block is an alphanumeric string surrounded by { and }. A corresponding parameter must be annotated with @Path using the same string.

```
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId);
```

Query parameters can also be added.


```
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId, @Query("sort") String sort);
```
For complex query parameter combinations a Map can be used.


```
@GET("group/{id}/users")
Call<List<User>> groupList(@Path("id") int groupId, @QueryMap Map<String, String> options);
```

#### REQUEST BODY
An object can be specified for use as an HTTP request body with the @Body annotation.
```
@POST("users/new")
Call<User> createUser(@Body User user);
```
The object will also be converted using a converter specified on the Retrofit instance. If no converter is added, only RequestBody can be used.

#### FORM ENCODED AND MULTIPART
Methods can also be declared to send form-encoded and multipart data.

Form-encoded data is sent when @FormUrlEncoded is present on the method. Each key-value pair is annotated with @Field containing the name and the object providing the value.
```
@FormUrlEncoded
@POST("user/edit")
Call<User> updateUser(@Field("first_name") String first, @Field("last_name") String last);
```
Multipart requests are used when @Multipart is present on the method. Parts are declared using the @Part annotation.
```
@Multipart
@PUT("user/photo")
Call<User> updateUser(@Part("photo") RequestBody photo, @Part("description") RequestBody description);
```
Multipart parts use one of Retrofit's converters or they can implement RequestBody to handle their own serialization.


#### HEADER MANIPULATION
You can set static headers for a method using the @Headers annotation.
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
Note that headers do not overwrite each other. All headers with the same name will be included in the request.

A request Header can be updated dynamically using the @Header annotation. A corresponding parameter must be provided to the @Header. If the value is null, the header will be omitted. Otherwise, toString will be called on the value, and the result used.


```
@GET("user")
Call<User> getUser(@Header("Authorization") String authorization)
```
Similar to query parameters, for complex header combinations, a Map can be used.


```
@GET("user")
Call<User> getUser(@HeaderMap Map<String, String> headers)
```
Headers that need to be added to every request can be specified using an OkHttp [interceptor](https://github.com/square/okhttp).

#### SYNCHRONOUS VS. ASYNCHRONOUS

Call instances can be executed either synchronously or asynchronously. Each instance can only be used once, but calling clone() will create a new instance that can be used.

On Android, callbacks will be executed on the main thread. On the JVM, callbacks will happen on the same thread that executed the HTTP request.

## Retrofit Configuration
Retrofit is the class through which your API interfaces are turned into callable objects. By default, Retrofit will give you sane defaults for your platform but it allows for customization.

#### CONVERTERS
By default, Retrofit can only deserialize HTTP bodies into OkHttp's ResponseBody type and it can only accept its RequestBody type for @Body.

Converters can be added to support other types. Six sibling modules adapt popular serialization libraries for your convenience.
- Gson: com.squareup.retrofit2:converter-gson
- Jackson: com.squareup.retrofit2:converter-jackson
- Moshi: com.squareup.retrofit2:converter-moshi
- Protobuf: com.squareup.retrofit2:converter-protobuf
- Wire: com.squareup.retrofit2:converter-wire
- Simple XML: com.squareup.retrofit2:converter-simplexml
- JAXB: com.squareup.retrofit2:converter-jaxb
- Scalars (primitives, boxed, and String): com.squareup.retrofit2:converter-scalars
Here's an example of using the GsonConverterFactory class to generate an implementation of the GitHubService interface which uses Gson for its deserialization.

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
