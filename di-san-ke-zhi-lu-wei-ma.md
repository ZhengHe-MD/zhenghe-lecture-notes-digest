# "指鹿为马" — 玩指针

#### 指 int 为 float

```c
int i = 37;
float f = *(float *)&i;
// &i            => 取 i 的地址，类型为 int *
// (float *)&i    => 指鹿为马 -- 假装它是个 float *
// *(float *)&i    => 取出 float* 所指的值，类型为 float
// 以上实现了指 int 为 float，因为本质上它们都是 4 个字节排列在一起

printf("%f \n", f); // 0.000000 

// i: 00000000 00000000 00000000 00100101
// f: 00000000 00000000 00000000 00100101

// 1.00000000xxxxxx * 2^(-127) 约等于 0
```

#### 指 float 为 short

```c
float f = 7.0;
short s = *(short *)&f;
// &f             => 取 f 的地址，类型为 float *
// (short *)&f     => 指鹿为马 -- 假装它是个 short *
// *(short *)&f => 取出 short * 所指的值，类型为 short

printf("%s \n", s); // 输出结果视 endianness 而定

// Big Endian:
// f: 01000000 11100000 00000000 00000000
// s: 01000000 11100000                 

// Little Endian:
// f: 01000000 11100000 00000000 00000000
// s:                   00000000 00000000
```

#### 指 double 为 char

```c
double d = 3.1416;
char ch = *(char *)&d;
// &d             => 取 d 的地址，类型为 double *
// (char *)&d    => 指鹿为马 -- 假装它是个 char *
// *(char *)&d     => 取出 char * 所指的值，类型为 char

printf("%c \n", ch); // 输出结果视 endianness 而定

// Big Endian:
// d:  01000000 | 00001001 | 00100001 | 11111111 | 01001000 | 11101000 | 10100111
// ch: 01000000

// Little Endian:
// d:  01000000 | 00001001 | 00100001 | 11111111 | 01001000 | 11101000 | 10100111
// ch:                                                                   10100111
```

#### 数组与指针算术（pointer arithmetic）

```
int array[10];
// array 等价于第一个元素的地址
array === &array[0]
array + k === &array[k]
*array === array[0]
*(array + k) === array[k]

// 指针算术是指，对指针做加减法时，会自动乘上指针所指数据类型所占的字节数

int array[5]; array[3] = 128;
((short *)array)[6] = 2;
// array 			=> 数组起始地址，类型为 int *
// (short *)array 	=> 指鹿为马 -- 假装它是个 short *，即 array 中的元素为 short 类型
// (short *)array[6]=> 将 short array 的第六个元素改成 2

// Big Endian:
// array:              => [][][][00000000 00000000 00000001 00000000][]
// (short *)array[6]=2 => [][][][00000000 00000010 00000001 00000000][]
// 输出 768

// Little Endian:
// array:              => [][][][00000000 00000000 00000001 00000000][]
// (short *)array[6]=2 => [][][][00000000 00000000 00000000 00000010][]
// 输出 2
```



