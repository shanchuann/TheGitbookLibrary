---
description: 使用结构体、动态内存、文件和多文件组织完成学生成绩管理系统。
icon: rss
---

# 学生成绩管理系统

这一章把前面学过的结构体、字符串、动态内存、文件、排序、错误处理和多文件编译组合成一个可以真正运行的项目。目标不是堆出一个很长的 `main` 函数，而是建立清晰的边界：数据模块管理学生记录，存储模块管理文件，菜单层管理交互，`main` 只负责组织生命周期。

## 需求和边界

程序提供以下功能：

* 添加、删除、修改学生；
* 按学号查询；
* 按成绩降序排列；
* 显示平均分、最高分和最低分；
* 从 CSV 文件加载，退出前保存；
* 通过命令行参数选择数据文件。

本实现把学号限制为字母、数字、下划线和短横线，把姓名限制为不含逗号和换行的文本，把成绩限制在 `0` 到 `100`。这些限制不是“偷懒”，而是先明确文件格式边界。若以后要允许逗号，应增加真正的 CSV 引号解析，而不能只修改一个 `strtok` 调用。

## 数据模型和所有权

每条记录使用固定长度字符数组，避免把菜单层的临时字符串地址保存到列表中。`StudentList.items` 是动态数组的唯一所有者；调用 `student_list_destroy` 后，调用者不得继续使用其中的指针。

```c
/* student.h */
#ifndef STUDENT_H
#define STUDENT_H

#include <stddef.h>

#define STUDENT_ID_CAP 16
#define STUDENT_NAME_CAP 64

typedef struct {
    char id[STUDENT_ID_CAP];
    char name[STUDENT_NAME_CAP];
    double score;
} Student;

typedef struct {
    Student *items;
    size_t size;
    size_t capacity;
} StudentList;

typedef enum {
    STUDENT_OK = 0,
    STUDENT_INVALID,
    STUDENT_DUPLICATE_ID,
    STUDENT_NOT_FOUND,
    STUDENT_OUT_OF_RANGE,
    STUDENT_NO_MEMORY
} StudentResult;

void student_list_init(StudentList *list);
void student_list_destroy(StudentList *list);
StudentResult student_validate(const Student *student);
StudentResult student_list_add(StudentList *list, const Student *student);
StudentResult student_list_remove_at(StudentList *list, size_t index);
StudentResult student_list_update(StudentList *list, size_t index,
                                  const Student *student);
const Student *student_list_get(const StudentList *list, size_t index);
long student_list_find_id(const StudentList *list, const char *id);
void student_list_sort_by_score(StudentList *list);
double student_list_average(const StudentList *list);

#endif
```

接口返回状态码而不是偷偷打印错误。这样同一个数据模块既可以被终端菜单调用，也可以被自动化测试调用。`student_list_find_id` 使用 `-1` 表示找不到，因此索引必须转换为 `long` 后再返回；列表过大时应进一步改成专门的结果结构。

## 实现动态列表

列表需要维护三个不变量：`items` 指向一块可容纳 `capacity` 个元素的区域，`size <= capacity`，有效记录只位于 `[0, size)`。扩容时先用临时指针接收 `realloc` 结果，失败时保留原数组。

