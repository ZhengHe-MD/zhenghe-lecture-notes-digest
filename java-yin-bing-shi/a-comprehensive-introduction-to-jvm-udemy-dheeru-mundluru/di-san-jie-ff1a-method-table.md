# 第三节 - Method Table

Method Table 存在于 Method Area 中，是 class data 的一部分，里面由 instance methods 的引用数组构成。

假设有三个 Class A、B、C，以及一个 Interface，它们之间的关系如下图所示：

![](/assets/Screen Shot 2018-04-30 at 10.22.08 AM.jpg)

这时候有如下一个方法：

```java
go (A a) {
    a.foo();
    a.staticMethod();
}
```

在运行时中，a 可能是 A、B、C 三种 Class 中的任意一个，那究竟是谁的 foo 被调用，这时候就需要 Method Table 来决定。

Method Table 里面存储着 instance methods 的引用数组，这个数组中每个引用的排列顺序服从以下规则：

* 父类的方法被储存在前面
* 在同一个类中，方法按照声明顺序被存储

因此 A、B、C 的 Method Table 如下图所示：

![](/assets/Screen Shot 2018-04-30 at 10.22.49 AM.jpg)

由于“父类的方法被存储在前面”，不论在 A、B 还是 C 的 Method Table 中，foo 都在相同的位置。即如图中所示，A、B、C 的 Method Table 的第一个引用都是 foo，这时 JVM 不需要搜索 Method Table 就可以找到 foo 方法。

如果 A 是 Interface 类型，那么刚才的规则就不使用，这时候 A、B、C 的 Method Table，foo 方法在 B、C 的 Method Table 中的位置就不同，这时 JVM 就需要在运行时搜索 a 类型的 Method Table。具体如下图所示：

![](/assets/Screen Shot 2018-04-30 at 10.23.08 AM.jpg)

##### static method

static method 不在 Method Table 中。由于 static method 在编译的时候就已经确定，如在刚才的例子中 a.staticMethod 就是确定地指向 A.staticMethod。



