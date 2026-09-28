---
description: 使用结构体、动态内存、文件和多文件组织完成学生成绩管理系统。
icon: rss
---

# 学生成绩管理系统

这是全书的综合项目。它把结构体、数组、指针、动态内存、文件、排序、函数指针、错误处理和多文件组织连接起来。

![学生成绩管理系统示例运行结果](https://raw.githubusercontent.com/shanchuann/TheGitbookLibrary/main/C%E8%AF%AD%E8%A8%80%E5%9C%A3%E7%BB%8F/.gitbook/assets/student_manager_demo.png)

```mermaid
flowchart LR
    U[用户输入] --> M[菜单层]
    M --> S[学生数据模块]
    S --> F[文件持久化模块]
    S --> A[排序与查询]
    F --> S
```

## 数据模型

```c
#include <stddef.h>
#include <stdint.h>

typedef struct {
    char id[16];
    char name[32];
    double score;
} Student;

typedef struct {
    Student *items;
    size_t size;
    size_t capacity;
} StudentList;
```

`StudentList` 的所有权约定应明确：`items` 由列表拥有，只能通过列表接口扩容和释放；调用者传入的字符串要么被复制到固定数组，要么由接口明确要求调用者保证其生命周期。固定长度字段可以避免悬空指针，但必须在输入时检查截断和终止空字符。

一个最小的列表接口可以这样设计：

```c
void student_list_init(StudentList *list);
void student_list_destroy(StudentList *list);
int student_list_add(StudentList *list, Student value);
int student_list_remove_at(StudentList *list, size_t index);
const Student *student_list_get(const StudentList *list, size_t index);
```

每个函数都应说明空指针、越界索引、容量溢出和分配失败时的行为。接口返回状态码后，菜单层就不需要猜测操作是否成功。

下面的单文件版本先完成“排序并保存”这条最小闭环，再逐步拆分为多个模块：

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct { char id[16]; char name[32]; double score; } Student;

static int compare_score_desc(const void *left, const void *right) {
    const Student *a = left, *b = right;
    return (a->score < b->score) - (a->score > b->score);
}

static int save_students(const char *path, const Student students[], size_t count) {
    FILE *file = fopen(path, "w");
    if (file == NULL) return -1;
    for (size_t i = 0; i < count; ++i) {
        if (fprintf(file, "%s,%s,%.1f\n", students[i].id,
                    students[i].name, students[i].score) < 0) {
            fclose(file);
            return -1;
        }
    }
    return fclose(file) == 0 ? 0 : -1;
}

int main(void) {
    Student students[] = {
        {"S002", "Li", 86.0}, {"S001", "Wang", 92.5}, {"S003", "Zhao", 78.0}
    };
    size_t count = sizeof students / sizeof students[0];
    qsort(students, count, sizeof students[0], compare_score_desc);
    if (save_students("students.csv", students, count) != 0) return 1;
    for (size_t i = 0; i < count; ++i)
        printf("%s %s %.1f\n", students[i].id, students[i].name, students[i].score);
    puts("saved students.csv");
    return 0;
}
```

动态数组扩容时，不能直接把 `realloc` 的结果覆盖唯一原指针：

```c
int student_list_reserve(StudentList *list, size_t capacity) {
    if (capacity <= list->capacity) return 0;
    if (capacity > SIZE_MAX / sizeof *list->items) return -1;
    Student *new_items = realloc(list->items, capacity * sizeof *new_items);
    if (new_items == NULL) return -1;
    list->items = new_items;
    list->capacity = capacity;
    return 0;
}
```

## 功能拆分

建议的多文件结构：

```
student-manager/
├── main.c
├── student.c
├── student.h
├── storage.c
├── storage.h
├── menu.c
└── menu.h
```

核心功能包括：

* 添加、删除、修改学生；
* 按学号查找；
* 按成绩排序；
* 统计平均分、最高分和最低分；
* 从文件加载；
* 保存到文件；
* 对输入和文件错误给出明确提示。

模块之间不要互相访问内部数组。`student.c` 管理列表的不变量，`storage.c` 只负责把记录编码或解码，`menu.c` 负责交互和提示，`main.c` 负责初始化、主循环和最终清理。这样文件格式改变时，不必重写菜单逻辑。

排序和查询也应使用接口而不是直接修改内部字段。例如成绩排序需要规定相同成绩的次序；如果希望结果稳定，可以把学号或原始位置作为第二排序键。按学号查询前应先规定学号是否唯一，重复学号应在添加阶段拒绝，而不是等到查询时随机返回一个。

## 第一阶段：可运行闭环

不要一开始就实现所有菜单。建议按下面顺序交付：

1. 用固定数组完成显示、排序和统计；
2. 把数组替换为 `StudentList`，补齐初始化、扩容和释放；
3. 增加按学号查找、添加和删除，并为每个操作定义成功/失败返回值；
4. 最后加入文件加载和保存，再把单文件实现拆成多个编译单元。

每个阶段都应保留一个可以编译运行的版本。这样出现问题时，可以区分数据模型、内存管理和文件格式分别引入的错误。

文本格式也需要明确约束。示例中的逗号分隔格式只适合学号、姓名不含逗号和换行的输入；如果允许任意姓名，应实现 CSV 转义，或改用长度前缀的二进制/文本记录格式。

## 项目边界

这个项目首先使用文本文件，便于观察和调试；完成基础版本后，再增加二进制文件和逐字段序列化版本。书中代码应始终检查内存分配、文件打开、读写返回值和输入长度。

文本文件的加载不能只依赖 `fscanf("%s,%s,%lf")`。它无法正确处理带空格、逗号或引号的姓名，也不能清楚地区分格式错误和文件结束。初版可以明确限制字段字符集；需要支持通用 CSV 时，应实现引号和转义规则，或使用经过验证的解析器。

保存操作建议采用“写临时文件、刷新并关闭、替换目标文件”的流程。直接覆盖原文件时，如果程序在写入中途退出，下一次启动可能只能看到半个数据库。读取时则要限制最大记录数，避免损坏文件中的恶意数量导致超大内存申请。

## 完成标准

项目完成并不只看“能运行”：

1. 空数据文件可以正常启动；
2. 文件不存在时能给出可理解的错误；
3. 添加、删除、查询、排序结果稳定；
4. 程序退出前释放所有动态内存；
5. 使用 `-Wall -Wextra -Wpedantic` 编译无警告；
6. 代码可以拆分为多个编译单元并独立测试。

建议至少准备以下测试：空文件、重复学号、满容量扩容、内存分配失败、损坏记录、姓名包含逗号、排序中存在相同成绩，以及保存文件时磁盘写入失败。测试应检查返回值和错误信息，而不只检查“程序没有崩溃”。

还可以把程序拆成三类测试：纯函数测试（比较器、统计函数、字段校验）、模块测试（列表增删改查和存储读写）、端到端测试（从命令行启动，输入菜单操作，再重新加载文件）。测试数据应包含空集合、单条记录、大量记录和非法 UTF-8 或超长输入等边界。完成这些测试后，综合项目才真正把前面章节的结构体、指针、动态内存、文件和错误处理连接起来。
