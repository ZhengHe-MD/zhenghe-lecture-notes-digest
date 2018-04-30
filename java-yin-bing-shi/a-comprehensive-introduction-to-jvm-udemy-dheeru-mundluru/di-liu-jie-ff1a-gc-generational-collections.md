# 第六节：GC Generational Collections

### Motivation

在实际应用中可以观察到，许多 objects 存活的时间很短，同时又一部分 objects 存活时间很长，甚至只会在应用停止运行时才消失。如果能够区别对待短时 objects 和长时 objects，就可以避免在 GC 的时候重复访问长时 objects，从而进一步缩减 GC 的暂停时间。

#### Generational Collections

把 Heap 内存空间分为 Young Generation 和 Old \(Tenured\) Generation 两部分，分别处理存活时间较短和较长的两类 objects。一般情况下，Young Generation 的吞吐量比较大，进行垃圾搜集的次数要比 Old Generation 的次数多一些，同时根据两类 objects 的特点可以使用不同的 GC algorithms 分别专门处理。一般来说，GC 一次 Young Generation Collection 被称为 minor GC；GC 一次 Young 和 Old 两个 Collections 以及 Permanent Generation/Metaspace 被称为 full GC。

##### Young Generation

所有新的 objects 都在 Young Generation



