---
description: 函数、参数传递、模块化、递归和函数接口。
icon: code
---

# 函数

## 为什么需要函数

在结构化程序设计中，函数是划分任务的基本单位。一个复杂问题可以拆成若干个职责明确的小问题，每个小问题由一个函数负责。这样能减少重复代码，也让测试和排错有了清晰边界。

C 语言不允许在一个函数内部再定义另一个函数。函数可以调用其他函数，也可以调用自己，但每个函数定义都必须位于文件作用域。

## 函数的基本形式

函数定义的一般形式如下：

~~~c
返回类型 函数名(形参列表)
{
    函数体
}
~~~

一次完整的函数使用通常包含三个步骤：

1. 函数声明：告诉编译器函数的接口；
2. 函数调用：传入实参并执行函数体；
3. 函数定义：提供函数的具体实现。

~~~mermaid
flowchart LR
    A[函数声明] --> B[函数调用]
    B --> C[计算实参]
    C --> D[传给形参]
    D --> E[执行函数体]
    E --> F[return 返回结果]
    F --> G[调用者继续执行]
~~~

## 函数声明、调用与定义

函数声明也叫函数原型，通常写在函数调用之前：

~~~c
int add(int a, int b);
~~~

形参名称在函数声明中可以省略：

~~~c
int add(int, int);
~~~

完整示例：

~~~c
#include <stdio.h>

int add(int a, int b);

int main(void)
{
    int result = add(2, 3);
    printf("result = %d\n", result);
    return 0;
}

int add(int a, int b)
{
    return a + b;
}
~~~

函数声明和定义的返回类型、函数名以及参数类型必须保持一致。参数名称可以不同，但参数类型和顺序不能随意改变。

## 使用函数计算三角形面积

下面的函数根据三条边计算三角形面积。半周长为 p = (a + b + c) / 2，面积为 sqrt(p * (p - a) * (p - b) * (p - c))。

~~~c
#include <math.h>
#include <stdio.h>

double triangle_area(double a, double b, double c);

int main(void)
{
    double a;
    double b;
    double c;

    printf("请输入三条边长：");

    if (scanf("%lf %lf %lf", &a, &b, &c) != 3)
    {
        printf("输入格式错误。\n");
        return 1;
    }

    double area = triangle_area(a, b, c);

    if (area < 0.0)
    {
        printf("这三条边不能构成三角形。\n");
    }
    else
    {
        printf("面积为：%.2f\n", area);
    }

    return 0;
}

double triangle_area(double a, double b, double c)
{
    if (a <= 0.0 || b <= 0.0 || c <= 0.0)
    {
        return -1.0;
    }

    if (a + b <= c || a + c <= b || b + c <= a)
    {
        return -1.0;
    }

    double p = (a + b + c) / 2.0;
    return sqrt(p * (p - a) * (p - b) * (p - c));
}
~~~

使用 GCC 编译时需要链接数学库：

~~~bash
gcc -std=c17 -Wall -Wextra -Wpedantic triangle_area.c -o triangle_area -lm
~~~

## 参数与值传递

形参是函数定义或声明中的参数变量，例如 int a 和 int b。实参是调用函数时传入的具体值，例如 add(2, 3) 中的 2 和 3。

调用函数时，程序会先计算实参表达式的值，再将结果传给对应的形参。参数通常按位置一一对应，类型需要兼容或能够进行合法转换。

C 语言的普通参数传递是值传递。函数接收的是实参值的副本，修改形参不会改变调用者中的原变量。

~~~c
#include <stdio.h>

void set_value(int value)
{
    value = 100;
}

int main(void)
{
    int number = 10;
    set_value(number);
    printf("number = %d\n", number);
    return 0;
}
~~~

输出仍然是 number = 10。值传递过程如下：

~~~mermaid
flowchart LR
    A[调用者中的 number = 10] -->|复制数值| B[函数形参 value = 10]
    B --> C[value = 100]
    C --> D[形参副本被销毁]
    D --> E[调用者中的 number 仍为 10]
~~~

图中的两个 10 属于两个不同的对象。函数修改的是形参副本，调用者中的变量不会跟着变化。

## 通过指针修改实参

C 语言没有单独的引用传递语法，但可以把变量的地址传给函数。函数通过指针访问原变量，从而修改调用者中的数据。

~~~c
#include <stdio.h>

void swap(int *left, int *right)
{
    int temporary = *left;
    *left = *right;
    *right = temporary;
}

int main(void)
{
    int x = 10;
    int y = 20;

    printf("交换前：x = %d, y = %d\n", x, y);
    swap(&x, &y);
    printf("交换后：x = %d, y = %d\n", x, y);
    return 0;
}
~~~

这里传递的仍然是值，只不过这个值是地址：

~~~mermaid
flowchart LR
    A[x 的地址] -->|传递地址副本| B[left]
    B --> C[*left 指向 x]
    C --> D[修改 x 的内容]
    E[y 的地址] -->|传递地址副本| F[right]
    F --> G[*right 指向 y]
    G --> H[修改 y 的内容]
~~~

指针本身仍然是按值传递的。函数拿到的是地址副本，但这个地址副本仍然指向调用者的原对象。

## 返回值与 return

return 有两个作用：结束当前函数，并把一个值返回给调用者。

~~~c
int square(int value)
{
    return value * value;
}
~~~

返回类型为 void 的函数可以不返回值，也可以使用不带表达式的 return 提前结束：

~~~c
void print_positive(int value)
{
    if (value <= 0)
    {
        return;
    }

    printf("%d\n", value);
}
~~~

在 main 函数中，return 会结束整个程序并把返回值交给操作系统。exit 定义在 stdlib.h 中，无论在哪个函数中调用都会结束程序：

~~~c
#include <stdlib.h>

exit(EXIT_SUCCESS);  /* 正常结束 */
exit(EXIT_FAILURE);  /* 异常结束 */
~~~

## 函数与模块化设计

好的函数通常具备以下特点：

- 单一职责：一个函数尽量只负责一件明确的事情；
- 参数检查：在入口处检查指针、范围和除数等条件；
- 接口清晰：函数名、参数类型和返回值表达基本用途；
- 高内聚：同一模块中的函数围绕同一个功能组织；
- 低耦合：模块之间通过清晰接口交互，不直接访问对方的内部数据。

~~~c
int divide(int dividend, int divisor, int *result)
{
    if (result == NULL || divisor == 0)
    {
        return 0;
    }

    *result = dividend / divisor;
    return 1;
}
~~~

## 二维数组作为函数参数

一维数组作为函数参数时，数组名通常转换为指向首元素的指针。二维数组则会转换为指向数组的指针。

~~~c
#include <stdio.h>

void print_matrix(int matrix[][4], int rows)
{
    for (int i = 0; i < rows; ++i)
    {
        for (int j = 0; j < 4; ++j)
        {
            printf("%d ", matrix[i][j]);
        }
        printf("\n");
    }
}

int main(void)
{
    int matrix[3][4] = {
        {1, 2, 3, 4},
        {5, 6, 7, 8},
        {9, 10, 11, 12}
    };

    print_matrix(matrix, 3);
    return 0;
}
~~~

int matrix[][4] 与 int (*matrix)[4] 在函数参数中含义相同。编译器必须知道每一行有多少列，才能计算 matrix[i][j] 的地址。

## 递归函数

递归函数会在函数体内直接或间接调用自己。设计递归函数时，必须回答两个问题：

1. 什么条件下停止递归？
2. 每次调用是否都更接近停止条件？

阶乘的递归实现如下：

~~~c
unsigned long long factorial(unsigned int n)
{
    if (n <= 1)
    {
        return 1;
    }

    return n * factorial(n - 1);
}
~~~

调用 factorial(4) 时，调用关系如下：

~~~mermaid
flowchart TD
    A[factorial 4] --> B[factorial 3]
    B --> C[factorial 2]
    C --> D[factorial 1]
    D --> E[返回 1]
    E --> F[返回 2]
    F --> G[返回 6]
    G --> H[返回 24]
~~~

每次递归调用都会产生新的函数栈帧。递归层数过深可能导致栈空间耗尽，因此简单计数通常使用循环更节省空间。

## 本节示例的运行环境与截图

本节示例均在 WSL2 中运行，终端工作目录为：

~~~text
/home/shanchuan/CStudy
~~~

例如：

~~~bash
cd /home/shanchuan/CStudy
gcc -std=c17 -Wall -Wextra -Wpedantic functions_demo.c -o functions_demo
./functions_demo
~~~

下面的截图展示了该目录中的函数示例编译命令和运行输出。图片文件已放入 GitHub 仓库，因此 GitBook 同步后可以直接显示。

![函数示例在 WSL2 中运行](https://raw.githubusercontent.com/shanchuann/TheGitbookLibrary/main/C%E8%AF%AD%E8%A8%80%E5%9C%A3%E7%BB%8F/.gitbook/assets/book-images/c-learning/functions-demo.png)
