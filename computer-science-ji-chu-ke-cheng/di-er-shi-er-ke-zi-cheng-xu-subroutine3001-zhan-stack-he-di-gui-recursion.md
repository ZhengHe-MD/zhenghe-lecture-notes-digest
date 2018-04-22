# 第二十二课 - 子程序 \(subroutine\)、栈 \(stack\) 和递归 \(recursion\)

上节课我们构建了迭代求解最大公约数的寄存器机器，本节我们将为寄存器机器新增两个组件子程序 \(subroutine\) 和栈 \(stack\)。本节要点：

* 每个程序与子程序之间需要遵守规定的契约
* 栈是实现递归算法的基石

#### 子程序

顾名思义，子程序就是一系列固定的指令：

* 以一个标记 \(label\) 为起点，进入子程序，结束后可以跳转回父程序继续执行
* 可以从许多地方进入子程序，从而达到复用的目的

为支持子程序，register machine 需要新增两个指令

* **\(assign continue \(label after-call-1\)\): **将父程序中，执行完子程序后的命令地址的标记 **\(label \(after-call-1\)\)** 记录到 continue 寄存器中
* **\(goto \(reg continue\)\):** 将寄存器 PC 的值修改成之前存储在 continue 当中标记指向的指令地址，从而达到返回父程序的目的

##### 例子：increment

```scheme
(controller
  (assign sum (const 0))
  (assign continue (label after-call-1))       ; 1
  (goto (label increment))                     ; 2
after-call-1
  (assign continue (label after-call-2))       ; 4
  (goto (label increment))                     ; 5
after-call-2
  (goto (label done))                          ; 6
increment
  (assign sum (op +) (reg sum) (const 1))
  (goto (reg continue))                        ; 3
done)                                          ; 7
```

1. 主程序将调用子程序 increment 结束后的指令地址标记 after-call-1 存入 continue 寄存器中
2. 主程序调用子程序 increment
3. 子程序 increment 执行完自增操作后，返回 continue 寄存器中存储的标记指向的返回地址
4. 主程序将调用子程序 increment 结束后的指令地址标记 after-call-2 存入 continue 寄存器中
5. 主程序调用子程序 increment
6. 子程序 increment 执行完自增操作后，返回 continue 寄存器中存储的标记指向的返回地址
7. 主程序继续执行 \(goto \(label done\)\) 后，程序运行结束

##### 子程序与父程序的契约包括：

* 存储子程序输入参数以及返回地址的寄存器
* 存储子程序输出返回值的寄存器
* 除了返回值所在的寄存器，其它可能被修改的寄存器

如果这些契约没有被遵守，子程序调用就会失效，甚至可能造成程序崩溃。

如，increment 子程序与父程序之间的契约：

* 输入参数和返回地址的寄存器: sum, continue
* 返回值寄存器: sum
* 其它寄存器：无

##### 为什么需要子程序？

* 复用程序段
* 提高程序可读性
* 支持递归

### 栈

Stack 支持两种操作，save 和 restore，对应 push 和 pop，它支持存储多个数值，且最后一个进栈的元素将第一个出栈 \(Last In First Out\)，举例如下：

```scheme
(controller
  (assign a (const 0))
  (assign b (const 5))
  (save a)
  (save b)
  (restore a)
  (restore b))
```

上面这段指令就是利用栈的 LIFO 特点，交换 a、b 寄存器中的数值。

#### 栈深 \(stack depth\)

栈的深度就是任意时刻，栈中所含元素的数量。

