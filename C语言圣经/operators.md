---
description: 算术、逻辑、关系和位运算。
icon: code
---

# 运算符与表达式

### 操作数与运算符

操作数是参与运算的数据，可以是常量、变量、函数返回值或表达式。

```c
a + 10
```

在上面的表达式中，`a` 和 `10` 是操作数，`+` 是运算符。

按照操作数的数量，运算符可以分为：

| 类型    | 说明       | 示例                  |
| ----- | -------- | ------------------- |
| 一元运算符 | 只需要一个操作数 | `-x`、`!flag`、`++i`  |
| 二元运算符 | 需要两个操作数  | `a + b`、`x == y`    |
| 三元运算符 | 需要三个操作数  | `condition ? a : b` |

C 语言只有一个三元运算符，即条件运算符 `?:` ，具体优先级见后文优先级表。

### 左值与右值

“左值”和“右值”不能简单理解为等号的左边和右边。

在赋值表达式中，左侧必须是一个可以被写入的对象，右侧通常是一个用于计算或传递的值。

```c
int value = 10;
int array[2] = {1, 2};
​
value = 20;
array[0] = value;
```

变量、数组元素以及指针解引用的结果，通常可以作为可修改左值：

```c
int number = 10;
int *pointer = &number;
​
*pointer = 20;
```

下面的写法是错误的：

```c
// 10 = number;
// (number + 1) = 20;
```

`const` 对象虽然有存储位置，但不能通过当前标识符修改：

```c
const int limit = 10;
​
// limit = 20;  // 错误
```

可以这样理解：

* 作用域决定一个名字在哪里可见；
* 左值通常表示一个具有存储位置的对象；
* 可修改左值可以作为赋值目标；
* 右值通常表示参与运算或传递的值。

```c
int value = 10;
int array[2] = {1, 2};

value = 20;
array[0] = value;
```

`const` 对象有存储位置，但不能通过当前标识符修改：

```c
const int limit = 10;
// limit = 20;  // 错误
```

