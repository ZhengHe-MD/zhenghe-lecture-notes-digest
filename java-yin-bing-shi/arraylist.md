# ArrayList

### 概述

ArrayList 是长度可变的 List 接口实现，它会随着 List 中的元素增加而扩大 List 的长度。ArrayList，顾名思义，它是利用 Array 构造的 List。但 Java 中的 Array 是定长的，那么 ArrayList 是如何支持长度可变的特性呢？实际上这里并没有黑科技：ArrayList 内部会保存一个定长 Array 来存储 ArrayList 中的元素，每当当前定长的 Array 放满，就重新申请一个更长的 Array，把当前的 Array 里的元素全部拷贝进去。

### 说明

* 为了避免混淆，Array 和 List 在本文中不作翻译，保留原单词
* capacity =&gt; 容量
* size =&gt; 长度
* reallocation =&gt; 重分配
* 本文应用范围 - Java8

### 导读

#### instance variables

```java
public class ArrayList<E> extends AbstractList<E> 
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    transient Object[] elementData;
    private int size;

    //...
}
```

ArrayList 只有两个 instance variable，其中

* elementData 用来保存元素
* size 用来记录 ArrayList 的长度/大小。

#### constructors

##### constructor without parameters

写代码时，十有八九我们会这样声明一个 ArrayList

```java
List<Object> list = new ArrayList<>();
```

在它的背后 ArrayList 做了哪些事情呢？

```java
private static final int DEFAULT_CAPACITY = 10;
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

public ArrayList() { this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA; }
```

每次初始化一个空的 ArrayList 实例，elementData 都指向 DEFAULTCAPACITY\__EMPTY_\_ELEMENTDATA，后者是一个以单例形式存在的空的 Object Array。由于现实的应用中充斥着空的、从未增加元素的 ArrayList，如果为这些空的 ArrayList 都分配内存空间，将造成不可忽视的空间浪费，而这里正是用单例代替预分配的策略来避免无谓的浪费。

##### constructor with initialCapacity

如果我们预先知道 list 的容量 \(capacity\)，就可以在初始化的时候设定容量大小：

```java
private static final Object[] EMPTY_ELEMENTDATA = {};

public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}
```

同样，这里的 EMPTY\_ELEMENTDATA 也是用作避免空间浪费。

这里需要区分一下容量和长度：容量指的是 ArrayList 在不做重分配的情况下，最多能容纳的元素数量；长度指的是 ArrayList 当前的元素数量。因此执行如下代码会抛错：

```java
ArrayList<Integer> arr = new ArrayList<Integer>(10);
arr.add(5, 10); // 当前 ArrayList 的容量为 10，长度为 0，无法直接将 10 插入到第 5 个位置
```

##### constructor with Collection

```java
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

如果没有 [bug 6260652](https://bugs.java.com/bugdatabase/view_bug.do?bug_id=6260652)，本段代码实际可以精简成：

```java
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if (elementData.length == 0) {
        elementData = EMPTY_ELEMENTDATA;
    }
}
```

一目了然，c.toArray\(\) 是 Collection 到 Array 的转化 api，如果长度为 0，则初始化为 EMPTY\_ELEMENTDATA。

#### 增长策略

```java
private static final int DEFAULT_CAPACITY = 10;

public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}

private static int calculateCapacity(Object[] elementData, int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        return Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    return minCapacity;
}

private void ensureCapacityInternal(int minCapacity) {
    ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
}

private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}
```

往 ArrayList 里添加元素前，需要先检查容量是否满足需求，若不足则应该扩容。这里的 calculateCapacity 保证了：如果是默认初始容量的 ArrayList，就直接将最小容量设置为 DEFAULT\_CAPACITY =&gt; 10。最后，在 ensureExplicitCapacity 中判断：若所需最小容量大于当前 elementData 的长度，就应该扩容，见下文中的 grow 函数：

```java
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    // oldCapacity >> 1 相当于 oldCapacity * 0.5
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
    // 令人费解的代码
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
        
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```

##### 扩容策略的复杂度分析

扩容的主要策略就是 **int newCapacity = oldCapacity + \(oldCapacity &gt;&gt; 1\)** 即 **int newCapacity = 1.5 \* oldCapacity **，这种扩容方式可以将插入操作的均摊 \(amortized\) 时间、空间复杂度控制在 O\(n\) 内，具体计算过程见[这里](http://www.wolframalpha.com/input/?i=sum%281.5**k%29+for+k%3D1,log%281.5,n%29)。

##### OutOfMemoryError

hugeCapacity 中存在一段令人费解的代码:





