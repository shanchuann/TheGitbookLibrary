---
description: malloc、calloc、realloc、free、所有权与内存安全。
icon: code
---

# 动态内存

## 为什么需要动态内存

局部变量和数组的大小通常在进入代码块前就已经确定。如果数据量由用户输入、文件内容或运行时状态决定，固定长度数组可能不够用，也可能浪费空间。动态内存允许程序在运行期间申请一块存储空间，并在不再需要时释放。

动态内存通常来自堆区，但 C 标准并不要求实现必须使用名为“堆”的特定区域。程序只需要遵守申请、使用和释放的规则。

## malloc、calloc 与 free

### malloc

malloc 申请指定字节数的内存，返回一个 void 指针。申请成功后，内存中的字节值是不确定的：

```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    size_t count = 5;
    int *values = malloc(count * sizeof *values);

    if (values == NULL) {
        fprintf(stderr, "memory allocation failed\n");
        return EXIT_FAILURE;
    }

    for (size_t i = 0; i < count; ++i) {
        values[i] = (int)(i * i);
        printf("%d ", values[i]);
    }
    putchar('\n');

    free(values);
    values = NULL;
    return EXIT_SUCCESS;
}
```

malloc 只负责申请空间，不负责初始化内容。需要清零时，应显式初始化，或者使用 calloc。

### calloc

calloc 接收元素个数和每个元素的大小，申请空间后将所有字节初始化为零：

```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    size_t count = 4;
    int *values = calloc(count, sizeof *values);

    if (values == NULL) {
        return EXIT_FAILURE;
    }

    for (size_t i = 0; i < count; ++i) {
        printf("%d ", values[i]);
    }
    putchar('\n');

    free(values);
    return EXIT_SUCCESS;
}
```

对整数类型来说，清零字节通常得到数值 0；但对于浮点数、指针或结构体，不能把“所有字节为零”简单等同于所有成员都具有语言层面的零值。需要可移植地初始化复杂对象时，应逐成员赋值。

### free

free 释放由 malloc、calloc 或 realloc 申请的内存：

```c
int *p = malloc(10 * sizeof *p);

if (p != NULL) {
    free(p);
    p = NULL;
}
```

free(NULL) 是安全的。释放后将指针设为 NULL，可以降低重复使用同一指针的风险，但不能让其他指向同一内存的指针自动失效。

## 动态内存的生命周期

动态对象通常经历以下阶段：

```mermaid
stateDiagram-v2
    [*] --> 未分配
    未分配 --> 已分配: malloc / calloc 成功
    已分配 --> 已分配: realloc 成功
    已分配 --> 已释放: free
    已释放 --> [*]
    未分配 --> 分配失败: 返回 NULL
    分配失败 --> [*]
```

申请失败时，malloc、calloc 和 realloc 会返回 NULL。申请失败不是“小概率异常”，程序必须检查返回值并决定如何处理。

## sizeof 与溢出检查

申请数组时，通常使用 sizeof \*pointer，避免类型修改后忘记同步：

```c
double *data = malloc(count * sizeof *data);
```

计算 count 乘以元素大小时可能发生整数溢出。溢出后传给 malloc 的字节数可能比预期小，后续写入就会越界：

```c
#include <stdint.h>
#include <stdio.h>
#include <stdlib.h>

int allocate_ints(size_t count, int **out) {
    if (out == NULL || count > SIZE_MAX / sizeof **out) {
        return 0;
    }

    int *p = malloc(count * sizeof *p);
    if (p == NULL) {
        return 0;
    }

    *out = p;
    return 1;
}
```

实际项目中，任何来自文件、网络或用户输入的数量都应该在乘法前进行范围检查。

## realloc：调整已有内存

realloc 用于调整已有动态内存的大小：

```c
int *tmp = realloc(values, new_count * sizeof *values);
if (tmp == NULL) {
    /*
     * 原来的 values 仍然有效。
     * 这里可以释放它，也可以保留并采用降级方案。
     */
    free(values);
    values = NULL;
    return EXIT_FAILURE;
}

values = tmp;
```

不要直接覆盖唯一的原指针：

```c
values = realloc(values, new_count * sizeof *values);
```

如果 realloc 失败，它会返回 NULL，而原来的内存仍然存在。直接覆盖 values 会丢失原地址，造成内存泄漏。

