>[和Jayce一起学习Jetpack -- 开篇](https://www.jianshu.com/p/0912bd8bb82f)
>[Google 官方解释](https://developer.android.google.cn/jetpack/compose/)

## 第一步 Jetpack Compose 是干嘛用的？
做过启动优化的同学一定会发现`LayoutInflater.inflate`实在是太忙了，可是优化xml却不会有太多的提升，真的是搞得人头疼。这时候就要分析 `LayoutInflater.inflate` 里面到底做了什么？研究一下吧。
```
    public View inflate(@LayoutRes int resource, @Nullable ViewGroup root, boolean attachToRoot) {
        final Resources res = getContext().getResources();
        if (DEBUG) {
            Log.d(TAG, "INFLATING from resource: \"" + res.getResourceName(resource) + "\" ("
                  + Integer.toHexString(resource) + ")");
        }

        View view = tryInflatePrecompiled(resource, res, root, attachToRoot);         // 1
        if (view != null) {
            return view;
        }
        XmlResourceParser parser = res.getLayout(resource);                           // 2
        try {
            return inflate(parser, root, attachToRoot);                               // 3
        } finally {
            parser.close();
        }
```
- 1 tryInflatePrecompiled 这是另外一个故事我们之后再说。
- 2 这里的代码也不细贴了主要就是io读取文件内容。 这是一个大耗时。
- 3 关键的调用是在这里，我们深入再看。
```

    public View inflate(XmlPullParser parser, @Nullable ViewGroup root, boolean attachToRoot) {
        synchronized (mConstructorArgs) {
            Trace.traceBegin(Trace.TRACE_TAG_VIEW, "inflate");
            //...
            try {
                if (TAG_MERGE.equals(name)) {
                    //  ...
                    rInflate(parser, root, inflaterContext, attrs, false);                            // 1
                } else {
                    // Temp is the root view that was found in the xml
                    final View temp = createViewFromTag(root, name, inflaterContext, attrs);         // 2
                    // ...

                    // Inflate all children under temp against its context.
                    rInflateChildren(parser, temp, attrs, true);                                     // 3

                    if (DEBUG) {
                        System.out.println("-----> done inflating children");
                    }

                    // We are supposed to attach all the views we found (int temp)
                    // to root. Do that now.
                    if (root != null && attachToRoot) {
                        root.addView(temp, params);                                                  // 4
                    }

      
                }

            } catch (XmlPullParserException e) {
                  // ... 
            }
            return result;
        }
    }
```
- 1 这里的代码就不贴了 主要是处理 merge的代码
- 2 根据当前的属性创建一个view
- 3 递归创建子view
- 4 递归结束是否attach

#####  `LayoutInflater.inflate` 做了什么我们现在大致也知道了，第一是读取io文件内容，第二是递归创建view组件，构建内存中的view树。这两步都是大耗时，而且看起来不可避免，因为你必然要用xml来定义界面，对吧。*但是，对嘛？不一定哦！Jetpack compose就是为了解决这个问题。*

## 第二步 Jetpack Compose 怎么用呢。
Jetpack Compose 是基于kotlin的，所以你的项目如果是java的。那么恭喜你，这是一个好时机将你的项目转化成kotlin，相信我，kotlin 真香！
##### 配置kotlin
```
plugins {
  id 'org.jetbrains.kotlin.android' version '1.4.0'
}
```
##### 配置gradle
```
android {
    defaultConfig {
        ...
        minSdkVersion 21
    }

    buildFeatures {
        // 这里这里这里这里这里这里这里这里这里这里
        compose true
    }
    ...

    // Set both the Java and Kotlin compilers to target Java 8.

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }

    kotlinOptions {
        jvmTarget = "1.8"
        // 这里这里这里这里这里这里这里这里这里这里
        useIR = true
    }

        // 这里这里这里这里这里这里这里这里这里这里
    composeOptions {
        kotlinCompilerVersion '1.4.0'
        kotlinCompilerExtensionVersion '1.0.0-alpha01'
    }
}
```
###### 配置依赖
```
dependencies {
    implementation 'androidx.compose.ui:ui:1.0.0-alpha01'
    // Tooling support (Previews, etc.)
    implementation 'androidx.ui:ui-tooling:1.0.0-alpha01'
    // Foundation (Border, Background, Box, Image, Scroll, shapes, animations, etc.)
    implementation 'androidx.compose.foundation:foundation:1.0.0-alpha01'
    // Material Design
    implementation 'androidx.compose.material:material:1.0.0-alpha01'
    // Material design icons
    implementation 'androidx.compose.material:material-icons-core:1.0.0-alpha01'
    implementation 'androidx.compose.material:material-icons-extended:1.0.0-alpha01'
    // Integration with observables
    implementation 'androidx.compose.runtime:runtime-livedata:1.0.0-alpha01'
    implementation 'androidx.compose.runtime:runtime-rxjava2:1.0.0-alpha01'

    // UI Tests
    androidTestImplementation 'androidx.ui:ui-test:1.0.0-alpha01'
}
```
