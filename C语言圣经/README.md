---
description: 这本 C 语言书的写作说明与配套练习。
icon: layer-plus
cover: .gitbook/assets/imagegen2-20260927-010428-1.png
coverY: -61.66703470031546
coverHeight: 501
layout:
  width: default
  cover:
    visible: true
    size: hero
    mask: radial
  title:
    visible: true
  description:
    visible: true
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
  tags:
    visible: true
  actions:
    visible: true
  anchors:
    visible: true
tags:
  - c
  - exercise
---

# 书籍介绍

当我读完一些关于 C 语言的书籍或教程后，发现他们要么过于全面，要么过于混乱，对于新手来说这并不是一件好事。所以这本书就因此而来——它可以帮助你系统化建立有关 C 语言的知识体系，甚至足够你后续对技术的更深一层次的探索。

这本书有一个配套的 [_**CStudy**_](https://github.com/shanchuann/CStudy.git) 练习系统，随着本书的更新它里面的习题也会逐步更新。代码会报错，这是正常现象，完全不报错的学习过程，通常只是还没按下编译键。

#### **编译诊断与可移植性**

教学示例**建议**使用明确的标准版本和警告选项：

```bash
gcc -std=c17 -Wall -Wextra -Wpedantic -Wconversion -g main.c -o main
```

其中：

* `-std=c17` 指定语言标准，避免编译器默认模式不同；
* `-Wall -Wextra -Wpedantic` 打开常见警告（编译器警告不是“编译器多管闲事”，而是把许多运行时问题提前暴露出来。应尽量做到零警告，再进入运行测试）；
* `-Wconversion` 帮助发现隐式窄化转换；
* `-g` 保留调试信息，便于使用 GDB 或 AddressSanitizer。

调试内存问题时，可以额外使用 `-fsanitize=address,undefined`；Windows、Linux 和 macOS 的编译器扩展不属于 ISO C 的可移植保证。

“可运行完整示例”可以直接复制到同名 `.c` 文件中编译；书中截图用于展示 WSL2 GCC 的一次验证结果，不代替本地编译。
