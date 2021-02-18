---
title: Golang Gin框架Clean Architecture探索
author: huimingz
category: [Golang]
tags: [Golang, Gin]
pin: false
future: true
---

这里主要记录一些关于我在探索Gin框架实战的设计，这里面的设计主要结合Clean Architecture。

下面是一张关于Clean Architecture最为经典的图：

![Clean Architecture](/assets/img/posts/2021/02-15-post-clean-architecture-01.jpeg)

| Layer                      | 内容                                          |
|----------------------------|-----------------------------------------------|
| Enterprise Business Rules  | Entities                                      |
| Application Business Rules | Use Cases / Interactor                        |
| Interface Adapters         | Controllers / Presenters / Gateways           |
| Frameworks & Drivers       | Web / UI / External INterfaces / Devices / DB |

Note: 以下内容为一些较为懒散的内容，等全部确定之后再做整理。

## 代码风格
你肯定不希望在一个团队的项目中出现各种不一样的风格，这不仅会增加团队之间交流，还可能出现一些工作中遇到的麻烦（比如代码合并时）。因此我们需要一个良好的统一的代码风格，这里推荐使用[Uber Go 语言编码规范](https://github.com/xxjwxc/uber_go_guide_cn)来作为标准，团队成员应该严格遵守这里面的风格指导。

当然在代码风格这件事上，除了开发人员的自我约束之外，咱们还可以借助在CI或者Git Hook中借助检查工具。以下是一些要求：

1. 配置git hook，确保在每次代码提交的时候都执行一次代码检查。为了简单起见和统一，这里推荐使用[pre-commit](https://pre-commit.com/)工具，关于具体的用法这里不细述（或许将来会记录）。
2. 我们不能相信每个开发人员都正确配置了我们所期待的git hook，就像后端服务器永远不能相信前端做了检查（即使前端往往都会做检查）一样，因而需要在CI工程中加入格式检查的过程，将那些不符合规则的commit暴露出来，特别是在merge到master或者dev分支的。

关于Golang的代码格式检查工具推荐：
1. 代码保存时: ``goimports``
2. 使用 ``golint`` 和 ``go vet`` 确保代码风格


## 组织
介绍和描述...

### 项目结构
项目目录和文件结构.code

模块之间的依赖关系和结构如下（草图）:
![依赖关系草图](https://hmz-storage.oss-cn-shenzhen.aliyuncs.com/static/img/2021/02-16-IMG_740B8CBE6179-1.jpeg)
※ 这里针对的是前后端分离的结构设计。

列出里面的细节和注意事项

要点：
1. 每个模块之间应该有一个明确的上下文边界和定义，以确保在进行开发时不会出现将功能定义到了不属于它的那块区域中，从未出现污染并危害整个项目架构。
2. 严责遵守 **单一职责原则**，每一个模块只因该有一个唯一的被改变的理由，只对一个actor负责。
3. 小心那些贫血模型，虽然它带来了不少好处，但是随着领域逻辑的复杂性的增加，系统的复杂性也将迅速增加，程序结构将变得极度混乱。下面给出了几点建议：
    * 不应该将 ``entity`` 分散到各个app中，应该将它们存放与项目根目录下的 ``model`` （不要使用复数命名）package内。
    * 如果你达到了上面这条要求，那么每个 ``entity`` 所对应的 ``Repository`` 也应该放到该目录下，或许目录结构会是下面这样：
    
        ```
        # 关于repository的位置还需要讨论
        .
        ├── appv1
        |   └── user
        |       ├── controller
        |       ├── service
        |       └── business
        ├── model
        |   ├── entity
        |   └── repository
        ...
        ```
4. 遵守SOLID原则。

### 依赖方向
从上面的模块依赖关系图可以看出整体的依赖方向，上层依赖于下层，但是下层觉得不能依赖于上层结构（应该说它是不知道上层是谁）。

#### 值传递
在各层的数据传输方面，Repository的传入值基本上是固定的基本类型、Entity或 ``bson.M`` 类型（这里使用MongoDB作为数据库），而范围值是Entity或者基本类型；Entity只是定义，并没有具体的方法，即使有也是返回基本类型。所以在Repository和Entity层之间的依赖关系是正向的。

我们需要关心的是是在Controller层、Business层、Service层之间的调用，这里面不能全部使用基本类型来作为输入和输入，因为部分调用的传入和返回的参数的数量会很多，虽然Golang支持多返回值，但不应该利用该特性来实现大量的返回值。那么解决办法就是引入Value Object，在Business中定义 ``BizRequestValue`` 和 ``BizResponseValue`` 供Controller使用，在Service中定义 ``SrvInputValue`` 和 ``SrvOutputValue`` 供Business使用。

这些Value Object的定义需要遵循下面的要求：
1. 只作为值传递的载体，不需要其他的行为。
2. Value Object的定义应该定义在同层，供上层调用者使用，本层被调用者的输入和输出值。
3. Value Object应该和被调用者处于同一个包内。
4. Value Object不应该被定义到被调用者的同一个文件内。
5. Value Object的文件名应该明确标识，比如Business层应该是 ``biz_dto``，Service层应该是 ``srv_vo`` （只是建议）。

示例目录文件结构:
```
.
├── controller
│   └── xxx.go
├── business
│   ├── xxx_dto.go  // Value Object定义
│   └── xxx.go
├── service
│   ├── xxx_vo.go  // Value Object定义
│   └── xxx.go
```

**如何确保从DB中独立？**

在上面中有提到 ``bson.M`` 作为Repository的传入类型的情况，这明显违反了Clean Architecture的原则，处于内部的Repository知晓了外部DB的情况，并且依赖于了MongoDB。这就无法确保Databse的独立性，无法实现数据库的切换（关系型到非关系型）。

虽然要做到这点很困难，但是我们可以尽力保持它的独立性，比如可以参考Django中的设（``Q`` 方法）计，它将各个条件生成对应的SQL语句，以实现对不同数据库的支持。

假设在Gin项目中可以通过下面的途径去实现：
```golang
// 查询条件的的定义
type Compare struct {
	Key string
	Value interface{}
	Operation string  // 可以定义一个常量
}

type Repo struct {
	// ...
}

func (r *Repo) FindBy(args ...interface{}) User {
	// args中遍历并获取值
}

user := repo.FindBy(Compare("Id", "sakura", "equal"))
```

虽然目前看来这种方案并不可行，但我们可以参考一下社区的一些解决方案。

#### Interface
Interface的定义和Value Object的定义位置稍微有点区别，需要定义在调用者那边，而不是被调用者那边。

满足以下要求：
1. 被调用者的Interface应该定义在调用者一侧。
2. Interface声明内容不应该和调用者处于同一个源代码文件内。
3. Interface的声明文件应该和调用者处于同一个包内（package）。
3. Interface的声明应该尽量简洁，满足需求即可。

### 注意

### 依赖注入
使用[wire](https://github.com/google/wire)来实现依赖注入，不要滥用依赖注入，你应该将依赖注入使用在诸如 ``Controller`` 这样的初始化上，而不是 ``Service`` 或者 ``Repository`` 的初始化上。

## 日志
这里推荐使用

## 指导原则

### 分层设计

### SOLID原则

### 贫血模型

### 充血模型

## 配置

