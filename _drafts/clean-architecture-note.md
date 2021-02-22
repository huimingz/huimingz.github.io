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
人们想重用软件组件的话，那么这些组件就应该有发布的历史记录，每个记录都有发布号。发布过程应该生成适当的通知和发布文档，以便用户可以决定是否使用和适应去集成新的版本。

比如在Python的第三方模块中，我们安装模块的时候要么是安装最新的，要么是因为一些原因而去安装那些旧的模块。当我们考虑升级模块的时候就必须去查看那些变更历史，是否存在破坏性的升级，避免因为直接升级而导致出现兼容问题。

从软件设计和架构上来看，类和模块必须属于一个聚合组，形成组件。组件不应该简单理解为保护随机类和模块，而是这些模块必须共有架构主旨或目的。

#### CCP（共同封闭原则）
建议将所有类中可能由于相同原因而发生改变的都聚集到同一个地方。当两个类关系紧密，如论是物理上还是概念上总是一起发生变化，那么它们就应当属于同一个组建。这样可以最大限度地减少与软件的发布、重新验证和重新部署相关的工作量。

这一原则与开放-封闭原则（OCP）密切相关。事实上，CCP所需要解决的正是OCP意义上的“封闭”。OCP指出，类应该为修改而封闭，但为扩展而开放。因为100%的封闭是不可能实现的，所以封闭必须是战略性的。我们在设计我们的类时，要使它们对我们预期的或经历过的最常见的变化是封闭的。

CCP是SRP（Single Responsibility Principle）的组件形式。SRP告诉我们，如果方法因为不同的原因发生变化，我们要把方法分成不同的类。CCP告诉我们，如果类因为不同的原因发生变化，要把类分成不同的组件。这两个原则可以用下面的口诀来概括。
> 把那些在相同时间和相同原因下发生变化的事物聚集在一起。把那些在不同时间或不同原因发生变化的事物分开。

#### CRP（共同重用原则）
> 不要强迫使用者去用它们不需要的组建。

CRP原则可以帮助我们决定哪些类和模块可以放到一个组件里，也告诉我们哪些类不应该放到一个组件里。

将哪些关联密不可分的类放到同一个组件中，同一个组件中的类或许是相互依赖的，高度聚合的。

※ CRP是ISP（Interface Separate Principle）关联：ISP建议我们在类中别依赖用不到的方法；CRP建议我们在组件中别依赖用不到的类。

#### 组件聚合张力图
这三个聚合原则有相互排斥的倾向。REP和CCP是包含型（inclusive）的原则：两者有让组件更庞大的趋势。CRP是排外型（exclusive）的原则，是得组件更加小。三个原则的平衡是好的架构师应该去解决的。

下面是一张组件聚合张力图，展示了三个聚合原则之间的联系。从顶点逆时针，每条边描述了放弃该原则的代价。