realloc 成功后有两种可能：

* 内存区域在原位置扩大或缩小；
* 分配新的区域，复制旧内容，再释放旧区域。

因此，realloc 成功后原指针不能继续使用，必须使用返回的新指针。

当 new\_size 为 0 时，realloc 的行为和实现有关，不应依赖它来代替明确的 free。需要释放内存时，直接调用 free 更清楚。

## 所有权与接口设计

动态内存最容易出错的地方不是 malloc 本身，而是“这块内存由谁负责释放”没有说清楚。

常见的所有权约定：

| 场景            | 约定                |
| ------------- | ----------------- |
| 函数内部申请并返回指针   | 调用者负责最终 free      |
| 调用者传入缓冲区      | 调用者负责缓冲区的生命周期     |
| 函数只读取指针       | 函数不释放，也不修改所有权     |
| 函数接收二级指针并替换对象 | 接口文档必须说明旧对象是否会被释放 |
| 结构体拥有成员指针     | 结构体销毁函数负责释放成员     |

下面的接口让所有权关系清楚：函数申请空间，成功后由调用者释放。

```c
#include <stdlib.h>

int create_message(char **out) {
    if (out == NULL) {
        return 0;
    }

    char *message = malloc(32);
    if (message == NULL) {
        return 0;
    }

    message[0] = 'o';
    message[1] = 'k';
    message[2] = '\0';
    *out = message;
    return 1;
}
```

调用方：

```c
char *message = NULL;

if (create_message(&message)) {
    puts(message);
    free(message);
    message = NULL;
}
```

## 动态数组示例

下面的程序读取数量，动态申请数组，计算平均值，并在所有路径上释放资源：

```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    size_t count;

    printf("count: ");
    if (scanf("%zu", &count) != 1 || count == 0) {
        fprintf(stderr, "invalid count\n");
        return EXIT_FAILURE;
    }

    int *values = malloc(count * sizeof *values);
    if (values == NULL) {
        fprintf(stderr, "memory allocation failed\n");
        return EXIT_FAILURE;
    }

    long long sum = 0;
    for (size_t i = 0; i < count; ++i) {
        printf("value[%zu]: ", i);
        if (scanf("%d", &values[i]) != 1) {
            fprintf(stderr, "invalid value\n");
            free(values);
            return EXIT_FAILURE;
        }
        sum += values[i];
    }

    printf("average = %.2f\n", (double)sum / count);
    free(values);
    return EXIT_SUCCESS;
}
```

如果申请了多块内存，发生错误时要按照已经成功申请的部分逐一释放。资源清理应覆盖正常路径和错误路径。

## 二维动态数组

二维数据有多种分配方式。最简单的是申请一整块连续内存：

```c
#include <stdlib.h>

size_t rows = 3;
size_t cols = 4;

int *matrix = malloc(rows * cols * sizeof *matrix);
if (matrix == NULL) {
    return EXIT_FAILURE;
}

matrix[1 * cols + 2] = 42;
free(matrix);
```

访问元素时使用 row \* cols + column。连续内存有利于缓存访问，也只需要释放一次。

如果需要使用 matrix\[row]\[column] 的写法，可以申请指针数组和每一行：

```c
int **matrix = malloc(rows * sizeof *matrix);
if (matrix == NULL) {
    return EXIT_FAILURE;
}

for (size_t row = 0; row < rows; ++row) {
    matrix[row] = malloc(cols * sizeof *matrix[row]);
    if (matrix[row] == NULL) {
        while (row > 0) {
            free(matrix[--row]);
        }
        free(matrix);
        return EXIT_FAILURE;
    }
}

/* 使用 matrix[row][column] */

for (size_t row = 0; row < rows; ++row) {
    free(matrix[row]);
}
free(matrix);
```

这种方式的每一行可能位于不同位置，释放时必须先释放每一行，再释放指针数组。它与真正的二维数组 int matrix\[rows]\[cols] 不是同一种类型。

## 常见错误

