# 第五节 - GC Algorithms

#### GC 的挑战

* 如何找出废弃 \(abandoned\) 的 objects？
  * 在你寻找废弃的 objects 时，可能有新的废弃的 objects 在产生。虽然可以通过暂停整个应用来消除这种情况，但暂停时间过长会影响用户体验。
* 如何减小 GC 的时长？

#### Mark and Sweep

Mark and Sweep 算法分为 Mark 和 Sweep 两个阶段，显然 Mark 阶段要标记处需要被回收的 objects，后者在 Sweep 阶段被统一回收。

##### Mark Phase

Mark 阶段会遍历环境内存在的对 objects 的直接饮用和间接引用，并把它们标记为 live。这些直接引用被称为 GC Root，举例如下图所示：

![](/assets/Screen Shot 2018-04-30 at 3.54.03 PM.jpg)

从图中的 GC Root 出发，每个圆圈表示一个 object，蓝色表示这些 objects 是 live 的，而没有被标记为 live 的 objects 用灰色表示，Mark 阶段结束后就可能出现图中所示的标记结果。

###### 如果在 Mark 阶段进行的时候有新的 objects 产生

我们没法把随时可能新产生的 objects 加入到 GC Roots 中，因此它们不会在 Mark 阶段被标记成 live。但如果它们不被标记为 live，Sweep 阶段就会把它们错误地回收，这是无法接受的。于是我们只能在 Mark 阶段暂停应用。

##### Sweep Phase

遍历 Heap Memory，把所有被标记为 live 的 objects 的标记删除，所有未被标记为 live 的 objects 回收。

##### Issue

Mark and Sweep 算法的一大问题是它会造成 Heap 空间碎片化，如下图所示：

![](/assets/Screen Shot 2018-04-30 at 3.54.25 PM.jpg)

碎片化会出现大块内存空间无法被分配的问题。为了解决这个问题，又出现了 Mark-Sweep-Compact 和 Mark and Copy 两种 GC 算法。

#### Mark-Sweep-Compact

顾名思义，Mark-Sweep-Compact 在 Mark 阶段和 Sweep 阶段之后又新设一个 Compact 阶段，整理 Heap 的碎片空间，如下图所示：

![](/assets/Screen Shot 2018-04-30 at 3.54.49 PM.jpg)

Mark-Sweep-Compact 算法会延长应用的暂停时间，进一步影响用户体验。

#### Mark and Copy

Mark and Copy 把 Heap 内存空间分为两部分大小相等的空间 A 和 B，A 空间用于分配，B空间保留不分配。在 Copy A 空间阶段，遇到标记为 live 的 objects 时，直接把它们复制到 B 空间，同时进行碎片整理，如下图所示：

![](/assets/Screen Shot 2018-04-30 at 3.55.10 PM.jpg)

Mark and Copy 由于可以在 Mark 阶段同时进行 Copy 操作，因此能够减少应用的暂停时间，同时可以做碎片整理，但缺点在于需要两倍的内存空间，是典型的空间换时间的算法。

#### 小结

从 GC 要解决的问题出发，可以很容易将这些算法联系起来：

* 检测废弃的 objects --- Mark and Sweep
* 减小 GC 时长 --- Mark and Copy
* 碎片整理 --- Mark and Copy、Mark-Sweep-Compact

本节介绍了基本 GC 算法，为了权衡不同因素，实际情况中可能将这些算法组合使用。