<table><thead><tr><th width="69.800048828125">优先级</th><th width="92.800048828125">运算符</th><th width="155.2000732421875">名称或含义</th><th width="213.9998779296875">使用形式</th><th width="108">结合方向</th><th width="108.800048828125">说明</th></tr></thead><tbody><tr><td rowspan="4">1</td><td><code>[]</code></td><td>数组下标</td><td><code>数组名[常量表达式]</code></td><td rowspan="4">左到右</td><td rowspan="4">--</td></tr><tr><td><code>()</code></td><td>圆括号、函数调用</td><td><code>(表达式)</code> / <code>函数名(形参表)</code></td></tr><tr><td><code>.</code></td><td>成员选择（对象）</td><td><code>对象.成员名</code></td></tr><tr><td><code>-></code></td><td>成员选择（指针）</td><td><code>对象指针->成员名</code></td></tr><tr><td rowspan="9">2</td><td><code>-</code></td><td>负号运算符</td><td><code>-表达式</code></td><td rowspan="9">右到左</td><td rowspan="7">单目运算符</td></tr><tr><td><code>~</code></td><td>按位取反运算符</td><td><code>~表达式</code></td></tr><tr><td><code>++</code></td><td>自增运算符</td><td><code>++变量名</code> / <code>变量名++</code></td></tr><tr><td><code>--</code></td><td>自减运算符</td><td><code>--变量名</code> / <code>变量名--</code></td></tr><tr><td><code>*</code></td><td>取值运算符</td><td><code>*指针变量</code></td></tr><tr><td><code>&#x26;</code></td><td>取地址运算符</td><td><code>&#x26;变量名</code></td></tr><tr><td><code>!</code></td><td>逻辑非运算符</td><td><code>!表达式</code></td></tr><tr><td><code>(类型)</code></td><td>强制类型转换</td><td><code>(数据类型)表达式</code></td><td rowspan="2">--</td></tr><tr><td><code>sizeof</code></td><td>长度运算符</td><td><code>sizeof(表达式)</code></td></tr><tr><td rowspan="3">3</td><td><code>/</code></td><td>除</td><td><code>表达式 / 表达式</code></td><td rowspan="18">左到右</td><td rowspan="18">双目运算符</td></tr><tr><td><code>*</code></td><td>乘</td><td><code>表达式 * 表达式</code></td></tr><tr><td><code>%</code></td><td>余数（取模）</td><td><code>整型表达式 % 整型表达式</code></td></tr><tr><td rowspan="2">4</td><td><code>+</code></td><td>加</td><td><code>表达式 + 表达式</code></td></tr><tr><td><code>-</code></td><td>减</td><td><code>表达式 - 表达式</code></td></tr><tr><td rowspan="2">5</td><td><code>&#x3C;&#x3C;</code></td><td>左移</td><td><code>变量 &#x3C;&#x3C; 表达式</code></td></tr><tr><td><code>>></code></td><td>右移</td><td><code>变量 >> 表达式</code></td></tr><tr><td rowspan="4">6</td><td><code>></code></td><td>大于</td><td><code>表达式 > 表达式</code></td></tr><tr><td><code>>=</code></td><td>大于等于</td><td><code>表达式 >= 表达式</code></td></tr><tr><td><code>&#x3C;</code></td><td>小于</td><td><code>表达式 &#x3C; 表达式</code></td></tr><tr><td><code>&#x3C;=</code></td><td>小于等于</td><td><code>表达式 &#x3C;= 表达式</code></td></tr><tr><td rowspan="2">7</td><td><code>==</code></td><td>等于</td><td><code>表达式 == 表达式</code></td></tr><tr><td><code>!=</code></td><td>不等于</td><td><code>表达式 != 表达式</code></td></tr><tr><td>8</td><td><code>&#x26;</code></td><td>按位与</td><td><code>表达式 &#x26; 表达式</code></td></tr><tr><td>9</td><td><code>^</code></td><td>按位异或</td><td><code>表达式 ^ 表达式</code></td></tr><tr><td>10</td><td><code>|</code></td><td>按位或</td><td><code>表达式 | 表达式</code></td></tr><tr><td>11</td><td><code>&#x26;&#x26;</code></td><td>逻辑与</td><td><code>表达式 &#x26;&#x26; 表达式</code></td></tr><tr><td>12</td><td><code>||</code></td><td>逻辑或</td><td><code>表达式 || 表达式</code></td></tr><tr><td>13</td><td><code>?:</code></td><td>条件运算符</td><td><code>表达式1 ? 表达式2 : 表达式3</code></td><td rowspan="12">右到左</td><td>三目运算符</td></tr><tr><td rowspan="11">14</td><td><code>=</code></td><td>赋值运算符</td><td><code>变量 = 表达式</code></td><td rowspan="12">--</td></tr><tr><td><code>/=</code></td><td>除后赋值</td><td><code>变量 /= 表达式</code></td></tr><tr><td><code>*=</code></td><td>乘后赋值</td><td><code>变量 *= 表达式</code></td></tr><tr><td><code>%=</code></td><td>取模后赋值</td><td><code>变量 %= 表达式</code></td></tr><tr><td><code>+=</code></td><td>加后赋值</td><td><code>变量 += 表达式</code></td></tr><tr><td><code>-=</code></td><td>减后赋值</td><td><code>变量 -= 表达式</code></td></tr><tr><td><code>&#x3C;&#x3C;=</code></td><td>左移后赋值</td><td><code>变量 &#x3C;&#x3C;= 表达式</code></td></tr><tr><td><code>>>=</code></td><td>右移后赋值</td><td><code>变量 >>= 表达式</code></td></tr><tr><td><code>&#x26;=</code></td><td>按位与后赋值</td><td><code>变量 &#x26;= 表达式</code></td></tr><tr><td><code>^=</code></td><td>按位异或后赋值</td><td><code>变量 ^= 表达式</code></td></tr><tr><td><code>|=</code></td><td>按位或后赋值</td><td><code>变量 |= 表达式</code></td></tr><tr><td>15</td><td><code>,</code></td><td>逗号运算符</td><td><code>表达式, 表达式, ...</code></td><td>左到右</td></tr></tbody></table>

## 算术运算符

常见的算术运算符包括：

