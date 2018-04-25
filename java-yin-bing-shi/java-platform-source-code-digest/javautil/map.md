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

#### 参考：

* Effective Java \(2017\): Item 20、21、22
* [What are the reasons why Map.get\(Object key\) is not \(fully\) generic](https://stackoverflow.com/questions/857420/what-are-the-reasons-why-map-getobject-key-is-not-fully-generic)



