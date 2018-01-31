# The Complete Performance Story

本书第一章的精华凝结就在这一节，其它内容都是些鸡毛蒜皮的小事，就不一一记录。

这 **The Complete Performance Story ，**一言以蔽之，就是在考虑应用的性能问题时，脑子里要有整体把握，不要认为自己 Java 牛逼就以为性能问题都出在 Java 代码身上。还有一些代码之外的方面，在影响着应用的性能：

## Write Better Algorithms

如果你要做的事情是多次在一组数据里面找出其中一个，使用 HashMap 永远要比遍历 List 来得快。当遇见性能问题，好的算法常常是解决问题的核心。

## Write Less Code

同样是实现一个应用的所有功能，一种包含更少代码的实现总是要优于一种包含更多代码的实现，因为更多的代码意味着：

1. 更多的代码要被编译
2. 代码运行的时间更长
3. 更多的对象被管理
4. 更繁重的垃圾回收工作
5. 更多的类要被加载，更长的应用启动时间
6. ...

## Oh Go Ahead, Prematurely Optimize

> 季文子三思而后行。子闻之曰：再，斯可矣 ——《论语·公冶长》

这里引一句论语，文中的金句"**三思而后行**"被人们口耳相传，殊不知孔子本来说的是，“思考两次就够了，为啥要思考三次？”

本节提到的也是一个经典的断章取义:

> We should forget about small efficiencies, say about 97% of the times; premature optimization is the root of all evil 
>
> --- Donald Knuth

Knuth 反对的是那些因为优化代码而严重损害代码可读性的行为，而不是 "premature optimization" 本身，然而这句话却被慵懒、不求甚解的程序员用来鼓励自己不考虑性能随便写代码。完成一个功能可能有多种方法，如果它们都差不多简单，就选择那个性能更好的！

一个经典的例子如下：

```java
log.log(Level.FINE, "I am here, and the value of X is " + calcX() + “ and Y is " + calcY());
```

如果 logging level 高于 FINE，那么这个日志信息将不被打印，然而 String concatenation、calcX\(\)、calcY\(\) 每次仍然都会被执行，记录日志是会被经常执行的代码，数量级上去了性能问题就会显现出来。一个合理的做法是：

```java
if (log.isLoggable(Level.FINE)） {
    log.log(Level.FINE, "I am here, and the value of X is " + calcX() + “ and Y is " + calcY());
}
```

代码可读性基本不变，却及时扼杀了潜在性能问题。

## Look Elsewhere: The Database is Always the Bottleneck

本节的主菜，虽然重要但其实它要传达的意思很明确：

如果整个系统的瓶颈在数据库上，那么无论你怎么优化应用代码，都是南辕北辙，甚至适得其反：原因在于如果数据库响应请求缓慢时，如果你优化了应用代码，可能使得发送给数据库的请求频率更高，导致整个系统性能进一步下降。这种优化错位适得其反的情况是广义的，对系统中的其它组成部分也同样适用。

## Optimize for the Common Case

抓主要矛盾，优化常见情况的处理





