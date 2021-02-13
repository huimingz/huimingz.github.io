---
title: Python列表推导式变量存储陷阱以及如何保存过程值
author: huimingz
date: 2021-02-10 18:50:03 +0800
category: [Python]
tags: [Python]
pin: false
---

最近面试的时候遇到了这样一道面试题:

```py
funcs = [lambda x: x+n for n in range(6)]

for func in funcs:
    print(func(3))
```

问打印的结果是什么，当时一看到就知道这是个陷阱，也知道答案是什么，结果最后写的时候居然写错了（粗心大意）。

正确的结果是：
```
8
8
8
8
8
```



**至于为什么结果都是一样**

在列表推导式的过程中`n`变量只有一个，而`lambda`表达式只是用了n变量，并没有在循环过程中保存`n`变量所指向的值，而`n`变量指向的值在这个过程中不断变化，最终是`5`。

那么`lambda`表达式调用时，所有的`n`指向的结果是最后一个结果`5`。

## 那么如何保存循环过程中的值呢？

方法如下:
```python
from functools import  partial
funcs = [partial(lambda x, y: x+y, y=n) for n in range(6)]

for func in funcs:
    print(func(3), end=" ")
```

结果：
```
3 4 5 6 7 8 
```

将循环中的过程值赋值给了匿名函数的形参数（借助`partial`），这样就保存了过程值。