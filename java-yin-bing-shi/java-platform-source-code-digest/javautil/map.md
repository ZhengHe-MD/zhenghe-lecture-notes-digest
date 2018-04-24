# Map

Map 是 Interface，定义了用来存储键值对 \(key-value pair\) 的抽象数据类型 \(ADT\) 的契约。一般来说，Map 中：

* 不存在重复的键
* 每个键最多只能与一个值对应
* 键的顺序一般不能保证，但部分实现如 TreeMap 可以保证键的顺序

一些值得注意的地方：

* 可变 \(mutable\) 对象被用作键时需要额外注意，由于可变对象的 hashCode 会随着 instance variables 的改变而改变。