```c
/* student.c */
#include "student.h"

#include <ctype.h>
#include <math.h>
#include <stdlib.h>
#include <string.h>

static int valid_id_char(unsigned char ch) {
    return isalnum(ch) || ch == '_' || ch == '-';
}

static int valid_id(const char *id) {
    size_t length;
    if (id == NULL || id[0] == '\0') return 0;
    length = strlen(id);
    if (length >= STUDENT_ID_CAP) return 0;
    for (size_t i = 0; i < length; ++i) {
        if (!valid_id_char((unsigned char)id[i])) return 0;
    }
    return 1;
}

static int valid_name(const char *name) {
    if (name == NULL || name[0] == '\0') return 0;
    if (strlen(name) >= STUDENT_NAME_CAP) return 0;
    for (const unsigned char *p = (const unsigned char *)name; *p; ++p) {
        if (*p == ',' || *p == '\n' || *p == '\r') return 0;
    }
    return 1;
}

StudentResult student_validate(const Student *student) {
    if (student == NULL || !valid_id(student->id) ||
        !valid_name(student->name) || !isfinite(student->score) ||
        student->score < 0.0 || student->score > 100.0) {
        return STUDENT_INVALID;
    }
    return STUDENT_OK;
}

void student_list_init(StudentList *list) {
    if (list != NULL) {
        list->items = NULL;
        list->size = 0;
        list->capacity = 0;
    }
}

void student_list_destroy(StudentList *list) {
    if (list != NULL) {
        free(list->items);
        student_list_init(list);
    }
}

static StudentResult reserve(StudentList *list, size_t needed) {
    size_t capacity;
    Student *new_items;
    if (needed <= list->capacity) return STUDENT_OK;
    capacity = list->capacity == 0 ? 8 : list->capacity;
    while (capacity < needed) {
        if (capacity > (size_t)-1 / 2) return STUDENT_NO_MEMORY;
        capacity *= 2;
    }
    if (capacity > (size_t)-1 / sizeof *list->items) return STUDENT_NO_MEMORY;
    new_items = realloc(list->items, capacity * sizeof *new_items);
    if (new_items == NULL) return STUDENT_NO_MEMORY;
    list->items = new_items;
    list->capacity = capacity;
    return STUDENT_OK;
}

long student_list_find_id(const StudentList *list, const char *id) {
    if (list == NULL || id == NULL) return -1;
    for (size_t i = 0; i < list->size; ++i) {
        if (strcmp(list->items[i].id, id) == 0) return (long)i;
    }
    return -1;
}

StudentResult student_list_add(StudentList *list, const Student *student) {
    StudentResult result;
    if (list == NULL) return STUDENT_INVALID;
    result = student_validate(student);
    if (result != STUDENT_OK) return result;
    if (student_list_find_id(list, student->id) >= 0) return STUDENT_DUPLICATE_ID;
    result = reserve(list, list->size + 1);
    if (result != STUDENT_OK) return result;
    list->items[list->size++] = *student;
    return STUDENT_OK;
}

StudentResult student_list_remove_at(StudentList *list, size_t index) {
    if (list == NULL || index >= list->size) return STUDENT_OUT_OF_RANGE;
    if (index + 1 < list->size) {
        memmove(&list->items[index], &list->items[index + 1],
                (list->size - index - 1) * sizeof list->items[0]);
    }
    --list->size;
    return STUDENT_OK;
}

StudentResult student_list_update(StudentList *list, size_t index,
                                  const Student *student) {
    StudentResult result;
    long duplicate;
    if (list == NULL || index >= list->size) return STUDENT_OUT_OF_RANGE;
    result = student_validate(student);
    if (result != STUDENT_OK) return result;
    duplicate = student_list_find_id(list, student->id);
    if (duplicate >= 0 && (size_t)duplicate != index) return STUDENT_DUPLICATE_ID;
    list->items[index] = *student;
    return STUDENT_OK;
}

const Student *student_list_get(const StudentList *list, size_t index) {
    if (list == NULL || index >= list->size) return NULL;
    return &list->items[index];
}

static int compare_score_desc(const void *left, const void *right) {
    const Student *a = left;
    const Student *b = right;
    if (a->score < b->score) return 1;
    if (a->score > b->score) return -1;
    return strcmp(a->id, b->id);
}

void student_list_sort_by_score(StudentList *list) {
    if (list != NULL && list->size > 1) {
        qsort(list->items, list->size, sizeof list->items[0], compare_score_desc);
    }
}

double student_list_average(const StudentList *list) {
    double total = 0.0;
    if (list == NULL || list->size == 0) return 0.0;
    for (size_t i = 0; i < list->size; ++i) total += list->items[i].score;
    return total / (double)list->size;
}
```

