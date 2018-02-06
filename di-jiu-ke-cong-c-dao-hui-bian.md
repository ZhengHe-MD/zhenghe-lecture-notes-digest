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
// M[R1+4] = 10;  		// store operation

// j = i + 7;
// R2 = M[R1+4]; 		// load operation
// R3 = R2 + 7; 		// ALU operation
// M[R1] = R3; 		        // store operation

// j++;
// R2 = M[R1];
// R2 = R2+1;
// M[R1] = R2;
```



