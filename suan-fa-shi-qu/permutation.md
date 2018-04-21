# Permutation

permutation 就是找到数列的全排列

### 问题描述：

**输入：**任意数组

如 \[1, 2, 3\]

**输出：**数组的全排列

如\[\[1, 2, 3\], \[1, 3, 2\], \[2, 1, 3\], \[2, 3, 1\], \[3, 1, 2\], \[3, 2, 1\]\]

### 分析：

任意长度为 n 的数组，全排列的数量为 n!，所以 permutation 的算法时空复杂度至少是 O\(n!\)。本题也可以使用将大问题拆分成小问题的思路去解决，如将问题描述中的输出示例格式化：

```
[[1, 2, 3], [1, 3, 2],
 [2, 1, 3], [2, 3, 1],
 [3, 1, 2], [3, 2, 1]]
```

每一行都是输入中的某个元素，和剩下元素的全排列分别组合的结果，顺着思路，我们用可以用递归来解决。

### 递归解：

#### scheme:

```scheme
(define (remove items item)
  (if (null? items) '()
    (if (equal? item (car items))
        (remove (cdr items) item)
        (cons (car items) (remove (cdr items) item)))))


(define (permute items)
  (if (null? items) '(())
    (apply append
      (map (lambda (item)
             (map (lambda (pm) (cons item pm))
           (permute (remove items item))))
      items))))
```

#### python:

```py
def permute(l):
    if len(l) == 0:
        return [[]]
    else:
        pmt = []
        for i, e in enumerate(l):
            ll = [x for x in l if x != e]
            pmtll = permute(ll)
            for pmtl in pmtll:
                pmtl.append(e)      # append(e) 是 O(1) 复杂度，insert(0, e) 是 O(n) 复杂度
                pmt.append(pmtl)
        return pmt
```

我们来 profile 一下 python 的递归方案：

```py
import cProfile

def permute(l):
    pass # copy the code from above

if __name__ == "__main__":
    cProfile.run('permute(list(range(5)))')
```

得到：

```
       2180 function calls (1855 primitive calls) in 0.001 seconds

   Ordered by: standard name

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.000    0.000    0.001    0.001 <string>:1(<module>)
      325    0.000    0.000    0.000    0.000 permute.py:10(<listcomp>)
    326/1    0.001    0.000    0.001    0.001 permute.py:4(permute)
        1    0.000    0.000    0.001    0.001 {built-in method builtins.exec}
      326    0.000    0.000    0.000    0.000 {built-in method builtins.len}
     1200    0.000    0.000    0.000    0.000 {method 'append' of 'list' objects}
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
```

5！= 120，然而 permute 被调用了 326 次，hmmmmmm....

仔细分析不同数量数列被 permute 执行的次数不同：

* 元素数量为0：permute\(\[\]\) 被调用 120 次
* 元素数量为1：permute\(\[1\]\) 被调用 24 次
* 元素数量为2：permute\(\[1, 2\]\) 被调用 6 次
* 元素数量为3：permute\(\[1, 2, 3\]\) 被调用 2 次
* 元素数量为4：permute\(\[1, 2, 3, 4\]\) 被调用 1 次
* 元素数量为5：permute\(\[1, 2, 3, 4, 5\]\) 被调用 1 次

因此 permute 被调用的总次数为：

120 \* 1 + 24 \* 5 + 6 \* 10 + 2 \* 10 + 5 \* 1 + 1 = 326

和 profile 的结果一致。不难推算，对于长度为 n 的数组，permute 被调用的次数为：

n! \* \(1 + 1/2! + 1/3! + ... + 1/\(n-1\)!\) ~= \(e - 1\)n!

具体数学推导过程可以参考 [wolframalpha](http://www.wolframalpha.com/input/?i=1+%2B+1%2F2!+%2B+1%2F3!+%2B+...+%2B+1%2Fn!)

##### 时间复杂度：

T\(n\) = n\*\(T\(n-1\) + T\(n-1\)\) = 2\*n T\(n-1\) = ... = 2^n \* n! = O\(2^n \* n!\)

##### 空间复杂度：

由于所用的临时空间综合都小于 pmt 的最终占用空间大小，因此：

S\(n\) = O\(n!\)

### 迭代解：

#### Heap's Algorithm （待续）









