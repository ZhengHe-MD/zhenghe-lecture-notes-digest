# 第十一课：C 与 C++ 代码生成

## Reference 与 Pointer

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

```cpp
void swap(int& a, int& b) {
    int temp = a;
    a = b;
    b = temp;
}
```

实际上，c++ swap 编译后的结果和 C 代码编译后的结果完全一致。只是 C++ 在编译过程中，识别了 reference 语法，完成相应的代码转换，使得最终代码与 C 版本代码生成的结果一致。

### Simpler Example

#### c code

```c
int x = 17;
int y = x;
int *z = &y;
```

#### c++ code

```cpp
int x = 17;
int y = x;
int& z = y;
```

二者编译后的结果是一样的，三个变量的关系如下图所示：![](/assets/Screen Shot 2018-02-27 at 9.32.21 PM.jpg)值得一提的是，reference 和 pointer 的区别在于，**reference 一旦绑定，就无法更改**。如本例所示，在 c++ 代码中，一旦 z 指向 y，z将无法被修改；而在 c 代码中，指针可以重新赋值。

## c++ Class

#### c++ code

```cpp
class binky {
    public:
        int dunky(int x, int y);
        char * minky(int *z) {
            int w = *z;
            return slinky + dunky(winkey, winkey);
            // return this.slinky + this.dunky(this.winkey, this.winkey);
        }
    private:
        int winky;
        char *blinky;
        char slinky[8];
}

int n = 17;
binky b;
b.minky(&n); // binky::minky(&b, &n);
```

class instance 在调用 instance method 的时候，会自动将实例的指针作为第 1 个参数传给 instance method，既 b.minky\(&n\) 实际调用过程是 binky::minky\(&b, &n\), 其运行时的函数栈如下图所示：

![](/assets/Screen Shot 2018-02-27 at 9.54.52 PM.jpg)

> c code 和 c++ code 只是在 assembly code 上加的一层语法。一旦 c 和 c++ 编译成 assembly，所有高级语言的概念如 class、reference、pointer、template 都将消失。



### 参考资料

* [Stanford CS107: lecture 11](https://www.youtube.com/watch?v=DwTXMjVkIUY&t=1200s&list=PL9D558D49CA734A02&index=11)



