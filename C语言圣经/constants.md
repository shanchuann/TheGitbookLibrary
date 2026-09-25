---
description: 字面量、宏常量、枚举和字符串常量。
icon: code
---

# 常量

## 字面常量

字面常量也叫字面量。它没有名字，直接写在源代码中，看到它的写法通常就能判断它表示的值。

例如：

* `10` 表示整数；
* `3.14` 表示浮点数；
* `'a'` 表示字符常量；
* `"Hello"` 表示字符串字面量。

```c
#include <stdio.h>

int main(void)
{
    10;                 // 整数字面量
    3.14;               // 浮点数字面量
    'c';                // 字符字面量
    "Hello world!";     // 字符串字面量

    int sum = 10 + 20;  // 10 和 20 都是字面量
    int value = 10;

    printf("%d\n", sum);
    printf("%d\n", value);

    return 0;
}
```

字面常量通常用于初始化变量、参与表达式计算或作为函数参数：

```c
int count = 10;
double price = 19.9;
char grade = 'A';
​
int total = count + 20;
```

需要注意，在 C 语言中：

* `'a'` 是字符常量，但它的类型实际上是 `int`；
* `"a"` 是字符串字面量，实际类型是包含两个元素的字符数组：`{'a', '\0'}`；
* C 语言没有内置的 `string` 字符串类型，字符串通常使用字符数组表示。

## 宏常量

宏常量使用 `#define` 定义：

```c
#define PI 3.1415926
#define MAX_SIZE 128
```

使用宏时，预处理器会在编译前进行文本替换：

```c
#include <stdio.h>
​
#define AGE 21
​
int main(void)
{
    printf("%d\n", AGE);
​
    int age = AGE;
    printf("%d\n", age);
​
    return 0;
}
```

输出：

```
21
21
```

宏通常使用大写字母命名，以便与普通变量区分。

### 宏的作用范围

宏从定义位置开始生效，只对后面的代码有效：

```c
#include <stdio.h>
​
int main(void)
{
    printf("%d\n", AGE);  // 错误：此处还没有定义 AGE
​
    return 0;
}
​
#define AGE 21
```

正确写法：

```c
#include <stdio.h>
​
#define AGE 21
​
int main(void)
{
    printf("%d\n", AGE);
    return 0;
}
```

宏本质上是预处理阶段的文本替换，不是变量，也不是函数。宏名在预处理完成后通常不会作为独立符号保留。

```c
#define SQUARE(x) ((x) * (x))
```

虽然这个宏看起来像函数，但参数可能被重复求值：

```c
int i = 3;
int value = SQUARE(i++);  // i++ 可能执行两次
```

因此：

* 简单的编译期常量可以使用宏；
* 带参数的计算通常优先使用函数或 `static inline` 函数；
* 宏中的表达式应使用括号，避免运算优先级导致错误。

```c
static inline int square_int(int x)
{
    return x * x;
}
```

如果确实需要取消宏定义，可以使用：

```c
#undef AGE
```

重新定义宏时，建议明确取消旧定义，避免不同头文件中的宏发生冲突。

## `const` 常量

使用 `const` 修饰的对象称为只读对象，也常被称为常变量：

```c
const int max_size = 128;
```

通过这个标识符不能直接修改对象的值：

```c
#include <stdio.h>
​
int main(void)
{
    const int limit = 100;
​
    // limit = 200;  // 错误：不能直接修改 const 对象
​
    printf("%d\n", limit);
    return 0;
}
```

<img src="https://raw.githubusercontent.com/shanchuann/TheGitbookLibrary/main/C%E8%AF%AD%E8%A8%80%E5%9C%A3%E7%BB%8F/.gitbook/assets/book-images/c-learning/c-learning-01.png" alt="ERROR" width="563">

`const` 的含义是“不能通过当前对象直接修改”，并不意味着这块内存绝对无法改变。通过强制类型转换修改 `const` 对象通常会产生未定义行为，因此不应这样做。

{% code overflow="wrap" %}
```c
int value = 100;
const int *ptr = &value;

value = 200;       // 合法
// *ptr = 300;     // 错误：不能通过 ptr 修改 value
```
{% endcode %}

### `const` 与数组长度

在 C 语言中，`const int` 变量通常不是编译期常量表达式：

```c
const int length = 10;
​
// 文件作用域下通常不允许这样定义固定长度数组
// int array[length];
```

在函数内部，C99 及之后的标准允许根据运行时值定义变长数组：

```c
#include <stdio.h>
​
int main(void)
{
    const int length = 10;
    int array[length];  // C99 中可作为变长数组
​
    for (int i = 0; i < length; ++i)
    {
        array[i] = i + 1;
    }
​
    printf("%d\n", array[9]);
    return 0;
}
```

如果需要编译期数组长度，可以使用枚举常量或宏：

```c
enum { ARRAY_SIZE = 10 };
​
int array[ARRAY_SIZE];
```

