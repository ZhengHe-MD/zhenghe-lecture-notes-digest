# Map

### 约定

* Map: 表示 Interface Map
* map: 表示 Interface Map 的具体实现的一个实例
* 每个方法后面的 **O\(f\(n\)\)** 表示大部分 map 的实现对应方法的实现的复杂度，但不绝对

### 引入

Map 是 Interface，定义了用来存储键值对 \(key-value pair\) 的抽象数据类型 \(ADT\) 的契约。一般来说，Map 中：

* 不存在重复的键
* 每个键最多只能与一个值对应
* 键的顺序一般不能保证，但部分实现如 TreeMap 可以保证键的顺序

一些值得注意的地方：

* 可变 \(mutable\) 对象被用作键时需要额外注意，specification 并没有明确规定对象变化时 map 的行为
* 所有的 Map 实现一般都有两个构造器
  * 空构造器 \(void constructor\)：构造空的 map
  * 转化构造器 \(conversion constructor\)：按照给定的 map 构造一个新的 map，新旧 map 的实现可以不同。该构造器常常用于不同类型的 map 之间互相转化

### 声明

```java
public interface Map<K, V> {}
```

Map 是 generic type，可以接受类别作为其参数，在声明一个新的 Map 类型时，我们可以同时指定其键、值的类别，如：

```java
Map<String, Integer> map = new HashMap<>();
```

### Instance Methods

#### Query Operations

##### size - O\(1\)

```java
int size();
```

返回 map 中键值对的数量，当 map 含有超过 Integer.MAX\_VALUE 数量的键值对时，返回后者

##### isEmpty - O\(1\)

```java
boolean isEmpty();
```

检查 map 是否为空

##### containsKey - O\(1\)

```java
boolean containsKey(Object key);
```

检查 map 中是否包含键为 key 的键值对

##### containsValue - O\(n\)

```java
boolean containsValue(Object value);
```

检查 map 中是否包含值为 value 的键值对

##### get - O\(1\)

```java
V get(Object key);
```

返回 key 对应的值，如果 key 不存在于 map 中则返回 null。对于这个方法，有两个值得注意的点：

###### 为什么不是下面这种形式？

```java
V get(K key);
```

