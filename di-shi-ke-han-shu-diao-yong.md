# 第十课 函数的活动记录 - More Activation Record

### 例：简单函数调用过程中，活动记录的产生和回收

```c
void foo(int bar, int *baz) {
    char snink[4];
    short *why;

    why = (short *) (snink + 2);
    *why = 50;
}
```

当函数 foo 在内存中运行时，我们可以大胆猜测 **bar**、**baz、snink、why **应该会被存储在一起，实际上也确实如此：![](/assets/Screen Shot 2018-02-01 at 9.09.24 PM.jpg)这就是 foo 函数的活动记录，它有以下几个特点：

1. 函数的参数从右往左，依次从高内存地址往低内存地址方向摞起来 --- 第一个参数在最低地址处
2. 函数的局部变量从上往下，依次从高内存地址往低内存地址方向摞起来 --- 第一个参数在最高地址处
3. 在函数的参数和局部变量之间留有一段空间 \(4 字节或 8 字节\)，用来存储 foo 函数执行完的返回地址，也称为 saved pc.

接下来再看看 foo 函数被其它函数调用时，函数的活动记录是如何变化的：

```c
int main(int argc, char **argv) {
    int i = 4;
    foo(i, &i);
    return 0;
}
```

main 函数执行过程中，函数栈里 main 和 foo 的活动记录如下图所示：![](/assets/Screen Shot 2018-02-01 at 9.49.28 PM.jpg)**①： **main 函数执行时，有两个参数 argc, argv 被放在了 main 的活动记录中，并在 saved pc 中保留了 main 函数执行结束后 pc 的位置

②：执行到 int i = 4; 时，main 函数需要为局部变量 i 分配空间，即 SP = SP - 4，并将 i 的值初始化，即 M\[SP\] = 4，

③：这时候 main 函数遇到了调用 foo 函数的指令，但在此之前，需要为 foo 函数生成一半活动记录，上面记录有 foo 函数的两个参数 bar 和 baz。在执行完 foo 之后，通过 SP = SP + 8 来回收分配给这两个参数的空间。因此，foo 函数的 saved pc 里面存的就是指令 SP = SP + 8；的地址。初始化了 foo 函数的 saved pc 后，就可以正式把控制权转交给 foo 函数

④：foo 函数先为它的两个局部变量分配空间，即 SP = SP - 8，然后再为两个局部变量 bar 和 baz 赋值。这里由于受到指令集及硬件的限制，无法实现 M\[SP\] = M\[SP + 8\] 以及 M\[SP + 4\] = SP + 8 这种指令，因此需要先将变量的值放入通用寄存器 R1, R2 中，再将值存入内存中，接着执行后两条语句，这里略过。

⑤：foo 函数体执行完毕后，执行 SP = SP + 8 回收局部变量空间，这时栈指针 SP 指向 foo 函数的 saved pc

⑥：执行 RET 会将 saved pc 的值存到 PC 中，foo 函数就将控制权转交回给了 main 函数

⑦：main 函数执行 SP = SP + 8，回收为 foo 函数参数分配的空间，然后将返回值 0 存入专用寄存器 RV 中，等待 main 函数返回

### 活动记录的一般模式

从上面的例子中可以抽象出活动记录的一般模式：![](/assets/Screen Shot 2018-02-01 at 10.29.33 PM.jpg)每个活动记录包含 **参数**、**返回地址 **和 **局部变量**，其中 **参数** 由函数调用方负责分配空间以及初始化；**局部变量 **则由函数执行方负责分配空间及初始化。原因如下：

1. 只有函数调用方知道参数的个数及取值
2. 只有函数的执行方知道局部变量的个数及取值

综上所述，活动记录由函数的调用方和执行方共同生成，当执行方完成工作后，会回收自己负责分配和初始化的局部变量，然后将控制权转交回给函数的调用方，由函数的调用方负责回收函数的参数，并最终完成整个活动记录的回收。

### 例：递归函数的执行过程中，活动记录的产生与回收

```c
int fact(int n) {
    if (n == 0) return 1;
    return n * fact(n - 1);
}
```

假设某函数在某处调用了 fact\(3\)，则从调用开始之后的活动记录变化过程如下图所示![](/assets/Screen Shot 2018-02-01 at 11.00.35 PM.jpg)

①：某函数在某处调用 fact，并将参数 n 的空间分配好且初始化为 3，将返回地址放入 saved pc 中后控制权转交 fact 函数

②③：fact 函数执行，读入参数 n 并利用 BNE 指令来判断是返回 1 还是继续递归执行 n \* fact\(n-1\)，值得注意的是，当递归的 fact 函数执行结束时，外层 fact 函数相信里层 fact 函数已经把它的返回值放入特殊寄存器 RV 中，因此它直接将参数 n 与 RV 相乘的值存入 RV 中，执行 RET 语句将控制权交给外层调用者。

④：fact 函数执行，读入参数 n，值为 0，因此 BNE 指令的跳转判断为否，继续执行之后的指令，把 1 放入 RV 中并将控制权转交外层 fact 函数

⑤⑥⑦：fact 函数继续执行 CALL&lt;fact&gt; 之后的指令，将参数 n 与 RV 中的返回结果相乘，并存回 RV 中，继续将控制权转交给外层函数。

### 参考资料

* [Stanford CS107: lecture 10](https://www.youtube.com/watch?v=FvpxXmEG1F8&index=10&t=3s&list=PL08D9FA018A965057)
* [Github: ZhengHe-MD - lecture codes](https://github.com/ZhengHe-MD/cs107-lecture-codes)