```c
#define ARRAY_SIZE 10
​
int array[ARRAY_SIZE];
```

### `const` 变量必须初始化吗？

在 C 语言中，`const` 对象可以不在定义时初始化：

```c
const int value;
```

但此时 `value` 的值是不确定的，而且之后不能通过普通赋值为它设置值。因此，实际编程中通常应该在定义时完成初始化：

```c
const int value = 10;
```

## 枚举常量

枚举用于把一组相关的整数常量组织在一起。关键字 `enum` 的本意就是“逐一列举”。

```c
#include <stdio.h>
​
enum color
{
    YELLOW,
    BLACK,
    GREEN,
    ORANGE
};
​
int main(void)
{
    enum color current_color = YELLOW;
​
    printf("%d\n", current_color);
​
    return 0;
}
```

默认情况下，枚举成员从 `0` 开始依次递增：

```
YELLOW  // 0
BLACK   // 1
GREEN   // 2
ORANGE  // 3
```

也可以手动指定枚举值：

```c
enum status
{
    STATUS_OK = 200,
    STATUS_NOT_FOUND = 404,
    STATUS_ERROR = 500
};
```

枚举常量适合表示状态、方向、星期或选项等固定集合：

```c
enum day
{
    MONDAY = 1,
    TUESDAY,
    WEDNESDAY,
    THURSDAY,
    FRIDAY
};
```

枚举变量的类型是枚举类型，枚举成员本身表示整数常量。

## **字符常量和字符串常量**

字符常量使用一对单引号括起来：

```c
char ch = 'a';
```

需要注意，**在 C 语言中，普通字符常量的类型是 `int`，不是 `char`**。

```
'a'
'\n'
'0'
```

这些字符常量的类型都是 `int`。例如：

```c
#include <stdio.h>

int main(void)
{
    printf("%zu\n", sizeof('a'));   // 通常输出 sizeof(int)，例如 4
    printf("%d\n", 'a');            // 通常输出 97
    printf("%c\n", 'a');            // 输出 a

    return 0;
}
```

这是因为字符常量参与表达式运算时，需要使用整数类型表示字符编码。

但下面的变量类型仍然是 `char`：

```c
char ch = 'a';
```

这里发生了从 `int` 到 `char` 的转换：

```
字符常量 'a'：int
变量 ch：char
```

`printf("%c", ...)` 要求传入 `int`，而不是 `char`，因为 `char` 作为函数参数传递时会进行整数提升。

需要特别区分：

| 写法                      | C 语言中的类型                    |
| ----------------------- | --------------------------- |
| `'a'`                   | `int`                       |
| `char ch = 'a'` 中的 `ch` | `char`                      |
| `"a"`                   | `char[2]`，包含 `'a'` 和 `'\0'` |
| `L'a'`                  | `wchar_t`                   |

这点与 C++ 不同：在 C++ 中，普通字符常量 `'a'` 的类型是 `char`。

**字符常量**只能表示一个字符：

```
'a'    // 正确
'7'    // 正确，表示字符 7
'\n'   // 正确，表示换行符
'ab'   // 不应作为普通字符使用
```

字符 `'7'` 和整数 `7` 不是同一个东西：

```
'7'  // 字符 7，ASCII 值通常为 55
7    // 整数 7
```

### ASCII码表（部分，详见附件）

ASCII (American Standard Code for Information Interchange)是美国信息交换标准代码，基于拉丁字母的一套电脑编码系统，主要用于显示现代英语和其他西欧语言。它是最通用的信息交换标准，并等同于国际标准 ISO/IEC 646。ASCII第一次以规范标准的类型发表是在1967年，最后一次更新则是在1986年，到目前为止共定义了128个字符。

其中**0～31及127(共33个)是控制字符或通信专用字符（其余为可显示字符）**，如控制符LF(换行)、CR(回车)、FF(换页)、DEL(删除)、BS(退格)、BEL(响铃)等；又如通信专用字符SOH（文头）、EOT（文尾）、ACK（确认）等；

