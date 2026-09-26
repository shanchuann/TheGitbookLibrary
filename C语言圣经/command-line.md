---
description: argc、argv、参数校验与命令行文件程序。
icon: code
---

# 命令行参数

命令行参数让程序可以由脚本和其他程序调用。`argc` 表示参数数量，`argv` 是以 `NULL` 结尾的字符串指针数组；`argv[0]` 通常是程序名。

```c
#include <stdio.h>

int main(int argc, char *argv[]) {
    for (int i = 0; i < argc; ++i)
        printf("argv[%d] = %s\n", i, argv[i]);
    return 0;
}
```

使用参数前必须检查数量和格式，不能直接把用户输入当成可信路径或整数：

```c
#include <errno.h>
#include <limits.h>
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "usage: %s number\n", argv[0]);
        return EXIT_FAILURE;
    }

    char *end = NULL;
    errno = 0;
    long value = strtol(argv[1], &end, 10);
    if (errno == ERANGE || end == argv[1] || *end != '\0' ||
        value < INT_MIN || value > INT_MAX) {
        fprintf(stderr, "invalid integer: %s\n", argv[1]);
        return EXIT_FAILURE;
    }
    printf("value=%ld\n", value);
    return EXIT_SUCCESS;
}
```

文件拷贝程序可以把 `file-engineering.md` 中的 `copy_file` 包装成：

```text
file-copy source.bin target.bin
```

这样就能把函数、错误处理和操作系统提供的命令行环境连接起来。参数解析属于输入校验，不应使用 `atoi` 代替 `strtol`，因为 `atoi` 无法可靠报告范围错误。
