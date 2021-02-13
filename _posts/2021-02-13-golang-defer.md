---
title: Golang defer知识点
author: huimingz
date: 2021-02-13 16:38:42 +0800
category: [Golang]
tags: [Golang]
pin: false
---



`defer`的特性：

1. `defer`在函数执行`return`或者`panic`之后执行，多个`defer`的执行顺序为“先进后出（FILO）”。
2. 匿名返回值在`return`执行时被声明；有名返回值则是在声明的同时被声明。`defer`语句只能访问有名返回值，不能直接访问匿名返回值。
3. `defer`、`return`和返回值的执行逻辑：`return`先执行，`return`将结果写入到返回值中；然后`defer`开始执行一些收尾工作；最后`RET`（汇编命令，用于结束一个函数）携带返回值退出函数。
4. 主动调用`os.Exit(int)`退出进程时，已声明的defer将不再被执行。
5. `defer`声明时会先计算确定参数的值，`defer`推迟执行的是其函数体。
6. `defer`的作用域：
    - 只对当前goroutine有效；
    - 当`panic`发生时仍然会执行当前goroutine中已声明的`defer`语句，如果未遇到`recover()`则会执行完所有的`defer`后引发**整个进程的崩溃**。



`return`执行步骤：

1. 给返回值赋值（若为有名返回值则直接赋值，若为匿名返回值则先声明再赋值）。
2. 调用RET返回指令并传入返回值，而RET则会检查defer是否存在，若存在就先逆序插播defer语句，最后RET携带返回值退出函数。

※ Golang不同于C语言，被调用函数的入参和返回值由调用者函数维护，这也就是为什么Golang支持多返回值的原因。



## 执行顺序

先进后出原则，可以看作是一个“栈”的结构关系。

![流程图](/assets/img/posts/2021-02-13-golang-defer-sequence.png)



## defer中出现panic时

在`defer`语句中执行`panic`和在外面执行效果时一样的，可以被位于其之后的`recover()`所捕获。

```go
package main

import (
	"fmt"
	"time"
)

func doo() {
	fmt.Println("execute the doo function.")

	defer func() {
		if v := recover(); v != nil {
			fmt.Printf("recover v: %v\n", v)
		}
	}()

	defer func(t time.Time) {
		fmt.Printf("time: %v\n", t)
	}(time.Now())

	defer func() {
		fmt.Println("doo.defer()")
		panic("yahoo!!!!!")
	}()
}

func main() {

	go doo()

	time.Sleep(1 * time.Second)
	println("end of main function.")
}
```

执行结果:

```
$ go run p.go execute the doo function.
doo.defer()
time: 2021-02-13 17:12:41.355246 +0800 CST m=+0.000121017
recover v: yahoo!!!!!
end of main function.
```



## 参考

1. [Golang 汇编入门知识总结](https://cloud.tencent.com/developer/article/1692904)
2. [汇编语言入门教程](https://www.ruanyifeng.com/blog/2018/01/assembly-language-primer.html)
3. [Golang中defer、return、返回值之间执行顺序的坑](https://my.oschina.net/henrylee2cn/blog/505535)

