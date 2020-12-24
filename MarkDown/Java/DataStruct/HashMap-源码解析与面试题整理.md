> 源码解析不会通篇解析，会通过面试题整理来针对性地解析其中的关键点

----
主要的几个面试题：
1. HashMap的底层原理是什么？线程安全么？ 
2. HashMap中put是如何实现的？ 
3. 谈一下hashMap中什么时候需要进行扩容，扩容resize()又是如何实现的？
4. 什么是哈希碰撞？怎么解决? 
5. HashMap和HashTable的区别 
6. HashMap中什么时候需要进行扩容，扩容resize()是如何实现的？ 
7. hashmap concurrenthashmap原理 
8. arraylist和hashmap的区别，为什么取数快？
---
#### 1. HashMap的底层原理是什么？线程安全么？ 
HashMap的底层原理是链表的数组， 通过hash算法计算出 key的hash值再根据数组长度取模得到index 如果index相同，那么就排在同一个链表中。链表中根据key的hash值大小有序排列方便查找![内部结构](https://upload-images.jianshu.io/upload_images/12975041-ccf59eb9c4be4b51.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> jdk 1.8 之后 androidsdk 24 之后 hashmap 的内部实现有链表改为 链表加红黑树实现，当一个链表的长度大于 8之后将会在后续添加的时候转变为红黑树
![红黑树](https://upload-images.jianshu.io/upload_images/12975041-ba926ee54c9b775a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- get： 先通过hash算法计算出key的hash值 然后再通过hash根据 链表数组的长度取模 获得index。根据index去循环链表或者红黑树取到相应的value。
- put：  先通过hash算法计算出key的hash值 然后再通过hash根据 链表数组的长度取模 获得index，再根据排列顺序将 entry放到链表或者红黑树中。

线程不安全，当多个线程操作的时候可能使读取的数据为错误数据。

#### 2. HashMap中put是如何实现的？ 
> 简单过程：先通过hash算法计算出key的hash值 然后再通过hash根据 链表数组的长度取模 获得index，再根据排列顺序将 entry放到链表或者红黑树中。
看代码：
```java
 public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length; 
// Step 1 获取hashmap中链表数组的长度（其中包含了一个重要的步骤 resize 之后在沟通）
        if ((p = tab[i = (n - 1) & hash]) == null)
// 此处 p指向了tab数值中对应index元素链表的头结点
            tab[i] = newNode(hash, key, value, null);
// Step 2 对key值的hash对 长度取模 （ (n - 1) & hash 等价于 hash % n ，此处采用位运算是为了性能考虑）得到i。
//             再检查数组i元素是否存在，如果不存在将node<k,v> 赋值
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
// Step 3 如果key相同 用e标记，指向p，后面将会把新的value 赋值给p节点
                e = p;
            else if (p instanceof TreeNode)
// 如果链表开始就是 树结构的 那么用树结构的逻辑将节点保存
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```
