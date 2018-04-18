# Method Area & Heap

Method area 通常用来存储 Class 相关的信息，如名称、构造器、方法等可以用 reflection API 看到的信息；Heap 通常用来存储各个instance 相关的信息，如各种 instance variables，因此通常 instance 的大小由 instance variables 的数量及类型相关。

##### 创建 instance

```java
SomeClass sc = new SomeClass();
```

当我们在程序中创建一个类的 instance 时，Method Area 和 Heap 的状态如下图所示：

![](/assets/Screen Shot 2018-04-18 at 10.41.19 PM.jpg)

如果此时 Heap 中不存在 SomeClass 的 Class object \(java.lang.Class 的实例\)，JVM 会在 Heap 中创建 SomeClass 的 Class object，同时在 Method Area 中存储 SomeClass 的元信息，注意这里 Class object 与 class data 之间都留有对方的联系方式。以上步骤完成后，JVM 才会在 Heap 中创建 SomeClass 的 Class instance \(图中的 object\)。

##### Method Area

接下来看一下 Method Area 中都包含着 Class 的哪些信息：

* 元信息 \(Meta Info\)
  * Names of type, superclass, super interfaces
  * Class or interface
  * Type modifiers: abstract, final public 
* 对 Class object 的引用 \(Reference to Class object\)
* 字段信息 \(Field Info\)
  * Name & type
  * Modifiers: static, final, access modifiers, transient, volatile
  * 原始类型的静态变量 \(primitive static variable\) ，其它任何类型的变量都存储在 Heap 中。对于对象类型的变量，如果是static 的，则对变量对象的引用存储在 Method Area 中；如果是实例变量，则对变量对象的引用存储在 Heap 中。
* Runtime Constant Pool: literals & symbolic references
* 方法信息 \(Method Info\)
  * Name
  * Return Type
  * Number and type of parameters
  * Modifiers: static, final, access modifiers, abstract, synchronized, native
  * Method bytecode
* Method Table: maintains array of references to instance methods。被用于实例方法调用

从 Java 8 开始，Method Area 被移到 Native Heap 中，被称为 Metaspace，移到 Native Heap 后 Metaspace 的大小不再受到 Java Heap 的限制。

##### Class data 在哪些地方会被用到

* 调用实例方法
* 检查 Type Cast 的正确性

```java
Superclass obj = new Subclass();
Subclass subObj = (Subclass) obj;
```

* instanceof 检查

```java
subObj instanceof Subclass
```



