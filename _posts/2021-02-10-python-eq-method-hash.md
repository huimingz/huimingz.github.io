---
title: Python类覆盖__eq__方法后的hash问题【旧】
author: huimingz
date: 2021-02-10 18:53:05 +0800
category: [Python]
tags: [Python]
pin: false
---

为了符合对象模型，定义了自己的平等方法的类也应该定义自己的哈希方法，或者说是不可哈希方法。如果没有定义哈希方法，那么就使用超级类的哈希方法。这不太可能导致预期的行为。

一个类可以通过将其 `__hash__` 属性设置为 `None` 来使其成为不可隐藏的。

在 Python 3 中，如果定义了一个类级的`__eq__`方法，并省略了 `__hash__` 方法，那么该类就会自动标记为不可隐藏。

## 示例

在下面的例子中，`Point`类定义了一个`__eq__`方法，但没有`__hash__`方法。如果在这个类上调用`hash`方法，那么就使用为对象定义的`hash`方法。这不太可能给出所需的行为。`PointUpdated`类比较好，因为它同时定义了一个`__eq__`方法和一个`__hash__`方法。如果`Point`不用于字典或集合，那么可以定义为`UnhashablePoint`如下。

```python
# Incorrect: equality method defined but class contains no hash method
class Point(object):
 
    def __init__(self, x, y):
        self._x = x
        self._y = y
 
    def __repr__(self):
        return 'Point(%r, %r)' % (self._x, self._y)
 
    def __eq__(self, other):
        if not isinstance(other, Point):
            return False
        return self._x == other._x and self._y == other._y
 
 
# Improved: equality and hash method defined (inequality method still missing)
class PointUpdated(object):
 
    def __init__(self, x, y):
        self._x = x
        self._y = y
 
    def __repr__(self):
        return 'Point(%r, %r)' % (self._x, self._y)
 
    def __eq__(self, other):
        if not isinstance(other, Point):
            return False
        return self._x == other._x and self._y == other._y
 
    def __hash__(self): 
        return hash(self._x) ^ hash(self._y)
 
# Improved: equality method defined and class instances made unhashable
class UnhashablePoint(object):
 
    def __init__(self, x, y):
        self._x = x
        self._y = y
 
    def __repr__(self):
        return 'Point(%r, %r)' % (self._x, self._y)
 
    def __eq__(self, other):
        if not isinstance(other, Point):
            return False
        return self._x == other._x and self._y == other._y
 
    #Tell the interpreter that instances of this class cannot be hashed
    __hash__ = None
```

## 结论

当你为类定义一个 `__eq__`方法时，记得要实现一个 `__hash__`方法，或者设置 ``__hash__ = None`。

## 参考

-   [Inconsistent equality and hashing](https://help.semmle.com/wiki/display/PYTHON/Inconsistent+equality+and+hashing)