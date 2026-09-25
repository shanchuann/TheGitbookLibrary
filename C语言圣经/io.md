---
description: 标准输入输出与格式化。
icon: code
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

这里的 `31` 用于限制最多读取的字符数，为字符串结尾的 `'\0'` 留出空间，避免写入越界。

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

面向整行文本的交互通常更适合使用 `fgets`，再用 `strtol` 解析。这样可以控制最大读入长度，也能保留并检查多余字符。任何来自用户、文件或网络的数据都应视为不可信输入，不能只依赖格式字符串“碰巧匹配”。