```
+  加法
-  减法
*  乘法
/  除法
%  取余
```

### 整数除法

如果两个操作数都是整数，执行的是整数除法，小数部分会被截去。

```c
#include <stdio.h>
​
int main(void)
{
    int a = 10;
    int b = 3;
    double result;
​
    result = a / b;
    printf("%.6f\n", result);  // 3.000000
​
    result = (double)a / b;
    printf("%.6f\n", result);  // 3.333333
​
    result = 1.0 * a / b;
    printf("%.6f\n", result);  // 3.333333
​
    return 0;
}
```

`a / b` 在计算时已经完成了整数除法，之后再把结果赋给 `double`，无法恢复已经被截去的小数部分。

只要除法表达式中至少有一个操作数是浮点类型，运算通常就会按照浮点规则进行。

整数除法和取余运算的除数不能为 `0`。取余运算的两个操作数必须是整数。

```c
printf("%d\n",  5 %  3);  //  2
printf("%d\n",  5 % -3);  //  2
printf("%d\n", -5 %  3);  // -2
printf("%d\n", -5 % -3);  // -2
```

C99 及之后的标准规定，整数除法向零截断，余数的符号与左操作数一致。

### 自增与自减

`++` 和 `--` 是一元运算符，同时会修改操作数。

* 前缀形式先修改，再产生表达式值；
* 后缀形式先产生原值，再修改。

```c
int i = 0;
​
printf("%d\n", ++i);  // 1：先加 1，再取值
printf("%d\n", i);    // 1
​
i = 0;
​
printf("%d\n", i++);  // 0：先取值，再加 1
printf("%d\n", i);    // 1
```

不要在同一个复杂表达式中多次修改同一个变量：

```c
// int value = i++ + ++i;  // 不要这样写
```

最稳妥的做法是把修改拆成多条语句，避免依赖复杂的求值规则。

## 赋值运算符

### 简单赋值运算符

`=`（简单赋值运算符），是一个二元运算符。 优先级很低（倒数第二低，除了逗号运算符，赋值运算符最低），结合性从右到左。

```c
int a, b;b = a = 10;
printf("%d\n", a); // 10
printf("%d\n", b); // 10
```

### 复合赋值运算符

复合赋值把运算和赋值合并起来：

```c
int value = 10;

value += 3;  // value = value + 3
value -= 2;  // value = value - 2
value *= 4;  // value = value * 4
value /= 2;  // value = value / 2
value %= 3;  // value = value % 3
```

位运算也有对应的复合赋值形式：

```c
flags &= mask;
flags |= mask;
flags ^= mask;
value <<= 1;
value >>= 1;
```

## 关系运算符

关系运算符用于比较两个值：

```
<   小于
<=  小于等于
>   大于
>=  大于等于
==  等于
!=  不等于
```

关系表达式的结果是 `int` 类型的 `0` 或 `1`：

```c
int result = 10 > 3;  // result 为 1
```

关系运算符的优先级低于算术运算符，高于赋值运算符：

```c
int result = 2 + 3 < 6;
```

上式等价于：

```c
int result = (2 + 3) < 6;
```

实际编程时，建议使用括号明确表达式的意图。

## 逻辑运算符

逻辑运算符包括：

| 运算符    | 名称  | 说明                        |
| ------ | --- | ------------------------- |
| `!`    | 逻辑非 | 条件为真时结果为假，条件为假时结果为真       |
| `&&`   | 逻辑与 | 两个操作数都为真时结果为真，否则为假        |
| `\|\|` | 逻辑或 | 至少一个操作数为真时结果为真，两个都为假时结果为假 |

在 C 语言中：

* `0` 表示假；
* 非 `0` 表示真；
* 逻辑运算的结果始终是 `0` 或 `1`；
* `&&` 和 `||` 具有短路求值特性。
* `++`、`--`、赋值运算符等会修改对象；其他一元运算符通常只产生计算结果。

```c
int number = 50;

int in_range = number >= 1 && number <= 99;
int out_of_range = number < 1 || number > 99;
int also_in_range = !(number < 1 || number > 99);
```

### 短路求值

逻辑运算符从左到右求值，并可能跳过右侧表达式：