删除使用 `memmove` 填补空洞，排序在成绩相同的时候再按学号排序，因此输出顺序是确定的。`qsort` 本身不保证稳定，但这里明确了第二关键字，不依赖“恰好保持原顺序”这种未写进接口的行为。

## 文件存储模块

文件格式为一行一条记录：`学号,姓名,成绩`，例如 `S001,Zhang San,92.5`。加载时必须区分三种情况：正常读到一行、文件结束、读取发生错误。`fgets` 返回 `NULL` 后还要检查 `ferror`，不能把磁盘错误误报成“文件读完了”。

```c
/* storage.h */
#ifndef STORAGE_H
#define STORAGE_H

#include "student.h"

int students_load(const char *path, StudentList *list);
int students_save(const char *path, const StudentList *list);

#endif
```

```c
/* storage.c */
#include "storage.h"

#include <errno.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

static void trim_newline(char *line) {
    line[strcspn(line, "\r\n")] = '\0';
}

static int parse_line(char *line, Student *student) {
    char *first = strchr(line, ',');
    char *second;
    char *end;
    double score;
    if (first == NULL) return -1;
    *first = '\0';
    second = strchr(first + 1, ',');
    if (second == NULL) return -1;
    *second = '\0';
    errno = 0;
    score = strtod(second + 1, &end);
    if (errno != 0 || end == second + 1 || *end != '\0') return -1;
    if (snprintf(student->id, sizeof student->id, "%s", line) >=
            (int)sizeof student->id ||
        snprintf(student->name, sizeof student->name, "%s", first + 1) >=
            (int)sizeof student->name) return -1;
    student->score = score;
    return student_validate(student) == STUDENT_OK ? 0 : -1;
}

int students_load(const char *path, StudentList *list) {
    FILE *file;
    char line[256];
    StudentList loaded;
    if (path == NULL || list == NULL) return -1;
    file = fopen(path, "r");
    if (file == NULL) return errno == ENOENT ? 1 : -1;
    student_list_init(&loaded);
    while (fgets(line, sizeof line, file) != NULL) {
        Student student;
        if (strchr(line, '\n') == NULL && !feof(file)) {
            fclose(file);
            student_list_destroy(&loaded);
            return -1;
        }
        trim_newline(line);
        if (line[0] == '\0') continue;
        if (parse_line(line, &student) != 0 ||
            student_list_add(&loaded, &student) != STUDENT_OK) {
            fclose(file);
            student_list_destroy(&loaded);
            return -1;
        }
    }
    if (ferror(file) || fclose(file) != 0) {
        student_list_destroy(&loaded);
        return -1;
    }
    student_list_destroy(list);
    *list = loaded;
    return 0;
}

int students_save(const char *path, const StudentList *list) {
    char temporary[512];
    FILE *file;
    if (path == NULL || list == NULL ||
        snprintf(temporary, sizeof temporary, "%s.tmp", path) >=
            (int)sizeof temporary) return -1;
    file = fopen(temporary, "w");
    if (file == NULL) return -1;
    for (size_t i = 0; i < list->size; ++i) {
        const Student *student = &list->items[i];
        if (fprintf(file, "%s,%s,%.17g\n", student->id, student->name,
                    student->score) < 0) {
            fclose(file);
            remove(temporary);
            return -1;
        }
    }
    if (fclose(file) != 0) {
        remove(temporary);
        return -1;
    }
    /* 简化的跨平台替换：生产程序还应处理 Windows 上目标文件已存在的情况。 */
    remove(path);
    if (rename(temporary, path) != 0) {
        remove(temporary);
        return -1;
    }
    return 0;
}
```

