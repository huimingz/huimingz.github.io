---
title: Golang在test中如何使用setUp和tearDown
author: huimingz
date: 2021-02-20 21:20:00 +0800
category: [Golang]
tags: [Golang]
pin: false
future: true
---

在进行单元测试时，我们有些时候需要在执行前和执行后做一些工作，比如准备测试数据，测试结束后的测试数据清理。


## 包范围的准备和清理
在Golang中并没有直接提供setUp和tearDown，单我们可以利用 ``TestMain(m *testing.M)`` 来实现：

```golang
package foo

import (
	"os"
	"testing"
)


func TestMain(m *testing.M) {
	// 退出状态码
	var exitCode int
	defer func() {
		os.Exit(exitCode)
	}()

	// 准备和清理工作
	setUp()
	defer tearDown()

	exitCode= m.Run()
}

func setUp() {
	println("SetUp called")
}

func tearDown() {
	println("TearDown called")
}

func TestFoo(t *testing.T) {
	// ...
}
```

运行测试:
```
$ go test -v ./foo/...
SetUp called
=== RUN   TestFoo
--- PASS: TestFoo (0.00s)
PASS
TearDown called
ok      demo/foo 0.004s
```

注意：
1. ``TestMain`` 只允许你处理在当前包进行测试前的准备和测试后的清理工作，并不允许你对每一个测试进行处理。
2. 这里的 ``TestMain`` 会对整个包下的test生效，所以在使用的时候要注意影响范围。

## 细粒度的准备和清理工作
在[Go 1.14更新](https://tip.golang.org/doc/go1.14#testing)之后，你可以在test或者benchmark中使用 ``T.Cleanup`` 和 ``B.Cleanup`` 来执行相应的清理工作。

> The testing package now supports cleanup functions, called after a test or benchmark has finished, by calling [T.Cleanup](https://tip.golang.org/pkg/testing#T.Cleanup) or [B.Cleanup](https://tip.golang.org/pkg/testing#B.Cleanup) respectively.

除了Go 1.14版本引入的功能之外，你还可以使用另外的一种解决方案，下面是一个表驱动测试的示例，来自于[gist](https://gist.github.com/kare/ec85d88b9b5f975936b67cd1bebdd621#file-go-unit-test-setup-and-teardown-math_setup_and_teardown_test-go):

```golang
package math

import "testing"

func setupTestCase(t *testing.T) func(t *testing.T) {
	t.Log("setup test case")
	return func(t *testing.T) {
		t.Log("teardown test case")
	}
}

func setupSubTest(t *testing.T) func(t *testing.T) {
	t.Log("setup sub test")
	return func(t *testing.T) {
		t.Log("teardown sub test")
	}
}

func TestAddition(t *testing.T) {
	cases := []struct {
		name     string
		a        int
		b        int
		expected int
	}{
		{"add", 2, 2, 4},
		{"minus", 0, -2, -2},
		{"zero", 0, 0, 0},
	}

	teardownTestCase := setupTestCase(t)
	defer teardownTestCase(t)

	for _, tc := range cases {
		t.Run(tc.name, func(t *testing.T) {
			teardownSubTest := setupSubTest(t)
			defer teardownSubTest(t)

			result := Sum(tc.a, tc.b)
			if result != tc.expected {
				t.Fatalf("expected sum %v, but got %v", tc.expected, result)
			}
		})
	}
}
```


## 参考
1. [setup and teardown for each test using std testing package](https://stackoverflow.com/questions/42310088/setup-and-teardown-for-each-test-using-std-testing-package)
2. [Go unit test setup and teardown](https://blog.karenuorteva.fi/go-unit-test-setup-and-teardown-db1601a796f2#.2aherx2z5)
