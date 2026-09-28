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

路径参数也不能只检查“字符串非空”。程序需要明确相对路径相对于当前工作目录解析，还是相对于可执行文件解析；需要创建输出文件时，还要处理目标已存在、父目录不存在和权限不足等情况。不要把用户提供的路径直接拼接到 shell 命令中，这会引入命令注入和空格转义问题；优先使用 C 标准库或操作系统提供的文件 API。

如果参数来自自动化脚本，输出格式应保持稳定。人类可读的提示写到 `stderr`，机器可读的结果写到 `stdout`，并为错误保留非零退出码。这样调用方可以把 `stdout` 重定向到文件，同时仍然看到失败原因。

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

选项解析通常需要处理以下情况：`--` 后面的内容全部视为位置参数；未知选项应立即报错；需要值的选项不能接受缺少值的形式；同一个选项重复出现时要规定“最后一次生效”还是“直接报错”。这些规则应写进 usage 文本，否则用户只能靠试错理解命令。

环境变量可以作为默认配置，但不能悄悄覆盖显式命令行参数。一个常见的优先级是：内置默认值 < 配置文件 < 环境变量 < 命令行选项。读取环境变量后仍要执行同样的格式和范围检查。

## 与文件程序组合

文件拷贝程序可以把 `file-engineering.md` 中的 `copy_file` 包装成：

```
file-copy source.bin target.bin
```

这样就能把函数、错误处理和操作系统提供的命令行环境连接起来。实际实现还应检查源路径和目标路径是否相同，并在覆盖目标前明确记录覆盖策略。参数解析属于输入校验，不应使用 `atoi` 代替 `strtol`，因为 `atoi` 无法可靠报告范围错误。

命令行程序还应记录终端行为：是否支持交互输入、遇到 `EOF` 是否正常结束、标准输入和标准输出是否可以重定向，以及输出中是否依赖颜色或光标控制。把这些约束写清楚，程序才能从“能在终端运行”变成“能被其他程序可靠调用”。