![聚合原则张力图](https://hmz-storage.oss-cn-shenzhen.aliyuncs.com/static/img/2021/02-21-Figure_13.1_Cohesion_principles_tension_diagram.png)

### Component Coupling（组件的耦合）
造成构建破坏的往往是那些技术性的、政策性的合不稳定性的力量。

> 组件依赖图应该是遵守无环的设计。

书中提到了the morning after syndrome（早餐综合症）

> 你是否有过工作了一整天，使得你写的东西跑起来了，然后回家，再第二天早上发现你写的东西不再能正常跑起来了？为什么？因为有些人比你工作的晚，改了你东西的依赖，我称作这种情况为“早晨综合征”。

这种情况大多数是发生在我们修改了同一份源代码文件，或者修改了相同的依赖导致。在小团队的开发中这种问题可以很快解决，但是如果项目变得很大，开发团队变多之后，这种问题会十分地棘手。

为了解决这个问题，书中介绍了两种解决方案：The Weekly Build（每周构建）；Acyclic Dependencies Principle（无环依赖原则）。

#### The Weekly Build（每周构建）
每周构建的工作流程是在每周的前四天，每个开发人员或者团队在自己的领域内进行单独的开发，然后在周五的时候将所有人的变更进行集成，解决冲突问题等，再进行构建。

这种方法虽然能够在一定程度上解决The Morning After Syndrome的问题，但是随着项目的膨胀，这种工作方式将不再有效，集成的时间会变得更很长且十分痛苦，当你不得不将每周集成一次更改为两周集成一次的时候，虽然能够暂时缓解疼痛，但是更长的开发时间意味着更多的变动，也意味着集成时将面临更大的挑战。并且测试工作也会变得十分困难。

所以我们不应该完全依赖于每周集成这样工作方式去解决造成综合症的问题，而是依赖于一个优秀的结构设计和原则，实现组件的低耦合。

#### The Acyclic Dependencies Principle（无环依赖原则）
什么样的依赖才叫无环依赖的，看下面的示例：

![典型的组件图](https://hmz-storage.oss-cn-shenzhen.aliyuncs.com/static/img/2021/02-21-Figure_14.1_Typical_component_diagram.png)

如上图中的样式所示，不管从哪个组件开始，不可能沿着依赖关系绕回起始地的组件，这种结构就是无环的。叫做无环图（DAG: Directed Acyclic Graph）。

当组件中不可避免地出现了循环依赖时怎么办呢？

![一个依赖循环示例](https://hmz-storage.oss-cn-shenzhen.aliyuncs.com/static/img/2021/02-21-Figure_14.2_A_dependency_cycle.png)

上面时一个依赖循环的示例，图中Entities、Authorizer、Interactors形成了循环依赖。

解决这种循环依赖的方法主要有两种：
1. 应用依赖反转原则（DIP）。
2. 创建一个新组件，双方均去依赖这个新的组件，把问题类都转移到这个新的组件中。

Entities和Authorizer组件间的依赖转置:
![依赖转置](https://hmz-storage.oss-cn-shenzhen.aliyuncs.com/static/img/2021/02-21-Figure_14.3_Inverting_the_dependency_between_Entities_and_Authorizer.png)

Entities和Authorizer组件都依赖于新组件
![新组件](https://hmz-storage.oss-cn-shenzhen.aliyuncs.com/static/img/2021/02-21-Figure_14.4_The_new_component_that_both_Entities_and_Authorizer_depend_on.png)

#### 自上而下设计
模块结构的设计不是设计系统的首要之一，而是随着系统的发展和变化逐渐演化。

如果我们一开始先于类的设计而去设计组件依赖，我们很可能会失败得很惨，我们并不知道公共的闭包，也不晓得任何一个可重用元素，我们几乎肯定会造出产生循环依赖的组件。因此组件依赖结构的增长和演变是随着系统逻辑设计而来的。

#### 稳定依赖原则和稳定抽象原则
被依赖的组件应该保持稳定，因为它需要那些依赖它的组件负责，不能随意发生变化。而那些很少被依赖或者不被依赖的组件是不稳定的。

组件在具备稳定性的同时也应该具备抽象性，以此来提高架构的灵活性。

在架构设计中，我们要尽量将那些稳定的组件保持在下层，不稳定的组件（比如表示层）放到上层。并且组件直接应该遵循之前提到的SOLID设计原则。

#### 指标

书中还提到了一些指标用于量化，比如“稳定指标”、“抽象度指标”。

稳定指标：
* 扇入传入依赖。这个指标是外部组件的类依赖该组件类的数量。
* 扇出传出依赖，这个指标是该组件内部的类依赖外部组件的类的数量
* 不稳定度I I=扇出/(扇出+扇入)，这个值的区间是[0,1]，I=0表示组件最为稳定，I=1表示组件最不稳定。


抽象度指标：
* Nc: 组件里的类的总数
* Na: 组件里的接口和抽象类的总数
* A: 抽象度A=Na/Nc

抽象度A值的区间为[0,1]，A=0意味着组件里没有抽象性的类，A=1意味着组件仅仅只包含抽象性的类。


**主次序线**
用某种方式定义不稳定度（I）和抽象度（A）的关系，我们建立一个A为纵坐标，I为横坐标的坐标系，如下图：

![I/A图](https://hmz-storage.oss-cn-shenzhen.aliyuncs.com/static/img/2021/02-22-A_graph.png)

不是所有的组件只分为这两类，因为组件会经常有抽象度和稳定度的矛盾。比如一个常见的例子，从一个抽象类派生出另一个抽象类，这个派生类也就有了依赖，因此虽然是最大抽象的，但并非最大稳定的，这个依赖关系降低了它的稳定。

![排除的区域](https://hmz-storage.oss-cn-shenzhen.aliyuncs.com/static/img/2021/02-22-Figure_14.13_Zones_of_exclusion.png)

我们要避免痛苦区和无用区，让组件尽可能地接近或处于主次序线。书中给出了一个距离衡量公示：

> D: 距离，D=|A+I-1|，这个值的取值区间是[0,1]，
> D值为0时表示这个组件表示的点就在主次序线上，
> D值为1时表示这个组件表示的点离主次序线最远。

![组件的散点图](https://hmz-storage.oss-cn-shenzhen.aliyuncs.com/static/img/2021/02-22-Figure_14.14_Scatterplot_of_the_components.png)


## 参考
1. [《Clean Architecture》](https://www.amazon.com/Clean-Architecture-Craftsmans-Software-Structure/dp/0134494164)
2. [【ボブおじさんのClean Architectureまとめ】オブジェクト指向 ~SOLIDの原則~](https://qiita.com/yoshinori_hisakawa/items/25576a62123607a696f6)
3. [実装から学ぶクリーンアーキテクチャ](https://gist.github.com/MegaBlackLabel/ef0f1ae19491c3d75823d6b26db97bd9)

