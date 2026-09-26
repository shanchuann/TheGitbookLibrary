---
description: 标准输入输出与格式化。
icon: cloud-check
---

# C语言输入输出

## `printf`：格式化输出

`printf` 是 C 语言标准库提供的格式化输出函数。函数名末尾的 `f` 是 `format` 的缩写，表示可以按照指定格式输出数据。

使用 `printf` 前，需要引入头文件：

```c
#include <stdio.h>
```

基本语法：

```c
printf("格式控制字符串", 输出参数);
```

例如：

```c
#include <stdio.h>

int main(void)
{
    int a = 10;
    float b = 1.5f;
    char ch = 'a';

    printf("浮点型：%f\n", b);
    printf("字符型：%c\n", ch);
    printf("十进制：%d\n", a);
    printf("八进制：%o\n", a);
    printf("十六进制：%x\n", a);

    return 0;
}
```

输出结果：

```
浮点型：1.500000
字符型：a
十进制：10
八进制：12
十六进制：a
```

### 常用格式说明符

_完整版见附件_

| 格式说明符 | 作用         | 示例         |
| ----- | ---------- | ---------- |
| `%d`  | 输出有符号十进制整数 | `-10`      |
| `%u`  | 输出无符号十进制整数 | `10`       |
| `%o`  | 输出八进制整数    | `12`       |
| `%x`  | 输出十六进制整数   | `a`        |
| `%c`  | 输出单个字符     | `A`        |
| `%s`  | 输出字符串      | `Hello`    |
| `%f`  | 输出浮点数      | `3.140000` |
| `%%`  | 输出百分号      | `%`        |

### 控制小数位数

可以在 `%f` 中指定小数位数：

```c
#include <stdio.h>

int main(void)
{
    double pi = 3.1415926535;

    printf("%f\n", pi);     // 3.141593
    printf("%.2f\n", pi);   // 3.14
    printf("%.4f\n", pi);   // 3.1416

    return 0;
}
```

其中，`%.2f` 表示保留两位小数。

## `scanf`：格式化输入

`scanf` 是 C 语言标准库提供的格式化输入函数，通常用于从标准输入流中读取键盘输入。

使用 `scanf` 前，需要引入：

```c
#include <stdio.h>
```

基本语法：

```c
scanf("格式控制字符串", 地址参数);
```

例如：

```c
#include <stdio.h>

int main(void)
{
    int age;

    printf("请输入年龄：");

    if (scanf("%d", &age) == 1)
    {
        printf("你的年龄是：%d\n", age);
    }
    else
    {
        printf("输入格式错误。\n");
    }

    return 0;
}
```

### 为什么要使用 `&`？

`scanf` 不仅需要知道变量的类型，还需要知道变量在内存中的地址，这样才能把输入的数据写入变量。

* `age` 表示变量当前保存的值；
* `&age` 表示变量 `age` 的地址。

```c
scanf("%d", &age);
```

这条语句可以理解为：

> 从输入中读取一个整数，并把它写入 `age` 所在的内存位置。

对于普通变量，通常需要使用地址运算符 `&`：

```c
int age;
float height;
char ch;

scanf("%d", &age);
scanf("%f", &height);
scanf(" %c", &ch);
```

读取字符串数组时，数组名本身通常已经表示首元素的地址，因此不需要再写 `&`：

```c
char name[32];

scanf("%31s", name);
```

这里的 `31` 用于限制最多读取的字符数，并且为字符串结尾的 `'\0'` 留出空间，避免写入越界。

### 检查 `scanf` 的返回值

`scanf` 的返回值表示成功转换并赋值的参数个数。

```c
int value;

if (scanf("%d", &value) == 1)
{
    printf("读取成功：%d\n", value);
}
else
{
    printf("请输入一个整数。\n");
}
```

不要假设用户一定会按照提示输入。用户可能输入字母、空行，甚至输入一段完全出乎意料的内容。程序应检查 `scanf` 的返回值，再决定后续操作。

### 输入格式必须匹配

输入数据的格式必须与格式说明符匹配：

