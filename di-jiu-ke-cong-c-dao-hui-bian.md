# 第九课 - 从 C 到汇编

本课 Jerry Chain 老师探讨了 C 代码编译以后的汇编语言的形式。C 代码编译后：

1. 所有 C 语言类型消失

2. 所有指针操作消失，如指针计算、&、\* 以及 cast 等等

3. ...

简而言之，C 语言向编程者提供的各种抽象都不复存在，仅剩赤裸裸的内存操作。

### 符号说明

本课中，R 指代 General Purpose Registers \(GPR\)，R1 即第一个 GPR，R2 即第二个 GPR 等等。PC 为 Program Counter Register，它永远指向当前正在执行的汇编指令的地址。为了方便，本课中，R1 永远指向当前 Activation Record 的最低地址。此外，本课中的各种汇编指令默认操纵 4 字节数据。

另外，本文所用的汇编指令不是真正的汇编指令，仅仅是为了课程讲解需要构造的，但实际情况与此出入不大。

#### 例1：简单计算和赋值

```c
int i;
int j;

i = 10;
j = i + 7;
j++;

// ===== compiled to
// R -> general purpose registers
// R1 -> stores the base address of the activation record

// i = 10;
// M[R1+4] = 10;         // store operation

// j = i + 7;
// R2 = M[R1+4];         // load operation
// R3 = R2 + 7;          // ALU operation
// M[R1] = R3;           // store operation

// j++;
// R2 = M[R1];
// R2 = R2+1;
// M[R1] = R2;
```

本例内存的栈结构如下图所示：![](/assets/cs107-9-example-1.jpg)R1 指向当前 Activation Record 的最低地址，即为整型变量 j 的地址，而 R1 + 4 即为整型变量 i 的地址。j = i + 7 被编译成三个步骤，将内存地址 R1+4 中的连续 4 个字节复制到 R2 中，利用 ALU \(Arithmetic Logic Unit\) 计算 R2 + 7 的结果并存储到 R3 中，再将 R3 中的 4 个字节复制到内存地址 R1 中。接着，读入内存地址 R1 存储的 4 个字节数据到 R2，利用 ALU 自增 1 并写回 R2，再将计算结果写回原地址。这个过程抛弃了所有类型信息，只对 4 个字节进行读取、计算、写出。

##### 问题1：为什么不直接将 j = i + 7 编译成 M\[R1\] = 10 + 7

##### 问题2：为什么不直接将 j++ 编译成 M\[R1\] = 17 + 1

为了使编译出来的汇编语言更具扩展性，即如果把**i = 10**改成**i = 11**，j = i + 7 被编译而成的三个步骤无需改变。同理，j++; 被编译而成的三个步骤也无需改变。

#### 例2：隐形类型转换

```c
int i;
short s1;
short s2;

i = 200;      
s1 = i;
s2 = s1 + 1;

// i = 200;
// M[R1+4] = 200;

// s1 = i;
// M[R1+2] = M[R1+4];
// R2 = M[R1+4];
// M[R1+2] = .2R2; // only copy 2 bytes

// s2 = s1 + 1;
// R2 = .2M[R1+2]
// R3 = R2 + 1;
// M[R1] = .2R3;
```

本例的内存栈结构如下图所示：![](/assets/cs107-9-example-2.jpg)

本例 **s1 = i **中 i 是整型，占 4 字节；s1 是短整型，占 2 字节，把整型赋数值赋给短整型，将只取其中两个字节，具体取哪两个字节取决于多字节数据类型的存储方式是大端还是小端。因此将 R2 中存储的 200 写回 R1+2 时，只写回原 4 个字节中的 2 个字节，因此 **M\[R1+2\] = .2R2**。s2 = s1 + 1，这里被编译成了 3 个汇编指令，分别是将 s1 写入 R2 中，利用 ALU 将 R2 自增的结果存到 R3 中，然后写回内存地址 R1 中。这里 **R2 = .2M\[R1+2\] **只将内存 R1+2 处的 2 个字节复制到 R2 中，而 R2 为 4 个字节，余下的两个字节会自动填 0，而写出时也只写出 2 个字节，因此数值溢出产生的结果也就可以理解了。

#### 例3：循环体

本例的内存栈结构如下图所示：

![](/assets/cs107-9-example-3.jpg)

本例中 R1 为整数 i 的地址，R1+4 为数组 array 的地址。这里为了复用循环体，使用了

1. 汇编指令 BGE \(branch if greater than\) ，当它的第一个操作数大于第二个操作数时，将往前跳转到第三个操作数所指的指令地址。其中 PC 为 Program Counter，它永远指向当前正在执行的汇编指令地址。

2. 汇编指令 JMP，将跳到唯一的操作数所指的指令地址。

```c
int array[4];
int i;

for (i=0; i < 4; i++) {
  array[i] = 0;
}
i--; 

// M[R1] = 0;
// R2 = M[R1];
// BGE = R2,4, PC + 40;

// array[i] = 0;
// R3 = M[R1];
// R4 = R3 * 4;
// R5 = R1 + 4;
// R6 = R4 + R5;
// M[R6] = 0;
// R2 = M[R1];
// R2 = R2 + 1;
// M[R1] = R2;
// JMP PC - 40;
// i--;
```

#### 例4：指针强制转换

```c
struct fraction {
  int num;
  int dnum;
};

struct fraction pi;
pi.num = 22;
pi.dnum = 7;
((struct fraction *)&pi.dum) -> dnum = 451;

// M[R1] = 22;
// M[R1+4] = 7;
// M[R1+8] = 451;
```

本例内存栈结构如下图所示:

![](/assets/cs107-9-example-4.jpg)

我们发现复杂的指针强制转换语句最后竟然只被编译成一个简单的汇编指令！原因在于，指针、指针类型等都是 C 语言在编程上添加的抽象，只要通过了编译器的检查，最终这些抽象都会消失，编程赤裸裸的字节数据操作。

#### 参考

* [Stanford CS107: lecture 9](https://www.youtube.com/watch?v=arjo2-JQeaY&list=PL9D558D49CA734A02&index=9&t=155s)

* [Github: ZhengHe-MD - lecture codes](https://github.com/ZhengHe-MD/cs107-lecture-codes)



