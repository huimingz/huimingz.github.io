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

![流程图](/assets/img/posts/2021/02-13-golang-defer-sequence.png)



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



## 有名返回值和匿名返回值遇见defer

`defer`声明的语句中是无法访问匿名返回值的，而匿名返回值的声明是在`return`语句执行的时候声明的（匿名返回值的情况时）。

示例代码：

```go
package main

func a() int {
	var i int
	defer func() {
		i++
		println("a defer: i =", i)
	}()
	i = 100
	return i
}

func b() (i int) {
	defer func() {
		i++
		println("b defer: i =", i)
	}()
	i = 100
	return i
}

func main() {
	println("a return:", a())
	println("b return:", b())
}
```

输出:

```
$ go run main_a.go
a defer: i = 101
a return: 100
b defer: i = 101
b return: 101
```

从结果中可以发现b函数的`defer`语句修改了返回的值，这是因为返回值一开始就被声明了，`return`语句执行将执行结果进行了赋值操作。

而a函数的`defer`虽然执行了，但是并没有修改函数最终的返回值，其原因就如最初所说的那样，匿名返回值类型是在`return`语句执行时声明的，a函数内最初声明`var i int`只是声明了一个函数内的局部变量，他只是存活于当前函数作用域内，与返回值并没有直接关系。

从汇编的角度来分析一下执行过程：

```shell
# 获取汇编代码
go build -gcflags "-N -l" main_a.go
```

汇编代码（删除了很多无用的输出）：

