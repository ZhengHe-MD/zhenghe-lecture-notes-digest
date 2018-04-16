# Power Set

Power Set 就是求一个集合的所有子集。一个集合的任意一个元素在任一子集中，要么存在，要么不存在，假设输入集合的大小为 N，那么该集合的子集总数为 2^N。直观地推理，由于至少要生成 2^N 个元素，这个算法的时空复杂度应该是 O\(2^N\)。

#### 思路：

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
def power_set(s):
    if len(s) == 0:
        return [[]]
    else:
        ps1 = power_set(s[1:])
        s1 = s[0]
        ps1_s1 = list(map(lambda x: s1 + x, ps1))
        return ps1 + ps1_s1 
```



