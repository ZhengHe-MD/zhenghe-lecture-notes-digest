# SOLID 原则

solid 原则之于编程就如美德之于人，它是一种**抽象的规范**，**不涉及具体的做法**，符合原则的行为更有利于提高代码质量。

## S - Single Responsibility Principle

Single Responsibility Principle 要求我们在每个实体 \(class、method、interface\) 中只完成一个事情，或者联系紧密的一类事情。这就好比流水车间，每个任务由不同的步骤构成，每个流水线工人只在一个步骤中完成一件或一类事情。对工人来说，可以使自己在某项工作专心研究，提高效率；对工厂来说，每个工人不需要了解所有步骤，培养成本低、工作职责明确、便于管理。管理工厂和管理程序实际上都是管理复杂系统，Single Responsibility Principle 在两个领域不约而同地形成、固化。

## O - Open/Close Principle

> software entities \(classes, modules, functions, etc.\) should be **open **for extension, but **closed **for modification

这条原则说的是写代码的时候需要注意代码本身的扩展性 — 在未来新增功能时，应尽可能少地修改当前代码。



