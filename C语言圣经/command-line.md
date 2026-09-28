---
description: argc、argv、参数校验与命令行文件程序。
icon: chess-clock
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

## 参数校验的边界

命令行参数仍然是外部输入。除了检查 `argc`，还要决定是否允许前导空白、正负号和额外字符；`strtol` 的 `end` 指针可以帮助调用者区分“没有数字”和“数字后还有垃圾”。数量较大的参数还应先检查是否会导致 `size_t` 乘法溢出。

程序应使用不同的退出状态表达不同失败原因：参数格式错误通常返回非零值，文件打开失败则应把路径和系统错误一起写到 `stderr`。不要把诊断信息混到标准输出中，否则脚本难以可靠解析结果。

## 选项与子命令

简单程序可以手动解析 `argv`；选项较多时，应把解析逻辑和业务逻辑分开：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

static void usage(const char *program) {
    fprintf(stderr, "usage: %s [-v] input\n", program);
}

int main(int argc, char *argv[]) {
    int verbose = 0;
    int index = 1;
    if (index < argc && strcmp(argv[index], "-v") == 0) {
        verbose = 1;
        ++index;
    }
    if (argc - index != 1) {
        usage(argv[0]);
        return EXIT_FAILURE;
    }
    if (verbose) fprintf(stderr, "input=%s\n", argv[index]);
    return EXIT_SUCCESS;
}
```

POSIX 环境可以使用 `getopt`，但它不是 ISO C 接口；需要跨 Windows、Linux 和 macOS 时，最好提供自己的小型解析器，或明确记录平台依赖。

## 与文件程序组合

文件拷贝程序可以把 `file-engineering.md` 中的 `copy_file` 包装成：

```
file-copy source.bin target.bin
```

这样就能把函数、错误处理和操作系统提供的命令行环境连接起来。实际实现还应检查源路径和目标路径是否相同，并在覆盖目标前明确记录覆盖策略。参数解析属于输入校验，不应使用 `atoi` 代替 `strtol`，因为 `atoi` 无法可靠报告范围错误。
