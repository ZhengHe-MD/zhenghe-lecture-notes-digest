# 第二十一课 - 寄存器机器 \(Register Machines\)

在本课之前，我们基于 Scheme 环境建立抽象程度更高的 Scheme\* Evaluator，已经对利用抽象级别低的工具构建抽象级别高的工具有了概念；从本课开始，我们从另一个方向出发，从构建一个 CPU \(central processing unit\) 开始，看一看 Scheme 环境的底层。

## 设计一个 CPU

接下来，我们利用以下工具来构建一个机器语言是 Scheme 的 CPU：

* 电线 \(wires\)
* 逻辑电路 \(logics\)
* 寄存器 \(registers\)
* 序列控制器 \(control sequencer\)

##### 本节的目标：

* 在硬件中实现迭代算法
* 在硬件中实现递归算法

##### 最终目标：

在硬件中支持机器语言 Scheme。举例如下：

![](/assets/Screen Shot 2018-04-20 at 11.12.09 AM.jpg)

最终我们可以利用这个 CPU 构建一个终极机器，它可以接受任意一个procedure，如求最大公约数 \(GCD\)，然后调整自己内部的状态使之与 GCD 的逻辑相同，同样的输入产生同样的输出。这正是二十课中提到过的通用机，只不过它的抽象级别很低，直接利用的是 CPU 的机器语言。

### GCD

#### GCD in Scheme

```scheme
(define (gcd a b)
  (if (= b 0)
      a
      (gcd b (remainder a b))))
```

#### GCD circuit diagram

![](/assets/Screen Shot 2018-04-20 at 11.14.55 AM.jpg)

上图是 GCD 的逻辑电路，也代表了数据通路 \(datapath\)。以下是逻辑电路中的各元素说明：

* Button \(图中的圈圈叉叉\): 当 Button 被按下后，输入线 \(input wire\) 上的值流过输出线 \(output wire\)
* Register: 
  * 当有新的值从输入线上进入时，改变内部存储的值
  * 不断输出当前内部存储的值
* Operation: 接收输入线进入的数值，经过某个固定的函数，由输出线输出
* Test:
  * 输出是 true or false 的操作，将结果输出到 condition register

#### GCD instructions in register machine

```scheme
(controller
test-b
  (test (op =) (reg b) (const 0))
  (branch (label gcd-done))
  (assign t (op rem) (reg a) (reg b))
  (assign a (reg b))
  (assign b (reg t))
  (goto (label test-b))
gcd-done)
```

#### Connect instructions with circuit diagram

![](/assets/Screen Shot 2018-04-20 at 10.20.55 AM.jpg)

仅仅有逻辑电路并不是一个完整的 register machine，我们还需要别的组件来控制命令的执行，这个东西就是图中的 sequencer。sequencer 接受 condition register 的输入，来决定下一个要执行 instruction，这个 instruction 的地址总是被存在 sequencer 内部的 program counter register 中。图中的 instructions 则与逻辑电路图中的 Button 一一对应。

结合 GCD 的 register machine instructions：

* controller 就是刚才提到的 sequencer，控制程序的执行过程
* test-b、gcd-done 是一个 label，指向其下的命令地址，用于 branch 命令
* test 命令用于判断某个条件是否成立，成立则把 True 放入 condition register，不成立则把 False 放入 condition register
* branch 命令用于有条件跳转，当 consition register 中是 True 时，则程序跳转至 label 指向的地址，否则继续顺序执行
* goto 命令是无条件跳转，直接跳转到 label 指向的地址

#### Less-abstract GCD instructions

在前面的 GCD instructions 中，我们用到了 rem instruction，这种抽象程度较高的 instruction，被称为 abstract instruction，它可以被进一步拆分成更小的，更基本的 instruction。以下是一个更具体的 GCD instructions:

```scheme
(controller
test-b
  (test (op =) (reg b) (const 0))
  (branch (label gcd-done))
  (assign t (reg a))
rem-loop
  (test (op <) (reg t) (reg b))
  (branch (label rem-done))
  (assign t (op -) (reg t) (reg b))
  (goto (label rem-loop))
rem-done
  (assign a (reg b))
  (assign b (reg t))
  (goto (label test-b))
gcd-done)
```

这里把 rem 展开成了更基本的 instructions。

#### 参考

* [Youtube: SICP-2004-Lecture-21](https://www.youtube.com/watch?v=ikvAQ_lu31s&t=0s&list=PL7BcsI5ueSNFPCEisbaoQ0kXIDX9rR5FF&index=21)