保存先写 `*.tmp`，写完并关闭后才替换旧文件，避免程序在中途退出时留下半个数据库。这里的 CSV 解析明确不支持带逗号的姓名；这是一个可测试的边界，而不是隐藏的限制。要支持完整 CSV，应加入引号、双引号转义和字段长度限制。

## 菜单和完整主程序

菜单层只做输入转换和提示，不直接操作 `items`。`read_line` 去掉换行但保留空格，`read_score` 使用 `strtod` 检查尾部是否还有非法字符。

```c
/* main.c */
#include "storage.h"

#include <errno.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

static void read_line(const char *prompt, char *buffer, size_t capacity) {
    int ch;
    fputs(prompt, stdout);
    if (fgets(buffer, (int)capacity, stdin) == NULL) {
        buffer[0] = '\0';
        return;
    }
    if (strchr(buffer, '\n') == NULL) {
        while ((ch = getchar()) != '\n' && ch != EOF) {}
    }
    buffer[strcspn(buffer, "\r\n")] = '\0';
}

static int read_index(const char *prompt, size_t *index) {
    char text[32];
    char *end;
    unsigned long value;
    read_line(prompt, text, sizeof text);
    errno = 0;
    value = strtoul(text, &end, 10);
    if (errno != 0 || end == text || *end != '\0') return 0;
    *index = (size_t)value;
    return 1;
}

static int read_score(const char *prompt, double *score) {
    char text[64];
    char *end;
    read_line(prompt, text, sizeof text);
    errno = 0;
    *score = strtod(text, &end);
    return errno == 0 && end != text && *end == '\0';
}

static void print_student(size_t index, const Student *student) {
    printf("%3zu  %-15s %-20s %6.2f\n", index, student->id,
           student->name, student->score);
}

static void print_all(const StudentList *list) {
    puts("编号  学号            姓名                   成绩");
    puts("---------------------------------------------------");
    for (size_t i = 0; i < list->size; ++i) print_student(i, &list->items[i]);
    printf("共 %zu 条，平均分 %.2f\n", list->size, student_list_average(list));
}

static void add_student(StudentList *list) {
    Student student = {0};
    read_line("学号: ", student.id, sizeof student.id);
    read_line("姓名: ", student.name, sizeof student.name);
    if (!read_score("成绩: ", &student.score)) {
        puts("成绩格式错误。");
        return;
    }
    switch (student_list_add(list, &student)) {
        case STUDENT_OK: puts("添加成功。"); break;
        case STUDENT_DUPLICATE_ID: puts("学号已经存在。"); break;
        case STUDENT_NO_MEMORY: puts("内存不足，添加失败。"); break;
        default: puts("记录无效，请检查字段长度和成绩范围。"); break;
    }
}

static void edit_student(StudentList *list) {
    size_t index;
    Student student;
    const Student *old;
    if (!read_index("编号: ", &index) || (old = student_list_get(list, index)) == NULL) {
        puts("编号不存在。");
        return;
    }
    student = *old;
    read_line("新学号: ", student.id, sizeof student.id);
    read_line("新姓名: ", student.name, sizeof student.name);
    if (!read_score("新成绩: ", &student.score)) {
        puts("成绩格式错误。");
        return;
    }
    puts(student_list_update(list, index, &student) == STUDENT_OK ?
         "修改成功。" : "修改失败，请检查字段或重复学号。");
}

static void delete_student(StudentList *list) {
    size_t index;
    if (!read_index("编号: ", &index) ||
        student_list_remove_at(list, index) != STUDENT_OK) {
        puts("编号不存在。");
        return;
    }
    puts("删除成功。");
}

static void find_student(const StudentList *list) {
    char id[STUDENT_ID_CAP];
    long index;
    read_line("学号: ", id, sizeof id);
    index = student_list_find_id(list, id);
    if (index < 0) puts("没有找到该学生。");
    else print_student((size_t)index, &list->items[index]);
}

static void menu(void) {
    puts("\n1. 显示全部  2. 添加  3. 修改  4. 删除");
    puts("5. 按学号查询  6. 按成绩排序  0. 保存并退出");
}

int main(int argc, char **argv) {
    const char *path = argc > 1 ? argv[1] : "students.csv";
    StudentList list;
    char choice[16];
    int loaded;
    student_list_init(&list);
    loaded = students_load(path, &list);
    if (loaded < 0) fprintf(stderr, "无法读取数据文件：%s\n", path);
    else if (loaded > 0) printf("文件不存在，将创建新文件：%s\n", path);
    for (;;) {
        menu();
        read_line("请选择: ", choice, sizeof choice);
        switch (choice[0]) {
            case '1': print_all(&list); break;
            case '2': add_student(&list); break;
            case '3': edit_student(&list); break;
            case '4': delete_student(&list); break;
            case '5': find_student(&list); break;
            case '6': student_list_sort_by_score(&list); puts("排序完成。"); break;
            case '0':
                if (students_save(path, &list) != 0) {
                    fputs("保存失败，数据仍在内存中。\n", stderr);
                    student_list_destroy(&list);
                    return 1;
                }
                student_list_destroy(&list);
                puts("已保存，程序结束。");
                return 0;
            default: puts("无效选项。"); break;
        }
    }
}
```

