---
description: 回调、表驱动和泛型接口。
icon: code
---

# 函数指针

> **学习路径**：普通指针保存对象地址，函数指针保存可调用代码的入口。它把“调用哪个函数”变成运行时数据，适合回调和表驱动；下一章将把指针与结构化数据结合起来。

## 什么是函数指针

在程序中定义了一个函数，当编译链接成功，运行程序时系统就会为这个函数代码分配一段存储空间，这段存储空间的首地址称为这个函数的地址。而且函数名表示的就是这个地址。既然是地址，我们就可以定义一个指针变量来存放，这个指针变量就叫作函数指针变量，简称函数指针。

函数指针的定义方式为：`函数返回值类型 (* 指针变量名)(函数参数列表);`

“函数返回值类型” 表示该指针变量可以指向具有什么返回值类型的函数；“函数参数列表” 表示该指针变量可以指向具有什么参数列表的函数。这个参数列表中只需要写函数的参数类型即可。

```c
int* funa(int a, int b);
int (*funb)(int, int);
```

这两个有什么区别呢？

`int* funa(int a, int b);`是函数的声明，`int (*funb)(int, int);`则是表示这是一个指向形参有`(int, int)`构成的函数的指针，与`int (*br)[4]`相似，`(*funb)`表示该标识符为指针类型。

函数由返回类型和形参列表构成，倘若函数具有相同的返回类型和参数列表，则说这两个函数类型一样。

我们在函数返回类型、形参列表后跟分号(`;`)表示这是函数的声明。后跟花括号(`{}`)表明这是函数的定义。当我们给出函数名后跟圆括号(`()`)时则说明这是函数的调用。

```c
int add(int a, int b); // 函数的声明
int sub(int a, int b);
int inc(int a);

int main() {
	int a = 10, b = 20,c = 0;
	c = add(a, b); // 函数的调用
}

int add(int a, int b) { // 函数的定义
	return a + b;
}
int sub(int a, int b) {
	return a - b;
}
int inc(int a) {
	return a + 1;
}
```

在C语言的表达中，add和\&add表达的意思完全相同，表示函数的地址。

