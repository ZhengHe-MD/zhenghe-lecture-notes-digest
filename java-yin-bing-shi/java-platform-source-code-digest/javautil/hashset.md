# HashSet

HashSet 在概念上就是 Set，而它被叫作 HashSet 的原因也很直观 --- 它的内部实现依赖于 HashTable \(HashMap\)，或者说它的背后实际上就是一个 HashTable。对 HashTable 我们会有一个基本的判断，只要 hash function 能够均匀地把元素分布到不同的 buckets 中，存取元素的操作的均摊 \(amortized\) 时空复杂度应该都在 O\(1\)。接下来，我们把 HashTable \(HashMap\) 当作是给定的组件，来阅读一下 HashSet 的源码。

注：HashTable 和 HashMap 功能上大致相同，主要区别在于 HashTable 是线程安全的而 HashMap 不是线程安全的。在很早以前只有 HashTable 的时候，许多数据结构都是线程安全的，但在后来迭代的过程中发现很多情况是不需要保证线程安全，而保证线程安全会有额外的性能开销，因此 Java 团队决定新的 API 默认不附加线程安全的特性。本文的 HashSet 背后是 HashMap，因此它也不是线程安全的。

### 说明

* 本文应用范围 Java8

### 导读

#### 声明

```java
public class HashSet<E> extends AbstractSet<E>
                        implements Set<E>, Cloneable, java.io.Serializable {}
```

我们观察到以下几点：

* HashSet 是 generic 类：显然 Set 里面放什么类型的对象是任意指定的，但一个集合里面只能放同一种类型的对象
* HashSet 继承 AbstractSet：这个可参考 AbstractSet 的源码阅读
* HashSet 实现 Set&lt;E&gt; 接口：Set 是 Collection 接口下的一个子接口，它定义了一般 Set 具体实现的契约
* HashSet 实现 Cloneable 接口: Cloneable 是一个标记接口 \(Marker Interface\) ，标记这个 HashSet 支持 clone
* HashSet 实现 Serializable 接口：Serializable 也是标记接口，标记这个 HashSet 支持序列化 \(serialization\)和反序列化 \(deserialization\)，具体可以参考[这篇文章](http://www.oracle.com/technetwork/articles/java/javaserial-1536170.html)

#### 静态变量

```java
static final long serialVersionUID = -5024744406713321676L;
private static final Object PRESENT = new Object();
```

serialVersionUID 用于序列化过程中核验对象的版本，具体请参考声明中提到的文章。只要不同版本 JDK 的 HashSet 实现没有较大的改动，这个 serialVersionUID 不会发生改变。

由于 HashSet 的背后是 HashMap。Map 由键值对组成，我们用 key 代表 Set 中的元素，那么有 key 就必须要有 value，如果我们用 null 作 value，就无法判断 value 对应的 key 是否存在，因此需要一个 nonNull 的值来作 Map 中已经存在的 key 的 value，于是就有了 PRESENT，PRESENT 是一个空的 Object 对象，HashSet 中所有的元素在背后的 HashMap 中都与之对应，PRESENT 的静态变量身份也让空间的浪费在最大程度上减小。

#### 参考

* [Oracle: Set Interface](https://docs.oracle.com/javase/tutorial/collections/interfaces/set.html)
* [Discover the secrets of the Java Serialization API](http://www.oracle.com/technetwork/articles/java/javaserial-1536170.html)



