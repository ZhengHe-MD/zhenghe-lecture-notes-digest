# 第一课 什么是程序

### 鱼和渔 \(Declarative Knowledge and Imperative Knowledge\)

Declarative Knowledge 是指正确的知识，而 Imperative Knowledge 指的是能够获取一类 Declarative Knowledge 的步骤。这与我们的俗语 “授人以鱼不如授人以渔” 中的鱼和渔暗合。再举一例，小学时背诵的乘法口诀表，就是 Declarative Knowledge，它让我们熟记 81 种一位数乘法的结果；而小学学习的乘法竖式计算法，就是 Imperative Knowledge，它让我们拥有计算任意整数之间的乘法结果。我们的记忆有限，无法在脑海中记诵无穷多种整数相乘的结果，但 Imperative Knowledge 能够让我们按需产生 Declarative Knowledge，简直完美。讲了这么多，其实就是想引出：

**Imperative Knowledge, 即获取一类 Declarative Knowldge 的步骤，被称为程序 — procedure；而计算机内部实际按照程序执行的步骤，成为进程 — process。**

### 描述程序 \(计算机语言\) 的要素

我们用自然语言与人交流，与计算机交流同样需要语言。对比自然语言，计算机语言拥有以下构成要素：

1. 词典 \(vocabulary\)：对应汉语字词，英语单词，计算机语言通常也有一定的保留词语 \(reserved keywords\)，比如：int, double, char, boolean 等等，这些词语是我们描述程序的基础。

2. 语法 \(syntax\)：有了汉语字词，但如果我们随意拼接这些字词，就无法准确描述意图，如：吗思意我懂你。词典中的字词按一定**方法**组合起来，才能有效表明意图，这里的方法即指为**语法**。

3. 语义 \(semantics\)：语义指的是在解释程序时的一套规则，语义层在合乎语法的程序与最终程序的执行方法之间。

### 描述程序的目的

描述程序的真实目的是**控制大型系统的复杂性**。为了要达到这个目的，通常我们会递归地重复下面的过程：

1. 创建一些语言的原始元素 \(primitives\) — 简单的数据和程序 \(simple data and sinple procedures\)

2. 创建组合语言元素的方法 \(means of combination\)

3. 创建抽象语言元素的规则 \(means of abstraction\) — 把利用原始元素构造出来的语言元素看作是新的原始元素，忽略其内部细节，以此为基础继续构造更抽象的语言元素。

为什么通过这种方式可以帮助我们实现控制大型系统的复杂性呢？

我们以烹饪为例，如果让一个厨师回到远古时代，没有油盐酱醋，那么他可能需要掌握：

* 生火技术

* 植物油提取技术

* 晒盐技术

* 黄豆种植及酱油提取技术

* 大米种植及醋的酿造技术

如果烹饪是一个大型系统，以上这些技术就是原始元素，而对应的 火、油、盐、酱、醋则是这些原始元素抽象出来的高级原始元素，当这些元素被抽象出来后，厨师就可以只需要掌握如何利用这些高级原始元素进行烹饪的技术，减少他所需掌握的技术以及工作量。而常人就可以使用抽象程度更高的老干妈，甚至酸菜鱼酱料包等等元素，轻松烹饪复杂菜品。

### 控制系统复杂性的工具

1. 抽象 \(Abstraction\)：利用简单元素构建复杂元素，同时对使用者隐藏背后的实现细节。
2. 标准接口 \(Conventional interfaces\) 和编程范式 \(Programming paradigms\)：二者为我们提供将不同组件结合起来的方式。
3. 元语言抽象 \(Meta-linguistic abstraction\)：为特定问题设计新的语言。



**有了可以执行的程序还需要有实体真正去执行，它就是计算机。那程序与计算机之间是如何沟通呢？**



### 从比特 \(bit\) 到原始对象 \(primitive objects\)

要与计算机交流，我们需要知道如何表达信息 \(information representation\) — 数字和符号。

##### 数字表达

计算机通过区分高低电压来表达基本信息，因此表达数字最原始的方法就是使用二进制变量，0或1，这个变量可以表达 1 比特信息。通过将比特组合成字节 \(byte\) 和字 \(word\)，我们可以表达更大的数字。通过符号位，我们可以表达正负数，通过指数位，我们可以表达小数。

##### 符号表达

我们不仅可以使用比特的组合表达数字，也可以用它们来编码符号。

尽管我们可以仅仅依靠比特来表达程序，但这种方式像结绳计数一般过于低级，严重降低表达效率，因此我们需要**抽象**的帮助 — 假设我们已经拥有一些原始对象 （基本数据结构），以及对这些对象的一些基本操作，然后利用它们来构造表达程序。它们一般包括：数字 \(Numbers\)、字符 \(Characters\) 和布尔 \(Booleans\)。

### Scheme 表达式的评价规则 \(Rules for evaluation\)

1. 自评价 \(self-evaluating\) 表达式：返回自评价值
2. 名字 \(name\) 表达式：返回环境中与之绑定的值
3. 特殊形式 \(special form\) 表达式：依照特殊规则进行评价
4. 组合 \(combination\) 表达式:
   * 评价所有子表达式 （顺序不定\)
   * 对子表达式评价结果执行操作后返回结果

### 参考

[Youtube: MIT6.001-SICP-2004-lecture-1](https://www.youtube.com/watch?v=FIUJd_ZFmGo&t=2s&list=PL7BcsI5ueSNFPCEisbaoQ0kXIDX9rR5FF&index=1)

[MIT6.006-SICP-2005-lecture-notes-1](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-001-structure-and-interpretation-of-computer-programs-spring-2005/lecture-notes/lecture1webhand.pdf)

