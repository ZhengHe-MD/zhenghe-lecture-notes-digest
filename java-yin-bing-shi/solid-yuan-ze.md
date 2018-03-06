# SOLID 原则

solid 原则之于编程就如美德之于人，它是一种**抽象的规范**，**不涉及具体的做法**，符合原则的行为更有利于提高代码质量。

## S - Single Responsibility Principle

> A **class** should have only a single responsibility \(i.e. changes to only one part of the software's specification should be able to affect the specification of the class\)

Single Responsibility Principle 要求我们在每个实体 \(class、method、interface\) 中只完成一个事情，或者联系紧密的一类事情。这就好比流水车间，每个任务由不同的步骤构成，每个流水线工人只在一个步骤中完成一件或一类事情。对工人来说，可以使自己在某项工作专心研究，提高效率；对工厂来说，每个工人不需要了解所有步骤，培养成本低、工作职责明确、便于管理。管理工厂和管理程序实际上都是管理复杂系统，Single Responsibility Principle 在两个领域不约而同地形成、固化。

## O - Open/Close Principle

> Software entities \(classes, modules, functions, etc.\) should be **open **for extension, but **closed **for modification

这条原则说的是写代码的时候需要注意代码本身的扩展性 — 在未来新增功能 \(open\) 时，应尽可能少地修改当前代码 \(close\)。这句话至少对代码提出了两个要求，一是不同的模块之前耦合度要低，二是代码构建过程中要考虑到新增功能的方向。

## L - Liskov Substitution Principle

> Objects in a program should be replaceable with instances of their subtypes without altering the correctness of that program

这条原则翻译成人话就是在代码中**子类可以完全被父类替换 \(substitution\)**。

## I - Interface Segregation Principle

> "many client-specific interfaces are better than one general-purpose interface

这条原则可以理解成 Interface 的 Single Responsibility Principle，我们在往 Interface 里面增加方法的时候，应仔细考虑新增的方法是否符合该 Interface 的主题，是否每个实现该 Interface 的 Class/Interface 都理应实现这个新增的方法，如果不是，就应该考虑 Interface 的隔离 \(segregation\)。

## D - Dependency Inversion Principle

> one should "depend upon abstractions, \[not\] concretions."

我们在开发软件时，通常会先从低阶模块 \(Low-level module\) 开始，逐渐往高阶模块 \(High-level module\) 开发，因为高阶模块的实现依赖于低阶模块。这条原则说的是，高阶模块不应当直接依赖于低阶模块，在高阶模块和低阶模块之间应当新增一层抽象，帮助高阶模块与低阶模块互相解耦合。

### 小结

SOLID 原则互相之间并不是独立的，而是紧密关联的。这有点冷幽默，SOLID 原则本身并不 SOLID， lol~.

