# jvm

### 垃圾回收机制

**何时触发垃圾回收？回收算法有哪些？如何判定可回收？垃圾回收过程？垃圾收集器有哪些？四种引用？**

垃圾回收有两种类型，新生代垃圾回收和老年代垃圾回收。

**新生代垃圾回收**

一般情况下，当新对象生成，并且在Eden申请空间失败时，就会触发新生代，对Eden区域进行GC，清除非存活对象，并且把尚且存活的对象移动到Survivor区。然后整理Survivor的两个区。这种方式的GC是对年轻代的Eden区进行，不会影响到年老代。因为大部分对象都是从Eden区开始的，同时Eden区不会分配的很大，所以Eden区的GC会频繁进行。因而，一般在这里需要使用速度快、效率高的算法，使Eden去能尽快空闲出来。

**老年代垃圾回收**

对整个堆进行整理，包括Young、Tenured和Perm。Full GC因为需要对整个堆进行回收，所以比Scavenge GC要慢，因此应该尽可能减少Full GC的次数。在对JVM调优的过程中，很大一部分工作就是对于FullGC的调节。有如下原因可能导致Full GC：
1.年老代（Tenured）被写满
2.持久代（Perm）被写满
3.System.gc()被显示调用
4.上一次GC之后Heap的各域分配策略动态变化

**分配担保机制**

创建新对象时eden区内存不足，同时survivor区不足以容纳存活对象时，通过分配分配担保机制将存活对象放到老年代。如果老年代内存不足，则会进行老年代垃圾回收。

在Serial+Serial Old的情况下直接启动担保机制。在Parallel Scavenge+Serial Old的情况下，却是先要去判断一下要分配的内存是不是**>=Eden区大小的一半**，如果是那么直接把该对象放入老生代，否则才会启动担保机制

**回收算法**

复制、标记-清除、标记-整理

**判定可回收**

**引用计数法**，被引用为0，可回收对象。无法解决循环引用的问题

**根搜索算法**，以一系列的GC  ROOTS作为根节点，至当前对象没有可达路径，则该对象不可达。不可达不等价于可回收，至少需要经过两次标记过程，两次标记后仍是可回收，则面临回收。

GC  ROOTS ：方法区中常量引用的对象，类的静态属性引用的对象。虚拟机栈中引用的对象。本地方法栈中引用的对象。

**四种引用**

