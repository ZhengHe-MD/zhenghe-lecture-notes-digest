# 第三课 - 替代模型

### 替代模型是什么

替代模型实际上是一种思维模型，让我们可以在心里推导一个**表达式**在计算机中的演进过程。理解替代模型可以帮助我们更合理地设计程序，从而使程序向我们所希望的方向演进。

Scheme 的替代模型可以概括如下：

1. 如果是自评价 \(self-evaluating\) 表达式，直接返回其值

2. 如果是名字表达式，使用名字对应的值替代原表达式

3. 如果是 lambda 表达式，创建程序 \(procedure\) 并将其返回

4. 如果是特殊形式表达式，依照特殊规则来评价子表达式 \(sub-expressions\)，如 if, cond 等

5. 如果是复合 \(compound or combination\) 表达式：

   5.1 任意顺序评价子表达式

   5.2 如果第一个子表达式是原始程序，直接应用到子表达式的评价值上

   5.3 如果第一个子表达式是复合程序，则将子表达式的评价值替换复合程序 \(lambda 表达式创造\) 的函数体中的对应变量，然后将整个函数体表达式替换原复合表达式，将其作为新的表达式重复评价过程

举个简单例子： \(注释对应替代规则\)

```scheme
(define square (lambda (x) (* x x)))

; 1. (square 4)                 // 5.1
;   1. 4 -> 4                   // 1
;   2. Square -> #procedure     // 2
; 2. (* 4 4)                    // 5.3
;   1. 4 -> 4                   // 1
;   2. 4 -> 4                   // 1
; 3. 16                         // 5.2
```

举个复杂的例子：\(注释对应替代规则\)

```scheme
(define square 
  (lambda (x) (* x x)))
(define average
  (lambda (x y) (/ (+ x y) 2)))

; 1. (average 5 (square 3))      // 5.1
;   1. average -> #procedure     // 2
;   2. 5 -> 5                    // 1
;   3. (square 3)                // 5.1
;       1. square -> #procedure  // 2
;       2. 3 -> 3                // 1
;       3. (* 3 3)               // 5.3
;       4. 9                     // 5.2
; 2. (/ (+ 5 9) 2)               // 5.3，5.1
;   1. (+ 5 9) -> 14             // 5.1
;     1. 5 -> 5                  // 1
;     2. 9 -> 9                  // 1
;   2. 2 -> 2                    // 1
; 3. 7                           // 5.2
```

递归例子：

```scheme
(define fact
  (lambda (n)
      (if (= n 1)
        1
        (* n (fact (- n 1))))))

; 1. (fact 3)                                         // 2, 5.3
; 2. (if (= 3 1) 1 (* 3 (fact (- 3 1))))              // 4 (if)
; 3. (if #f 1 (* 3 (fact (- 3 1))))                   // 4 (if)
; 4. (* 3 (fact (- 3 1)))                             // 5.2
; 5. (* 3 (fact 2))                                   // 2, 5.3
; 6. (* 3 (if (= 2 1) 1 (* 2 (fact (- 2 1)))))        // 4 (if)
; 7. (* 3 (if #f 1 (* 2 (fact (- 2 1)))))             // 4 (if)
; 8. (* 3 (* 2 (fact (- 2 1))))                       // 5.2
; 9. (* 3 (* 2 (fact (1))))                           // 2, 5.3
; 10.(* 3 (* 2 (if (= 1 1) 1 (* 1 (fact (- 1 1))))))  // 4(if)
; 11.(* 3 (* 2 (if #t 1 (* 1 (fact (- 1 1))))))       // 4(if)
; 12.(* 3 (* 2 1))                                    // 5.2
; 13.(* 3 2)                                          // 5.2
; 14.6
```

注意到递归的过程中有两个子过程

1. 充分展开表达式至基本情况（base case），对应 1, 5, 9, 12 步

2. 合并表达式的过程，对应 12, 13, 14 步

### 利用替代模型设计递归 \(Recursive\) 算法

如果一个问题可以被分成一个或多个同样的但规模更小的问题，然后再将这些小问题的结果通过简单的计算汇总，那么我们就可以用递归的方式来描述这样的过程。以 fact 为例：

1. 假设解决问题的程序已经存在, fact \(n\) 存在

2. 分解问题成同样的但规模更小的问题 \( recursive case \), fact\(n\) = n \* fact\(n - 1\)

3. 找到最小的问题，利用简单的计算合并结果 \( base case \), fact\(1\) = 1

### 利用替代模型设计迭代 \(Iterative\) 算法

从 fact 的递归算法中，我们可以看到在抵达 base case 之前，算法内存储着所有需要最终和 base case 结果相乘的乘子，这需要占用 O\(n\) 的空间。如果利用乘法的结合律，在计算的过程中，记录当下之前的所有乘子的乘积，就可以使得算法的空间占用达到 O\(1\)，这就是另一种算法设计思路 — 迭代。

```scheme
(define ifact
  (lambda (n)
    (ifact-helper 1 1 n)))

(define ifact-helper
  (lambda (product counter n)
    (if (> counter n)
        product
        (ifact-helper (* product counter) (+ counter 1) n))))
```