| 错误                  | 后果                   |
| ------------------- | -------------------- |
| 解引用 malloc 返回的 NULL | 未定义行为或程序崩溃           |
| 使用 malloc 后未初始化就读取  | 读取不确定值               |
| 申请字节数计算溢出           | 分配空间过小，后续越界          |
| 直接覆盖 realloc 原指针    | realloc 失败时发生内存泄漏    |
| free 后继续读写          | use-after-free，未定义行为 |
| 对同一地址重复 free        | double free，未定义行为    |
| 忘记 free             | 内存泄漏                 |
| 返回局部数组地址            | 函数结束后指针悬空            |
| 释放不是动态申请得到的地址       | 未定义行为                |
| 把二维数组当作 int \*\*    | 指针类型和步长不匹配           |
| 多行分配只释放外层指针         | 每行内存泄漏               |
| 把内存所有权交给多个模块        | 容易重复释放或遗漏释放          |

动态内存的三个检查问题：

1. 申请是否成功？
2. 这块内存现在由谁负责？
3. 所有退出路径是否都能释放它？

## 调试工具

GCC 和 Clang 可以使用 AddressSanitizer 检查越界、释放后使用和重复释放：

```bash
gcc -std=c17 -Wall -Wextra -Wpedantic \
    -g -fsanitize=address,undefined dynamic_memory.c \
    -o dynamic_memory

./dynamic_memory
```

Linux 下还可以使用 Valgrind 检查内存泄漏：

```bash
valgrind --leak-check=full ./dynamic_memory
```

这些工具不能替代边界设计和所有权约定，但能把许多“运行一会儿才崩”的问题提前暴露出来。

## 从 malloc 到操作系统

malloc、calloc、realloc 和 free 是 C 标准库接口。它们通常由用户态的内存分配器实现，分配器再向操作系统申请更大的虚拟内存区域。因此，一次 malloc 不一定对应一次系统调用：

```
C 程序
  |
  v
malloc / calloc / realloc / free
  |
  v
C 运行库中的分配器
  |
  +--> brk / sbrk       管理传统数据段末端
  |
  +--> mmap / munmap    映射或解除映射虚拟内存
  |
  v
操作系统虚拟内存管理
```

分配器会在已经取得的大块区域中切分小块，记录空闲块、大小和对齐信息。这样可以减少系统调用，但也会带来内部碎片和外部碎片。free 通常先把内存归还给分配器，不一定立即归还给操作系统。

### brk 与 sbrk

在 Unix 系统中，程序有一个称为 program break 的位置，传统堆空间位于数据段末端附近。brk 和 sbrk 可以调整这个位置：

| 接口              | 作用                             |
| --------------- | ------------------------------ |
| brk(address)    | 把 program break 设置为指定地址        |
| sbrk(increment) | 按字节数移动 program break，并返回移动前的位置 |

这两个接口属于 Unix/POSIX 系统接口，不是 ISO C 标准的一部分。现代程序通常不应直接调用它们，原因包括：

* 它们可能与 malloc 使用的分配器元数据发生冲突；
* program break 只能描述一段连续区域，不适合所有分配模式；
* 不同系统和运行库对它们的支持不同；
* 直接移动 program break 后，原有 malloc 指针可能全部失效。

示例仅用于观察系统接口，不要与 malloc 混用：

```c
#define _DEFAULT_SOURCE
#include <unistd.h>
#include <stdio.h>

int main(void) {
    void *old_break = sbrk(0);
    if (old_break == (void *)-1) {
        perror("sbrk");
        return 1;
    }

    printf("program break: %p\n", old_break);
    return 0;
}
```

sbrk(0) 只查询当前位置。即使在 Linux 上可以调用，也不能据此推断 malloc 的全部内存都来自 program break。

### mmap 与 munmap

mmap 可以把文件或匿名内存映射到进程的虚拟地址空间。Linux 下申请匿名可读写内存的示例：

```c
#define _GNU_SOURCE
#include <sys/mman.h>
#include <stdio.h>
#include <unistd.h>

int main(void) {
    size_t page_size = (size_t)sysconf(_SC_PAGESIZE);
    size_t length = page_size * 2;

    void *memory = mmap(
        NULL,
        length,
        PROT_READ | PROT_WRITE,
        MAP_PRIVATE | MAP_ANONYMOUS,
        -1,
        0
    );

    if (memory == MAP_FAILED) {
        perror("mmap");
        return 1;
    }

    int *values = memory;
    values[0] = 42;
    values[1] = 84;
    printf("%d %d\n", values[0], values[1]);

    if (munmap(memory, length) != 0) {
        perror("munmap");
        return 1;
    }
    return 0;
}
```

