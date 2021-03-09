---
title: '从一个例子研究Binder'
date: 2018-11-28T15:14:39+10:00
weight: 2
summary: "Binder 面试问题"
---

# binder 内容很多，我们从一个非常简单的例子开始研究

## 我们的例子一

### 具体代码
一般创建一个binder 都是通过aidl实现，那么我们也用aidl来实现 [PagesDemo/BinderDemo 具体代码可以从这里下载](https://github.com/bokmark/PagesDemo#BinderDemo)
{{<details "IMyAidlInterface.aidl">}}
```AIDL
// IMyAidlInterface.aidl
package com.jaycema.binderdemo;

// Declare any non-default types here with import statements

interface IMyAidlInterface {
    /**
     * Demonstrates some basic types that you can use as parameters
     * and return values in AIDL.
     */
    String basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat,
            double aDouble, String aString);
}
```
{{</details>}}

{{<details "RemoteService.kt">}}
```kotlin
class RemoteService : Service() {

    val b = object : IMyAidlInterface.Stub() {
        override fun basicTypes(
            anInt: Int,
            aLong: Long,
            aBoolean: Boolean,
            aFloat: Float,
            aDouble: Double,
            aString: String?
        ): String {
            return "anInt$anInt, aLong$aLong, aBoolean$aBoolean, aFloat$aFloat, aDouble$aDouble, aString$aString"
        }

    }

    override fun onBind(intent: Intent): IBinder {
        return b
    }
}
```
{{</details>}}

{{<details "MainActivity.kt">}}
```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        val tv = findViewById<TextView>(R.id.tv)
        bindService(
            Intent(MainActivity@ this, RemoteService::class.java),
            object : ServiceConnection {
                override fun onServiceConnected(name: ComponentName?, service: IBinder?) {
                    Log.i("mainAct", "onServiceConnected name $name, service $service")
                    val mybinder = IMyAidlInterface.Stub.asInterface(service)
                    tv.setText(mybinder.basicTypes(1, 2, false, 0.1f, 2.0, "xxx"))
                }

                override fun onServiceDisconnected(name: ComponentName?) {
                    Log.i("mainAct", "onServiceDisconnected name $name")
                }

            },
            BIND_AUTO_CREATE
        )

    }
}
```
{{</details>}}

{{<details "AndroidMenifest.xml">}}
```XML
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.jaycema.binderdemo">

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.BinderDemo">

        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <service
            android:name=".RemoteService"
            android:process=":remote"
            android:enabled="true"
            android:exported="true"/>
    </application>

</manifest>
```
{{</details>}}
这是一个简单到不能再简单的例子，其中我们通过调用mybinder的basicTypes方法获取string显示到textview上。
但是跨进程真的这么简单嘛。我们逐步分析。

### 真正的binder代码

我们知道aidl文件最后将会编译成java代码编译进我们的apk中。那么我们就来找一下这个编译好的文件。
![寻找的路径](/images/blog_image/binder/aidl_java_path.jpg) 可以从这里找到编译好的文件

{{<details "IMyAidlInterface.java">}}
```JAVA
/*
 * This file is auto-generated.  DO NOT MODIFY.
 */
package com.jaycema.binderdemo;
// Declare any non-default types here with import statements

public interface IMyAidlInterface extends android.os.IInterface
{
  /** Default implementation for IMyAidlInterface. */
  public static class Default implements com.jaycema.binderdemo.IMyAidlInterface
  {
    /**
         * Demonstrates some basic types that you can use as parameters
         * and return values in AIDL.
         */
    @Override public java.lang.String basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, java.lang.String aString) throws android.os.RemoteException
    {
      return null;
    }
    @Override
    public android.os.IBinder asBinder() {
      return null;
    }
  }
  /** Local-side IPC implementation stub class. */
  public static abstract class Stub extends android.os.Binder implements com.jaycema.binderdemo.IMyAidlInterface
  {
    private static final java.lang.String DESCRIPTOR = "com.jaycema.binderdemo.IMyAidlInterface";
    /** Construct the stub at attach it to the interface. */
    public Stub()
    {
      this.attachInterface(this, DESCRIPTOR);
    }
    /**
     * Cast an IBinder object into an com.jaycema.binderdemo.IMyAidlInterface interface,
     * generating a proxy if needed.
     */
    public static com.jaycema.binderdemo.IMyAidlInterface asInterface(android.os.IBinder obj)
    {
      if ((obj==null)) {
        return null;
      }
      android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
      if (((iin!=null)&&(iin instanceof com.jaycema.binderdemo.IMyAidlInterface))) {
        return ((com.jaycema.binderdemo.IMyAidlInterface)iin);
      }
      return new com.jaycema.binderdemo.IMyAidlInterface.Stub.Proxy(obj);
    }
    @Override public android.os.IBinder asBinder()
    {
      return this;
    }
    @Override public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException
    {
      java.lang.String descriptor = DESCRIPTOR;
      switch (code)
      {
        case INTERFACE_TRANSACTION:
        {
          reply.writeString(descriptor);
          return true;
        }
        case TRANSACTION_basicTypes:
        {
          data.enforceInterface(descriptor);
          int _arg0;
          _arg0 = data.readInt();
          long _arg1;
          _arg1 = data.readLong();
          boolean _arg2;
          _arg2 = (0!=data.readInt());
          float _arg3;
          _arg3 = data.readFloat();
          double _arg4;
          _arg4 = data.readDouble();
          java.lang.String _arg5;
          _arg5 = data.readString();
          java.lang.String _result = this.basicTypes(_arg0, _arg1, _arg2, _arg3, _arg4, _arg5);
          reply.writeNoException();
          reply.writeString(_result);
          return true;
        }
        default:
        {
          return super.onTransact(code, data, reply, flags);
        }
      }
    }
    private static class Proxy implements com.jaycema.binderdemo.IMyAidlInterface
    {
      private android.os.IBinder mRemote;
      Proxy(android.os.IBinder remote)
      {
        mRemote = remote;
      }
      @Override public android.os.IBinder asBinder()
      {
        return mRemote;
      }
      public java.lang.String getInterfaceDescriptor()
      {
        return DESCRIPTOR;
      }
      /**
           * Demonstrates some basic types that you can use as parameters
           * and return values in AIDL.
           */
      @Override public java.lang.String basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, java.lang.String aString) throws android.os.RemoteException
      {
        android.os.Parcel _data = android.os.Parcel.obtain();
        android.os.Parcel _reply = android.os.Parcel.obtain();
        java.lang.String _result;
        try {
          _data.writeInterfaceToken(DESCRIPTOR);
          _data.writeInt(anInt);
          _data.writeLong(aLong);
          _data.writeInt(((aBoolean)?(1):(0)));
          _data.writeFloat(aFloat);
          _data.writeDouble(aDouble);
          _data.writeString(aString);
          boolean _status = mRemote.transact(Stub.TRANSACTION_basicTypes, _data, _reply, 0);
          if (!_status && getDefaultImpl() != null) {
            return getDefaultImpl().basicTypes(anInt, aLong, aBoolean, aFloat, aDouble, aString);
          }
          _reply.readException();
          _result = _reply.readString();
        }
        finally {
          _reply.recycle();
          _data.recycle();
        }
        return _result;
      }
      public static com.jaycema.binderdemo.IMyAidlInterface sDefaultImpl;
    }
    static final int TRANSACTION_basicTypes = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
    public static boolean setDefaultImpl(com.jaycema.binderdemo.IMyAidlInterface impl) {
      // Only one user of this interface can use this function
      // at a time. This is a heuristic to detect if two different
      // users in the same process use this function.
      if (Stub.Proxy.sDefaultImpl != null) {
        throw new IllegalStateException("setDefaultImpl() called twice");
      }
      if (impl != null) {
        Stub.Proxy.sDefaultImpl = impl;
        return true;
      }
      return false;
    }
    public static com.jaycema.binderdemo.IMyAidlInterface getDefaultImpl() {
      return Stub.Proxy.sDefaultImpl;
    }
  }
  /**
       * Demonstrates some basic types that you can use as parameters
       * and return values in AIDL.
       */
  public java.lang.String basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, java.lang.String aString) throws android.os.RemoteException;
}
```
{{</details>}}

这其中最关键的就是**Stub**

```java
public static abstract class Stub extends android.os.Binder implements com.jaycema.binderdemo.IMyAidlInterface
```  

Stub 类继承了[android.os.Binder](http://androidxref.com/6.0.0_r1/xref/frameworks/base/core/java/android/os/Binder.java)。还实现了一个proxy。

#### 我们先来分析proxy。
Proxy类是我们客户端的代理类，负责包装了客户端和服务端的通信。

{{<details "Stub.Proxy">}}
```java
private static class Proxy implements com.jaycema.binderdemo.IMyAidlInterface
{
  private android.os.IBinder mRemote;
  Proxy(android.os.IBinder remote)
  {
    mRemote = remote;
  }
  @Override public android.os.IBinder asBinder()
  {
    return mRemote;
  }
  public java.lang.String getInterfaceDescriptor()
  {
    return DESCRIPTOR;
  }
  // proxy代理类，即客户端的代理。我们来看客户端实际上是怎么和服务器进程间通信的。
  @Override public java.lang.String basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, java.lang.String aString) throws android.os.RemoteException
  {
    android.os.Parcel _data = android.os.Parcel.obtain();
    android.os.Parcel _reply = android.os.Parcel.obtain();
    java.lang.String _result;
    try {
      _data.writeInterfaceToken(DESCRIPTOR);
      _data.writeInt(anInt);
      _data.writeLong(aLong);
      _data.writeInt(((aBoolean)?(1):(0)));
      _data.writeFloat(aFloat);
      _data.writeDouble(aDouble);
      _data.writeString(aString);
      boolean _status = mRemote.transact(Stub.TRANSACTION_basicTypes, _data, _reply, 0);
      if (!_status && getDefaultImpl() != null) {
        return getDefaultImpl().basicTypes(anInt, aLong, aBoolean, aFloat, aDouble, aString);
      }
      _reply.readException();
      _result = _reply.readString();
    }
    finally {
      _reply.recycle();
      _data.recycle();
    }
    return _result;
  }
  public static com.jaycema.binderdemo.IMyAidlInterface sDefaultImpl;
}
```
{{</details>}}
从这里可以看到我们是如何通信的：
- 1：创建一个请求数据的包裹和响应数据的包裹。
- 2：将请求数据写入包裹。
- 3：调用binder提供的transact将请求数据包裹和响应数据包裹和code 一起发送。
- 4：解析返回值。
- 5：如果返回false，返回Default的实现。
- 6：如果返回true，解析响应数据包裹的内容。
- 7：返回结果并回收包裹。

#### 看完Proxy，我们再来看Remote。
```kotlin
val mybinder = IMyAidlInterface.Stub.asInterface(service)
```
我们通过`Stub`的`asInterface` 方法来获取`proxy`，`proxy`的`remote`在创建的时候传入，那么我们看这个 `asInterface` 方法肯定可以看到remote怎么获取。
```JAVA
public static com.jaycema.binderdemo.IMyAidlInterface asInterface(android.os.IBinder obj)
{
  if ((obj==null)) {
    return null;
  }
  android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
  if (((iin!=null)&&(iin instanceof com.jaycema.binderdemo.IMyAidlInterface))) {
    return ((com.jaycema.binderdemo.IMyAidlInterface)iin);
  }
  return new com.jaycema.binderdemo.IMyAidlInterface.Stub.Proxy(obj);
}
```
> 我调试过 `queryLocalInterface` 这里返回的是null，我们这里先不管。

所以，remote其实就是
```
override fun onServiceConnected(name: ComponentName?, service: IBinder?)
```
中得到的service。
那么我们是如何获取到这个service的呢？
#### 哪来的remote？
看 remoteservice 的代码，我们可以猜测出这个`service` 应该就是`val b = object : IMyAidlInterface.Stub()` 。
我们发挥打破砂锅问到底的精神，从bindservice查起。
{{<details "bindService">}}
```JAVA
// ContextWrapper.java -------------------------
@Override
  public boolean bindService(Intent service, ServiceConnection conn,
          int flags) {
      return mBase.bindService(service, conn, flags);
  }
// mBase ContextImpl[http://androidxref.com/6.0.0_r1/xref/frameworks/base/core/java/android/app/ContextImpl.java] ------------------------------------------------
@Override
    public boolean bindService(Intent service, ServiceConnection conn,
            int flags) {
        warnIfCallingFromSystemProcess();
        return bindServiceCommon(service, conn, flags, Process.myUserHandle());
    }
    private boolean bindServiceCommon(Intent service, ServiceConnection conn, int flags,
              UserHandle user) {
            IServiceConnection sd;
            if (conn == null) {
                throw new IllegalArgumentException("connection is null");
            }
            if (mPackageInfo != null) {
                sd = mPackageInfo.getServiceDispatcher(conn, getOuterContext(),
                        mMainThread.getHandler(), flags);
            } else {
                throw new RuntimeException("Not supported in system context");
            }
            validateServiceIntent(service);
            try {
                IBinder token = getActivityToken();
                if (token == null && (flags&BIND_AUTO_CREATE) == 0 && mPackageInfo != null
                        && mPackageInfo.getApplicationInfo().targetSdkVersion
                        < android.os.Build.VERSION_CODES.ICE_CREAM_SANDWICH) {
                    flags |= BIND_WAIVE_PRIORITY;
                }
                service.prepareToLeaveProcess();
                int res = ActivityManagerNative.getDefault().bindService(
                    mMainThread.getApplicationThread(), getActivityToken(), service,
                    service.resolveTypeIfNeeded(getContentResolver()),
                    sd, flags, getOpPackageName(), user.getIdentifier());
                if (res < 0) {
                    throw new SecurityException(
                            "Not allowed to bind to service " + service);
                }
                return res != 0;
            } catch (RemoteException e) {
                throw new RuntimeException("Failure from system", e);
            }
        }
```
{{</details>}}
关键在这句
```java
int res = ActivityManagerNative.getDefault().bindService(
    mMainThread.getApplicationThread(), getActivityToken(), service,
    service.resolveTypeIfNeeded(getContentResolver()),
    sd, flags, getOpPackageName(), user.getIdentifier());
```

{{<details "IActivityManager">}}
ActivityManagerNative.getDefault()

```java
// ActivityManagerNative.java
    @UnsupportedAppUsage
    static public IActivityManager getDefault() {
        return ActivityManager.getService();
    }
// ActivityManager.java
    @UnsupportedAppUsage
    public static IActivityManager getService() {
        return IActivityManagerSingleton.get();
    }
    @UnsupportedAppUsage
    private static final Singleton<IActivityManager> IActivityManagerSingleton =
          new Singleton<IActivityManager>() {
              @Override
              protected IActivityManager create() {
                  final IBinder b = ServiceManager.getService(Context.ACTIVITY_SERVICE);
                  final IActivityManager am = IActivityManager.Stub.asInterface(b);
                  return am;
              }
          };

// ActivityManagerService.java
  public int bindService(IApplicationThread caller, IBinder token, Intent service,
          String resolvedType, IServiceConnection connection, int flags, String callingPackage,
          int userId) throws TransactionTooLargeException {
      enforceNotIsolatedCaller("bindService");

      // Refuse possible leaked file descriptors
      if (service != null && service.hasFileDescriptors() == true) {
          throw new IllegalArgumentException("File descriptors passed in Intent");
      }

      if (callingPackage == null) {
          throw new IllegalArgumentException("callingPackage cannot be null");
      }

      synchronized(this) {
          return mServices.bindServiceLocked(caller, token, service,
                  resolvedType, connection, flags, callingPackage, userId);
      }
  }
// ActiveServices.java

    int bindServiceLocked(IApplicationThread caller, IBinder token, Intent service,
            String resolvedType, IServiceConnection connection, int flags,
            String callingPackage, int userId) throws TransactionTooLargeException {
        if (DEBUG_SERVICE) Slog.v(TAG_SERVICE, "bindService: " + service
                + " type=" + resolvedType + " conn=" + connection.asBinder()
                + " flags=0x" + Integer.toHexString(flags));
        final ProcessRecord callerApp = mAm.getRecordForAppLocked(caller);
        if (callerApp == null) {
            throw new SecurityException(
                    "Unable to find app for caller " + caller
                    + " (pid=" + Binder.getCallingPid()
                    + ") when binding service " + service);
        }

        ActivityRecord activity = null;
        if (token != null) {
            activity = ActivityRecord.isInStackLocked(token);
            if (activity == null) {
                Slog.w(TAG, "Binding with unknown activity: " + token);
                return 0;
            }
        }

        int clientLabel = 0;
        PendingIntent clientIntent = null;

        if (callerApp.info.uid == Process.SYSTEM_UID) {
            // Hacky kind of thing -- allow system stuff to tell us
            // what they are, so we can report this elsewhere for
            // others to know why certain services are running.
            try {
                clientIntent = service.getParcelableExtra(Intent.EXTRA_CLIENT_INTENT);
            } catch (RuntimeException e) {
            }
            if (clientIntent != null) {
                clientLabel = service.getIntExtra(Intent.EXTRA_CLIENT_LABEL, 0);
                if (clientLabel != 0) {
                    // There are no useful extras in the intent, trash them.
                    // System code calling with this stuff just needs to know
                    // this will happen.
                    service = service.cloneFilter();
                }
            }
        }

        if ((flags&Context.BIND_TREAT_LIKE_ACTIVITY) != 0) {
            mAm.enforceCallingPermission(android.Manifest.permission.MANAGE_ACTIVITY_STACKS,
                    "BIND_TREAT_LIKE_ACTIVITY");
        }

        final boolean callerFg = callerApp.setSchedGroup != Process.THREAD_GROUP_BG_NONINTERACTIVE;

        ServiceLookupResult res =
            retrieveServiceLocked(service, resolvedType, callingPackage,
                    Binder.getCallingPid(), Binder.getCallingUid(), userId, true, callerFg);
        if (res == null) {
            return 0;
        }
        if (res.record == null) {
            return -1;
        }
        ServiceRecord s = res.record;

        final long origId = Binder.clearCallingIdentity();

        try {
            if (unscheduleServiceRestartLocked(s, callerApp.info.uid, false)) {
                if (DEBUG_SERVICE) Slog.v(TAG_SERVICE, "BIND SERVICE WHILE RESTART PENDING: "
                        + s);
            }

            if ((flags&Context.BIND_AUTO_CREATE) != 0) {
                s.lastActivity = SystemClock.uptimeMillis();
                if (!s.hasAutoCreateConnections()) {
                    // This is the first binding, let the tracker know.
                    ProcessStats.ServiceState stracker = s.getTracker();
                    if (stracker != null) {
                        stracker.setBound(true, mAm.mProcessStats.getMemFactorLocked(),
                                s.lastActivity);
                    }
                }
            }

            mAm.startAssociationLocked(callerApp.uid, callerApp.processName,
                    s.appInfo.uid, s.name, s.processName);

            AppBindRecord b = s.retrieveAppBindingLocked(service, callerApp);
            ConnectionRecord c = new ConnectionRecord(b, activity,
                    connection, flags, clientLabel, clientIntent);

            IBinder binder = connection.asBinder();
            ArrayList<ConnectionRecord> clist = s.connections.get(binder);
            if (clist == null) {
                clist = new ArrayList<ConnectionRecord>();
                s.connections.put(binder, clist);
            }
            clist.add(c);
            b.connections.add(c);
            if (activity != null) {
                if (activity.connections == null) {
                    activity.connections = new HashSet<ConnectionRecord>();
                }
                activity.connections.add(c);
            }
            b.client.connections.add(c);
            if ((c.flags&Context.BIND_ABOVE_CLIENT) != 0) {
                b.client.hasAboveClient = true;
            }
            if (s.app != null) {
                updateServiceClientActivitiesLocked(s.app, c, true);
            }
            clist = mServiceConnections.get(binder);
            if (clist == null) {
                clist = new ArrayList<ConnectionRecord>();
                mServiceConnections.put(binder, clist);
            }
            clist.add(c);

            if ((flags&Context.BIND_AUTO_CREATE) != 0) {
                s.lastActivity = SystemClock.uptimeMillis();
                if (bringUpServiceLocked(s, service.getFlags(), callerFg, false) != null) {
                    return 0;
                }
            }

            if (s.app != null) {
                if ((flags&Context.BIND_TREAT_LIKE_ACTIVITY) != 0) {
                    s.app.treatLikeActivity = true;
                }
                // This could have made the service more important.
                mAm.updateLruProcessLocked(s.app, s.app.hasClientActivities
                        || s.app.treatLikeActivity, b.client);
                mAm.updateOomAdjLocked(s.app);
            }

            if (DEBUG_SERVICE) Slog.v(TAG_SERVICE, "Bind " + s + " with " + b
                    + ": received=" + b.intent.received
                    + " apps=" + b.intent.apps.size()
                    + " doRebind=" + b.intent.doRebind);

            if (s.app != null && b.intent.received) {
                // Service is already running, so we can immediately
                // publish the connection.
                try {
                    c.conn.connected(s.name, b.intent.binder);
                } catch (Exception e) {
                    Slog.w(TAG, "Failure sending service " + s.shortName
                            + " to connection " + c.conn.asBinder()
                            + " (in " + c.binding.client.processName + ")", e);
                }

                // If this is the first app connected back to this binding,
                // and the service had previously asked to be told when
                // rebound, then do so.
                if (b.intent.apps.size() == 1 && b.intent.doRebind) {
                    requestServiceBindingLocked(s, b.intent, callerFg, true);
                }
            } else if (!b.intent.requested) {
                requestServiceBindingLocked(s, b.intent, callerFg, false);
            }

            getServiceMap(s.userId).ensureNotStartingBackground(s);

        } finally {
            Binder.restoreCallingIdentity(origId);
        }

        return 1;
    }
```
{{</details>}}

















## 从栗子研究 binder 分层 以及底层逻辑








https://github.com/bokmark/PagesDemo
![binder](/images/blog_image/binder/binder_arch.webp)

- **Framework**
- **JNI**
- **Native**
- **Kernal**

### Kernal
- binder_init 注册binder驱动
- binder_open 打开binder驱动
- binder_ioctl 读写
- binder_mmap 映射物理内存


### Native
- ServiceManager

### JNI

### Framework
