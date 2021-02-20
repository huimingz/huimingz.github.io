---
title: Clean Architecture笔记
author: huimingz
category: [architect]
tags: [architect]
pin: false
future: true
---

Clean Architecture 是Robert C. Martin(uncle Bob)在2012年提出的结构设计，目的是为了保证DB和framework的独立性。

![](/assets/img/posts/2021/02-15-post-clean-architecture-01.jpeg)

## Object-Oriented
面向对象的三大特性：Encapsulation（封装）、Inheritance（继承）、Polymorphism（多态）。

### Encapsulation（封装）
是指利用抽象数据类型将数据和基于数据的操作封装在一起，使其构成一个不可分割的独立实体，数据被保护在抽象数据类型的内部，尽可能地隐藏内部的细节，只保留一些对外接口使之与外部发生联系。

意义：
1. 将属性和方法放到一起做为一个整体，然后通过实例化对象来处理；
2. 隐藏内部实现细节，只需要和对象及其属性和方法交互就可以了；
3. 对类的属性和方法增加 访问权限控制。

好处：
1. 良好的封装能够减少耦合。
2. 类内部的结构可以自由修改。
3. 可以对成员进行更精确的控制。
4. 隐藏信息，实现细节。

实现对比：C语言对封装的实现很好，在 `.h` 文件中你只能看到被定义的结构体名称和方法定义。但是C++作为一门面向对象的语言反而做得不够好，它将成员变量也定义到了 `.h` 文件中，这在一定程度上破坏了封装，而其他面向对象的语言，比如Java、Python等更糟糕，声明和实现都在一个文件中。

### Inheritance（继承）

继承是使用已存在的类的定义作为基础建立新类的技术，新类的定义可以增加新的数据或新的功能，也可以用父类的功能，但不能选择性地继承父类。

特性：
1. 子类拥有父类非private的属性和方法。
2. 子类可以拥有自己属性和方法，即子类可以对父类进行扩展。
3. 子类可以用自己的方式实现父类的方法。

好处：
1. 避免代码重复，提高代码的复用性。
2. 类与类之间产生了关系，是多态的前提。
3. 修改父类，影响所有的子类，便于修改，修改一次接口。

### Polymorphism（多态）
指程序中定义的引用变量所指向的具体类型和通过该引用变量发出的方法调用在编程时并不确定，而是在程序运行期间才确定，即一个引用变量倒底会指向哪个类的实例对象，该引用变量发出的方法调用到底是哪个类中实现的方法，必须在由程序运行期间才能决定。

好处：
1. 减少了重载方法的数量
2. 符合开闭原则，即使增加子类，不需要提供额外的方法

### OO是什么？
虽然对于面向对象语言存在很多定义，不同的人有不同的看法和答案，但是对于软件架构师而言这个问题的答案是十分明确的：OO是利用多态的能力来获得对系统中每一个源代码依赖的绝对控制能力，它允许架构师创建一个插件架构。包含高层策略的模块独立于包含低层细节的模块。低级细节被归入插件模块，可以独立于包含高级策略的模块进行部署和开发。

## SOLID原则

五大原则：
1. SRP: Single Responsibility Principle（单一职责原则）
2. OCP: The Open-Closed Principle（开放-封闭原则）
3. LSP: The Liskov Substitution Principle（里氏替代原则）
4. ISP: The Interface Segregation Principle（接口隔离原则）
5. DIP: The Dependency Inversion Principle（依赖性反转原则）

### Single Respinsibility Principle（单一职责原则）
一个系统的最佳结构是在很大程度上受到使用它的组织社会所影响的，所以 **每一个软件模块有且只能有一个改变它的理由**。

### The Open-Close Principle（开放-封闭原则）
要使系统易于改变，就必须在设计上允许通过追加新的代码来改变这些系统的行为，而不是改变现有的代码。

对拓展开放，对修改保守。在支持新功能时，可以很好的利用拓展来进行实现，而不需要修改代码。

> The OCP is one of the driving forces behind the architecture of systems. The goal is to make the system easy to extend without incurring a high impact of change. This goal is accomplished by partitioning the system into components, and arranging those components into a dependency hierarchy that protects higher-level components from changes in lower-level components.
>
> OCP是系统架构的驱动力之一。其目标是使系统易于扩展，而不产生较大的变化影响。实现这一目标的方法是将系统分割成若干组件，并将这些组件安排成一个依赖层次，以保护较高级别的组件不受较低级别的组件变化的影响。

### The Liskov Substitution Principle（里氏代换原则）
要想从可互换的部件中构建软件系统，这些部件必须遵守一个契约，允许这些部件被一个个替换掉。

LSP应该被拓展到架构的层面上。一个简单的违反可替代性的行为，就会导致系统的架构受到大量额外机制的污染（造成开发和维护负担等）。

### The Interface Segregation Principle（接口隔离原则）
这个原则建议软件设计者避免依赖那些他们不用的东西。无论从架构层面还是代码层面来看，依赖那些无用的东西都是有害的，往往会造成一些你不期待的一些问题。

### The Dependency Inversion Principle（依赖反转原则）
实现高层次策略的代码不应该依赖于实现低层次细节的代码。相反，细节应该依赖于策略。

依赖反转的原则：
1. 高层次的模块不能依赖于低层次的模块。
2. 所有的模块都应该依赖于抽象。
3. 抽象不依赖于实现。
4. 实现应该依赖于抽象。

![](https://hmz-storage.oss-cn-shenzhen.aliyuncs.com/static/img/2021/02-16-Xnip2021-02-16_16-35-53.png)

## Components

### Component Cohesion（组合的内聚）
三个原则：
1. REP: The Reuse/Release Equivalence Principle（复用/发布等效原则）
2. CCP: The Common Closure Principle（共同封闭原则）
3. CRP: The Common Reuse Principle（共同重用原则）

#### REP（复用/发布等效原则）

#### CCP（共同封闭原则）
建议将所有类中可能由于相同原因而发生改变的都聚集到同一个地方。当两个类关系紧密，如论是物理上还是概念上总是一起发生变化，那么它们就应当属于同一个组建。这样可以最大限度地减少与软件的发布、重新验证和重新部署相关的工作量。

这一原则与开放-封闭原则（OCP）密切相关。事实上，CCP所需要解决的正是OCP意义上的“封闭”。OCP指出，类应该为修改而封闭，但为扩展而开放。因为100%的封闭是不可能实现的，所以封闭必须是战略性的。我们在设计我们的类时，要使它们对我们预期的或经历过的最常见的变化是封闭的。

CCP是SRP的组件形式。SRP告诉我们，如果方法因为不同的原因发生变化，我们要把方法分成不同的类。CCP告诉我们，如果类因为不同的原因发生变化，要把类分成不同的组件。这两个原则可以用下面的口诀来概括。
> 把那些在相同时间和相同原因下发生变化的事物聚集在一起。把那些在不同时间或不同原因发生变化的事物分开。

#### CRP（共同重用原则）

## 参考
1. [《Clean Architecture》](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164)
2. [【ボブおじさんのClean Architectureまとめ】オブジェクト指向 ~SOLIDの原則~](https://qiita.com/yoshinori_hisakawa/items/25576a62123607a696f6)
3. [実装から学ぶクリーンアーキテクチャ](https://gist.github.com/MegaBlackLabel/ef0f1ae19491c3d75823d6b26db97bd9)