编译：

```bash
gcc -std=c17 -Wall -Wextra -Wpedantic mmap_demo.c -o mmap_demo
./mmap_demo
```

mmap 失败时返回 MAP\_FAILED，而不是 NULL。解除映射必须使用 mmap 返回的起始地址和对应长度，解除映射后不能继续访问该区域。

常用参数：

| 参数             | 含义             |
| -------------- | -------------- |
| PROT\_READ     | 页面可读           |
| PROT\_WRITE    | 页面可写           |
| PROT\_EXEC     | 页面可执行          |
| MAP\_PRIVATE   | 写入时使用私有副本      |
| MAP\_SHARED    | 修改可与其他映射者共享    |
| MAP\_ANONYMOUS | 不对应磁盘文件，初始内容为零 |

mmap 是 POSIX/Linux 接口，不应写进只要求 ISO C 的通用库接口中。需要跨平台申请普通动态内存时，优先使用 malloc 系列函数。

### 虚拟内存与页面

操作系统以页为单位管理虚拟内存。mmap 通常以页为粒度建立映射，页大小可以通过 sysconf 查询，常见值是 4096 字节，但不能写死。

申请虚拟地址空间和实际占用物理内存是两个概念。操作系统可能采用按需分配：只有程序第一次访问某个页面时，才建立对应的物理映射。Linux 的 overcommit 策略还可能让申请成功和最终可用的物理内存之间存在差异，因此程序仍需限制总申请量并正确处理异常。

可以使用 mprotect 修改已映射页面的访问权限。例如，在分配区域两端设置不可访问的保护页，可以帮助发现越界访问：

```
可读写页面 | 可读写页面 | 保护页
           ^
           越界访问在这里触发异常
```

保护页和内存映射属于系统级调试与安全技术，具体实现依赖操作系统。

## 对齐与特殊分配

malloc 返回的地址满足普通对象的对齐要求。需要更高对齐要求时，可以使用 aligned\_alloc：

```c
#include <stdlib.h>

size_t alignment = 64;
size_t size = 1024;

void *memory = aligned_alloc(alignment, size);
if (memory == NULL) {
    return EXIT_FAILURE;
}

/* 使用满足 64 字节对齐的内存 */
free(memory);
```

aligned\_alloc 属于 C11。size 必须是 alignment 的整数倍；不满足时，调用不符合函数要求。Windows 或 POSIX 环境也提供其他对齐接口，但释放方式必须遵循对应平台的规定，不能混用释放函数。

## 分配器边界

动态内存接口必须成对使用：

| 申请方式           | 对应释放方式 |
| -------------- | ------ |
| malloc         | free   |
| calloc         | free   |
| realloc 返回的地址  | free   |
| aligned\_alloc | free   |
| mmap           | munmap |

不要使用 free 释放 mmap 返回的地址，也不要使用 munmap 释放 malloc 返回的地址。它们的内部元数据和生命周期管理完全不同。

从系统角度看，动态内存问题可以分为三层：

1. C 代码的边界、类型和生命周期是否正确；
2. 分配器是否正确处理空闲块、碎片和并发；
3. 操作系统是否成功提供虚拟页和物理页。

初学阶段先把第一层做好：检查返回值，记录所有权，避免越界和释放后使用。理解 brk 与 mmap 后，再去观察分配器和操作系统如何完成后两层工作。

## 分配大小的溢出检查

动态分配前必须确认元素个数乘以元素大小没有溢出：

```c
#include <stdint.h>
#include <stdlib.h>

void *array_alloc(size_t count, size_t size) {
    if (size != 0 && count > SIZE_MAX / size) {
        return NULL;
    }
    return malloc(count * size);
}
```

`malloc(0)` 的行为允许返回空指针，也允许返回一个不能解引用但可以传给 `free` 的特殊指针。实际接口应避免把零长度申请当作普通对象使用。

`brk/sbrk` 是传统进程堆边界接口，`mmap/munmap` 以虚拟内存区域为单位管理映射。应用程序通常不直接调用它们，而是通过 C 运行库分配器获得内存；分配器可能从堆或匿名映射取得更大的区域，再切分给调用者。

## 原稿图示