```c
int age;
float score;

scanf("%d", &age);    // 例如输入：20
scanf("%f", &score);  // 例如输入：95.5
```

如果使用 `%d` 读取浮点数，或者使用 `%f` 读取整数，程序的行为可能不符合预期。

`scanf` 会从标准输入流中读取字符，并按照格式字符串进行解析。终端显示用户输入的内容，通常是终端本身的回显行为；`scanf` 负责读取和转换数据，并不负责把输入内容重新显示到屏幕上。

## 使用海伦公式计算三角形面积

已知三角形三边长度 `a`、`b`、`c`，可以使用海伦公式计算三角形面积。

半周长为：

$$
p = \frac{a+b+c}{2}
$$

面积为：

$$
S = \sqrt{p(p-a)(p-b)(p-c)}
$$

在计算面积之前，必须先判断三条边能否构成三角形：

```
a + b > c
a + c > b
b + c > a
```

完整程序如下：

{% code title="triangle_area.c" %}
```c
#include <math.h>
#include <stdio.h>

int main(void)
{
    double a;
    double b;
    double c;
    double half_perimeter;
    double area;

    printf("请输入三角形的三条边长：");

    if (scanf("%lf %lf %lf", &a, &b, &c) != 3)
    {
        printf("输入格式错误，请输入三个数字。\n");
        return 1;
    }

    if (a <= 0 || b <= 0 || c <= 0)
    {
        printf("边长必须大于 0。\n");
        return 1;
    }

    if (a + b <= c || a + c <= b || b + c <= a)
    {
        printf("这三条边不能构成三角形。\n");
        return 0;
    }

    half_perimeter = (a + b + c) / 2.0;

    area = sqrt(
        half_perimeter
        * (half_perimeter - a)
        * (half_perimeter - b)
        * (half_perimeter - c)
    );

    printf("三角形的面积为：%.2f\n", area);

    return 0;
}
```
{% endcode %}

运行示例：

```
请输入三角形的三条边长：3 4 5
三角形的面积为：6.00
```

使用 GCC 编译时，需要链接数学库：

```bash
gcc -std=c17 -Wall -Wextra -Wpedantic triangle_area.c -o triangle_area -lm
```

运行程序：

```bash
./triangle_area
```

这里使用 `double` 而不是 `float`，是因为面积计算包含多次浮点运算，`double` 通常能够提供更高的精度。在简单示例中，`float` 也可以完成计算；但在连续计算中，多保留一些有效数字通常更稳妥。

## 输入校验与错误处理

`scanf` 的返回值表示成功转换的项目数，不应忽略：

```c
int age;
if (scanf("%d", &age) != 1) {
    fprintf(stderr, "invalid integer\n");
    return 1;
}
```

`scanf` 适合格式固定、输入边界已经明确的场景；交互式程序通常先用 `fgets` 读入一整行，再用 `strtol`、`strtod` 解析。这样才能区分空输入、非法字符和数值溢出，也能处理数字后多输入的内容。**读取和解析分开，**&#x5148;拿到一整行，再决定它是什么意思。

### `scanf` 的适用面

适合：格式严格的数据（日期 `2024-01-02`、判题输入、配置文件），格式由生成方保证的场合。

不适合交互式输入，原因：

* **返回值信息量不够**：只返回成功赋值的项数，无法区分"读到 0"、"读到字母"、"空行"、"EOF"。
* **溢出是未定义行为**：`%d` 转换结果超出 `int` 范围时行为未定义且不报错；`strtol` 会置 `errno = ERANGE`。
* **输入残留导致死循环**：`while (scanf("%d", &n) != 1)` 遇到字母后，垃圾一直留在缓冲区，无限循环。
* **`%s` 没有宽度限制**，会缓冲区溢出（要写 `%31s`）。
* **格式串末尾的 `\n` 会"卡住"**：它表示"跳过任意空白"，会一直等到下一个非空白字符。
* **`%c` 会读到上一行留下的换行符**（用 `" %c"`，或先 `fgets`）。

