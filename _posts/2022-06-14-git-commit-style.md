---
title: Git commit style
author: huimingz
date: 2022-06-14 12:50:10 +0800
category: [git]
tags: [git]
pin: false
---

## Commit Message

Commit 的内容结构应当如下：

```
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

type 类型如下：
| Type 	| 说明 	|
|---	|---	|
| feat 	| 新增特性 (feature) 	|
| fix 	| 修复 Bug(bug fix) 	|
| docs 	| 修改文档 (documentation) 	|
| style 	| 代码格式修改(white-space, formatting, missing semi colons, etc) 	|
| refactor 	| 代码重构(refactor) 	|
| perf 	| 改善性能(A code change that improves performance) 	|
| test 	| 测试(when adding missing tests) 	|
| build 	| 变更项目构建或外部依赖（例如 scopes: webpack、gulp、npm 等） 	|
| ci 	| 更改持续集成软件的配置文件和 package 中的 scripts 命令，例如 scopes: Travis, Circle 等 	|
| chore 	| 变更构建流程或辅助工具(比如更改测试环境) 	|
| revert 	| 代码回退 	|

## Reference

- [How to Write Better Git Commit Messages – A Step-By-Step Guide](https://www.freecodecamp.org/news/how-to-write-better-git-commit-messages/)
