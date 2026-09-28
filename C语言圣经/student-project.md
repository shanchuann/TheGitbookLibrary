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

## 完成标准

项目完成并不只看“能运行”：

1. 空数据文件可以正常启动；
2. 文件不存在时能给出可理解的错误；
3. 添加、删除、查询、排序结果稳定；
4. 程序退出前释放所有动态内存；
5. 使用 `-Wall -Wextra -Wpedantic` 编译无警告；
6. 代码可以拆分为多个编译单元并独立测试。

建议至少准备以下测试：空文件、重复学号、满容量扩容、内存分配失败、损坏记录、姓名包含逗号、排序中存在相同成绩，以及保存文件时磁盘写入失败。测试应检查返回值和错误信息，而不只检查“程序没有崩溃”。
