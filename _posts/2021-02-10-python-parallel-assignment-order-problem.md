---
title: Python并行赋值的前后顺序问题
author: huimingz
date: 2021-02-10 18:50:01 +0800
category: [Python]
tags: [Python]
pin: false
---

我们需要借助描述器来验证这个问题

代码如下：
```python
class MyInteger(object):

    def __set_name__(self, owner, name):
        print(f"__set_name__, name:{name}, owner:{owner}")
        self.name = name
    
    def __get__(self, instance, type_=None) -> object:
        print(f"__get__, instance: {instance}, name: {self.name}")
        return instance.__dict__.get(self.name, 0)

    def __set__(self, instance, value) ->None:
        print(f"__set__, instance: {instance}, name: {self.name}, value: {value}")
        instance.__dict__[self.name] = value

    def __delete__(self, instance) -> None:
        print(f"__delete__, instance: {instance}, name: {self.name}")
        del instance.__dict__[self.name]


class A:
    idx = MyInteger()


a1 = A()
a1.idx = 1
a2 = A()
a2.idx = 2
a3 = A()
a3.idx = 3


a1.idx, a2.idx, a3.idx = a3.idx, a1.idx, a2.idx
```

输出结果：
```
__set_name__, name:idx, owner:<class '__main__.A'>
__set__, instance: <__main__.A object at 0x10c74eb50>, name: idx, value: 1
__set__, instance: <__main__.A object at 0x10c728550>, name: idx, value: 2
__set__, instance: <__main__.A object at 0x10c728580>, name: idx, value: 3
__get__, instance: <__main__.A object at 0x10c728580>, name: idx
__get__, instance: <__main__.A object at 0x10c74eb50>, name: idx
__get__, instance: <__main__.A object at 0x10c728550>, name: idx
__set__, instance: <__main__.A object at 0x10c74eb50>, name: idx, value: 3
__set__, instance: <__main__.A object at 0x10c728550>, name: idx, value: 1
__set__, instance: <__main__.A object at 0x10c728580>, name: idx, value: 2
```


结论：
- 其执行顺序先执行=右边的表达式，再依次将值赋给左边
- Python解释器会将其组成为一个tuple，赋值的时候再unpack
- 右边表达式从左往右开始执行
- 赋值时从左往右依次赋值给变量