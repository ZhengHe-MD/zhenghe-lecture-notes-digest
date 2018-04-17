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

#### Class Variables

```java
static final long serialVersionUID = -5024744406713321676L;
private static final Object PRESENT = new Object();
```

serialVersionUID 用于序列化过程中核验对象的版本，具体请参考声明中提到的文章。只要不同版本 JDK 的 HashSet 实现没有较大的改动，这个 serialVersionUID 不会发生改变。

由于 HashSet 的背后是 HashMap。Map 由键值对组成，我们用 key 代表 Set 中的元素，那么有 key 就必须要有 value，如果我们用 null 作 value，就无法判断 value 对应的 key 是否存在，因此需要一个 NonNull 的值来作 Map 中已经存在的 key 的 value，于是就有了 PRESENT。PRESENT 是一个空的 Object 对象，HashSet 中所有的元素在背后的 HashMap 中的 value 都是 PRESENT。PRESENT 是 Class Variable ，因此空间的浪费在最大程度上被减小。值得一提的是，PRESENT 的本意就是**存在**的意思。

#### Instance Variables

```java
private transient HashMap<E, Object> map;
```

这里除了 transient 其它无需说明。transient 告诉 Java 的序列化 API 这个字段不需要被序列化，在这里实际上是因为默认的 serializer 在性能或者安全性等方面不满足需求，因此 JDK 的作者希望自己来实现 map 的序列化。综上，这里的 transient 是在宣告 “序列化的时候别动它，放着我来！”

#### Constructors

```java
/* 1 */
public HashSet() { 
    map = new HashMap<>();
}

/* 2 */
public HashSet(Collection<? extends E> c) {
    map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
    addAll(c);
}

/* 3 */
public HashSet(int initialCapacity, float loadFactor) {
    map = new HashMap<>(initialCapacity, loadFactor);
}

/* 4 */
public HashSet(int initialCapacity) {
    map = new HashMap<>(initialCapacity);
}

/* 5 */
HashSet(int initialCapacity, float loadFactor, boolean dummy) {
    map = new LinkedHashMap<>(initialCapacity, loadFactor);
}
```

1. 在不知道集合中元素的总量时最常用的 constructor，内部的 HashMap 使用默认的 initialCapacity - **16** 与 loadFactor - **0.75**。
2. Collection 的所有实现必须有的 constructor，用作不同 collection 之间的转化。因为知道集合中的元素总量，因此使用与总量相匹配的 initialCapacity，避免重复再分配。
3. 自定义内部 HashMap 的两个参数，在明确知道其含义的情况下才使用。
4. 自定义 initialCapacity，使用场景见 2

#### Instance Methods

以下几个 instance methods 复用了 HashMap 中的实现，减少重复代码，提高 JDK code base 的可维护性

##### iterator

```java
public Iterator<E> iterator() {
    return map.keySet().iterator();
}
```

map.keySet\(\) 返回 HashMap 中定义的 KeySet 类，后者实现了 Set 接口，可以理解成最简版的 Set 实现。

##### size

```java
public int size() { return map.size(); }
```

##### isEmpty

```java
public boolean isEmpty() { return map.isEmpty(); }
```

##### contains

```java
public boolean contains(Object o) { return map.containsKey(o); }
```

##### add

```java
public boolean add(E e) { return map.put(e, PRESENT) == null; }
```

如果 map 中已经含有 key e, 则 put 返回 e 之前绑定的值；如果 map 中尚未有 key e，则 put 返回 null。因此如果 HashSet 已经含有元素 e，返回 false；如果 HashSet 尚未含有元素 e，返回 true。

##### remove

```java
public boolean remove(Object o) { return map.remove(o) == PRESENT; }
```

注意这里的输入是 Object 类型，map.remove 的返回值与 map.put 类似。

##### clear

```java
public void clear() { map.clear(); }
```

##### clone

```java
public Object clone() {
    try {
        HashSet<E> newSet = (HashSet<E>) super.clone();
        newSet.map = (hashMap<E, Object>) map.clone();
        return newSet;
    } catch (CloneNotSupportedException e) {
        throw new InternalError(e);
    }
}
```

super.clone 是 shallow clone，因此在执行 try block 内第一行代码后，newSet.map 指向 map 本身，这并不是使用者想要的，因此需要第二行代码来 shallow clone map。

##### writeObject

```java
private void writeObject(java.io.ObjectOutputStream s) throws java.io.IOException {
    s.defaultWriteObject();
    s.writeInt(map.capacity());
    s.writeFloat(map.loadFactor());
    s.writeInt(map.size());

    for (E e : map.keySet())
        s.writeObject(e);
    }
}
```

s.defaultWriteObject\(\) 执行默认的序列化程序，由于 map 被声明为 transient，因此 map 需要手动序列化，这里手动序列化了 map 的 capacity、loadFactor、size、Set 等信息

##### readObject

```java
private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    s.defaultReadObject();
    
    int capacity = s.readInt();
    if (capacity < 0) {
        throw new InvalidObjectException("Illegal capacity: " +
                                         capacity);
    }

    float loadFactor = s.readFloat();
    if (loadFactor <= 0 || Float.isNaN(loadFactor)) {
        throw new InvalidObjectException("Illegal load factor: " +
                                         loadFactor);
    }

    int size = s.readInt();
    if (size < 0) {
        throw new InvalidObjectException("Illegal size: " +
                                         size);
    }

    capacity = (int) Math.min(size * Math.min(1 / loadFactor, 4.0f),
            HashMap.MAXIMUM_CAPACITY);

    SharedSecrets.getJavaOISAccess()
                 .checkArray(s, Map.Entry[].class, HashMap.tableSizeFor(capacity));

    map = (((HashSet<?>)this) instanceof LinkedHashSet ?
           new LinkedHashMap<E,Object>(capacity, loadFactor) :
           new HashMap<E,Object>(capacity, loadFactor));

    for (int i=0; i<size; i++) {
        @SuppressWarnings("unchecked")
        E e = (E) s.readObject();
        map.put(e, PRESENT);
    }
}
```

基本上很清晰，依次把 map 的信息取出来，然后重新初始化 HashMap。 这里 SharedSecrets 一句没有理解，待日后补充。

#### 参考

* [Oracle: Set Interface](https://docs.oracle.com/javase/tutorial/collections/interfaces/set.html)
* [Discover the secrets of the Java Serialization API](http://www.oracle.com/technetwork/articles/java/javaserial-1536170.html)



