---
author: Bokmark Ma
title: "Android JNI \ Ndk"
date: 2022-10-26
slug: fwk/jni
description: 什么是jni、native开发 
categories:
    - Android
tags:
    - Framework
    - Android
---

# jni、ndk？什么是native开发

- jni（Java Native Interface）。
什么是jni? jni是java虚拟机提供的用于java与c\c++ 相互操作的接口。  
这是Java的语言特性和Android没有关系，java后端服务器开发也能用jni。

- ndk （Native Development Kit）
ndk是由Android提供的用于native开发的一套工具集合。  

- [Android：JNI 与 NDK到底是什么？（含实例教学）](https://cloud.tencent.com/developer/article/1394293)

- 交叉编译：在一个平台下编译出在另一个平台中可以执行的二进制代码。ndk提供给了相关功能。


## JavaVM 和 JNIEnv
JavaVM 就是java的虚拟机在jni中的实例，一个jvm中只有一个JavaVM对象。JavaVM可以在各线程中 **共享**。

### JavaVM的前世今生
[AndroidRuntime.cpp](http://aospxref.com/android-12.0.0_r3/xref/frameworks/base/core/jni/AndroidRuntime.cpp)
```c++
// frameworks/base/cmds/app_process/app_main.cpp#runtime
// init进程
int main(int argc, char* const argv[]) {
    if (zygote) {
        runtime.start("com.android.internal.os.ZygoteInit", args, zygote);
    } else if (className) {
        runtime.start("com.android.internal.os.RuntimeInit", args, zygote);
    } else { 
    }
}
// frameworks/base/core/jni/AndroidRuntime.cpp
void AndroidRuntime::start(const char* className, const Vector<String8>& options, bool zygote) {
    /* start the virtual machine */
    JniInvocation jni_invocation;
    // 创建JavaVM所需要的 三个jni方法进行注册，后面要用
//    jint (*JNI_GetDefaultJavaVMInitArgs)(void*);
//    jint (*JNI_CreateJavaVM)(JavaVM**, JNIEnv**, void*);
//    jint (*JNI_GetCreatedJavaVMs)(JavaVM**, jsize, jsize*);
    jni_invocation.Init(NULL);


    JNIEnv* env;
    // 启动javavm
    if (startVm(&mJavaVM, &env, zygote, primary_zygote) != 0) {
        return;
    }
    // 回调vm创建完成，这里是空实现。
    onVmCreated(env);

    /*
     * 注册一大堆jni方法，
        static const RegJNIRec gRegJNI[] = {
            REG_JNI(register_android_util_Log),
            ...
            REG_JNI(register_android_app_Activity),
            REG_JNI(register_android_app_ActivityThread),
        }
        比如我们比较熟悉的几个方法
     */
    if (startReg(env) < 0) {
        ALOGE("Unable to register all android natives\n");
        return;
    }
}
// 创建art版本的javavm 
int AndroidRuntime::startVm(JavaVM** pJavaVM, JNIEnv** pEnv, bool zygote, bool primary_zygote) {
    if (JNI_CreateJavaVM(pJavaVM, pEnv, &initArgs) < 0) {
        ALOGE("JNI_CreateJavaVM failed\n");
        return -1;
    }
}
```
从这里我们就可以看到在这里启动了一个javavm。  
这个vm由于fork机制和其他进程中的vm就是同一个。

#### 在我们的jni开发过程中，如何获取JavaVM
```c++
// 第一种方法：
// 在加载so库时，会调用本方法，将这里的vm保存下来。
jint JNI_OnLoad(JavaVM * vm, void * reserved){
	JNIEnv * env = NULL;
	jint result = -1;

	if (vm->GetEnv((void**) &env, JNI_VERSION_1_4) != JNI_OK) {
	    LOGE(TAG, "ERROR: GetEnv failed\n");
	    goto bail;
	}
	result = JNI_VERSION_1_4;
	bail:
	return result; 
}
// 第二种方法：
// 通过JNIEnv获取。
JNIEXPORT void JNICALL Java_com_xxx(JNIEnv * env, jobject object)
{ 
	env->GetJavaVM(&gJavaVM); 
}
```

- [jni.h](https://android.googlesource.com/platform/libnativehelper/+/brillo-m9-dev/include/nativehelper/jni.h)


### JNIEnv（Java Native Interface Environment）
Java 本地接口环境，说人话就是 访问当前线程的一个接口环境，可以访问到当前线程的那些数据。  
理解JNIEnv 我们先看一段代码:

```Kotlin
// 这是kotlin代码
class NativeLib {
    var index: Int = 0
  
    external fun stringFromJNI(): Int
 
}
fun onCreate() {
    val nativeLib = NativeLib()
    for (i in 0..10) {
        Executors.newCachedThreadPool().submit {
            nativeLib.index = i
            Log.i("TAG-${Thread.currentThread().id}", nativeLib.stringFromJNI().toString())
        }
    }
}
```

```c++
// 这是native代码
extern "C" JNIEXPORT jint JNICALL
Java_io_bokmark_nativedemo_NativeLib_stringFromJNI(
        JNIEnv *env,
        jobject thisObj) {
    jclass clazz = env->GetObjectClass(thisObj);
    jfieldID fid = env->GetFieldID(clazz, "index", "I");
    jint count = env->GetIntField(thisObj, fid);
    return count;
}

```
```
// 这是结果
2023-03-15 22:04:55.570 17048-17079 TAG-253           0
2023-03-15 22:04:55.570 17048-17080 TAG-254           1
2023-03-15 22:04:55.570 17048-17081 TAG-255           2
2023-03-15 22:04:55.583 17048-17083 TAG-257           3
2023-03-15 22:04:55.584 17048-17082 TAG-256           3
2023-03-15 22:04:55.585 17048-17084 TAG-258           5
2023-03-15 22:04:55.587 17048-17089 TAG-263           8
2023-03-15 22:04:55.588 17048-17087 TAG-261           6
2023-03-15 22:04:55.588 17048-17085 TAG-259           6
2023-03-15 22:04:55.588 17048-17086 TAG-260           7
2023-03-15 22:04:55.589 17048-17088 TAG-262           9
```
看完有啥感触没。

Jni就是当前线程的一个环境，从这个环境中获取到的就是当前线程能获取到的内容。

。。。 未完待续


## 阅读资料

- [jni ndk](https://blog.csdn.net/tkwxty/article/details/103454842)
