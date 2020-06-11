# 第2讲：垃圾收集器

> GC三问：
>
> 哪些内存需要回收？
>
> 什么时候回收？
>
> 如何回收？

程序计数器、虚拟机栈、本地方法栈随线程而生，随线程而灭，栈帧的内存分配在类结构确定下来就已知，在方法结束或者线程结束时就会回收。所以垃圾回收关注的是动态的堆内存。

ps. 方法区也能被回收，主要回收废弃常量和无用类，但性价比高，不过多描述。

## 1.哪些内存需要回收

> 这个问题的关键就是确定哪些内存是存活着，哪些内存死去（不再会被用到的）

- 引用计数算法

  有引用时就+1，引用失效就-1，计数器为0则可回收

  **无法回收相互引用的情况**

  > 引用分为强引用、软引用、弱引用、虚引用，引用强度递减
  >
  > - 强引用
  >
  >   - 普遍存在的，Object obj = new Object()
  >
  >   - 只要强引用存在，垃圾收集器永远不会回收掉被引用的对象
  >
  > - 软引用
  >   - 对象在有用但非必须
  >
  >   - 内存不足时才会回收
  >
  >   - 实现高速缓存
  >
  >   - ```java
  >     String str = new String("abc");
  >     SoftReference<String> softRef = new SoftReference<String>(str);
  >     ```
  >
  > - 弱引用
  >
  >   - 非必须，比软引用弱
  >
  >   - GC时会被回收(概率不大，优先级低)
  >
  >   - 适用于偶尔被使用不影响垃圾收集的对象
  >
  >   - ```java
  >     String str = new String("abc");
  >     WeakReference<String> weakRef = new WeakReference<String>(str);
  >     ```
  >
  > - 虚引用
  >
  >   - 不决定对象生命周期
  >
  >   - 任何时候可回收
  >
  >   - 跟踪对象被垃圾收集器回收的活动
  >
  >   - 必须和引用队列ReferenceQueue联合使用
  >
  >   - ```java
  >     String str = new String("abc");
  >     ReferenceQueue queue = new ReferenceQueue();
  >     PhantomReference<String> phantomRef = new PhantomReference<String>(str,queue);
  >     ```

  

- 可达性分析

  从GC Roots作为起始点向下，搜索走过的路径称为引用链，当一个对象到GC Roots没有任何引用链则为不可达，判定为可回收对象。

  > **什么对象可以作为GC Roots**
  >
  > - 虚拟机栈（栈帧中的本地变量表）中引用的对象
  > - 方法区的类静态属性引用的对象
  > - 方法区中常量引用的对象
  > - 本地方法栈中native方法引用的对象

  要判定一个对象的死亡，需要经过两次标记：第一次未与GC Roots相连的节点会经过第一次标记并进行一次筛选。筛选的条件是此对象是否有必要执行finalize()方法（对象没有覆盖finalize()或者finalize()已经被调用过则为没有必要执行）。经过第一次标记后的对象会被放入F-Queue的队列中，由虚拟机自动创建、优先级低的Finalizer线程去执行他。对象可以在finalize()方法中实现自救，如果自救成功会被移出队列，不再回收。



## 2.垃圾回收算法



