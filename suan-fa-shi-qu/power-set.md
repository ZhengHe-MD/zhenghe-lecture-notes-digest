# Power Set

一个集合的所有子集构成这个集合的 Power Set。

#### 描述：

输入：任意一个集合

输出：输入集合的 Power Set

#### 分析：

一个集合的任意一个元素在任一子集中，要么存在，要么不存在，假设输入集合的大小为 N，那么该集合的子集总数为 2^N。直观地推理，由于至少要生成 2^N 个元素，这个算法的时空复杂度应该在 O\(2^N\)。

把大问题拆成小问题。假设输入集合 S 的大小为 n，如果我们已经知道该集合 S\[1: n\]，记为 S1， 的 Power Set_，_记为 PS1_，_那么整个集合的 Power Set 就可以看作是 S1 的每个子集分别与 S\[0\] 的合取后形成的集合，与 PS1 集合本身的合取。

#### 实现：

##### Scheme:

```scheme
(define (power-set s)
  (if (null? s) '(())
    (let ((ps1 (power-set (cdr s))))
      (append ps1
              (map (lambda (ele)
                     (cons (car s) ele))
                   ps1))))
```

##### Python:

```py
def power_set(s):                                            # complexity (#calls * complexity/op)
    if len(s) == 0:                                          #    n * 1
        return [[]]                                          #      1
    else:
        ps1 = power_set(s[1:])                               #    F(n-1)
        s1 = s[0]                                            #    n * 1
        ps1_s1 = list(map(lambda x: s1 + x, ps1))            #    F(n-1)
        return ps1 + ps1_s1                                  #    2F(n-1)
```

#### 复杂度分析：

由于 scheme 中的 append 、python 中的 ps1 + ps1\_s1 操作都是 θ\(m+n\) ，设计算集合 （大小为 n）的 power-set 的时空复杂度为 F\(n\)，则可以推导出：

F\(n\) = 4F\(n-1\) + n + c ~= 4 F\(n-1\) &gt; 4^n F\(0\) = c \* O\(4^n\)

具体分析：

* append 以及 + 操作都被执行 2^n 次，且每次执行的时间复杂度为 F\(n-1\)，从推导可以看出时间复杂度为 O\(4^n\)

* append 以及 + 操作每次执行的空间复杂度为 F\(n-1\)，从推导可以看出空间复杂度为 O\(4^n\)

#### cProfile

执行 cProfile

```py
import cProfile

cProfile.run('power_set(list(range(20)))')
```

可以得到

```
1048620 function calls (1048600 primitive calls) in 1.374 seconds

   Ordered by: standard name

   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.083    0.083    1.374    1.374 <string>:1(<module>)
  1048575    1.038    0.000    1.038    0.000 powerset.py:10(<lambda>)
     21/1    0.253    0.012    1.291    1.291 powerset.py:4(power_set_1)
        1    0.000    0.000    1.374    1.374 {built-in method builtins.exec}
       21    0.000    0.000    0.000    0.000 {built-in method builtins.len}
        1    0.000    0.000    0.000    0.000 {method 'disable' of '_lsprof.Profiler' objects}
```

可以发现 lambda 确实被执行了 2^20 次。