![函数的地址](https://raw.githubusercontent.com/shanchuann/TheGitbookLibrary/main/C%E8%AF%AD%E8%A8%80%E5%9C%A3%E7%BB%8F/.gitbook/assets/book-images/typora/image-20260225111430698.png)

我们都知道在C语言程序编译链接的可执行文件由代码和数据两部分构成，数据部分保留的是全局变量的数据。当函数运行后将分配四个区域：代码区、数据区、堆区和栈区。在代码运行中，主函数在栈区开辟的栈帧用于存放局部变量，funptr分别指向两个函数。

```c
int main() {
	int a = 10, b = 20,c = 0;
	int (*funptr)(int, int) = add;
	funptr = sub;
	return 0;
}
```

> 真实情况是本书将会有一个函数表用于记录函数的地址，指针将指向函数表中的地址，因此即使add和\&add的地址可能不相同，但依旧指向同一个地址。

## 函数指针的使用

那么我们又该怎样使用函数指针呢？

无论是`z = (*funptr)(x,y);`还是`z = funptr(x,y);`都是可以的，但`(*funptr)(x,y)`可以直观的看出这是一个指针，更为推荐。函数指针和普通指针的功能类似，操作的都是指向的函数或数据。

```c
int main() {
	int a = 10, b = 20,c = 0;
	c = add(a, b);
	int (*funptr)(int, int) = add; // 存放有两个int参数和一个int返回值的函数地址
	printf("add: %d\n", funptr(a, b)); // 通过函数指针调用函数
	funptr = sub; // 将函数指针指向sub函数
	printf("sub: %d\n", funptr(a, b)); // 通过函数指针调用函数
	//funptr = inc;
	//funptr(a); // 错误：inc函数需要一个参数，而funptr期望两个参数
	int (*incptr)(int) = inc; // 存放有一个int参数和一个int返回值的函数地址
	printf("inc: %d\n", incptr(a)); // 通过函数指针调用函数

	return 0;
}
```

需要注意的是指向的函数必须是同一类型的。

```
1>D:\Code\C_Code\C\FunctionPointers.c(17,9): warning C4113: “int (__cdecl *)(int)”和“int (__cdecl *)(int,int)”的参数列表不同
1>D:\Code\C_Code\C\FunctionPointers.c(18,2): error C2198: “int (__cdecl *__cdecl funptr)(int,int)”: 用于调用的参数太少
```

## 回调函数

可以将函数指针作为函数的形参传入，通过其指向的函数地址实现相应的功能。

```c
int add(int a, int b) {
	return a + b;
}
int max(int a, int b) {
	return (a > b) ? a : b;
}
void callFunction(int (*func)(int, int), int x, int y) {
	int result = func(x, y);
	printf("Result: %d\n", result);
}
```

```c
int main() {
	int a = 10, b = 20;
	callFunction(add, a, b); // 输出: Result: 30
	callFunction(max, a, b); // 输出: Result: 20
	return 0;
}
```

当我们传入不同的函数名时就能实现不同功能，如上述代码所示。我们还可以使用`typedef int (*PFUN)(int,int);`代表 “接收两个 int、返回 int 的函数指针类型”来更加直观的使用函数指针。**typedef 是对一种数据类型的 “重命名”，而非简单的文本替换**。宏替换是预处理阶段的机械文本替换，不涉及类型检查；而 `typedef int (*PFUN)(int,int);` 是在编译阶段为 “指向接收两个 int 型参数、返回 `int` 型值的函数的指针” 这一特定数据类型，创建了一个简洁的别名 `PFUN`。

```c
void callFunction(PFUN func, int x, int y) {
	int result = func(x, y);
	printf("Result: %d\n", result);
}
```

再举一个例子，当我们进行数组的冒泡排序时，我们需要控制其为递增或是递减，除了开发功能大致相同的两个函数外，最佳解决方案应为对其额外的传入一个函数指针指向两个不同的比较函数，幸运的是C 标准库的 `qsort()`（C++ 的 `sort()`）正是基于这个思路设计的。

> `qsort` 是 C 标准库中提供的通用排序函数，在 `<stdlib.h>` 中声明。C 标准规定了接口和结果，但没有规定内部必须使用哪一种排序算法；具体实现可能使用快速排序、堆排序或其他策略，不能依赖某个特定算法。

```c
void bubbleSort(int arr[], int n, bool (*compare)(int, int)) {
	for (int i = 0; i < n - 1; i++) {
		for (int j = 0; j < n - i - 1; j++) {
			if (compare(arr[j], arr[j + 1])) {
				// Swap arr[j] and arr[j+1]
				int temp = arr[j];
				arr[j] = arr[j + 1];
				arr[j + 1] = temp;
			}
		}
	}
}
bool biggerThan(int a, int b) {
	return a > b;
}
bool smallerThan(int a, int b) {
	return a < b;
}
```

```c
int main() {
	int arr[5] = { 10, 21, 23, 54, 59 };
	int n = sizeof(arr) / sizeof(arr[0]);

	bubbleSort(arr, n, biggerThan); // 升序排序
	printf("升序排序：");
	for (int i = 0; i < n; i++) {
		printf("%d ", arr[i]);
	}
	printf("\n");
	bubbleSort(arr, n, smallerThan); // 降序排序
	printf("降序排序：");
	for (int i = 0; i < n; i++) {
		printf("%d ", arr[i]);
	}
	return 0;
}
```

最终输出该函数的升序、降序排序后的结果。

不过需要注意的是C语言并不能直接使用`bool`类型，需要引入`#include <stdbool.h>`头文件，这也是`qsort()`函数使用`int`作为返回类型的原因。

```c
// qsort()中的比较函数
int cmpfunc (const void * a, const void * b)
{
   return ( *(int*)a - *(int*)b );
}
```

## 表驱动

表驱动法本质是**用数据代替逻辑判断**，将分散的条件分支、映射关系、功能逻辑封装到一个结构化的 “表” 中，程序通过查询表来获取结果或执行操作。

```c
int add(int a, int b) { return a + b; }
int sub(int a, int b) { return a - b; }
int mul(int a, int b) { return a * b; }
int div(int a, int b) {
	if (b != 0) return a / b;
	else {
		printf("Error: Division by zero!\n");
		return 0; // Return 0 or handle as needed
	}
}
```

我们想在主函数中通过传入不同数字来调用不同函数，该怎么实现？

可以定义一个函数指针数组用于存放四个函数的地址：

```c
int (*pfun[4])(int, int) = { add, sub, mul, div }; // 定义一个函数指针数组，存放四个函数的地址

typedef int (*PFUN)(int,int);
PFUN arr[4] = { add, sub, mul, div }; // 更为直观
```

这样只需要在主函数中传入索引，即可将函数指针指向对应的函数。

![函数指针指向对应的函数](https://raw.githubusercontent.com/shanchuann/TheGitbookLibrary/main/C%E8%AF%AD%E8%A8%80%E5%9C%A3%E7%BB%8F/.gitbook/assets/book-images/typora/image-20260303132149637.png)

```c
// typedef int (*PFUN)(int, int); // 定义一个函数指针类型，指向有两个int参数和一个int返回值的函数
int main() {
	int index = 0;
	int (*pfun[4])(int, int) = { add, sub, mul, div }; // 定义一个函数指针数组，存放四个函数的地址
	// PFUN arr[4] = { add, sub, mul, div }; // 也可以使用PFUN类型定义函数指针数组
	scanf("%d", &index);
	if (index >= 0 && index < 4) {
		int a = 10, b = 5;
		int result = pfun[index](a, b); // 通过函数指针调用函数
		printf("Result: %d\n", result);
	} 
	else printf("Invalid index! Please enter a number between 0 and 3.\n");
	return 0;
}
```

## 泛型编程

我们在指针章节有提到过泛型编程：当我们遇见想使用同一个函数来处理不同类型数据的时候，无类型指针将派上用场。

```c
int Iarr[5] = { 10, 21, 23, 54, 59 };
int In = sizeof(Iarr) / sizeof(Iarr[0]);
double Darr[5] = { 10.5, 21.3, 23.7, 54.2, 59.9 };
int Dn = sizeof(Darr) / sizeof(Darr[0]);
char Carr[5] = { 'a', 'b', 'c', 'd', 'e' };
int Ci = sizeof(Carr) / sizeof(Carr[0]); // 也可以不除sizeof(Carr[0])，因为char类型的大小是1字节
```

如果要设计一个printnum函数用于打印所有类型的内容，该怎么做呢？

在数组打印中，我们不仅仅要传入数组大小，如果使用void\*，还需要通过sizeof int传入数据的类型。但当我们输出时又该如何确定以何种类型输出呢？float和int都为4字节大小……

```c
void printInt(const void* vp) {
	printf("%d ", *(const int*)vp);
}
void printDouble(const void* vp) {
	printf("%lf ", *(const double*)vp);
}
void printChar(const void* vp) {
	printf("%c ", *(const char*)vp);
}
```

以上三个处理不同类型数据的函数都是接受一个const void\*形参，返回void的函数，由此发现可以在主打印函数中通过函数指针来实现不同数据类型的处理。

```c
void printArray(const void* arr, int n, int size, void (*printFunc)(const void*)) {
	for (int i = 0; i < n; i++) {
		printFunc((const char*)arr + i * size); // 计算每个元素的地址并调用打印函数
	}
	printf("\n");
}
```

```c
printArray(Iarr, In, sizeof(int), printInt); 		// 输出整数数组
printArray(Darr, Dn, sizeof(double), printDouble); 	// 输出双精度浮点数组
printArray(Carr, Ci, sizeof(char), printChar); 		// 输出字符数组
```

这样通过以上方式调用函数后即可实现对任意数据类型的处理。

函数指针本身是可以被sizeof求值的，在32位系统中结果为4，但\*pfun则不能，系统无法确定函数的空间大小，那么函数指针也就无法进行加一的操作。

函数指针也可以作返回值。

```c
void (*g_pfun)(void) = NULL; // 定义一个全局函数指针，指向一个无参数无返回值的函数
void funa() {
	printf("funa\n");
}
void funb() {
	printf("funb\n");
}
void (*get_pfun(void (*pfun)(void)))(void) {
	void(*old)(void) = g_pfun;	// 保存旧的函数指针
	g_pfun = pfun;				// 更新全局函数指针
	return old;					// 返回旧的函数指针
}
int main() {
	void (*pfun)(void) = NULL; // 定义一个函数指针，指向一个无参数无返回值的函数
	pfun = get_pfun(funb);
	return 0;
}
```

这段代码定义一个**全局的函数指针** `g_pfun`（指向 “无参数、无返回值” 的函数）；实现两个简单的目标函数 `funa`、`funb`；实现一个工具函数 `get_pfun`，接收一个函数指针参数，更新全局函数指针为该参数，并返回更新前的旧函数指针；主函数中调用 `get_pfun`，传入 `funb` 来更新全局指针，同时接收返回的旧指针。

```c
void (*g_pfun)(void) = NULL; // 定义一个全局函数指针，指向一个无参数无返回值的函数
```

`void (*g_pfun)(void)`是函数指针的标准声明方式，作为全局作用域的函数指针，整个程序都能访问或修改它，用于存储 “当前生效” 的函数地址：

* `g_pfun` 是一个**指针变量**；
* `(*g_pfun)` 表示这个指针指向一个**函数**；
* 外层的 `void`：表示该函数**无返回值**；
* 括号内的 `void`：表示该函数**无参数**。

```c
void (*get_pfun(void (*pfun)(void)))(void) {
    void(*old)(void) = g_pfun;	// 保存旧的函数指针
    g_pfun = pfun;				// 更新全局函数指针
    return old;					// 返回旧的函数指针
}
```

对于`void (*get_pfun(void (*pfun)(void)))(void)` 函数，让我们分步解读：**从函数名往外层拆解**，函数声明的格式是 `返回值 函数名 (参数列表)`。

* `get_pfun`：是函数名；
* `get_pfun(void (*pfun)(void))`：表示 `get_pfun` 的参数是 `void (*pfun)(void)`（一个无参无返回的函数指针）；
* `void (*...)(void)`：表示 `get_pfun` 的返回值是指向无参无返回函数的指针。

用 `typedef` 简化理解，如果先定义函数指针类型别名，声明会更清晰：

```c
// 定义无参无返回的函数指针类型
typedef void (*PFUN_VOID)(void);
// 此时get_pfun的声明可简化为：
PFUN_VOID get_pfun(PFUN_VOID pfun) { ... }
```

本质和原声明完全一致，只是更易读。

## 函数指针与内存地址的直接操作

除了通过函数名赋值、作为参数或返回值使用函数指针外，我们还可以直接对内存地址进行类型转换，将指定地址强转为函数指针类型并调用——但这种操作绕开了编译器的类型校验，存在极高的风险，仅用于理解函数指针的本质，实际开发中应严格避免。

先定义一个基础的加法函数作为示例：

```c
// 基础函数：接收两个int，返回int
int Add(int a, int b) {
	return a + b;
}
```

接下来通过内存地址直接操作函数指针：

```c
int x = 10;
// 打印变量x的内存地址、Add函数的入口地址
printf("x的地址 = %p \n", &x);
printf("Add函数地址 = %p \n", Add); // 函数名等价于函数指针，直接打印地址

// 1. 常规函数指针赋值与调用
int (*pfun)(int, int) = Add; // 函数名赋值给函数指针
x = (*pfun)(12, 23);         // 解引用函数指针调用（等价于pfun(12,23)）
printf("常规调用结果：%d\n", x); // 输出 35

// 2. 直接将内存地址强转为函数指针并调用
// 注：0x00401005为示例地址，需替换为实际运行时Add函数的地址
x = (*(int (*)(int, int))0x00401005)(10, 20); 
printf("地址强转调用结果：%d\n", x); // 输出 30

// 用typedef简化地址强转的写法
typedef int (*PFUN)(int, int);
(*(PFUN)0x00401005)(10, 20); // 功能与上一行完全一致，无返回值接收

// 【危险操作】调用空指针地址（会触发段错误）
// (*(void (*)())0)(); // 地址0是系统保护区域，解引用会导致程序崩溃
// typedef void (*VPF)();
// (*(VPF)0)(); // 等价写法，同样触发崩溃
```

函数名（如`Add`）在编译后会被映射为内存中的一个固定入口地址，`printf("%p", Add)`可直接打印该地址；`int (*pfun)(int, int) = Add`是编译器认可的安全写法，通过函数名赋值保证了类型匹配；`(int (*)(int, int))0x00401005`将十六进制地址强制转换为“接收两个int、返回int的函数指针”类型，再解引用调用——此时编译器无法验证该地址是否真的指向合法函数，若地址错误/无执行权限，程序会直接崩溃；

在实际使用中，“强转函数指针”和“返回函数指针”是两种完全不同的操作。

|          | 强转函数指针               | 返回函数指针              |
| -------- | -------------------- | ------------------- |
| **操作本质** | 将内存地址（数值）强制转换为函数指针类型 | 函数执行后返回一个合法的函数指针变量  |
| **类型校验** | 绕开编译器类型校验，无类型安全保障    | 编译器严格校验返回值与函数声明的匹配性 |
| **合法性**  | 仅当地址指向合法函数时有效，否则崩溃   | 基于已定义的函数/指针返回，天然合法  |
| **使用场景** | 极特殊场景（如底层系统开发），几乎不用  | 通用场景（动态切换函数逻辑、回调管理） |
| **风险等级** | 极高（地址错误直接导致程序崩溃）     | 低（仅需校验指针是否为NULL）    |

**强转函数指针**是“被动适配类型”：把无意义的内存地址强行解释为函数指针，完全依赖开发者对内存地址的精准把控，属于底层、高风险操作，仅用于理解原理或极特殊的系统级开发；

**返回函数指针**是“主动传递指针”：函数通过返回值将已定义的、合法的函数指针（如全局指针`g_pfun`的旧值）传递给调用者，编译器全程参与类型校验，是函数指针的常规、安全用法；

实际开发中，我们优先使用“返回函数指针”实现逻辑复用（如前文的`get_pfun`函数），绝对避免无依据的“强转函数指针”——除非明确知道目标地址指向合法的可执行函数，且有必须这么做的业务场景。

## 回调接口的契约

回调函数的声明应固定参数类型、返回值和调用时机。还要说明比较函数是否允许修改元素、上下文指针是否可以为 `NULL`，以及回调执行期间数据是否可能被释放。函数指针只是类型，不会自动管理这些生命周期问题。
