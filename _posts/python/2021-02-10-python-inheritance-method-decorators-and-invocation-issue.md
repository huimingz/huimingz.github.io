---
title: Python继承关系时的方法装饰器与调用
author: huimingz
date: 2021-02-10 18:50:04 +0800
category: [Python]
tags: [Python]
pin: false
---

## 被装饰后的类方法在子类被重写的情况

思考：当父类的方法`a`被`staticmethod`装饰后，再被子类继承时会发生什么？子类需要使用装饰器么？

关于装饰器实现细节，请参考: [Descriptor](https://docs.python.org/3/howto/descriptor.html)。

可以利用描述器的特性去实现自己的`staticmethod`等装饰器，可以参考我的实现：[使用描述器实现自定义property、classmethod和staticmethod](https://gist.github.com/huimingz/ddcbb92d8bee5e265571e6e15649c30b#file-property-classmethod-staticmethod)

```python
class A:

    @staticmethod
    def say(self, name):
        print(f"{name} A say")
    
class B(A):

    # @staticmethod
    def say(self, name):
        print(f"{name} B say")

b = B()
b.say("yahoo")
# 输出结果如下
# yaoo B say
```

**结论**

-   父类的类方法在初始化的时候就会绑定，在绑定的时候会调用`staticmethod`装饰器包装一次该方法，而这里的装饰器只是修改了该函数（方法）的一些参数行为，其`A.__dict__`中存储的`say`键对应的值发生了一些变化。
-   子类覆盖父类方法的行为只是改变了调用行为，默认优先调用子类的方法，即使父类存在。
-   如何直接跨过子类调用父类的方法呢？`B.__base__.say("A self", "A")`

## 继承关系下的方法重写后的调用问题

思考下面代码

```python
class Parent:
    def whoami(self):
        print(f"I am Parent")
        self.saybye("Parent")

    def saybye(self, name):
        print(f"Parent: {name} say bye")

class Child(Parent):
    def whoami(self):
        print(f"I am Child")
        super().whoami()
    
    def saybye(self, name):
        print(f"Child: {name} say bye")

c = Child()
c.whoami()
```

输出：
```
I am Child, <__main__.Child object at 0x114342b50>
I am Parent, <__main__.Child object at 0x114342b50>
Child: Parent say bye, <__main__.Child object at 0x114342b50>
```

-   当使用`super`调用父类的普通类方法的时候，默认会将当前实例对象作为第一个参数调用父类的指定方法；
-   父类方法中调用`self.saybye("Parent")`的时候，会首先调用当前实例类的`saybye`方法，因为`self`就是当前类的实例化对象；
-   当当前实例类没有实现`saybye`方法的话，再去父类寻找。



## 参考

- [Understanding Python metaclasses](https://blog.ionelmc.ro/2015/02/09/understanding-python-metaclasses/#data-descriptors)
-  [Descriptor HowTo Guide](https://docs.python.org/3/howto/descriptor.html#id1)