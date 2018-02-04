# 第二课 - 程序抽象

### 定义计算模式 - lambda 表达式

```scheme
> (lambda (x) (* x x))
#<procedure>
```

lambda 表达式是 Scheme 中的一种特殊形式表达式，它返回的不是原始对象，不是名字与某对象的绑定，也不是组合，而是一个通用的计算模式 \(common pattern of computation\)，也可以称为一段程序 \(procedure\)。在这段程序中，有参数 \(parameters\) 和程序体 \(body\)。我们称 Scheme 中原始环境中存在的许多程序段，称为原始程序 \(primitive procedure\)，使用 lambda 表达式生成的程序段称为复合程序 \(compound procedure\)。

因此，在 Scheme 表达式的评价规则中，在执行程序段时，我们可以加入以下两条：

1. 如果是原始程序，就按照其定义的方式执行。

2. 如果是复合程序，那么我们将 lambda 函数体中的变量全部替换成输入的参数，然后对其进行评价。

举例如下：

```scheme
> ((lambda (x) (* x x)) 5) ; evaluated as (* 5 5)
25
```

结合名字表达式的评价规则，我们可以将复合程序与某个名字绑定，避免每次重新输入复合程序：

```scheme
> (define square (lambda (x) (* x x)))
> (square 5)
25
```

**总结**：lambda 表达式的评价值可以被理解为程序对象 \(procedure object\)，它的内部存有对参数和函数体的表达，而这个程序对象在被应用时 \(即：在组合表达式中\)，会将程序体重的参数替换成实际给定的参数，然后将函数体评价后的结果返回。

### lambda 表达式与程序抽象

lambda 表达式本身即是程序抽象的描述，即描述获取 declarative knowledge 的步骤。如果这些步骤比较复杂，我们就需要将步骤进一步划分为多个模块 \(module\) 或者阶段 \(stages\)。这样做包括但不局限于以下两个好处：

* 有些模块或阶段可以在其它程序中重复使用

* 模块或阶段将程序的细节和程序的使用隔离，使用者无须了解具体实现。

举例如下：

```scheme
; 利用勾股定理计算直角三角形斜边的长度

> (define square (lambda (x) (* x x)))
> (define sum-squares
    	(lambda (x y) (+ (square x) (square y))))
> (define pythagoras
    	(lambda (y x) (sqrt (sum-squares y x))))
```

抽象程序的过程，通常可以总结为一下 4 个步骤：

1. 找出程序中的模块或阶段

2. 利用程序抽象来描述这些模块或阶段

3. 建立一个大的程序来联系模块或阶段之间的输入输出

4. 在模块或阶段中递归地重复以上步骤

##### 更复杂的例子 - 牛顿法求平方根

1. 猜一个数，设为G

2. 如果 G 和目标足够接近，停止

3. 否则，更新猜测的值为 \(G + X/G\) / 2

判断是否足够接近的模块：

```scheme
> (define squre
    (lambda (x) (* x x)))
> (define close-enuf?
    (lambda (guess x)
      (< (abs (- (square guess) x)) 0.001)))
```

更新猜测值的模块：

```scheme
> (define average
    (lambda (a b) (/ (+ a b) 2)))

> (define improve
    (lambda (guess x)
      (average guess (/ x guess))))
```

整合整个程序：

```scheme
> (define sqrt-loop (lambda G X)
  	(if (close-enuf? G X)
        G
        (sqrt-loop (improve G X) X)))

> (define sqrt
    (lambda (x)
      (sqrt-loop 1.0 x)))
```

#### 参考

* [Youtube: MIT6.001-SICP-2004-lecture-2](https://www.youtube.com/watch?v=51gPEp0hRoQ&list=PL7BcsI5ueSNFPCEisbaoQ0kXIDX9rR5FF&index=2)

* [MIT6.006-SICP-2005-lecture-notes-2](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-001-structure-and-interpretation-of-computer-programs-spring-2005/lecture-notes/lecture2webhand.pdf)