利用替代模型分析一个简单的例子，合并部分评价步骤：

```
1. (ifact 4)                               // 2
2. (ifact-helper 1 1 4)                    // 5.3, 4(if), 5.2
3. (ifact-helper 1 2 4)                    // 5.3, 4(if), 5.2
4. (ifact-helper 2 3 4)                    // 5.3, 4(if), 5.2
5. (ifact-helper 6 4 4)                    // 5.3, 4(if), 5.2
6. (ifact-helper 24 5 4)                   // 5.3, 4(if), 1
7. 24
```

对比递归版本的 fact 算法，我们可以看到整个过程占用空间没有增大，达到了 O\(1\)

注意：递归和迭代的区别不在于自我调用，迭代也可能会使用自我调用的写法。它们的本质区别在于问题是否被分解成相同的**但规模更小的问题**，同时**有一个已知解的最小问题**存在。

#### 附：证明递归法的正确性

有了算法，严谨的人依然会心有不安，凭什么我要相信这些步骤拼出来的程序能告诉我正确答案呢？通常我们有集中选择：

1. 权威证明 \(proof by authority\)：大牛说的都是对的

2. 统计证明 \(proof by statistics\)：用测试覆盖能想象到的所有情况

3. 精神胜利法 \(proof by faith\)：相信自己没错的

4. 形式证明 \(formal proof\)：使用数学逻辑确定正确性

在编码时，随意的团队会使用法 1、3， 严谨的团队会使用法 1、2，学术的团队会使用法 4 + 2。接下来我们尝试用法 4 证明 fact 的递归算法的正确性。

##### 从命题逻辑 \(Propositional Logic\) 到谓词逻辑 \(Predicate Logic\)

命题是一段正确或错误的描述，原子命题 \(atomic propositions\) 是不可分解的事实陈述，而更有趣，不那么显而易见的命题通常是由简单命题组合而成的复杂命题 \(compound propositions\)。建立复合命题有五种标准方法：

1. 合取 \(Conjunction, and\)**P^Q**

2. 析取 \(Disjunction, or\)**PvQ**

3. 否定 \(Negation, not\)**¬P**

4. 蕴涵 \(Implication, if P then Q\)**P → Q**

5. 等价 \(Equivalence, P if only if Q\)**P ⇔ Q**

它们对应的真值表如下：

| 真值表 |
| :---: |


| P | Q | P and Q | P or Q | not P | If P, then Q | P iff Q |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| F | F | F | F | T | T | T |
| F | T | F | T | T | T | F |
| T | F | F | T | F | F | F |
| T | T | T | T | F | T | T |

其中蕴涵的真值表比较令人费解。举例如下：

P：n &gt; 2

Q: n \* n &gt; 4

我们来看复杂命题 P → Q

如果 P 正确，Q 也正确，此时 P → Q 正确，符合蕴涵本身的含义

如果 P 正确，Q 不正确，此时 P → Q 错误，符合蕴涵本身的含义

如果 Q 正确，P 也正确，此时 Q → P 正确，但这不影响 P → Q 的正确性，后者依然正确

如果 Q 不正确，P 不正确，同理，不影响 P → Q 的正确性，后者依然正确

以此为基础，我们可以做一些逻辑推理，这里不细究

##### 从演绎推理到归纳法 — 证明 fact 递归算法的正确性

演绎推理 \(deduction\) 从基本原理出发，将一系列逻辑推理串联起来，是一种自底向上的证明方式。这种方式通常从一般情况到特殊情况。但 fact 递归算法有 \[1, +infinity\] 无穷多种输入情况，我们无法通过证明每一种情况正确来证明算法本身的正确性，因此我们需要另一种证明方法 — 归纳 \(induction\)。归纳法很简单：

```
∵ P(0)
∀n: P(n)→ P(n+1) 
∴ ∀n: P(n)
```

我们发现这个结构和递归的结构有着惊人的相似，证明如下：

1. fact 的基本情况，当 n=1 时，返回 1，正确

2. 假设 fact\(n\) 正确，\(n+1\) \* fact\(n\) = fact\(n + 1\)，即 n+1 的情况下也正确

可以看出，将问题分解成形式一样，规模更小的问题，重复直到基本情况的思路恰好就是归纳法的逆过程，因此用归纳法证明递归算法的正确性，是天生一对。或者说，你在写递归算法时，脑海里已经用归纳法证明了算法的正确性。

#### 参考

* [Youtube: MIT6.001-SICP-2004-lecture-3](https://www.youtube.com/watch?v=Yj1fm4PVQPM&list=PL7BcsI5ueSNFPCEisbaoQ0kXIDX9rR5FF&index=3)

* [[MIT6.006-SICP-2005-lecture-notes-](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-001-structure-and-interpretation-of-computer-programs-spring-2005/lecture-notes/lecture2webhand.pdf)3](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/6-001-structure-and-interpretation-of-computer-programs-spring-2005/lecture-notes/lecture3webhand.pdf)