* `left && right`：左侧为假时，右侧不会执行；
* `left || right`：左侧为真时，右侧不会执行。

```c
#include <stdio.h>

int main(void)
{
    int a = 1;
    int b = 1;

    printf("%d\n", ++a || ++b);
    printf("a = %d, b = %d\n", a, b);

    return 0;
}
```

输出：

```
1
a = 2, b = 1
```

短路求值经常用于指针检查：

```c
if (pointer != NULL && *pointer > 0)
{
    /* 只有 pointer 非空时才会解引用 */
}
```

## 条件运算符

条件运算符是 C 语言唯一的三元运算符：

```
条件 ? 条件为真时的表达式 : 条件为假时的表达式
```

示例：

```c
#include <stdio.h>

int main(void)
{
    int number;

    scanf("%d", &number);

    printf("%s\n", number % 2 == 0 ? "偶数" : "奇数");

    return 0;
}
```

条件运算符适合表示简单的二选一结果。嵌套多个条件运算符会降低可读性，此时应改用 `if...else`。

### 位运算符

位运算直接处理整数的二进制位，常用于标志位、权限集合、协议字段和底层设备控制。

| 运算符  | 名称   | 使用形式           | 说明                    |
| ---- | ---- | -------------- | --------------------- |
| `&`  | 按位与  | `表达式1 & 表达式2`  | 两个位都为 `1` 时结果位才为 `1`  |
| `^`  | 按位异或 | `表达式1 ^ 表达式2`  | 两个位不同时结果位为 `1`        |
| `\|` | 按位或  | `表达式1 \| 表达式2` | 只要有一个位为 `1`，结果位就是 `1` |
| `~`  | 按位取反 | `~表达式`         | 将每一位的 `0` 和 `1` 互换    |
| `<<` | 左移   | `表达式1 << 表达式2` | 向左移动指定的位数，右侧通常补 `0`   |
| `>>` | 右移   | `表达式1 >> 表达式2` | 向右移动指定的位数；无符号数左侧补 `0` |

#### 位掩码

可以为每个选项分配一个独立的二进制位，再使用位运算进行设置、清除、测试和翻转：

```c
enum
{
    FLAG_READ  = 1u << 0,
    FLAG_WRITE = 1u << 1,
    FLAG_DEBUG = 1u << 2
};
​
unsigned flags = 0;
​
flags |= FLAG_READ | FLAG_WRITE;   // 设置标志
flags &= ~FLAG_DEBUG;              // 清除标志
​
if ((flags & FLAG_WRITE) != 0)     // 测试标志
{
    puts("writable");
}
​
flags ^= FLAG_READ;                // 翻转标志
```

#### 移位运算

移位量必须是非负值，并且小于左操作数提升后类型的位宽。

* 无符号整数左移时，移出的位丢弃，右侧补 `0`；
* 无符号整数右移时，左侧补 `0`；
* 负数右移的结果由具体实现决定；
* 有符号整数左移溢出可能产生未定义行为；
* `a << b` 只有在类型和范围满足条件时，才可以近似理解为 `a * 2^b`。

因此，不应把移位无条件当作乘法或除法的替代品。

```c
unsigned value = 8;
​
printf("%u\n", value << 2);  // 通常输出 32
printf("%u\n", value >> 1);  // 输出 4
```

### 逗号运算符

逗号运算符 `,` 按从左到右的顺序计算多个表达式，整个逗号表达式的结果是最后一个表达式的值。

```c
int number;
​
number = 100, 200;
printf("%d\n", number);  // 100
​
number = (100, 200);
printf("%d\n", number);  // 200
​
number = (100, 200, 300);
printf("%d\n", number);  // 300
```

第一条语句等价于：

```c
(number = 100), 200;
```

逗号运算符与函数参数列表中的逗号不是一回事。函数参数列表中的逗号只用于分隔参数。

### 运算符优先级

运算符优先级决定表达式如何分组，但不等于所有操作数的实际求值顺序。

涉及自增、自减、函数调用或其他副作用时，最好拆成多条语句。优先级表可以帮助理解表达式，但括号通常是更可靠的说明方式。

## 原稿图示
