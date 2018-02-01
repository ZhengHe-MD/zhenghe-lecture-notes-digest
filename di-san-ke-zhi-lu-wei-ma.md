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

```c
int array[10];
// array 等价于第一个元素的地址
array === &array[0]
array + k === &array[k]
*array === array[0]
*(array + k) === array[k]

// 指针算术是指，对指针做加减法时，会自动乘上指针所指数据类型所占的字节数

int array[5]; array[3] = 128;
((short *)array)[6] = 2;
// array             => 数组起始地址，类型为 int *
// (short *)array     => 指鹿为马 -- 假装它是个 short *，即 array 中的元素为 short 类型
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

#### 结构体

```c
struct student {
    char *name;
    char suid[8];
    int numUnits;
};

int main(int argc, char **argv) {
    struct student pupils[4];           // 初始化了长度为 4 的 student 结构体数组
    pupils[0].numUnits = 21;            // 赋予第一个 student 的 numUnits 字段
    pupils[2].name = strdup("Adam");    // 将第三个 student 的 name 指向 heap 中初始化的一个字符串 "Adam\0"
    pupils[3].name = pupils[0].suid + 6;// 将第四个 student 的 name 指向第一个 student 的 suid 往后移 6 位之处
    strcpy(pupils[1].suid, "40415xx");  // 将字符串 "40415xx" 放入第一个 student 的 suid 中
    strcpy(pupils[3].name, "123456");   // 将字符串 "123456" 放入第四个 student 的 name, 即第一个 student 的 suid 往后移 6 位之处
}
```

如下图所示：

![](/assets/Stanford-CS107-3-pupils.jpg)

### 参考资料

* [Stanford CS107: lecure 3](https://www.youtube.com/watch?v=H4MQXBF6FN4&list=PL9D558D49CA734A02&index=3)

* [Github: ZhengHe-MD - lecture codes](https://github.com/ZhengHe-MD/cs107-lecture-codes)



