---
title: 'thread local'
date: 2020-02-11T19:30:08+10:00
weight: 3
summary: "Thread Local 实现方式"
---

# 多线程--thread local

## ThreadLocal 为什么可以每个线程保存一份自己的变量


```java

public class ThreadLocal<T> {
    /*...*/

    public T get() {
        Thread var1 = Thread.currentThread();
        ThreadLocal.ThreadLocalMap var2 = this.getMap(var1); // 获取thread的 localmaps 成员属性
        if (var2 != null) {
            ThreadLocal.ThreadLocalMap.Entry var3 = var2.getEntry(this);
            if (var3 != null) {
                Object var4 = var3.value;
                return var4;
            }
        }

        return this.setInitialValue();
    }

    private T setInitialValue() {
        Object var1 = this.initialValue();
        Thread var2 = Thread.currentThread();
        ThreadLocal.ThreadLocalMap var3 = this.getMap(var2);
        if (var3 != null) {
            var3.set(this, var1);
        } else {
            this.createMap(var2, var1);
        }

        return var1;
    }

    public void set(T var1) {
        Thread var2 = Thread.currentThread();
        ThreadLocal.ThreadLocalMap var3 = this.getMap(var2);
        if (var3 != null) {
            var3.set(this, var1);
        } else {
            this.createMap(var2, var1);
        }

    }

    /*...*/

    static class ThreadLocalMap {
        private static final int INITIAL_CAPACITY = 16;
        private ThreadLocal.ThreadLocalMap.Entry[] table;
        private int size;
        private int threshold;
        /*...*/

        private void set(ThreadLocal<?> var1, Object var2) {
            ThreadLocal.ThreadLocalMap.Entry[] var3 = this.table;
            int var4 = var3.length;
            int var5 = var1.threadLocalHashCode & var4 - 1;

            for(ThreadLocal.ThreadLocalMap.Entry var6 = var3[var5]; var6 != null; var6 = var3[var5 = nextIndex(var5, var4)]) {
                ThreadLocal var7 = (ThreadLocal)var6.get();
                if (var7 == var1) {
                    var6.value = var2;
                    return;
                }

                if (var7 == null) {
                    this.replaceStaleEntry(var1, var2, var5);
                    return;
                }
            }

            var3[var5] = new ThreadLocal.ThreadLocalMap.Entry(var1, var2);
            int var8 = ++this.size;
            if (!this.cleanSomeSlots(var5, var8) && var8 >= this.threshold) {
                this.rehash();
            }

        }


        static class Entry extends WeakReference<ThreadLocal<?>> {
            Object value;

            Entry(ThreadLocal<?> var1, Object var2) {
                super(var1);
                this.value = var2;
            }
        }
    }

  static final class SuppliedThreadLocal<T> extends ThreadLocal<T> {/*...*/}
}
```
