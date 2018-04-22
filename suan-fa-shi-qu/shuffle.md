# Shuffle

Shuffle 就是随机打乱一个数组，比如洗牌。

#### 问题描述：

输入：任意数组

输出：随机打乱顺序后的数组

#### 分析：

这里首先要明确什么叫做 "**随机打乱顺序**"：就是经过处理后，原数组中的任意一个元素在新数组中任意位置的概率相等。因此本题的算法需要严谨地证明。

##### 法1：

我们可以创建一个临时数组 tmpArr，它保存着原数组的副本。每次从其中无放回的选一个出来，依次放到输出的数组当中，直到 tmpArr 为空：

```py
import random

def shuffle(arr):                                                   # complexity (calls * complexity/op)
    tmp_arr = arr[:]                                                #   1 * n
    new_arr = []                                                    #   1 * 1

    while len(tmp_arr) > 0:
        rand_int = random.randint(0, len(tmp_arr) - 1)              #   n * 1
        new_arr.append(tmp_arr.pop(rand_int))                       #   n * k ~= n * (n/2)

    return new_arr
```

根据右边注释对每行代码复杂度的分析，可以得出本算法的时间复杂度为 O\(n^2\)。由于整个过程只分配 tmpArr 和 newArr 用于存储数组，因此空间复杂度为 O\(n\)。

**证明**：设 arr 的长度为 n，对 arr 中任意一个元素 e

1. e 被放在 newArr 第一个位置的概率为 1/n
2. e 被放在 newArr 第二个位置的概率为 \(n-1\)/n \* \(1 / \(n-1\)\) = 1/n
3. ...
4. e 被放在 newArr 第 n 个位置的概率为 \(n-1\)/n \* \(n-2\)/\(n-1\) \* .... \* \(1/2\) = 1/n

**法2**：

是否可以实现 O\(n\) 复杂度？看一下 Fisher-Yates shuffle Algorithm：

```py
import random

def shuffle(arr):                                                    # complexity (calls * complexity/op)
    for i in reversed(range(1, len(arr))):                           
        j = random.randint(0, i)                                     #   n * 1
        arr[i], arr[j] = arr[j], arr[i]                              #   n * 1
```

根据右边注释对每行代码的复杂度分析，可以得出本算法的时间复杂度为 O\(n\)，整个过程中没有用到额外的存储空间，因此空间复杂度为 O\(1\)。插个题外话，arr\[i\], arr\[j\] = arr\[j\], arr\[i\] 的是否有用到额外的空间：

```py
>>> import dis
>>> def f(a, b):
...    a, b = b, a
...
>>> dis.dis(f)
  2           0 LOAD_FAST                1 (b)
              2 LOAD_FAST                0 (a)
              4 ROT_TWO
              6 STORE_FAST               0 (a)
              8 STORE_FAST               1 (b)
             10 LOAD_CONST               0 (None)
             12 RETURN_VALUE
```

可以看出，从 bytecode 中可以看出并没有用到额外的空间，而只是旋转了一下栈顶部的两个元素。

**证明**：设 arr 的长度为 n，对 arr 中任意元素 e

1. e 被放在第 n 个位置的概率为
   * 若 e 为原 arr 第 n 个元素：则概率为第一轮与自己交换的概率 1/n
   * 若 e 不是 arr 第 n 个元素：则概率为第一轮与第 n 个元素交换的概率 1/n
2. e 被放在第 n-1 个位置的概率为 第一轮不被选中而第二轮被选中的概率，即 \(n-1\)/n \* \(1/\(n-1\)\) = 1/n
3. ...
4. e 被放在第 1 个位置的概率为前 n-1 轮都不被选中而第 n 轮被选中的概率，即 \(n-1\)/n \* \(n-2\)/\(n-1\) \* ... \* \(1/2\) = 1/n

#### 参考

* [GeeksforGeeks: Shuffle a given array](https://www.geeksforgeeks.org/shuffle-a-given-array/)



