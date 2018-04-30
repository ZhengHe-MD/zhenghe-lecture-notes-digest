# 第六节：GC Generational Collections

### Motivation

在实际应用中可以观察到，许多 objects 存活的时间很短，同时又一部分 objects 存活时间很长，甚至只会在应用停止运行时才消失。如果能够区别对待短时 objects 和长时 objects，就可以避免在 GC 的时候重复访问长时 objects，从而进一步缩减 GC 的暂停时间。

#### Generational Collections

把 Heap 内存空间分为 Young Generation 和 Old \(Tenured\) Generation 两部分，分别处理存活时间较短和较长的两类 objects。一般情况下，Young Generation 的吞吐量比较大，进行垃圾搜集的次数要比 Old Generation 的次数多一些，同时根据两类 objects 的特点可以使用不同的 GC algorithms 分别专门处理。

##### Young Generation

所有新的 objects 都在 Young Generation 这里分配产生，并且年纪慢慢长大，当 Young Generation 被填满时，就会激发一次 minor garbage collection \(minor GC\)。在多次 minor GC 中存活下来的 objects 最终会被移动到 Old Generation 中去。Minor GC 是 "Stop the World" 事件，在 minor GC 结束之前，所有与应用本身有关的线程都会被停止。

##### Old Generation

所有年长的 objects 最终会被存储在 Old Generation 中。通常会有一个界限来定义所谓“年长”，比如连续在 15 次 minor GC 下生存的 objects。最终，Old Generation 也会被执行 GC，这个事件被称为 major garbage collection \(major GC\)。Major GC 同样也是 "Stop the World" 事件，在 GC 期间所有的应用线程都会停止，等待 GC 执行完毕。

##### The Generational Garbage Collection Process

1. Young Generation 被划分成三个部分，Eden space、"from" survivor space 以及 "to" suvivor space，所有新产生的对象都被分配到 Eden space。值得一提的是，Eden 就是伊甸园 \(the garden of eden\)，此外 "from" 和 “to” spaces 也被称为 S0 和 S1，当 copy 的方向改变时，"from"、"to" 与 S0、S1 的对应关系也会发生改变。
   ![](/assets/Screen Shot 2018-04-30 at 9.21.02 PM.jpg)

2. 当 eden space 被填满时，触发 minor GC
   ![](/assets/Screen Shot 2018-04-30 at 9.59.18 PM.jpg)

3. 所有被标记为 live 的 objects 被移动到 S0，并且 age 记为 1，剩下的 objects 则被删除回收
   ![](/assets/Screen Shot 2018-04-30 at 9.59.44 PM.jpg)

4. 当 eden space 再度被填满时，再次触发 minor GC，发生与 3 类似的过程，但这时候 eden space 和 S0 中的被标记为 live 的 objects 全部被 copy 到 S1 中，并且之前在 S0 中的 objects age 记为 2，之前在 eden space 中被标记为 live 的 objects age 记为 1。
   ![](/assets/Screen Shot 2018-04-30 at 10.00.10 PM.jpg)

5. 当 eden space 再度被填满时，又一次触发 minor GC，这时候再次置换 S0 与 S1 的角色，所有存活下来的 objects 被 copy 到 S0 中，并且来自于 S1 中的 objects age 记为 3。
   ![](/assets/Screen Shot 2018-04-30 at 10.00.26 PM.jpg)

6. 中间省略多次 minor GC，当个别 objects 的 age 超过阈值 \(这里假设为8\) 时，这些 objects 将被升级移动到 old generation 中。
   ![](/assets/Screen Shot 2018-04-30 at 10.00.41 PM.jpg)

7. 随着 minor GC 继续发生，不断有新的 objects 被移动到 old generation
   ![](/assets/Screen Shot 2018-04-30 at 10.00.57 PM.jpg)

8. 最终，major GC 会在 old generation 上触发。以上便是 Generational Garbage Collection Process 的大致情况。

#### Types of Garbage Collectors

|  | Young | Old |
| :--- | :--- | :--- |
| Serial GC | Mark & Copy | Mark-Sweep-Compact |
| Parallel GC | Mark & Copy | Mark-Sweep-Compact |
| CMS | Mark & Copy | Concurrent Mark-Sweep |

虽然三种 GC 的 Young Generation 都使用 Mark & Copy 算法，但 Serial GC 只使用一个线程，而 Parallel GC 及 CMS 则使用多线程并发，在多核环境下能更有效地减少 GC 暂停时间。

还有一种 Garbage Collector 叫做 G1，它并没有使用 Generational Garbage Collection 的 Heap Memory Model。它的特点是：

* 并发 & 并行
* 使用更少的 heap memory
* 减少 old GC 时间

G1 在 Java 9 中已经成为默认垃圾收集器，但 G1 本身原理不属于本节讨论的范围。

#### 参考

* [Java Garbage Collection Basics](http://www.oracle.com/webfolder/technetwork/tutorials/obe/java/gc01/index.html)



