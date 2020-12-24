##  java 虚拟机所管理的内存将会包括以下几个内存区域
- 方法区
- 虚拟机栈
- 本地方法栈
- 堆内存
- 程序计数器
##### 程序计数器
- 线程私有
- 通过改变程序计数器来控制 下一条需要执行的字节码指令
- 一个内核只会控制一个线程，每个线程都有一个程序计数器用来控制当前线程的逻辑
- 每个线程之间的程序计数器没有关联，独立存储，所有这一类内存被称为程序计数器内存
- 如果当前在java线程上执行，那么程序计数器记录的就是正在执行的虚拟机字节码指令地址
- 如果在执行native代码 那么程序计数器记录的就是空
- 程序计数器不会oom

##### java虚拟机栈
- 线程私有
- 虚拟机栈的生命周期与线程相同
- 每个方法在执行时都会创建一个栈帧（栈帧会在第八章讲到）（局部变量表，操作数栈，动态链接，方法出口）
- 一个方法调用就是一个栈帧入虚拟机栈的过程，一个方法退出就是一个栈帧退出虚拟机栈的过程
- 一个栈帧的局部变量表是编译期间就确定的，运行期间是不会变的
- 当线程请求的栈深度超过最大值时将会抛出stackoverflowerror
```java
public class ExampleUnitTest {
    
    @Test
    public void testStackOverflow() {
        testStack(1);
    }

    private void testStack(int i) {
        System.out.println(i);
        testStack(i + 1);
    }
}
// 9073
// java.lang.StackOverflowError
```
- java虚拟机栈申请不到足够的内存是，将会抛出 oom
```java
public class ExampleUnitTest {
    @Test
    public void testOOM() {
        testNewArray(0);
    }

    private void testNewArray(long last) {
        System.out.println(last);
        double[] doubles = new double[1024 * 1024 * 10];
        last += doubles.length;
        testNewArray(last);
    }
}
//513802240
//java.lang.OutOfMemoryError: Java heap space
```
##### 本地方法栈
- 线程私有
- 功能与java虚拟机栈十分相似
- 具体的实现由具体的虚拟机来实现，没有明确的强制规定
- 本地方法栈也会抛出 oom和stackoverflow
- hotspot 将本地方法栈与java虚拟机栈合二为一。

##### java堆
- 所有线程共享
- 此区域的目的就是存放对象实例
- gc的主要工作区域
- thread local allocation buffer 堆中可能划分出多个线程私有的分配缓冲区
- 还可以细分为 新生代（eden， from survivor， to survivor），老年代，
- 详细内容看第三章
- 将抛出 oom当堆内存不足以分配新的实例，或者无法扩展时

##### 方法区
- 所有现场共享的内存区域
- 记录加载到虚拟机的类信息，常量，静态变量，即时编译器编译后的代码等
- 不等同于永久代，在最新的规范中，方法区或将使用 native memory 来使用
- 也会发生内存泄漏
- 也会发生oom

##### 运行时常量池
- 能throw出oom
- 用于存放 在编译器就确定的那些 常量信息
- string intern 可以将string 在运行过程中存放到 常量次中 [参考这篇文章](https://blog.csdn.net/u011635492/article/details/81048150)
- 1.7之后将常量池放到了堆中，从方法区中移动出

##### 直接内存
- newio 通过使用channel等方式 在nativememory上开辟了一块直接内存，java可以通过 directbytebuffer 操作nativemomery 提高效率，避免了重复拷贝
- 受限于native 内存也有大小，也会爆出oom

-----
## hotspot 虚拟机内容
##### 对象的创建
- 
