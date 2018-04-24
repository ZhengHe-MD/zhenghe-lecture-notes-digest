# 第二十三课 - Eval in register machine

在上一节课中，我们的 register machine 增加了对 subroutine 和 stack 的支持，可以顺利执行一些过程 \(procedure\)，如 gcd、factorial 等。既然如此，我们是否可以用它来构建通用机 \(universal machine\)，即之前的 evaluator？构建完 evaluator 后，我们的 简易 CPU 也就完成了。

### Tail-recursive Evaluator

#### 例子：sfact

```scheme
(define sfact (lambda (n prod)
  (display prod)
  (if (= n 1) prod
      (sfact (- n 1) (* n prod)))))
```

###### sfact 是 iterative process 还是 recursive process？

sfact 虽然调用了自己，但由于它在调用 subroutine 前把 prod 当作参数传递，因此它的状态无需保存在栈中，这时候栈深是常数级别，因此整个过程是 iterative process。

> 输入一个 iterative procedure，如果一个 Evaluator 的栈深没有随着程序的自调用而增大，那么这个 Evaluator 被称为 Tail-recursive Evaluator

#### Tail-call Optimization

Tail-recursive Evaluator 的核心技术就是 Tail-call Optimization \(尾递归优化\)。

### Machine for EC-EVAL

#### 组件：

* 7 registers
  * exp           保存表达式 \(expression to be evaluated\)
  * env           保存当前环境 \(current environment\)
  * continue  保存返回地址 \(return point\)
  * val            保存输出结果 \(resulting value\)
  * unev         还未执行的表达式列表 \(list of unevaluated expressions\)、临时寄存器 \(temporary register\)
  * proc          过程 \(operator value\)
  * argl           参数 \(arguments values\)
* Many abstract operations
  * 假设一些与 syntax、environment model 有关的操作都以 primitive procedures 的形式存在，它们可以由更基本的操作构成，但这里为了描述方便，假设它们已经存在

#### Main entry point

EC-EVAL 的 entry point 是 eval-dispatch，如下所示：

```scheme
; inputs:    exp          expression to evaluate
;            env          environment
;            continue     return point
; output:    val          value of expression
; writes:    all          (except continue)
; stack:     unchanged

eval-dispatch
  (test (op self-evaluating?) (reg exp))
  (branch (label ev-self-eval))
  (test (op variable?) (reg exp))
  (branch (label ev-variable))
  ...
  (goto (label unkown-expression-type))
```

eval-dispatch 与调用者之间的契约在注释中注明。eval-dispatch 的形式也与十七课中的 eval 一致，按顺序判断表达式类型，然后把表达式传递给相应的 eval-helpers，这些过程与调用者之间的契约与 eval-dispatch 一致，以下是几个例子：

```scheme
ev-self-eval
  (assign val (reg exp))
  (goto (reg continue)
ev-variable
  (assign val (op lookup-variable-value)
              (reg exp) (reg env))
  (goto (reg continue))
ev-lambda
  (assign unev (op lambda-parameters)
               (reg exp))
  (assign exp (op lambda-body) (reg exp))
  (assign val (op make-procedure)
              (reg unev) (reg exp) (reg env))
  (goto (reg continue))
```

* ev-self-eval 负责处理 self-evaluating 表达式，因为 self-evaluating 表达式的值就是表达式本身，因此 ev-self-eval 直接将 exp 寄存器中的信息直接放入 val 寄存器中，然后返回到 continue 指向的返回地址
* ev-variable 负责在环境中查询变量的 binding，因此 ev-variable 取出 exp 中的变量表达式以及 env 中的环境，利用 lookup-variable-value 过程从环境中取出对应的变量值，放入 val 寄存器中，然后返回 continue 指向的返回地址
* ev-lambda 负责处理 lambda 表达式，lambda 表达式需要存储参数、过程体以及当时的环境，因此 ev-lambda 依次从 exp 中取出参数和过程体，然后从 exp 中取出环境，用 make-procedure 将三者打包，放入 val 寄存器中，最后返回 continue 指向的返回地址。

##### ev-definition: 递归调用 eval-dispatch

与上文中 eval-helpers 的例子 ev-self-eval、ev-variable 和 ev-lambda 不同，ev-definition 需要递归调用 eval-dispatch：

```scheme
ev-definition
  (assign unev (op definition-variable) (reg exp))
  (save unev)
  (save env)
  (save continue)
  (assign exp (op definition-value) (reg exp))
  (assign continue (label ev-definition-1))
  (goto (label eval-dispatch))
ev-definition-1
  (restore continue)
  (restore env)
  (restore unev)
  (perform (op define-variable!)
           (req unev) (reg val) (reg env))
  (assign val (const ok))
  (goto (reg continue))
```

由于 eval-dispatch 可能会修改 uenv、env、continue 寄存器，同时在递归调用结束后 ev-definition 还需要原先这些寄存器里的数据，因此在递归调用之前，需要把它们压入栈中，调用完成中再从栈中弹出。

##### ev-if: 递归调用 eval-dispatch 与尾递归优化

```scheme
ev-if
  (save exp)
  (save env)
  (save continue)
  (assign continue (label ev-if-decide))
  (assign exp (op if-predicate) (reg exp))
  (goto (label eval-dispatch))
ev-if-decide
  (restore continue)
  (restore env)
  (restore exp)
  (test (op true?) (reg val))
  (branch (label ev-if-consequent))
ev-if-alternative
  (assign exp (op if-alternative) (reg exp))
  (goto (label eval-dispatch))
ev-if-consequent
  (assign exp (op if-consequent) (reg exp))
  (goto (label eval-dispatch))
```

* 递归调用 eval-dispatch 获取 predicate 的结果，同样在递归调用之前将 uenv、env、continue 压入栈中，递归调用之后再将它们从栈中弹出
* 在调用 alternative 和 consequent 时，由于 ev-if 无需对 alternative 和 consequent 的返回值做后处理，因此 ev-if 使用尾递归优化 \(Tail-call optimization\)。如果没有尾递归优化，它们的写法如下所示：

```scheme
ev-if-alternative
  (save continue)
  (assign continue (label alternative1))
  (assign exp (op if-alternative) (reg exp))
  (goto (label eval-dispatch))
alternative1
  (restore continue)
  (goto (label continue))
ev-if-consequent
  (save continue)
  (assign continue (label consequent1))
  (assign exp (op if-consequent) (reg exp))
  (goto (label eval-dispatch))
consequent1
  (restore continue)
  (goto (label continue))
```



