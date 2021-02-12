---
title: 领域驱动设计（笔记）
author: huimingz
date: 2021-02-12 13:34:10 +0800
category: [architect]
tags: [architect]
pin: false
---

最近读完了《领域驱动设计》，以下是个人的一些笔记和理解。

<hr>

首先什么是**领域驱动设计**，它的定义是什么？

1. 应该关注领域中的核心问题以及机会；
2. 开发人员应该和领域专家一起探索模型；
3. 开发人员应该写出能够表明模型意图的代码；
4. 在界限上下文（bounded context）中尽量使用**统一语言**（ubiquitous language）。



软件的核心是为用户解决领域相关问题的能力，所有的其它特性，不管多么重要，都应该服务于这个目的。

大多数情况下开发人员更偏向于技术上的追求，更愿意从事精细的框架工作，企图使用技术来解决领域问题。这是一个错误的观点，开发人员同样需要面对软件的核心问题，否则只会让工作的重点偏离。（保持工作方向不偏离）

模型和实现应该紧密结合，不应该将其区分开（过度分工）。参与建模的人员同样应该了解代码，开发人员也一样，如果可能应该让开发人员也参与建模的过程。

## Ubiquitpus Language

在软件开发过程中，我们应该尽量使用ubiqitous language来进行交流，如果没有，那么应该发现并使用它。

使用统一语言能够帮助我们减少交流上的误解，这种交流不应该只是开发人员之间，而是整个团队乃至整个项目发工作人员，还应该体现在代码和文档之中。

对ubiquitous language的修改就是对模型的修改。

文档应该尽量使用领域模型，减少不必要的解释和说明，提高文档的可阅读性。当出现维护文档只是出于意志和自律，且并未产生其它大的价值时，应该将它归档并舍弃。

## 分离领域

最近开始关注《Clean Architeture》,可能更多的对分层设计有一些感触和想法。

分层结构大致为： 用户界面层（或表示层）=> 应用层 => 领域层（或模型层） => 基础设施层。

上层依赖于下层，如果下层要影响上层结构，可以使用代理或者回掉等方式。各层之间不应该之间关联，而应该使用抽象层。

优点：

1. 低耦合，分离关注点，讲中心放到领域层；

2. 各层只用关心当前层，不用关心上下层如何处理，减少认知过载的情况发生；
3. 项目结构清晰。

与之相反的是SMART UI模式，不过它只适用于项目较小、简单和需要快速开发的项目。

※ 切记不要过度分层，使用过多的分层设计只会让你的项目变得复杂和难以维护。

## 软件中使用的模型

- ENTITY（REFERENCE OBJECT）: 有标示定义的对象，entity是领域模型中的一个根本特性。注意建模时不要被过多的属性所干扰，应该抓住entity最根本的特性。
- VALUE OBJECT: 用于描述领域的某个方面而本身没有概念标示的对象（关心是什么，而不是关心是谁）。VO不应该有双向关联。
- SERVICE: 作为接口提供的一种操作，它在模型中是独立的，不像ENTITY和VALUE OBJECT那样具有封装状态。强调的是与对象的关系，关心的是做什么。（使用动词定义）
- MODULE（或PACKAGE）: 方便观察module的内部细节，而不会被整个模型埋没；观察module之间的关系，而不用关心内部细节。采用低耦合高内聚的原则。

## 领域对象的生命周期

挑战：

1. 在整个生命周期中维护完整性。
2. 防止模型陷入管理生命周期复杂性造成的困境当中。

### AGGREGATE模式

> 使用一个抽象来封装模型的引用，AGGREGATE就是一组相关对象的集合，我们把它作为数据修改的单元。每个AGGREGATE都有一个跟（root）和一个边界（boundary）。边界定义了AGGREGATE的内部都有什么。根则是AGGREGATE所包含的一个特定ENTITY。对外部而言只可以引用根，而边界内部的对象之间则可以相互引用。除根以外的其它ENTITY都有本地标识，但这些标识只在AGGREGATE内部才需要加以区别，因为外部对象除了根ENTITY之外看不到其它对象。
>
> 我们应该将ENTITY和VALUE OBJECT分门别类地聚集到AGGREGATE中，并定义每个AGGREGATE的边界。在每个AGGREGATE中，选择一个ENTITY作为根，并通过根来控制对边界内其它对象的所有访问。只允许外部对象保持对根的引用。对内部成员的临时引用可以被传递出去，但仅在一次操作中有效。由于根控制访问，因此不能绕过它来修改内部对象。这种设计有利于保持AGGREGATE中的对象满足所有固定规则，也可以确保在任何状态变化时AGGREGATE作为一个整体满足规则。

### FACTORY模式

解决复杂对象复杂自身创建问题时，导致的职责过载问题。

> 将创建复杂对象的实例和AGGREGATE的职责转移给单独的对象，这个对象本身可能没有承担领域模型中的职责，但它仍是领域设计中的一部分。提供一个封装所有复杂装配操作的接口，而且这个接口不需要客户引用要被实例化的对象的具体类。在创建AGGREGATE时要把它作为一个整体，并确保它满足固定规则。

两个基本要求：

1. 每个创建方法都是原子的，而且要保证被创建对象或AGGREGATE的所有固定规则。
2. FACTORY应该被抽象为所需的类型，而不是所要创建的具体的类。

※ 可以借助依赖注入技术来解决创建中的依赖问题。

### REPOSITORY模式

>  REPOSITORY将某种类型的所有对象表示为一个概念集合（通常是模拟的）。他的行为类似于集合，只是具有更复杂的查询功能。

开发人员需要了解REPOSITORY内部的实现，因此在实现REPOSITORY时需要保持对外的一致



REPOSITORY和FACTORY是不同的，他们所处生命周期阶段不同。FACTORY处于生命周期的开始，而REPOSITORY处于生命周期的中期和结束。

## 重构

通过不断的重构来不断加深对领域模型的理解，通过不对的对话来发现问题并改进模型，最终得到深层模型。这个过程中也同样伴随着设计的改进，最终设计也将抵达柔性设计。

※ 模型是系统重构的基础。

方法：

1. 突破：不断的小范围改进而最终产生质变。
2. 将隐式概念转变为显示概念：发掘那些未被发现的重要概念，并建模。
3. 柔性设计：软件应当首先为开发人员服务，最终服务于用户。
4. 应用分析模式：寻找方向和线索，降低成本（站在巨人的肩膀上）。
5. 将设计模式应用于模型：设计模式是指导。代码层面是技术设计模式，模型层面是感念概念模式。
6. 通过重构得到更深的理解。

## 战略设计

目的是解决分析膨胀的系统的成本高昂问题。

从概念和实现上将复杂的系统进行拆分，从而得到容易理解和分析的较小的部分，避免认知过载的发生。

> 战略设计原则必须知道设计决策，以便减少各个部分之间的互相依赖，在使设计意图更为清晰的同时而又不失去关键的互操作性和协同性。战略设计原则必须把模型的中带你放在捕获系统的核心概念，也就是系统的“远景”上。

三大主题：

1. 上下文：最根本的原则。
2. 精炼：关注重点，不被大量的次要问题所淹没。
3. 大型结构：全局视角，解决“只见树木，不见森林”。

※ 在采用领域驱动设计的时候，模型要与实现相结合。