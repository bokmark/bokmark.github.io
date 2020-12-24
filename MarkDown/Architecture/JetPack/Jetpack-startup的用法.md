>  [G oogle 官方解释](https://developer.android.google.cn/topic/libraries/app-startup)

这是官方对于startup这个库的解释。

> App startup 库 提供了 一个直截了当、高性能的方式去初始化你的应用。第三方库和app的开发者都可以使用app startup这个库以数据流的方式去显示的安排启动的各个步骤。
>
> app startup这个库允许你只创建一个contentprovider 去初始化你的各个 initializer，这将有助与提升你的app启动时间。

从这段话中，我们可以看到几个重要的点。
- contentprovider
- 数据流的方式，显示的安排各个启动的步骤
###配置
配置在你的app或者library的moudle中的build.gradle 中
```gradle
dependencies {
    implementation "androidx.startup:startup-runtime:1.0.0-alpha02"
}
```
### 注册初始化组件
定义了一个默认的content provider，所有的 组件都由这个content provider的create 里面创建流程。
如果想要自动化的创建组件，需要在manifest中显示的写入你的组件。

### 组件的创建
实现 `Initializer<T>` 接口。该接口有如下两个方法：
- T create(Context)
返回创建好的实例
- dependencies()
创建该组件之前需要创建的其他组件。
---
以workmanager举例说明
```java
// Initializes WorkManager.
class WorkManagerInitializer implements Initializer<WorkManager> {

    @Override
    public WorkManager create(Context context) {
        Configuration configuration = Configuration.Builder().build();
        WorkManager.initialize(context, configuration);
        return WorkManager.getInstance(context);
    }

    @Override
    public List<Class<Initializer<?>>> dependencies() {
        // No dependencies on other libraries.
        return emptyList();
    }

}
```
> 这个例子中间，我们可以看到由于不需要依赖与别的组件，dependencies 直接返回一个空的list

### 有依赖的组件的创建
```java
// Initializes ExampleLogger.
class ExampleLoggerInitializer implements Initializer<ExampleLogger> {

    @Override
    public ExampleLogger create(Context context) {
        // WorkManager.getInstance() is non-null only after
        // WorkManager is initialized.
        return ExampleLogger(WorkManager.getInstance(context));
    }

    @Override
    public List<Class<Initializer<?>>> dependencies() {
        // Defines a dependency on WorkManagerInitializer so it can be
        // initialized after WorkManager is initialized.
        return Arrays.asList(WorkManagerInitializer.class);
    }
}
```
这里在`ExampleLogger` 的create期间，workmanager已经创建好了，在这里可以直接是使用。
### manifest的规范
startup 库依赖与InitializationProvider
```xml
<provider
    android:name="androidx.startup.InitializationProvider"
    android:authorities="${applicationId}.androidx-startup"
    android:exported="false"
    tools:node="merge">
    <!-- This entry makes ExampleLoggerInitializer discoverable. -->
    <meta-data  android:name="com.example.ExampleLoggerInitializer"
          android:value="androidx.startup" />
</provider>
```
再在provider的metadata中定义好自己的Initializer。就会自动注册组件。

### startup的另一种用法
首先在xml中这样定义
```xml
<provider
    android:name="androidx.startup.InitializationProvider"
    android:authorities="${applicationId}.androidx-startup"
    tools:node="remove" />
```
其次 调用这个方法，将会开始注册组件，而不会自动注册，同理，由于 `workmangerinitializer`是`ExampleLoggerInitializer` 的依赖，所以这里会自动的创建`workmangerinitializer`
```java
AppInitializer.getInstance(context)
    .initializeComponent(ExampleLoggerInitializer.class);
```
----
这次解析我们从 定义在 manifest中的 InitializationProvider 入手

```java
@RestrictTo(RestrictTo.Scope.LIBRARY)
public final class InitializationProvider extends ContentProvider {
    @Override
    public boolean onCreate() {
        Context context = getContext();
        if (context != null) {
            AppInitializer.getInstance(context).discoverAndInitialize();
        } else {
            throw new StartupException("Context cannot be null");
        }
        return true;
    }

    @Nullable
    @Override
    public Cursor query(
            @NonNull Uri uri,
            @Nullable String[] projection,
            @Nullable String selection,
            @Nullable String[] selectionArgs,
            @Nullable String sortOrder) {
        throw new IllegalStateException("Not allowed.");
    }

    @Nullable
    @Override
    public String getType(@NonNull Uri uri) {
        throw new IllegalStateException("Not allowed.");
    }

    @Nullable
    @Override
    public Uri insert(@NonNull Uri uri, @Nullable ContentValues values) {
        throw new IllegalStateException("Not allowed.");
    }

    @Override
    public int delete(
            @NonNull Uri uri,
            @Nullable String selection,
            @Nullable String[] selectionArgs) {
        throw new IllegalStateException("Not allowed.");
    }

    @Override
    public int update(
            @NonNull Uri uri,
            @Nullable ContentValues values,
            @Nullable String selection,
            @Nullable String[] selectionArgs) {
        throw new IllegalStateException("Not allowed.");
    }
}
```

这个provider 也很简单 关键的代码都在 AppInitializer.getInstance(context).discoverAndInitialize(); 这一句。

```java
    void discoverAndInitialize() {
        try {
            Trace.beginSection(SECTION_NAME);
            ComponentName provider = new ComponentName(mContext.getPackageName(),
                    InitializationProvider.class.getName());
            ProviderInfo providerInfo = mContext.getPackageManager()
                    .getProviderInfo(provider, GET_META_DATA);
            Bundle metadata = providerInfo.metaData;
            String startup = mContext.getString(R.string.androidx_startup);
            if (metadata != null) {
                Set<Class<?>> initializing = new HashSet<>();
                Set<String> keys = metadata.keySet();
                for (String key : keys) {
                    String value = metadata.getString(key, null);
                    if (startup.equals(value)) {
                        Class<?> clazz = Class.forName(key);
                        if (Initializer.class.isAssignableFrom(clazz)) {
                            Class<? extends Initializer<?>> component =
                                    (Class<? extends Initializer<?>>) clazz;
                            mDiscovered.add(component);
                            if (StartupLogger.DEBUG) {
                                StartupLogger.i(String.format("Discovered %s", key));
                            }
                            doInitialize(component, initializing);
                        }
                    }
                }
            }
        } catch (PackageManager.NameNotFoundException | ClassNotFoundException exception) {
            throw new StartupException(exception);
        } finally {
            Trace.endSection();
        }
    }
```

- 这个方法，首先第一步 获取定义在 provider 里面的 metadata。
- 第二步就是便利 metadata 获取所有value为 `R.string.androidx_startup` 的key值。
- 第三步 通过name 获取class
- 执行 `doInitialize(component, initializing);`

```java
<T> T doInitialize(
            @NonNull Class<? extends Initializer<?>> component,
            @NonNull Set<Class<?>> initializing) {
        synchronized (sLock) {
            boolean isTracingEnabled = Trace.isEnabled();
            try {
                if (isTracingEnabled) {
                    // Use the simpleName here because section names would get too big otherwise.
                    Trace.beginSection(component.getSimpleName());
                }
                if (initializing.contains(component)) {
                    String message = String.format(
                            "Cannot initialize %s. Cycle detected.", component.getName()
                    );
                    throw new IllegalStateException(message);
                }
                Object result;
                if (!mInitialized.containsKey(component)) {
                    initializing.add(component);
                    try {
                        Object instance = component.getDeclaredConstructor().newInstance();
                        Initializer<?> initializer = (Initializer<?>) instance;
                        List<Class<? extends Initializer<?>>> dependencies =
                                initializer.dependencies();

                        if (!dependencies.isEmpty()) {
                            for (Class<? extends Initializer<?>> clazz : dependencies) {
                                if (!mInitialized.containsKey(clazz)) {
                                    doInitialize(clazz, initializing);
                                }
                            }
                        }
                        if (StartupLogger.DEBUG) {
                            StartupLogger.i(String.format("Initializing %s", component.getName()));
                        }
                        result = initializer.create(mContext);
                        if (StartupLogger.DEBUG) {
                            StartupLogger.i(String.format("Initialized %s", component.getName()));
                        }
                        initializing.remove(component);
                        mInitialized.put(component, result);
                    } catch (Throwable throwable) {
                        throw new StartupException(throwable);
                    }
                } else {
                    result = mInitialized.get(component);
                }
                return (T) result;
            } finally {
                Trace.endSection();
            }
        }
    }
```

从这个方法中可以看到这里面的逻辑是

- 判断是否有依赖，如果有首先创建依赖，并且调用create
- 如果没有则创建当前的实例，并且调用create
- 判断去重，避免重复创建。

### 不自动加载组件的逻辑

```java
AppInitializer.getInstance(context)
    .initializeComponent(ExampleLoggerInitializer.class);

######

  public <T> T initializeComponent(@NonNull Class<? extends Initializer<T>> component) {
        return doInitialize(component, new HashSet<Class<?>>());
    }

```

这里的逻辑就是不自动冲xml中读取自动创建， 改为手动调用`doInitialize` ，内部逻辑是一样。
是不是很简单。

## 总结

结合content provider的特性，我们很容易可以知道 这些创建的组件调用u会在 application 的oncreatae之前调用，但是content provider的oncreate 也是在主线程调用，还是会占用主线程的资源。
综合来说，startup还是以解偶 启动流程，规范化启动流程为主。而独立模块化的启动组件，也方便各个击破地提高我们的启动速度。