> stack depth = \(\# of saves\) - \(\# of restores\)

而最大栈深 \(max stack depth\) 指的是某程序在执行时，所需栈深的最大值。栈深最小值为 0，最大值与所能获取的最大内存相等。

#### 栈与父子程序之间的契约

有了栈以后，父子程序之间的契约还应该包括栈相关的信息：

##### 标准契约

一般情况下，子程序在被调用期间不会修改栈信息，以 increment 为例：

* 输入参数和返回地址的寄存器: sum, continue
* 返回值寄存器: sum
* 其它寄存器：无
* 栈：不变

##### 非标准契约

特殊情况下，子程序可以在被调用期间修改栈信息，举例如下：

```scheme
strange
  (assign val (op *) (reg val) (const 2))
  (restore continue)
  (goto (reg continue))
```

* 输入参数和返回地址的寄存器： val, return point on top of stack
* 返回值寄存器：val
* 其它寄存器：continue
* 栈：栈顶的元素出栈

该契约将原本存储在 continue 中的信息压入栈中，子程序通过出栈操作获取调用返回地址。

#### 尾递归优化

通常递归需要占用和递归深度一致的空间，但如果递归的返回值并不需要额外后处理的话，我们可以更机智地使用栈，将递归的过程优化成迭代的过程，从而使得空间占用达到常数级别，但依然可以保持递归程序的可读性。由于尚未介绍递归，这里暂不举例。

### 递归

以阶乘为例：

```scheme
(define (fact n)
  (if (= n 1) 1
      (* n (fact (- n 1)))))
```

利用替代模型，我们可以看到 \(fact 3\) 的执行过程：

```scheme
(fact 3)
(* 3 (fact 2))
(* 3 (* 2 (fact 1)))
(* 3 (* 2 1))
(* 3 2)
6
```

要完成这个过程，register machine 需要：

* 记住每次递归调用的返回地址
* 记住每个中间结果

正好子程序和栈可以帮助 register machine 做到：

```scheme
(controller
  (assign continue (label halt))
fact
  (test (op =) (reg n) (const 1))
  (branch (label b-case))
  (save continue)
  (save n)
  (assign n (op -) (reg n) (const 1))
  (assign continue (label r-done))
  (goto (label fact))
r-done
  (restore n)
  (restore continue)
  (assign val (op *) (reg n) (reg val))
  (goto (reg continue))
b-case 
  (assign val (const 1))
  (goto (reg continue))
halt)
```

##### base case:

```scheme
;...
(if (= n 1) 1
   ; ...
```

对应 register machine instructions:

```scheme
;...
fact
  (test (op =) (reg n) (const 1))
  (branch (label b-case))
;...
b-case
  (assign val (const 1))
 (goto (reg continue))
```

从中，我们也可以看出 fact 与父程序间的部分契约：

* 输入参数寄存器: n
* 返回地址寄存器: continue
* 返回值寄存器: val

##### recursive case:

```scheme
(define (fact n)
  ; ...
      (fact (- n 1))
  ; ...)
```

对应 register machine instructions:

```scheme
    ;...
    (assign n (op -) (reg n) (const 1))
    (assign continue (label r-done))
    (goto (label fact))
r-done
    (assign val (op *) (reg n) (reg val))
    (goto (reg continue))
```

与 base case 中提到的契约一样，这里在递归调用 fact 之前，按照契约，把参数 n 和返回标记 \(label r-done\) 放到对应的寄存器中。在执行完递归调用的 fact 后，再把 val 与当前的 n 相乘，结果存入 val 寄存器中作为结果输出。

但是，在上述过程中，我们多次覆盖寄存器 n 与 continue 中存储的数值，因此在覆盖之前，我们要将会被覆盖的寄存器的值压入栈中，然后在需要的时候从栈中弹出。具体体现在了 save 和 restore 命令中。

##### fact 与父程序间的完整契约：

* 输入参数与返回地址寄存器：n, continue
* 返回值寄存器：val
* 其它被修改的寄存器：无
* 栈：不变

###### 明明修改了 n 和 continue，为什么说没有其它被修改的寄存器？

因为虽然 fact 中修改了 n 和 continue，但是由于父程序在调用 fact 之前已经将原先的 n 和 continue 压入栈中，待子程序返回时从栈中弹出，从整个过程来看，fact 并没有真正修改 n 和 continue。

###### 明明有出入栈的操作，为什么 stack 不变？

因为在 fact 递归调用前入栈的信息，在调用完成后全部出栈，整个过程来看栈并没有发生变化。



#### 参考

* [Youtube: SICP-2004-Lecture-22](https://www.youtube.com/watch?v=JX9K8OISaKk&index=22&list=PL7BcsI5ueSNFPCEisbaoQ0kXIDX9rR5FF&t=0s)