这里有一个重要的生命周期顺序：先初始化列表，再尝试加载；退出时先保存，确认保存成功后再释放列表。保存失败不能假装成功退出，否则用户会以为数据已经持久化。

## 编译和运行

把四个文件放在同一个目录中，使用 C11 编译：

```text
cc -std=c11 -Wall -Wextra -Wpedantic -O2 main.c student.c storage.c -o student-manager
./student-manager students.csv
```

Windows 下使用 MinGW：

```text
gcc -std=c11 -Wall -Wextra -Wpedantic -O2 main.c student.c storage.c -o student-manager.exe
student-manager.exe students.csv
```

第一次运行时文件不存在属于正常情况，程序会从空列表开始；退出选择 `0` 后会创建 CSV 文件。也可以不传参数，程序默认使用当前目录的 `students.csv`。路径来自命令行时不要用 `system` 拼接命令，文件操作应始终通过 `fopen`、`rename` 等库函数完成。

## 测试清单

至少手动或自动覆盖下列情况：

1. 空文件启动，再添加一条记录；
2. 重复学号被拒绝，修改时也不能制造重复学号；
3. 成绩为 `0`、`100`、负数、超过 `100` 和非数字；
4. 学号、姓名达到长度上限，以及输入超长后的下一次菜单读取；
5. 删除第一条、最后一条和不存在的编号；
6. 相同成绩的多条记录，检查排序结果按学号确定；
7. 文件包含空行、损坏行、重复学号和超长行；
8. 数据文件路径所在目录不可写，确认保存失败会报告错误；
9. 使用 AddressSanitizer 检查内存错误：

```text
cc -std=c11 -Wall -Wextra -g -fsanitize=address,undefined \
   main.c student.c storage.c -o student-manager-asan
```

`student.c` 的列表操作还可以单独测试，不必每次模拟菜单输入。测试应断言返回码、列表大小、记录内容和文件重新加载后的结果，而不只是观察程序有没有崩溃。

## 可扩展方向

当前实现故意把领域限制写清楚，便于学习和验证。继续扩展时可以：

* 把存储接口改成支持完整 RFC 4180 CSV；
* 增加课程数组和总评计算，而不是只保存一个成绩；
* 增加按姓名、成绩区间和分页查询；
* 把菜单层替换成命令行子命令，便于脚本批量导入；
* 为存储模块增加版本号、校验和和备份文件；
* 把 `StudentList` 隐藏在不透明结构体后，进一步减少模块耦合。

完成这些扩展前，应先保持当前版本的接口契约和测试。一个能编译、能恢复、能报告错误的朴素程序，比功能很多但无法判断数据是否丢失的程序更适合作为后续工程的基础。
