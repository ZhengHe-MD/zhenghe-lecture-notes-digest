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

##### Sequences: 递归调用 eval-dispatch 与尾递归优化

```scheme
ev-begin
  (save continue)
  (assign unev (op begin-actions) (reg exp))
  (goto (label ev-sequence))

; ev-sequence: used by begin and apply (lambda bodies)
;
; inputs:  unev      list of expressions
;          env       environment in which to evaluate
;          stack     top value is return point
; writes:  all       (calls eval without saving)
; output:  val
; stack:   top value removed
ev-sequence
  (assign exp (op first-exp) (reg unev))
  (test (op last-exp?) (reg unev))
  (branch (label ev-sequence-last-exp))
  (save unev)
  (save env)
  (assign continue (label ev-sequence-continue))
  (goto (label eval-dispatch))
ev-sequence-continue
  (restore env)
  (restore unev)
  (assign unev (op rest-exps) (reg unev))
  (goto (label ev-sequence))
ev-sequence-last-exp
  (restore continue)
  (goto (label eval-dispatch))
```

* ev-sequence-last-exp 没有再进行额外的入栈出栈开销，做了尾递归优化，使得 sfact 这样尾递归表达式能够按照迭代过程的方式来执行。但尾递归优化无法对 env 和 unev 使用，原因在于二者在递归调用的过程中可能被修改，如果不压入栈中就会丢失对应的信息。
* ev-sequence-continue 没有使用 val 寄存器，原因在于 sequence 语句，只以最后一条语句的返回值作为最终返回结果，中间表达式的返回结果都忽略不计。

##### Applications

###### apply-dispatch

```scheme
; inputs:    proc    procedure to be applied
;            argl    list of arguments
;            stack   top value is return point
; writes:    all     (calls ev-sequence)
; output:    val
; stack:     top value removed

apply-dispatch
  (test (op primitive-procedure?) (reg proc))
  (branch (label primitive-apply))
  (test (op compound-procedure?) (reg proc))
  (branch (label compound-apply))
  (goto (label unknown-procedure-type))
```

###### primitive-apply

```scheme
primitive-apply
  (assign val (op apply-primitive-procedure) (reg proc) (reg argl))
  (restore continue)
  (goto (reg continue))
```

###### compound-apply

```scheme
compound-apply
  (assign unev (op procedure-parameters) (reg proc))
  (assign env (op procedure-environment) (reg proc))
  (assign env (op extend-environment)
              (reg unev) (reg argl) (reg env))
  (assign unev (op procedure-body) (reg proc))
  (goto (label ev-sequence))
```

###### ev-application

```scheme
ev-application
  (save continue)

ev-appl-operator
  (assign unev (op operands) (reg exp))
  (save env)  
  (save unev)
  (assign exp (op operator) (reg exp))
  (assign continue (label ev-appl-did-operator))
  (goto (label eval-dispatch))
ev-appl-did-operator
  (restore unev)
  (restore env)
  (assign proc (reg val))
  ; eval argl
  (assign argl (op empty-arglist))
  (test (op no-operands?) (reg unev))
  (branch (label apply-dispatch))
  (save proc)
ev-appl-operand-loop
  (save argl)
  (assign exp (op first-operand) (reg unev))
  (test (op last-operand?) (reg unev))
  (branch (label ev-appl-last-arg))
  ;; eval one operand
  (save env)
  (save unev)
  (assign continue (label ev-appl-accumulate-arg))
  (goto (label eval-dispatch))
ev-appl-accumulate-arg
  (restore unev)
  (restore env)
  (restore argl)
  (assign argl (op adjoin-arg) (reg val) (reg argl))
  (assign unev (op rest-operands) (reg unev))
  (goto (label ev-appl-operand-loop))
ev-appl-last-arg
  (assign continue (label ev-appl-accum-last-arg))
  (goto (label eval-dispatch))
ev-appl-accum-last-arg
  (restore argl)
  (assign argl (op adjoin-arg) (reg val) (reg argl))
  (restore proc)
  (goto (label apply-dispatch))
```

由于 operator 自身可能包含 application，它们可能会 extend-environment，也会使用到 unev 寄存器，因此在 eval operator 的时候，需要把当前的 env、uenv 寄存器压入栈中保存起来。eval operator 结束时，结果会被存在 val 寄存器中，ev-appl-did-operator 把它放入 proc 寄存器中，等待下一步被调用。

接下来 ev-appl-operand-loop、ev-appl-last-arg 和 ev-appl-accum-last-arg 一同 eval application 的参数，最后执行 application。

现在我们完成了 EC-EVAL 的主要组件，也能体验到用 register machine 构建 universal machine，也即 evaluator 的过程。

### 小结

本节通过大量代码讲解了如何用 register machine 构建 universal machine 的过程，让我们对 scheme evaluator 的底层拥有最基本的了解。非研究目的代码不必细扣。

#### 参考

* [Youtube: SICP-2004-Lecture-23](https://www.youtube.com/watch?v=1eQpcms7c98&t=1538s)