请参考 [What are the reasons why Map.get\(Object key\) is not \(fully\) generic](https://stackoverflow.com/questions/857420/what-are-the-reasons-why-map-getobject-key-is-not-fully-generic)，本问题也适用于 containsKey、containsValue、remove 等

###### 如果返回值是 null，怎么判断返回的是值还是告诉我们键值对不存在？

用 containsKey

#### Modifications

##### put - O\(1\)

```java
V put(K key, V value);
```

如果 map 中不存在键为 key 的键值对，则插入新的键值对 \(key, value\)，否则覆盖原 key 对应的 value。

##### remove - O\(1\)

```java
V remove(Object key);
```

如果 map 中不存在键为 key 的键值对，则返回 null；如果存在则删除该键值对，同时返回被删除键值对的 value。同样，因为可能存在 value 为 null 的情况，因此返回 null 并不代表 map 中一定不存在键为 key 的键值对。

#### Bulk Operations

##### putAll - O\(k\)

```java
void putAll(Map<? extends K, ? extends V> m);
```

本方法相当于对 m 中的每个键值对调用一次 put，如果 putAll 的过程中 m 发生改变，则行为是不可预测的。注意这里 m 的类型声明。

##### clear\(\)  - O\(1\) 或 O\(n\)

```java
void clear();
```

清空 map 的键值对。注意这里不同类型的 map 相应方法的复杂度不同，请参考 [Java: Why does clear\(\) on a HashMap take O\(n\) time while clear\(\) on a TreeMap takes only O\(1\) time?](https://www.quora.com/Java-Why-does-clear-on-a-HashMap-take-O-n-time-while-clear-on-a-TreeMap-takes-only-O-1-time)

#### Views

##### keySet - O\(n\)

```java
Set<K> keySet();
```

返回 map 中所有键的集合 \(Set\) 的视图 \(view\)，这个视图是以 map 为基础构建的，因此当 map 发生变化时 keySet 也将发生变化。但如果在 keySet 的使用过程中，map 发生变化，keySet 的行为不确定。keySet 支持删除元素，对应的键值对也会被删除；keySet 不支持新增元素。

##### values - O\(n\)

```java
Collection<V> values();
```

返回 map 中所有值的集合 \(Collection\) 的视图，剩下的说明与 keySet 相同。

##### entrySet - O\(n\)

```java
Set<Map.Entry<K, V>> entrySet();
```

返回 map 中所有键值对的集合 \(Set\) 的视图，剩下的说明与 keySet 相同。

##### Interface Entry:

Interface entry 是 entrySet 的元素所需实现的 Interface，因此也放在 Views 中讨论：

##### Entry: getKey - O\(1\)

```java
K getKey();
```

返回 entry 的键，如果 map 中对应的键值对被删除，则行为不确定。

##### Entry: getValue - O\(1\)

```java
V getValue();
```

返回 entry 的值，如果 map 中对应的键值对被删除，则行为不确定。

##### Entry: setValue - O\(1\)

```java
V setValue(V value);
```

将 entry 中 key 对应的值修改成 value，因为 map 是 entrySet 的基础，map 中的键值对也要相应地发生改变。如果 map 中对应的键值对被删除，则本操作行为不确定。

##### Entry: equals - O\(1\)

```java
boolean equals(Object o);
```

判断两个 entry 中的 key 和 value 是否两两相等

##### Entry: hashCode - O\(1\)

```java
int hashCode();
```

计算 entry 的 hashCode。specification 中指定了 hashCode 的计算方式，该方式同时保证了如果 entry1.equals\(entry2\) == true，那么 entry1.hashCode\(\) == entry2.hashCode\(\)。

##### Entry: static comparingByKey - O\(1\)

```java
public static <K extends Comparable<? super K>, V> Comparator<Map.Entry<K,V>> comparingByKey() {
    return (Comparator<Map.Entry<K, V>> & Serializable)
        (c1, c2) -> c1.getKey().compareTo(c2.getKey());
}

public static <K, V> Comparator<Map.Entry<K, V>> comparingByKey(Comparator<? super K> cmp) {
    Objects.requireNonNull(cmp);
    return (Comparator<Map.Entry<K, V>> & Serializable)
        (c1, c2) -> cmp.compare(c1.getKey(), c2.getKey());
}
```

* **&lt;K extends Comparable&lt;? super K&gt;, V&gt;** 用来指定 Comparator 参数的类别，K 需要实现 Comparable Interface
* **\(&lt;Comparator&lt;Map.Entry&lt;K, V&gt;&gt; & Serializable\)** 指定返回的参数不仅实现 Comparator Interface，而且实现 Serializable Interface

用法示例如下：

```java
// 1.
Map<String, Integer> res1 = unsortMap.entrySet().stream()
    .sorted(Map.Entry.comparingByKey())
    .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue,
            (oldValue, newValue) -> oldValue, LinkedHashMap::new));
// 2.
Map<String, Integer> res2 = new LinkedHashMap<>();
unsortedMap.entrySet().stream()
    .sorted(Map.Entry.comparingByKey())
    .forEachOrdered(x -> res2.put(x.getKey(), x.getValue()));
```

##### Entry: static comparingByValue - O\(1\)

```java
public static <K, V extends Comparable<? super V>> Comparator<Map.Entry<K, V>> comparingByValue() {
    return (Comparator<Map.Entry<K, V>> & Serializable)
        (c1, c2) -> c1.getValue().compareTo(c2.getValue());
}

public static <K, V> Comparator<Map.Entry<K, V>> comparingByValue(Comparator<? super V> cmp) {
    Objects.requireNonNull(cmp);
    return (Comparator<Map.Entry<K, V>> & Serializable)
        (c1, c2) -> cmp.compare(c1.getValue(), c2.getValue());
}
```

与 comparingByKey 类似。

Map.Entry Interface 的讨论到此结束，继续回到 Map Interface

#### Comparison and hashing

##### equals - O\(1\)

```java
boolean equals(Object o);
```

判断两个 map 是否相等。specification 里面定义，如果 Map m1 与 Map m2 中储存相同的键值对，也即 **m1.entrySet\(\).equals\(m2.entrySet\(\)\) **返回 true，那么两个 map 相等。同时这也表明，两个 map 是否相等并不取决于它们的实现，不同实现的 map 可能也含有相同的键值对。

##### hashCode - O\(1\)

```java
int hashCode();
```

specification 中定义一个 map 的 hashCode 应该等于它的所有 entry 的 hashCode 的和，这也保证两个相等的 map，它们的 hashCode 也相等。

#### Defaultable methods

default method 是 Java 8 新增的语言特性，允许在 Interface 定义默认方法，从而减小 Interface 实现者所需做的工作。

##### getOrDefault - O\(1\)

```java
default V getOrDefault(Object key, V defaultValue) {
    V v;
    return (((v = get(key)) != null) || containsKey(key))
        ? v
        : defaultValue;
}
```

此方法意义明确，实现非常精致：

1. get\(key\) 的同时赋值来避免二次调用 get\(key\)
2. **\(\(\(v = get\(key\)\) != null\) \|\| containsKey\(key\)\)** 很巧妙地表达了两种 case

##### forEach - O\(n\)

```java
default void forEach(BiConsumer<? super K, ? super V> action) {
    Objects.requireNonNull(action);
    for (Map.Entry<K, V> entry : entrySet()) {
        K k;
        V v;
        try {
            k = entry.getKey();
            v = entry.getValue();
        } catch (IllegalStateException ise) {
            // this usually means the entry is no longer in the map
            throw new ConcurrentModificationException(ise);
        }
        action.accept(k, v);
    }
}
```

forEach 依次对 entrySet 中的每一个 entry 执行 action 操作，当 entry.getKey\(\) 或 entry.getValue\(\) 发生错误时，一般是背后 map 的键值被修改或删除。

##### replaceAll - O\(n\)

```java
default void replaceAll(BiFunction<? super K, ? super V, ? extends V> function) {
    Objects.requireNonNull(function);
    for (Map.Entry<K, V> entry : entrySet()) {
        K k;
        V v;
        try {
            k = entry.getKey();
            v = entry.getValue();
        } catch(IllegalStateException ise) {
            throw new ConcurrentModificationException(ise);
        }

        v = function.apply(k, v);

        try {
            entry.setValue(v);
        } catch(IllegalStateException ise) {
            throw new ConcurrentModificationException(ise);
        }
    }
}
```

replaceAll 依次将 entrySet 的每个 entry 的 value 用 function 转化成另一个与 value 类别兼容的值，两个 try catch 尽最大努力做到及早发现并发修改导致的错误。

##### putIfAbsent - O\(1\)

```java
default V putIfAbsent(K key, V value) {
    V v = get(key);
    if (v == null) {
        v = put(key, value);
    }
    return v;
}
```

顾名思义

* 当 key 不在 map 中时，将 key, value 插入到 map 中
* 当 key 在 map 但对应的值为 null 时，用 value 覆盖原来的 null
* 当 key 在 map 中且对应的值不为 null 时，不变

##### remove - O\(1\)

```java
default boolean remove(Object key, Object value) {
    Object curValue = get(key);
    if (!Objects.equals(curValue, value) ||
         (curValue == null && !containsKey(key)) {
        return false;
    }
    remove(key);
    return true;
}
```

当 map 中存在 key, value 时，则把它们的 mapping 关系从 map 中删除。

##### replace \(K key, V oldValue, V newValue\) - O\(1\)

```java
default boolean replace(K key, V oldValue, V newValue) {
    Object curValue = get(key);
    if (!Objects.equals(curValue, oldValue) ||
        (curValue == null && !containsKey(key))) {
        return false;
    }
    put(key, newValue);
    return true;
}
```

当 map 中存在 key, oldValue 时，则把它们的 mapping 关系改成 key, value。

##### replace \(K key, V value\) - O\(1\)

```java
default V replace(K key, V value) {
    V curValue;
    if (((curValue = get(key)) != null) || containsKey(key)) {
        curValue = put(key, value);
    }
    return curValue;
}
```

当 map 中存在 key 时，则把 key 对应的值替换成 value

##### computeIfAbsent\(K key, Function&lt;? super K, ? extends V&gt; mappingFunction\) - O\(1\)

```java
default V computeIfAbsent(K key, Function<? super K, ? extends V> mappingFunction) {
    Objects.requireNonNull(mappingFunction);
    V v;
    if ((v = get(key)) == null) {
        V newValue;
        if ((newValue = mappingFunction.apply(key)) != null) {
            put(key, newValue);
            return newValue;
        }
    }
    return v;
}
```

当 map 中不存在键 key 或 key 对应的值为 null，且 mappingFunction.apply\(key\) 的返回值不为 null 时，将 key 与 newValue 的 mapping 关系存入 map 中

##### computeIfAbsent\(K key, BiFunction&lt;? super K, ? super V, ? extends V&gt; remappingFunction\) - O\(1\)

```java
default V computeIfAbsent(K key, BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
    Objects.requireNonNull(remappingFunction);
    V oldValue;
    if ((oldValue = get(key)) != null) {
        V newValue = remappingFunction.apply(key, oldValue);
        if (newValue != null) {
            put(key, newValue);
            return newValue;
        } else {
            remove(key);
            return null;
        }
    } else {
        return null;
    }
}
```

当 map 中不存在键 key 或 key 对应的值为 null 时，计算 remappingFunction.apply\(key, oldValue\) 的结果，如果返回 null，则从 map 中删除 key, oldValue 的 mapping 关系；如果返回非 null 值 newValue，则将 key, newValue 的 mapping 关系存入 map 中。

##### compute\(K key, BiFunction&lt;? super K, ? super V, ? extends V&gt; remappingFunction\) - O\(1\)

```java
default V compute(K key, BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
    Objects.requireNonNull(remappingFunction);
    V oldValue = get(key);
    
    V newValue = remappingFunction.apply(key, oldValue);
    if (newValue == null) {
        // delete mapping
        if (oldValue != null || containsKey(key)) {
            remove(key);
            return null;
        } else {
            return null;
        }
    } else {
        put(key, newValue);
        return newValue;
    }
}
```

如果 newValue 是 null，则删除 key, oldValue 的 mapping 关系；否则存入 key, newValue 的 mapping 关系。

##### merge\(K key, V value, BiFunction&lt;? super V, ? super V, ? extends V&gt; remappingFunction - O\(1\)

```java
default V merge(K key, V value, BiFunction<? super V, ? super V, ? extends V> remappingFunction) {
    Objects.requireNonNull(remappingFunction);
    Objects.requireNonNull(value);
    
    V oldValue = get(key);
    V newValue = (oldValue == null) ? value :
               remappingFunction.apply(oldValue, value);
    
    if (newValue == null) {
        remove(key);
    } else {
        put(key, newValue);
    } 
    return newValue;
}
```

这个操作常常被用来 merge 两个含有共同 key 不同 value 的 map。如果 key 尚不存在，则直接添加 mapping，否则使用 remappingFunction 对同 key 对应的不同 value 进行合理组合。

#### 参考：

* Effective Java \(2017\): Item 20、21、22
* [What are the reasons why Map.get\(Object key\) is not \(fully\) generic](https://stackoverflow.com/questions/857420/what-are-the-reasons-why-map-getobject-key-is-not-fully-generic)



