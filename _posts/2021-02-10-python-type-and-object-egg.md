---
title: Python的type与object谁先存在
author: huimingz
date: 2021-02-10 18:50:05 +0800
category: [Python]
tags: [Python]
pin: false
---

首先理一下 `type`与`object`的关系：

- `object`是`type`的实例。
- `object`没有父类，是`no object。
- `type`是其自身的实例。
- `type`是`object` 的子类。
- Python中存在两种类型的`objects`，为了便于区分就叫他们`types`和`non-types`。`non-types`可以被称为实例(instances)，但这个术语也可以指类型，因为一个类型总是另一个类型的实例，`types`也被称之为类(classes)，我也时不时地称它们为类。



## 基础

- Python中存在两种类型的对象：
    - `Type objects`，可创建实例化对象，可作为子类（metaclass和class）
    - `Non-type objects`不能创建实例化对象，不能作为子类（instance）
- `type`和`object`是两个起源对象；
- `objectname.__class__`存在于每个对象中，并指向对象的类型；
- `objectname.__bases__`存在于每个类型对象中，并指向该对象的超类。只有`object`的`__bases___`为空；
- 使用`class`声明语句创建的是`type`对象，基类可以是类或者`type`；
- 类实例化对象是`Non-type`类型；
- 一些`Non-type`类型可以使用Python提供的特殊语法创建，比如: `[1,2,3]`；
- 内在的，Python通常使用`type`对象去创建一个对象。新创建的对象是这个`type`对象的实例化对象。Python通过查看`class`声明时的基类来确认类型；
- `issubclass(A, B)`是用来判断继承关系的：
    - `B is in A.__bases__`返回True
    - `issubclass(Z, B)`是True， 那么`Z in A.__bases__`

## 鸡蛋问题

从语言语义学的角度来看，从程序开始的那一刻起，类型和对象都是存在的，完全初始化的。无论实现怎样做，让事情进入那个状态，都不一定要遵循实现允许你做的规则。

从`CPython`实现的角度来看，类型和对象都是在C级静态分配，两者都不是先创建的。你可以在`Object/typeobject.c`中看到变量定义。

object:

```C
PyTypeObject PyBaseObject_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)
    "object",                                   /* tp_name */
    sizeof(PyObject),                           /* tp_basicsize */
    0,                                          /* tp_itemsize */
    object_dealloc,                             /* tp_dealloc */
    0,                                          /* tp_print */
    0,                                          /* tp_getattr */
    0,                                          /* tp_setattr */
    0,                                          /* tp_reserved */
    object_repr,                                /* tp_repr */
    ...
};
```

type:

```c
PyTypeObject PyType_Type = {
    PyVarObject_HEAD_INIT(&PyType_Type, 0)
    "type",                                     /* tp_name */
    sizeof(PyHeapTypeObject),                   /* tp_basicsize */
    sizeof(PyMemberDef),                        /* tp_itemsize */
    (destructor)type_dealloc,                   /* tp_dealloc */
    0,                                          /* tp_print */
    0,                                          /* tp_getattr */
    0,                                          /* tp_setattr */
    0,                                          /* tp_reserved */
    (reprfunc)type_repr,                        /* tp_repr */
    ...
};
```

当解释器初始化开始时，`type`和`object`都处于半初始化状态，`PyType_Ready`函数负责完成它们的初始化。类型指针是在变量定义中设置的，但设置超类指针是`PyType_Ready`工作的一部分，还有很多其他的初始化需要由`PyType_Ready`来处理--例如，类型还没有`__dict__`。

顺便说一下，利用一些奇怪的元类和 Python 允许你在用户定义的类的实例上重新分配 `__class__`，我们可以设置自己的类 `A` 和` B`，其中 `B` 是 `A `的子类，`A `和` B` 都是` B `的实例，这和 `object` 和 `type` 的情况很像。不过，这与对象和类型的实际创建方式完全不同。

```python
class DummyMeta(type):
    pass

class A(type, metaclass=DummyMeta):
    pass

class B(A):
    pass

B.__class__ = B
A.__class__ = B

print(isinstance(A, B))
print(isinstance(B, B))
print(issubclass(B, A))
```

输出:

```py
True
True
True
```

## 参考

-  [Chicken or Egg paradox with type and object in Python](https://stackoverflow.com/questions/56502581/chicken-or-egg-paradox-with-type-and-object-in-python)

