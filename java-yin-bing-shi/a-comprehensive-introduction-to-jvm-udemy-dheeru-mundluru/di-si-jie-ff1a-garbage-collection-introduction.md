# 第四节：Garbage Collection Introduction

由于 Heap 内存空间有限，当空间被分配完时，就需要回收一部分不用的空间。这个过程有两种方式：Explicit Memory Management 和 Implicit Memory Management。如 C 、C++ 这类语言就需要程序员控制内存的分配和回收，而像 Java、Scheme 这类语言就支持自动管理内存的分配和回收，而负责回收的模块就被称作垃圾收集器 \(Garbage Collector\)。

#### Memory Corruption Errors

当内存的分配和回收操作不当时，就会出现内存破坏、程序崩溃，通常可以分为以下几种情况：

* 内存泄露 \(Memory Leak\): 不再使用的 objects 没有回收，将永远在 Heap 内存空间中 “占着茅坑不拉屎”。内存泄露持续发生将导致 Heap 内存可用空间不断减少，在短时 \(short running \) 应用中影响不大且不易察觉，到了像 web 服务这样的应用中就会显现出来。
* 悬挂引用 \(Dangling Reference\): 运行时中还存在对已经回收的空间的引用。这时候，由于被回收空间内的信息不可预测，直接使用可能导致程序出现不确定的行为。

#### Garbage Collector

自动内存管理 \(Automatic Memory Management\) 是解决主动内存管理容易出错问题的一种方案

* 回收不再被引用的 objects --- 解决内存泄露问题
* 不回收尚被引用的 objects --- 解决悬挂引用问题

##### 什么时候 objects 不再被引用

```java
// 1. 离开作用域
void go() {
    Book b = new Book();
}

// 2. 引用被赋予新值
Book b = new Book();
b = new Book();

// 3. 引用被赋予 null
Book b = new Book();
b = null;
```

##### GC 的几个特点：

1. 它可能作为内置模块或者插件存在于不同语言环境
2. 它一般在后台以低优先级执行
3. gc 的运行时机不被保证
4. gc 可能导致应用暂停 \(stop-the-world\)