`fgets` 读整行（带容量，安全）、去掉换行、解析。用一个枚举把失败原因传出去：

```c
typedef enum {
    READ_OK = 0,
    READ_EOF,       /* 输入结束 */
    READ_EMPTY,     /* 空行 */
    READ_INVALID,   /* 非法字符，或数字后有多余内容 */
    READ_RANGE,     /* 数值超出范围 */
    READ_TOOLONG,   /* 行太长 */
    READ_IO         /* 读取错误 */
} ReadStatus;
```

```c
/* 读一整行并去掉换行符 */
ReadStatus read_line(char *buf, size_t cap, size_t *out_len)
{
    if (fgets(buf, (int)cap, stdin) == NULL) {
        return ferror(stdin) ? READ_IO : READ_EOF;
    }

    size_t n = strlen(buf);
    if (n > 0 && buf[n - 1] == '\n') {
        buf[--n] = '\0';
        if (out_len) *out_len = n;
        return READ_OK;
    }

    /* 没有换行符：要么刚好读满，要么行太长被截断 */
    int ch = getchar();
    if (ch == '\n' || ch == EOF) {
        if (ch == EOF && ferror(stdin)) return READ_IO;
        if (out_len) *out_len = n;
        return READ_OK;                 /* 最后一行没有换行符，也算完整 */
    }
    while (ch != '\n' && ch != EOF) ch = getchar();   /* 丢弃本行剩余部分 */
    if (out_len) *out_len = n;
    return READ_TOOLONG;
}
```

```c
ReadStatus read_int(int *out)
{
    char line[64];
    size_t len = 0;

    ReadStatus st = read_line(line, sizeof line, &len);
    if (st != READ_OK) return st;
    if (len == 0) return READ_EMPTY;

    errno = 0;                                   /* errno 不会自动清零 */
    char *end = NULL;
    long value = strtol(line, &end, 10);

    if (end == line) return READ_INVALID;        /* 一个字符都没解析出来 */
    while (*end == ' ' || *end == '\t') ++end;
    if (*end != '\0') return READ_INVALID;       /* 数字后面还有内容 */

    if (errno == ERANGE || value < INT_MIN || value > INT_MAX) return READ_RANGE;

    *out = (int)value;
    return READ_OK;
}
```

浮点数同理，`strtol` 换成 `strtod`（`errno == ERANGE` 覆盖上溢和下溢）。

### 三种错误要分开

| 情况   | 例子                 | 反应               |
| ---- | ------------------ | ---------------- |
| 空输入  | 回车、Ctrl+D / Ctrl+Z | 结束，或提示"不能为空"     |
| 非法字符 | `abc`、`12abc`      | 提示"请输入数字"        |
| 数值溢出 | `999999999999`     | 提示"超出范围"，不能当正常值用 |
| 业务越界 | 年龄 `-5`            | 提示"年龄不能为负"       |

`scanf` 把这几种全塌缩成"返回值不是 1"。另外注意：类型范围（`INT_MIN`\~`INT_MAX`）和业务范围是两回事，后者要自己校验。

#### `strtol` / `strtod` 要点

* `errno` 必须先清零，它只在出错时被**设置**。
* `endptr == line` 表示没解析出任何字符。
* 跳过尾部空白后必须 `*endptr == '\0'`，否则 `12abc` 会被当成 12。
* `strtol` 返回 `long`，放进 `int` 前要再比一次 `INT_MIN` / `INT_MAX`。
* `base = 0` 才会自动识别 `0x` / `0` 前缀。
* `strtod` 还接受 `inf`、`nan`、`0x1p3`，不想要得自己拦。
* 别用 `atoi`：`atoi("0")` 和 `atoi("abc")` 都返回 0，溢出还是未定义行为。

### `scanf` 仍可用的场合

格式严格时它很合适，但要检查返回值、`%s` 写宽度、别在格式串末尾写 `\n`。

折中方案：`fgets` 读一行 + `sscanf` 解析，兼顾安全和便利（但溢出仍是未定义行为）。