```
"".a STEXT size=161 args=0x8 locals=0x68  ;; A函数
        0x0000 00000 (main_a.go:3)      TEXT    "".a(SB), ABIInternal, $104-8
        0x0000 00000 (main_a.go:3)      MOVQ    (TLS), CX
        0x0009 00009 (main_a.go:3)      CMPQ    SP, 16(CX)
        0x000d 00013 (main_a.go:3)      PCDATA  $0, $-2
        0x000d 00013 (main_a.go:3)      JLS     151
        0x0013 00019 (main_a.go:3)      PCDATA  $0, $-1
        0x0013 00019 (main_a.go:3)      SUBQ    $104, SP
        0x0017 00023 (main_a.go:3)      MOVQ    BP, 96(SP)
        0x001c 00028 (main_a.go:3)      LEAQ    96(SP), BP
        0x0021 00033 (main_a.go:3)      PCDATA  $0, $-2
        0x0021 00033 (main_a.go:3)      PCDATA  $1, $-2
        0x0021 00033 (main_a.go:3)      FUNCDATA        $0, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
        0x0021 00033 (main_a.go:3)      FUNCDATA        $1, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
        0x0021 00033 (main_a.go:3)      FUNCDATA        $2, gclocals·9fb7f0986f647f17cb53dda1484e0f7a(SB)
        0x0021 00033 (main_a.go:3)      PCDATA  $0, $0
        0x0021 00033 (main_a.go:3)      PCDATA  $1, $0
        0x0021 00033 (main_a.go:3)      MOVQ    $0, "".~r0+112(SP)  ;; 返回值初始化
        0x002a 00042 (main_a.go:4)      MOVQ    $0, "".i+8(SP)      ;; 局部变量i的初始化
        0x0033 00051 (main_a.go:5)      MOVL    $8, ""..autotmp_3+16(SP)
        0x003b 00059 (main_a.go:5)      PCDATA  $0, $1
        0x003b 00059 (main_a.go:5)      LEAQ    "".a.func1·f(SB), AX  ;; 将defer函数地址存储于‘临时内存区域’
        0x0042 00066 (main_a.go:5)      PCDATA  $0, $0
        0x0042 00066 (main_a.go:5)      MOVQ    AX, ""..autotmp_3+40(SP)
        0x0047 00071 (main_a.go:5)      PCDATA  $0, $1
        0x0047 00071 (main_a.go:5)      LEAQ    "".i+8(SP), AX
        0x004c 00076 (main_a.go:5)      PCDATA  $0, $0
        0x004c 00076 (main_a.go:5)      MOVQ    AX, ""..autotmp_3+88(SP)
        0x0051 00081 (main_a.go:5)      PCDATA  $0, $1
        0x0051 00081 (main_a.go:5)      LEAQ    ""..autotmp_3+16(SP), AX
        0x0056 00086 (main_a.go:5)      PCDATA  $0, $0
        0x0056 00086 (main_a.go:5)      MOVQ    AX, (SP)
        0x005a 00090 (main_a.go:5)      CALL    runtime.deferprocStack(SB)
        0x005f 00095 (main_a.go:5)      TESTL   AX, AX
        0x0061 00097 (main_a.go:5)      JNE     135
        0x0063 00099 (main_a.go:5)      JMP     101
        0x0065 00101 (main_a.go:9)      MOVQ    $100, "".i+8(SP)  ;; 对局部变量i执行赋值操作
        0x006e 00110 (main_a.go:10)     MOVQ    $100, "".~r0+112(SP)  ;; 给返回值赋值
        0x0077 00119 (main_a.go:10)     XCHGL   AX, AX
        0x0078 00120 (main_a.go:10)     CALL    runtime.deferreturn(SB)
        0x007d 00125 (main_a.go:10)     MOVQ    96(SP), BP
        0x0082 00130 (main_a.go:10)     ADDQ    $104, SP
        0x0086 00134 (main_a.go:10)     RET
        0x0087 00135 (main_a.go:5)      XCHGL   AX, AX
        0x0088 00136 (main_a.go:5)      CALL    runtime.deferreturn(SB)
        0x008d 00141 (main_a.go:5)      MOVQ    96(SP), BP
        0x0092 00146 (main_a.go:5)      ADDQ    $104, SP
        0x0096 00150 (main_a.go:5)      RET
        0x0097 00151 (main_a.go:5)      NOP
        0x0097 00151 (main_a.go:3)      PCDATA  $1, $-1
        0x0097 00151 (main_a.go:3)      PCDATA  $0, $-2
        0x0097 00151 (main_a.go:3)      CALL    runtime.morestack_noctxt(SB)
        0x009c 00156 (main_a.go:3)      PCDATA  $0, $-1
        0x009c 00156 (main_a.go:3)      JMP     0
"".b STEXT size=139 args=0x8 locals=0x60
        0x0000 00000 (main_a.go:13)     TEXT    "".b(SB), ABIInternal, $96-8
        0x0000 00000 (main_a.go:13)     MOVQ    (TLS), CX
        0x0009 00009 (main_a.go:13)     CMPQ    SP, 16(CX)
        0x000d 00013 (main_a.go:13)     PCDATA  $0, $-2
        0x000d 00013 (main_a.go:13)     JLS     129
        0x000f 00015 (main_a.go:13)     PCDATA  $0, $-1
        0x000f 00015 (main_a.go:13)     SUBQ    $96, SP
        0x0013 00019 (main_a.go:13)     MOVQ    BP, 88(SP)
        0x0018 00024 (main_a.go:13)     LEAQ    88(SP), BP
        0x001d 00029 (main_a.go:13)     PCDATA  $0, $-2
        0x001d 00029 (main_a.go:13)     PCDATA  $1, $-2
        0x001d 00029 (main_a.go:13)     FUNCDATA        $0, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
        0x001d 00029 (main_a.go:13)     FUNCDATA        $1, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
        0x001d 00029 (main_a.go:13)     FUNCDATA        $2, gclocals·9fb7f0986f647f17cb53dda1484e0f7a(SB)
        0x001d 00029 (main_a.go:13)     PCDATA  $0, $0
        0x001d 00029 (main_a.go:13)     PCDATA  $1, $0
        0x001d 00029 (main_a.go:13)     MOVQ    $0, "".i+104(SP)  ;; 初始化有名返回值i，注意这里没有和a的区别
        0x0026 00038 (main_a.go:14)     MOVL    $8, ""..autotmp_2+8(SP)
        0x002e 00046 (main_a.go:14)     PCDATA  $0, $1
        0x002e 00046 (main_a.go:14)     LEAQ    "".b.func1·f(SB), AX
        0x0035 00053 (main_a.go:14)     PCDATA  $0, $0
        0x0035 00053 (main_a.go:14)     MOVQ    AX, ""..autotmp_2+32(SP)
        0x003a 00058 (main_a.go:14)     PCDATA  $0, $1
        0x003a 00058 (main_a.go:14)     LEAQ    "".i+104(SP), AX
        0x003f 00063 (main_a.go:14)     PCDATA  $0, $0
        0x003f 00063 (main_a.go:14)     MOVQ    AX, ""..autotmp_2+80(SP)
        0x0044 00068 (main_a.go:14)     PCDATA  $0, $1
        0x0044 00068 (main_a.go:14)     LEAQ    ""..autotmp_2+8(SP), AX
        0x0049 00073 (main_a.go:14)     PCDATA  $0, $0
        0x0049 00073 (main_a.go:14)     MOVQ    AX, (SP)
        0x004d 00077 (main_a.go:14)     CALL    runtime.deferprocStack(SB)
        0x0052 00082 (main_a.go:14)     TESTL   AX, AX
        0x0054 00084 (main_a.go:14)     JNE     113
        0x0056 00086 (main_a.go:14)     JMP     88
        0x0058 00088 (main_a.go:18)     MOVQ    $100, "".i+104(SP)  ;; 给i赋值100（其实就是相当于给返回值赋值100）
        0x0061 00097 (main_a.go:19)     XCHGL   AX, AX
        0x0062 00098 (main_a.go:19)     CALL    runtime.deferreturn(SB)
        0x0067 00103 (main_a.go:19)     MOVQ    88(SP), BP
        0x006c 00108 (main_a.go:19)     ADDQ    $96, SP
        0x0070 00112 (main_a.go:19)     RET
        0x0071 00113 (main_a.go:14)     XCHGL   AX, AX
        0x0072 00114 (main_a.go:14)     CALL    runtime.deferreturn(SB)
        0x0077 00119 (main_a.go:14)     MOVQ    88(SP), BP
        0x007c 00124 (main_a.go:14)     ADDQ    $96, SP
        0x0080 00128 (main_a.go:14)     RET
        0x0081 00129 (main_a.go:14)     NOP
        0x0081 00129 (main_a.go:13)     PCDATA  $1, $-1
        0x0081 00129 (main_a.go:13)     PCDATA  $0, $-2
        0x0081 00129 (main_a.go:13)     CALL    runtime.morestack_noctxt(SB)
        0x0086 00134 (main_a.go:13)     PCDATA  $0, $-1
        0x0086 00134 (main_a.go:13)     JMP     0
"".main STEXT size=189 args=0x0 locals=0x20  ;; 主函数main
        0x0000 00000 (main_a.go:22)     TEXT    "".main(SB), ABIInternal, $32-0
        0x0000 00000 (main_a.go:22)     MOVQ    (TLS), CX
        0x0009 00009 (main_a.go:22)     CMPQ    SP, 16(CX)
        0x000d 00013 (main_a.go:22)     PCDATA  $0, $-2
        0x000d 00013 (main_a.go:22)     JLS     179
        0x0013 00019 (main_a.go:22)     PCDATA  $0, $-1
        0x0013 00019 (main_a.go:22)     SUBQ    $32, SP
        0x0017 00023 (main_a.go:22)     MOVQ    BP, 24(SP)
        0x001c 00028 (main_a.go:22)     LEAQ    24(SP), BP
        0x0021 00033 (main_a.go:22)     PCDATA  $0, $-2
        0x0021 00033 (main_a.go:22)     PCDATA  $1, $-2
        0x0021 00033 (main_a.go:22)     FUNCDATA        $0, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
        0x0021 00033 (main_a.go:22)     FUNCDATA        $1, gclocals·33cdeccccebe80329f1fdbee7f5874cb(SB)
        0x0021 00033 (main_a.go:22)     FUNCDATA        $2, gclocals·9fb7f0986f647f17cb53dda1484e0f7a(SB)
        0x0021 00033 (main_a.go:23)     PCDATA  $0, $0
        0x0021 00033 (main_a.go:23)     PCDATA  $1, $0
        0x0021 00033 (main_a.go:23)     CALL    "".a(SB)
        0x0026 00038 (main_a.go:23)     MOVQ    (SP), AX
        0x002a 00042 (main_a.go:23)     MOVQ    AX, ""..autotmp_0+16(SP)
        0x002f 00047 (main_a.go:23)     CALL    runtime.printlock(SB)
        0x0034 00052 (main_a.go:23)     PCDATA  $0, $1
        0x0034 00052 (main_a.go:23)     LEAQ    go.string."a return: "(SB), AX
        0x003b 00059 (main_a.go:23)     PCDATA  $0, $0
        0x003b 00059 (main_a.go:23)     MOVQ    AX, (SP)
        0x003f 00063 (main_a.go:23)     MOVQ    $10, 8(SP)
        0x0048 00072 (main_a.go:23)     CALL    runtime.printstring(SB)
        0x004d 00077 (main_a.go:23)     MOVQ    ""..autotmp_0+16(SP), AX
        0x0052 00082 (main_a.go:23)     MOVQ    AX, (SP)
        0x0056 00086 (main_a.go:23)     CALL    runtime.printint(SB)
        0x005b 00091 (main_a.go:23)     CALL    runtime.printnl(SB)
        0x0060 00096 (main_a.go:23)     CALL    runtime.printunlock(SB)
        0x0065 00101 (main_a.go:24)     CALL    "".b(SB)
        0x006a 00106 (main_a.go:24)     MOVQ    (SP), AX
        0x006e 00110 (main_a.go:24)     MOVQ    AX, ""..autotmp_0+16(SP)
        0x0073 00115 (main_a.go:24)     CALL    runtime.printlock(SB)
        0x0078 00120 (main_a.go:24)     PCDATA  $0, $1
        0x0078 00120 (main_a.go:24)     LEAQ    go.string."b return: "(SB), AX
        0x007f 00127 (main_a.go:24)     PCDATA  $0, $0
        0x007f 00127 (main_a.go:24)     MOVQ    AX, (SP)
        0x0083 00131 (main_a.go:24)     MOVQ    $10, 8(SP)
        0x008c 00140 (main_a.go:24)     CALL    runtime.printstring(SB)
        0x0091 00145 (main_a.go:24)     MOVQ    ""..autotmp_0+16(SP), AX
        0x0096 00150 (main_a.go:24)     MOVQ    AX, (SP)
        0x009a 00154 (main_a.go:24)     CALL    runtime.printint(SB)
        0x009f 00159 (main_a.go:24)     CALL    runtime.printnl(SB)
        0x00a4 00164 (main_a.go:24)     CALL    runtime.printunlock(SB)
        0x00a9 00169 (main_a.go:25)     MOVQ    24(SP), BP
        0x00ae 00174 (main_a.go:25)     ADDQ    $32, SP
        0x00b2 00178 (main_a.go:25)     RET
        0x00b3 00179 (main_a.go:25)     NOP
        0x00b3 00179 (main_a.go:22)     PCDATA  $1, $-1
        0x00b3 00179 (main_a.go:22)     PCDATA  $0, $-2
        0x00b3 00179 (main_a.go:22)     CALL    runtime.morestack_noctxt(SB)
        0x00b8 00184 (main_a.go:22)     PCDATA  $0, $-1
        0x00b8 00184 (main_a.go:22)     JMP     0
"".a.func1 STEXT size=127 args=0x8 locals=0x20  ;; a函数内defer语句的函数
        0x0000 00000 (main_a.go:5)      TEXT    "".a.func1(SB), ABIInternal, $32-8
        0x0000 00000 (main_a.go:5)      MOVQ    (TLS), CX
        0x0009 00009 (main_a.go:5)      CMPQ    SP, 16(CX)
        0x000d 00013 (main_a.go:5)      PCDATA  $0, $-2
        0x000d 00013 (main_a.go:5)      JLS     120
        0x000f 00015 (main_a.go:5)      PCDATA  $0, $-1
        0x000f 00015 (main_a.go:5)      SUBQ    $32, SP
        0x0013 00019 (main_a.go:5)      MOVQ    BP, 24(SP)
        0x0018 00024 (main_a.go:5)      LEAQ    24(SP), BP
        0x001d 00029 (main_a.go:5)      PCDATA  $0, $-2
        0x001d 00029 (main_a.go:5)      PCDATA  $1, $-2
        0x001d 00029 (main_a.go:5)      FUNCDATA        $0, gclocals·1a65e721a2ccc325b382662e7ffee780(SB)
        0x001d 00029 (main_a.go:5)      FUNCDATA        $1, gclocals·69c1753bd5f81501d95132d08af04464(SB)
        0x001d 00029 (main_a.go:5)      FUNCDATA        $2, gclocals·1cf923758aae2e428391d1783fe59973(SB)
        0x001d 00029 (main_a.go:6)      PCDATA  $0, $1
        0x001d 00029 (main_a.go:6)      PCDATA  $1, $0
        0x001d 00029 (main_a.go:6)      MOVQ    "".&i+40(SP), AX  ;; 获取i的值并存放到寄存器中（记住这里同样会影响外部函数作用域内i的值，但并没有影响外部的返回值，因为i并不是返回值）
        0x0022 00034 (main_a.go:6)      PCDATA  $0, $0
        0x0022 00034 (main_a.go:6)      MOVQ    (AX), AX
        0x0025 00037 (main_a.go:6)      PCDATA  $0, $2
        0x0025 00037 (main_a.go:6)      MOVQ    "".&i+40(SP), CX
        0x002a 00042 (main_a.go:6)      INCQ    AX  ;; 自增1
        0x002d 00045 (main_a.go:6)      PCDATA  $0, $0
        0x002d 00045 (main_a.go:6)      MOVQ    AX, (CX)
        0x0030 00048 (main_a.go:7)      CALL    runtime.printlock(SB)
        0x0035 00053 (main_a.go:7)      PCDATA  $0, $1
        0x0035 00053 (main_a.go:7)      LEAQ    go.string."a defer: i = "(SB), AX
        0x003c 00060 (main_a.go:7)      PCDATA  $0, $0
        0x003c 00060 (main_a.go:7)      MOVQ    AX, (SP)
        0x0040 00064 (main_a.go:7)      MOVQ    $13, 8(SP)
        0x0049 00073 (main_a.go:7)      CALL    runtime.printstring(SB)
        0x004e 00078 (main_a.go:7)      PCDATA  $0, $1
        0x004e 00078 (main_a.go:7)      PCDATA  $1, $1
        0x004e 00078 (main_a.go:7)      MOVQ    "".&i+40(SP), AX
        0x0053 00083 (main_a.go:7)      PCDATA  $0, $0
        0x0053 00083 (main_a.go:7)      MOVQ    (AX), AX
        0x0056 00086 (main_a.go:7)      MOVQ    AX, ""..autotmp_1+16(SP)
        0x005b 00091 (main_a.go:7)      MOVQ    AX, (SP)
        0x005f 00095 (main_a.go:7)      CALL    runtime.printint(SB)
        0x0064 00100 (main_a.go:7)      CALL    runtime.printnl(SB)
        0x0069 00105 (main_a.go:7)      CALL    runtime.printunlock(SB)
        0x006e 00110 (main_a.go:8)      MOVQ    24(SP), BP
        0x0073 00115 (main_a.go:8)      ADDQ    $32, SP
        0x0077 00119 (main_a.go:8)      RET
        0x0078 00120 (main_a.go:8)      NOP
        0x0078 00120 (main_a.go:5)      PCDATA  $1, $-1
        0x0078 00120 (main_a.go:5)      PCDATA  $0, $-2
        0x0078 00120 (main_a.go:5)      CALL    runtime.morestack_noctxt(SB)
        0x007d 00125 (main_a.go:5)      PCDATA  $0, $-1
        0x007d 00125 (main_a.go:5)      JMP     0
"".b.func1 STEXT size=127 args=0x8 locals=0x20  ;; b函数内的defer语句的函数
        0x0000 00000 (main_a.go:14)     TEXT    "".b.func1(SB), ABIInternal, $32-8
        0x0000 00000 (main_a.go:14)     MOVQ    (TLS), CX
        0x0009 00009 (main_a.go:14)     CMPQ    SP, 16(CX)
        0x000d 00013 (main_a.go:14)     PCDATA  $0, $-2
        0x000d 00013 (main_a.go:14)     JLS     120
        0x000f 00015 (main_a.go:14)     PCDATA  $0, $-1
        0x000f 00015 (main_a.go:14)     SUBQ    $32, SP
        0x0013 00019 (main_a.go:14)     MOVQ    BP, 24(SP)
        0x0018 00024 (main_a.go:14)     LEAQ    24(SP), BP
        0x001d 00029 (main_a.go:14)     PCDATA  $0, $-2
        0x001d 00029 (main_a.go:14)     PCDATA  $1, $-2
        0x001d 00029 (main_a.go:14)     FUNCDATA        $0, gclocals·1a65e721a2ccc325b382662e7ffee780(SB)
        0x001d 00029 (main_a.go:14)     FUNCDATA        $1, gclocals·69c1753bd5f81501d95132d08af04464(SB)
        0x001d 00029 (main_a.go:14)     FUNCDATA        $2, gclocals·1cf923758aae2e428391d1783fe59973(SB)
        0x001d 00029 (main_a.go:15)     PCDATA  $0, $1
        0x001d 00029 (main_a.go:15)     PCDATA  $1, $0
        0x001d 00029 (main_a.go:15)     MOVQ    "".&i+40(SP), AX
        0x0022 00034 (main_a.go:15)     PCDATA  $0, $0
        0x0022 00034 (main_a.go:15)     MOVQ    (AX), AX
        0x0025 00037 (main_a.go:15)     PCDATA  $0, $2
        0x0025 00037 (main_a.go:15)     MOVQ    "".&i+40(SP), CX
        0x002a 00042 (main_a.go:15)     INCQ    AX
        0x002d 00045 (main_a.go:15)     PCDATA  $0, $0
        0x002d 00045 (main_a.go:15)     MOVQ    AX, (CX)
        0x0030 00048 (main_a.go:16)     CALL    runtime.printlock(SB)
        0x0035 00053 (main_a.go:16)     PCDATA  $0, $1
        0x0035 00053 (main_a.go:16)     LEAQ    go.string."b defer: i = "(SB), AX
        0x003c 00060 (main_a.go:16)     PCDATA  $0, $0
        0x003c 00060 (main_a.go:16)     MOVQ    AX, (SP)
        0x0040 00064 (main_a.go:16)     MOVQ    $13, 8(SP)
        0x0049 00073 (main_a.go:16)     CALL    runtime.printstring(SB)
        0x004e 00078 (main_a.go:16)     PCDATA  $0, $1
        0x004e 00078 (main_a.go:16)     PCDATA  $1, $1
        0x004e 00078 (main_a.go:16)     MOVQ    "".&i+40(SP), AX
        0x0053 00083 (main_a.go:16)     PCDATA  $0, $0
        0x0053 00083 (main_a.go:16)     MOVQ    (AX), AX
        0x0056 00086 (main_a.go:16)     MOVQ    AX, ""..autotmp_1+16(SP)
        0x005b 00091 (main_a.go:16)     MOVQ    AX, (SP)
        0x005f 00095 (main_a.go:16)     CALL    runtime.printint(SB)
        0x0064 00100 (main_a.go:16)     CALL    runtime.printnl(SB)
        0x0069 00105 (main_a.go:16)     CALL    runtime.printunlock(SB)
        0x006e 00110 (main_a.go:17)     MOVQ    24(SP), BP
        0x0073 00115 (main_a.go:17)     ADDQ    $32, SP
        0x0077 00119 (main_a.go:17)     RET
        0x0078 00120 (main_a.go:17)     NOP
        0x0078 00120 (main_a.go:14)     PCDATA  $1, $-1
        0x0078 00120 (main_a.go:14)     PCDATA  $0, $-2
        0x0078 00120 (main_a.go:14)     CALL    runtime.morestack_noctxt(SB)
        0x007d 00125 (main_a.go:14)     PCDATA  $0, $-1
        0x007d 00125 (main_a.go:14)     JMP     0
```



## 参考

1. [Golang 汇编入门知识总结](https://cloud.tencent.com/developer/article/1692904)
2. [汇编语言入门教程](https://www.ruanyifeng.com/blog/2018/01/assembly-language-primer.html)
3. [Golang中defer、return、返回值之间执行顺序的坑](https://my.oschina.net/henrylee2cn/blog/505535)
4. [彻底弄懂return和defer的微妙关系](https://juejin.cn/post/6844903877033066509)

