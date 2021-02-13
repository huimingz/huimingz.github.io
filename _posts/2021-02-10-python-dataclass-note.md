---
title: Python的dataclass笔记
author: huimingz
date: 2021-02-10 18:50:02 +0800
category: [Python]
tags: [Python]
pin: false
---

`dataclasses`模块是在3.7版本版本引入的, [What’s New In Python 3.7](https://docs.python.org/3/whatsnew/3.7.html), 相比使用传统的`type`创建类，`dataclasses`提供了一些更高效和高级的办法。

`dataclass`的官方文档: [https://docs.python.org/3/library/dataclasses.html](https://docs.python.org/3/library/dataclasses.html)

## 正常定一个类:

```py
class Person(object):
    def __init__(self, name: str, age: int) -> None:
        self.name = name
        self.age = age
```

```
>>> help(Person)
Help on class Person in module __main__:

class Person(builtins.object)
 |  Person(name: str, age: int) -> None
 |  
 |  Methods defined here:
 |  
 |  __init__(self, name: str, age: int) -> None
 |      Initialize self.  See help(type(self)) for accurate signature.
 |  
 |  ----------------------------------------------------------------------
 |  Data descriptors defined here:
 |  
 |  __dict__
 |      dictionary for instance variables (if defined)
 |  
 |  __weakref__
 |      list of weak references to the object (if defined)

# 查看类属性
>>> Person.__dict__
mappingproxy({'__module__': '__main__', '__init__': <function Person.__init__ at 0x106526dc0>, '__dict__': <attribute '__dict__' of 'Person' objects>, '__weakref__': <attribute '__weakref__' of 'Person' objects>, '__doc__': None})

# 查看类继承
>>> Person.__mro__
(<class '__main__.Person'>, <class 'object'>)

>>> dir(Person)
['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__']

# 实例化
>>> p = Person("zhangsan", 17)

>>> p
<__main__.Person object at 0x1063c8b20>

>>> p.__dict__
{'name': 'zhangsan', 'age': 17}

>>> dir(p)
['__class__', '__delattr__', '__dict__', '__dir__', '__doc__', '__eq__', '__format__', '__ge__', '__getattribute__', '__gt__', '__hash__', '__init__', '__init_subclass__', '__le__', '__lt__', '__module__', '__ne__', '__new__', '__reduce__', '__reduce_ex__', '__repr__', '__setattr__', '__sizeof__', '__str__', '__subclasshook__', '__weakref__', 'age', 'name']
```

## 使用Type创建一个类

如何创建： `type(name, bases, dict) -> a new type`

```
# 创建类
Person = type("Person", (object, ), {"name": "", "age": 0, "say_hello": lambda: print("hello")})

# 查看
>>> help(Person)
Help on class Person in module __main__:

class Person(builtins.object)
 |  Methods defined here:
 |  
 |  say_hello lambda (...)
 |  
 |  ----------------------------------------------------------------------
 |  Data descriptors defined here:
 |  
 |  __dict__
 |      dictionary for instance variables (if defined)
 |  
 |  __weakref__
 |      list of weak references to the object (if defined)
 |  
 |  ----------------------------------------------------------------------
 |  Data and other attributes defined here:
 |  
 |  age = 0
 |  
 |  name = ''
 
>>> Person.__dict__
mappingproxy({'name': '', 'age': 0, 'say_hello': <function <lambda> at 0x106526c10>, '__module__': '__main__', '__dict__': <attribute '__dict__' of 'Person' objects>, '__weakref__': <attribute '__weakref__' of 'Person' objects>, '__doc__': None})
```

## 使用dataclass

```py
import dataclasses
from typing import Callable

@dataclasses.dataclass
class Person(object):
    name: str
    age: int = dataclasses.field(default=16, init=True)
    gender: str = dataclasses.field(default="F", repr=False, init=False)
    get_gender: Callable[[Person], str] = dataclasses.field(default=lambda self: self.gender, init=False, repr=False)
    
    def age_plus(self):
        self.age += 1
```

查看类：
```
>>> import pprint
>>> pprint.pprint(Person.__dict__)
mappingproxy({'__annotations__': {'age': <class 'int'>,
                                  'gender': <class 'str'>,
                                  'get_gender': typing.Callable[[__main__.Person], str],
                                  'name': <class 'str'>},
              '__dataclass_fields__': {'age': Field(name='age',type=<class 'int'>,default=16,default_factory=<dataclasses._MISSING_TYPE object at 0x10680b670>,init=True,repr=True,hash=None,compare=True,metadata=mappingproxy({}),_field_type=_FIELD),
                                       'gender': Field(name='gender',type=<class 'str'>,default='F',default_factory=<dataclasses._MISSING_TYPE object at 0x10680b670>,init=False,repr=False,hash=None,compare=True,metadata=mappingproxy({}),_field_type=_FIELD),
                                       'get_gender': Field(name='get_gender',type=typing.Callable[[__main__.Person], str],default=<function Person.<lambda> at 0x10686de50>,default_factory=<dataclasses._MISSING_TYPE object at 0x10680b670>,init=False,repr=False,hash=None,compare=True,metadata=mappingproxy({}),_field_type=_FIELD),
                                       'name': Field(name='name',type=<class 'str'>,default=<dataclasses._MISSING_TYPE object at 0x10680b670>,default_factory=<dataclasses._MISSING_TYPE object at 0x10680b670>,init=True,repr=True,hash=None,compare=True,metadata=mappingproxy({}),_field_type=_FIELD)},
              '__dataclass_params__': _DataclassParams(init=True,repr=True,eq=True,order=False,unsafe_hash=False,frozen=False),
              '__dict__': <attribute '__dict__' of 'Person' objects>,
              '__doc__': 'Person(name: str, age: int = 16)',
              '__eq__': <function __create_fn__.<locals>.__eq__ at 0x106878c10>,
              '__hash__': None,
              '__init__': <function __create_fn__.<locals>.__init__ at 0x1068789d0>,
              '__module__': '__main__',
              '__repr__': <function __create_fn__.<locals>.__repr__ at 0x106878af0>,
              '__weakref__': <attribute '__weakref__' of 'Person' objects>,
              'age': 16,
              'age_plus': <function Person.age_plus at 0x106878940>,
              'gender': 'F',
              'get_gender': <function Person.<lambda> at 0x10686de50>})

           
>>> help(Person)

Help on class Person in module __main__:

class Person(builtins.object)
 |  Person(name: str, age: int = 16) -> None
 |  
 |  Person(name: str, age: int = 16)
 |  
 |  Methods defined here:
 |  
 |  __eq__(self, other)
 |  
 |  __init__(self, name: str, age: int = 16) -> None
 |  
 |  __repr__(self)
 |  
 |  age_plus(self)
 |  
 |  get_gender lambda self
 |  
 |  ----------------------------------------------------------------------
 |  Data descriptors defined here:
 |  
 |  __dict__
 |      dictionary for instance variables (if defined)
 |  
 |  __weakref__
 |      list of weak references to the object (if defined)
 |  
 |  ----------------------------------------------------------------------
 |  Data and other attributes defined here:
 |  
 |  __annotations__ = {'age': <class 'int'>, 'gender': <class 'str'>, 'get...
 |  
 |  __dataclass_fields__ = {'age': Field(name='age',type=<class 'int'>,def...
 |  
 |  __dataclass_params__ = _DataclassParams(init=True,repr=True,eq=True,or...
 |  
 |  __hash__ = None
 |  
 |  age = 16
 |  
 |  gender = 'F'
```

```
>>> p = Person("sakura")
>>> p
Person(name='sakura', age=16)
>>> p.age_plus()
>>> p
Person(name='sakura', age=17)
>>> p.get_gender()
'F'
>>> p.gender
'F'
```

如果需要定义隐藏类的话只需要在声明的时候才变量名前加`_` or `__`即可。如果是`__`，则获取变量值的方式为: `instance._ClassName__atrributeName`。

### 使用动态的方式创建类

```
# 创建类
>>> Person = dataclasses.make_dataclass("Person", [("name", "str"), ("age", "int", dataclasses.field(default=16, init=True)), ("gender", "str", dataclasses.field(default="F", repr=False, init=False)), ("get_gender", "Callable[[Person], str]", dataclasses.field(default=lambda self: self.gender, repr=False, init=False))], namespace={"age_plus": lambda self: exec("self.age += 1")})
>>> 
>>> p = Person("sakura")
>>> p
Person(name='sakura', age=16)
>>> p.age_plus()
>>> p
Person(name='sakura', age=17)

>>> p.gender
'F'

>>> p.get_gender()
'F'

>>> pprint.pprint(Person.__dict__)
mappingproxy({'__annotations__': {'age': 'int',
                                  'gender': 'str',
                                  'get_gender': 'Callable[[Person], str]',
                                  'name': 'str'},
              '__dataclass_fields__': {'age': Field(name='age',type='int',default=16,default_factory=<dataclasses._MISSING_TYPE object at 0x10680b670>,init=True,repr=True,hash=None,compare=True,metadata=mappingproxy({}),_field_type=_FIELD),
                                       'gender': Field(name='gender',type='str',default='F',default_factory=<dataclasses._MISSING_TYPE object at 0x10680b670>,init=False,repr=False,hash=None,compare=True,metadata=mappingproxy({}),_field_type=_FIELD),
                                       'get_gender': Field(name='get_gender',type='Callable[[Person], str]',default=<function <lambda> at 0x106880430>,default_factory=<dataclasses._MISSING_TYPE object at 0x10680b670>,init=False,repr=False,hash=None,compare=True,metadata=mappingproxy({}),_field_type=_FIELD),
                                       'name': Field(name='name',type='str',default=<dataclasses._MISSING_TYPE object at 0x10680b670>,default_factory=<dataclasses._MISSING_TYPE object at 0x10680b670>,init=True,repr=True,hash=None,compare=True,metadata=mappingproxy({}),_field_type=_FIELD)},
              '__dataclass_params__': _DataclassParams(init=True,repr=True,eq=True,order=False,unsafe_hash=False,frozen=False),
              '__dict__': <attribute '__dict__' of 'Person' objects>,
              '__doc__': "Person(name: 'str', age: 'int' = 16)",
              '__eq__': <function __create_fn__.<locals>.__eq__ at 0x106880b80>,
              '__hash__': None,
              '__init__': <function __create_fn__.<locals>.__init__ at 0x1068809d0>,
              '__module__': 'types',
              '__repr__': <function __create_fn__.<locals>.__repr__ at 0x106880a60>,
              '__weakref__': <attribute '__weakref__' of 'Person' objects>,
              'age': 16,
              'age_plus': <function <lambda> at 0x106880700>,
              'gender': 'F',
              'get_gender': <function <lambda> at 0x106880430>})
```

## 参考

1. [What’s New In Python 3.7](https://docs.python.org/3/whatsnew/3.7.html)
2. [`dataclasses` — Data Classes](https://docs.python.org/3/library/dataclasses.html)

