---
description: 这本 C 语言书的写作说明与配套练习。
icon: code
---

# 书籍介绍

我写这本书，是因为很多 C 语言资料在两个极端之间摇摆，要么是由标准文档堆砌而成，要么只给出能运行代码。初学者真正需要的是一条能走通的路。先看语法，再看内存和编译器怎样处理它，最后用小程序把结论跑出来。

本书配套一个练习仓库 [CStudy](https://github.com/shanchuann/CStudy.git)。书里的示例尽量保持短小，练习仓库则负责把它们放进可以反复编译、调试和修改的环境里。代码会报错，这是正常现象；完全不报错的学习过程，通常只是还没按下编译键。

## 示例编译约定

书中 C 代码默认按 C11 编译，并开启常见警告：

```sh
gcc -std=c11 -Wall -Wextra -Wpedantic -g source.c -o program
```

新增章节的完整示例和运行截图位于 `/home/shanchuan/CStudy/book_examples/`。调试内存问题时，可以额外使用 `-fsanitize=address,undefined`；Windows、Linux 和 macOS 的编译器扩展不属于 ISO C 的可移植保证。

新增章节中的“可运行完整示例”可以直接复制到同名 `.c` 文件中编译；截图用于展示 WSL2 GCC 的一次验证结果，不代替本地编译。
