# ArrayList

### 概述

ArrayList 是长度可变的 List 接口实现，它会随着 List 中的元素增加而扩大 List 的长度。ArrayList，顾名思义，它是利用 Array 构造的 List。但 Java 中的 Array 是定长的，那么 ArrayList 是如何支持长度可变的特性呢？实际上这里并没有黑科技：ArrayList 内部会保存一个定长 Array 来存储 ArrayList 中的元素，每当当前定长的 Array 放满，就重新申请一个更长的 Array，把当前的 Array 里的元素全部拷贝进去。

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

```java
private static final int DEFAULT_CAPACITY = 10;
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

public ArrayList() { this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA; }
```