<table><thead><tr><th width="112">Bin(二进制)</th><th width="100.2000732421875">Oct(八进制)</th><th width="99.199951171875">Dec(十进制)</th><th width="112">Hex(十六进制)</th><th width="156.5999755859375">缩写/字符</th><th>解释</th></tr></thead><tbody><tr><td>0000 0000</td><td>00</td><td><strong>0</strong></td><td>0x00</td><td>NUL(null)</td><td>空字符</td></tr><tr><td>0000 0001</td><td>01</td><td>1</td><td>0x01</td><td>SOH(start of headline)</td><td>标题开始</td></tr><tr><td>0000 0011</td><td>03</td><td>3</td><td>0x03</td><td>ETX (end of text)</td><td>正文结束</td></tr><tr><td>……</td><td>……</td><td>……</td><td>……</td><td>……</td><td>……</td></tr><tr><td>0100 0000</td><td>0100</td><td>64</td><td>0x40</td><td>@</td><td>电子邮件符号</td></tr><tr><td>0100 0001</td><td>0101</td><td><strong>65</strong></td><td>0x41</td><td>A</td><td>大写字母A</td></tr><tr><td>0100 0100</td><td>0104</td><td>68</td><td>0x44</td><td>D</td><td>大写字母D</td></tr><tr><td>……</td><td>……</td><td>……</td><td>……</td><td>……</td><td>……</td></tr><tr><td>0101 1011</td><td>0133</td><td>91</td><td>0x5B</td><td>[</td><td>开方括号</td></tr><tr><td>0101 1100</td><td>0134</td><td>92</td><td>0x5C</td><td>\</td><td>反斜杠</td></tr><tr><td>0110 0001</td><td>0141</td><td><strong>97</strong></td><td>0x61</td><td>a</td><td>小写字母a</td></tr><tr><td>……</td><td>……</td><td>…………</td><td>……</td><td>……</td><td>……</td></tr></tbody></table>

### 转义字符

字母前加" \ "来表示常见的那些不能显示的ASCII字符，如 `\0`，`\t`，`\n`等，因为后面的字符都不是它本来的ASCII字符意思,称为转义字符

<table><thead><tr><th width="160">转义字符</th><th width="346.2000732421875">意义</th><th>ASCII码值（十进制）</th></tr></thead><tbody><tr><td><code>\a</code></td><td>响铃(BEL)</td><td>007</td></tr><tr><td><code>\b</code></td><td>退格(BS) ，将当前位置移到前一列</td><td>008</td></tr><tr><td><code>\f</code></td><td>换页(FF)，将当前位置移到下页开头</td><td>012</td></tr><tr><td><code>\n</code></td><td>换行(LF) ，将当前位置移到下一行开头</td><td>010</td></tr><tr><td><code>\r</code></td><td>回车(CR) ，将当前位置移到本行开头</td><td>013</td></tr><tr><td><code>\t</code></td><td>水平制表(HT) （跳到下一个TAB位置）</td><td>009</td></tr><tr><td><code>\v</code></td><td>垂直制表(VT)</td><td>011</td></tr><tr><td><code>\\</code></td><td>代表一个反斜线字符" \ "</td><td>092</td></tr><tr><td><code>\'</code></td><td>代表一个单引号（撇号）字符</td><td>039</td></tr><tr><td><code>\"</code></td><td>代表一个双引号字符</td><td>034</td></tr><tr><td><code>?</code></td><td>代表一个问号</td><td>063</td></tr><tr><td><code>\0</code></td><td>空字符(NUL)</td><td>000</td></tr><tr><td><code>\ddd</code></td><td>1到3位八进制数所代表的任意字符</td><td>三位八进制</td></tr><tr><td><code>\xhh</code></td><td>十六进制所代表的任意字符</td><td>十六进制</td></tr></tbody></table>

```c
#include <stdio.h>
​
int main(void)
{
    printf("第一行\n第二行\n");
    printf("姓名\t年龄\n");
    printf("他说：\"Hello\"\n");
​
    return 0;
}
```

`\0` 是值为 0 的空字符，常用来标记字符串的结尾。它和 `NULL` 不是同一个概念：

* `\0` 是字符；
* `NULL` 是空指针常量。

### 字符串字面量

字符串使用一对双引号括起来：

```
"Hello"
```

在 C 语言中，字符串实际上是以 `'\0'` 结尾的字符数组：

```c
char str[6] = "hello";
```

内存中的内容为：

```
'h' 'e' 'l' 'l' 'o' '\0'
```

也可以写成：

```c
char str[] = {'h', 'e', 'l', 'l', 'o', '\0'};
```

这两种写法得到的结果相同。

#### `strlen` 与 `sizeof`

`strlen` 用于计算字符串中有效字符的数量，不包括结尾的 `'\0'`。

`sizeof` 用于计算对象占用的字节数，在数组仍然是数组的上下文中，会把结尾的 `'\0'` 也计算在内。

```c
#include <stdio.h>
#include <string.h>
​
int main(void)
{
    char str[] = "hello";
​
    size_t length = strlen(str);
    size_t size = sizeof(str);
​
    printf("字符串长度：%zu\n", length);
    printf("数组大小：%zu\n", size);
​
    return 0;
}
```

输出：

```
字符串长度：5
数组大小：6
```

这里：

* `strlen(str)` 的结果是 `5`；
* `sizeof(str)` 的结果是 `6`，因为还包括字符串结尾的 `'\0'`。

字符串必须以 `'\0'` 结尾，否则许多字符串函数无法判断字符串在哪里结束。少了这个字符，程序可能会继续读取后面的内存，结果就不再是“字符串”，而是一场小型的内存探险。

## 原稿图示
