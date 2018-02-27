# 第十一课：C 与 C++ 代码生成

### Swap Example

#### C code generation

```c
void foo() {
    int x;
    int y;
    
    x = 11;
    y = 17;
    
    swap(&x, &y);
}

void swap (int *ap, int *bp) {
    int temp = *ap;
    *bp = *ap;
    *ap = temp;
}
```

* 调用 foo

foo 函数执行时，运行时中的函数栈如下图所示：![](/assets/Screen Shot 2018-02-27 at 1.36.49 PM.jpg)

saved pc 中保存着 foo 执行完成后返回的地址。前三行初始化 foo 函数的局部变量，中间省略调用 swap 函数的过程，在最后一行回收局部变量，sp 重新指向 saved pc 然后回到调用地址。

* 调用 swap

swap 函数执行时，运行时中的函数栈如下图所示：![](/assets/Screen Shot 2018-02-27 at 1.40.10 PM.jpg)

在调用 swap 函数之前，初始化 swap 函数的参数 ap 和 bp，然后才开始调用 swap，中间省略 swap 函数执行的过程，在最后一行回收局部变量，sp 重新指向 saved pc 然后回到调用地址。

* 执行 swap![](/assets/Screen Shot 2018-02-27 at 1.43.16 PM.jpg)

swap 函数执行，首先分配局部变量 temp，完成交换功能后，回收局部变量 temp 并回到 saved pc 所指的调用地址。

#### C++ code generation

```c
void swap(int& a, int& b) {
    int temp = a;
    a = b;
    b = temp;
}
```



