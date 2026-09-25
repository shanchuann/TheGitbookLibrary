---
description: 用 C 组织一个小型终端练习程序。
icon: code
---

# 打字母游戏

> **学习路径**：本章把数组、结构体、函数、随机数、终端输入输出和内存布局组合成一个小型程序。阅读时应把它当作一次工程回顾：先看数据模型，再看更新循环，最后检查平台相关代码。

在经过一段实践的学习后，我们急切的需要一个练习来巩固之前所学，因此本节向大家介绍一个练习：打字母游戏。

### 打字母v1.0

项目需要随机产生一个字母从屏幕上方向下落，玩家输入字母，如果和显示的字母相同，就消去字母；游戏会再随机产生一个字母，继续游戏，如果字母落出屏幕，玩家失败，游戏结束。项目由显示模块和处理模块构成，显示模块由二维数组构成，把随机产出的字母赋值到二维数组中，处理模块随机产出字母，输入字母比较，字母是否落出屏幕以及字母下降功能。

首先需要宏定义数据类型和基本的数值：

```c
#define ROWSIZE 20
#define COLSIZE 70
#define LETSIZE 1

struct LetterNode {
    char ch;
    int row;
    int col;
};

typedef char GridArray[ROWSIZE][COLSIZE + 1];
```

我们创建了一个长70，高20的画布，并初始化下落的字母数为1。并定义了一个字符的结构体，包含字符的数据和行列的信息。我们把 `char[ROWSIZE][COLSIZE + 1]` 这样的二维字符数组通过 `typedef` 定义成 `GridArray` 这个新类型，便于后续编写。

```c
// 不使用 typedef
// 定义变量时，要写完整的二维数组类型
char grid[ROWSIZE][COLSIZE + 1];
// 函数参数时，要重复写类型
void initGrid(char grid[ROWSIZE][COLSIZE + 1]) { ... }
void showGrid(char grid[ROWSIZE][COLSIZE + 1], struct LetterNode* px, int n) { ... }

// 使用 typedef
// 先给类型起别名
typedef char GridArray[ROWSIZE][COLSIZE + 1];
// 定义变量时，直接用别名
GridArray grid;
// 函数参数时，直接用别名
void initGrid(GridArray grid) { ... }
void showGrid(GridArray grid, struct LetterNode* px, int n) { ... }
```

接下来按照模块分别编写初始化函数`initGrid`和打印函数`showGrid`，用 `#` 初始化整个二维数组，并在末尾补 `\0`。

```c
void initGrid(GridArray grid) {
    for (int i = 0; i < ROWSIZE; i++) {
		memset(grid[i], '#', COLSIZE);
        grid[i][COLSIZE] = '\0';
    }
}
void showGrid(GridArray grid,struct LetterNode* px,int n) {
	assert(px != NULL);
	system("cls");
	initGrid(grid);
    for (int i = 0; i < n; i++) grid[px[i].row][px[i].col] = px[i].ch;
    for (int i = 0; i < ROWSIZE; i++) printf("%s \n", grid[i]);
}
```

`showGrid`函数中引入`#include <windows.h>`中的`system("cls");`，用于刷新输出，在这里需要注意的是在每一次刷新后需要初始化一下二维数组，否则会有残留数据导致一整列都是字母，而非字母下落的效果。

在每次字母下落时，因为调用了system("cls");，所以输出界面会有明显的闪烁。在这里我们使用`hideConsoleCursor()`（隐藏光标）和`setConsoleCursorPos()`（定位光标）来优化显示效果。

```c
// 隐藏光标、设置光标位置
void hideConsoleCursor() {
    HANDLE hOut = GetStdHandle(STD_OUTPUT_HANDLE);
    CONSOLE_CURSOR_INFO cci;
    GetConsoleCursorInfo(hOut, &cci);
    cci.bVisible = FALSE;
    SetConsoleCursorInfo(hOut, &cci);
}

void setConsoleCursorPos(int x, int y) {
    COORD pos = { x, y };
    SetConsoleCursorPosition(GetStdHandle(STD_OUTPUT_HANDLE), pos);
}

void initGrid(GridArray grid) {
    for (int i = 0; i < ROWSIZE; i++) {
        memset(grid[i], '#', COLSIZE);  // 填充数组
        grid[i][COLSIZE] = '\0';        // 末尾置空
    }
}

void showGrid(GridArray grid, struct LetterNode* px, int n) {
    assert(px != NULL);
    // 注释掉导致闪烁的system("cls")，改为光标定位到左上角
    // system("cls");
    setConsoleCursorPos(0, 0); // 光标移到左上角，覆盖输出
    hideConsoleCursor();       // 隐藏光标，减少闪烁

    initGrid(grid);
    for (int i = 0; i < n; i++) {
        grid[px[i].row][px[i].col] = px[i].ch;
    }
    for (int i = 0; i < ROWSIZE; i++) {
        printf("%s \n", grid[i]);
    }
}
```

如此就可以实现基本内容：打印画布，显示字母，字母下落。

```c
int main() {
	GridArray grid;
    struct LetterNode letter[LETSIZE] = { 0 };
	letter[0].ch = 'X';
	letter[0].row = 0;
	letter[0].col = 2;
	while (letter[0].row < ROWSIZE)
	{
		showGrid(grid, letter, LETSIZE);
		letter[0].row++;
		Sleep(1000);
	}
}
```

另外我们还需要让字母在第一行的随即列生成随机字母，因此需要实现一个随机函数。

```c
void RandLetter(struct LetterNode* px, int n) {
    assert(px != NULL);
	srand((unsigned int)time(NULL));    // 设置随机种子
    for (int i = 0; i < n; i++) {
        px[i].ch = 'A' + rand() % 26;   // 随机生成一个大写字母
        px[i].row = 0;                  // 从第一行开始
        px[i].col = rand() % COLSIZE;   // 随机列位置
    }
}
```

当有了以上基本的函数后，就可以在main主程序里实现游戏的循环和退出逻辑。

```c
while (1)
{
    showGrid(grid, letter, LETSIZE);
	Sleep(500);     // 暂停200毫秒，控制字母下落速度
	if (_kbhit())   // 检测键盘输入
    {
		//input = getchar(); // 直接使用getchar()会导致输入缓冲区问题，改为_getch()，它不会等待回车
		input = _getch(); // 获取输入的字符
        if (input == letter[0].ch)
        {
            letter[0].ch = 'A' + rand() % 26;   // 重新生成一个随机字母
            letter[0].col = rand() % COLSIZE;   // 重新生成一个随机列位置
            letter[0].row = -1;                 // 重置为第一行
        }
    }
    letter[0].row++;
    if (letter[0].row >= ROWSIZE)  // 如果字母落到底部，重新生成
    {
		//Game Over
        printf("Game Over! The letter was '%c'.\n", letter[0].ch);
		break;
	}
}
```

需要注意的是，当我们使用getchar()函数时，程序会因为等待用户的回车输入而造成卡顿，因此我们需要将其替换为`_getch()`。`_getch()` 是 Windows 平台下的**无回显、无缓冲字符输入函数**，`_kbhit()`（全称：keyboard hit）则是 Windows 平台下的**非阻塞式按键检测函数**，二者均定义在 `<conio.h>` 头文件中。

如果我们不使用`if (_kbhit())`的话，程序会卡在 `_getch()` 处，直到我们按按键才会继续，因此需要在主循环里添加这一判断条件。以下是代码全部：

```c
#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <string.h>
#include <assert.h>
#include <stdlib.h>
#include <windows.h>
#include <time.h>
#include <conio.h>

#define ROWSIZE 20
#define COLSIZE 70
#define LETSIZE 1

struct LetterNode {
    char ch;
    int row;
    int col;
};

typedef char GridArray[ROWSIZE][COLSIZE + 1];

// 隐藏光标、设置光标位置
void hideConsoleCursor() {
    HANDLE hOut = GetStdHandle(STD_OUTPUT_HANDLE);
    CONSOLE_CURSOR_INFO cci;
    GetConsoleCursorInfo(hOut, &cci);
    cci.bVisible = FALSE;
    SetConsoleCursorInfo(hOut, &cci);
}

void setConsoleCursorPos(int x, int y) {
    COORD pos = { x, y };
    SetConsoleCursorPosition(GetStdHandle(STD_OUTPUT_HANDLE), pos);
}

void initGrid(GridArray grid) {
    for (int i = 0; i < ROWSIZE; i++) {
        memset(grid[i], '#', COLSIZE);  // 填充数组
        grid[i][COLSIZE] = '\0';        // 末尾置空
    }
}

void showGrid(GridArray grid, struct LetterNode* px, int n) {
    assert(px != NULL);
    // 注释掉导致闪烁的system("cls")，改为光标定位到左上角
    // system("cls");
    setConsoleCursorPos(0, 0); // 光标移到左上角，覆盖输出
    hideConsoleCursor();       // 隐藏光标，减少闪烁

    initGrid(grid);
    for (int i = 0; i < n; i++) {
        grid[px[i].row][px[i].col] = px[i].ch;
    }
    for (int i = 0; i < ROWSIZE; i++) {
        printf("%s \n", grid[i]);
    }
}
void RandLetter(struct LetterNode* px, int n) {
    assert(px != NULL);
	srand((unsigned int)time(NULL));    // 设置随机种子
    for (int i = 0; i < n; i++) {
        px[i].ch = 'A' + rand() % 26;   // 随机生成一个大写字母
        px[i].row = 0;                  // 从第一行开始
        px[i].col = rand() % COLSIZE;   // 随机列位置
    }
}

int main() {
    GridArray grid;
	char input;
    struct LetterNode letter[LETSIZE] = { 0 };
	RandLetter(letter, LETSIZE);
    while (1)
    {
        showGrid(grid, letter, LETSIZE);
		Sleep(500);     // 暂停200毫秒，控制字母下落速度
		if (_kbhit())   // 检测键盘输入
        {
			//input = getchar(); // 直接使用getchar()会导致输入缓冲区问题，改为_getch()，它不会等待回车
			input = _getch(); // 获取输入的字符
            if (input == letter[0].ch)
            {
                letter[0].ch = 'A' + rand() % 26;   // 重新生成一个随机字母
                letter[0].col = rand() % COLSIZE;   // 重新生成一个随机列位置
                letter[0].row = -1;                 // 重置为第一行
            }
        }
        letter[0].row++;
        if (letter[0].row >= ROWSIZE)  // 如果字母落到底部，重新生成
        {
			//Game Over
            printf("Game Over! The letter was '%c'.\n", letter[0].ch);
			break;
		}
    }
	return 0;
}
```

![image-20260304131550146](https://raw.githubusercontent.com/shanchuann/TheGitbookLibrary/main/C%E8%AF%AD%E8%A8%80%E5%9C%A3%E7%BB%8F/.gitbook/assets/book-images/typora/image-20260304131550146.png)

### 打字母v2.0

在v1.0的基础上我们要对游戏进行一些升级，我们不再满足于只打印一个字母，而是将数量提升到10甚至更高，这不仅仅需要修改宏定义，也需要对打印和判定函数做出修改。

```c
#define LETSIZE 10

int main() {
    GridArray grid;
    char input;
    struct LetterNode letter[LETSIZE] = { 0 };
    RandLetter(letter, LETSIZE);
    while (1)
    {
        showGrid(grid, letter, LETSIZE);
        Sleep(1000);   // 控制字母下落速度
        if (_kbhit())   // 检测键盘输入
        {
            //input = getchar(); // 直接使用getchar()会导致输入缓冲区问题，改为_getch()，它不会等待回车
            input = _getch(); // 获取输入的字符
            for (int i = 0; i < LETSIZE; i++) {
				if (input == letter[i].ch) { // 如果输入的字符与某个字母匹配
                    letter[i].ch = 'A' + rand() % 26;   // 重新生成一个随机字母
                    letter[i].col = rand() % COLSIZE;   // 重新生成一个随机列位置
                    letter[i].row = -1;
                }
            }
        }
		for (int i = 0; i < LETSIZE; i++) {
            letter[i].row++; // 字母下落一行
            if (letter[i].row >= ROWSIZE) { // 如果字母掉出屏幕底部
                printf("Game Over!\n", letter[i].ch);
                return 0; // 结束游戏
            }
        }
    }
    return 0;
}
```

到这里打字母练习就基本结束了，不过这仅仅是一个雏形，还有很多内容值得修改，不过这里就不做多说。

下面是原稿中的运行结果截图。它只用于展示交互效果；不同终端的光标控制、字体和窗口大小可能导致画面略有差异。

![打字母游戏运行结果](https://files.seeusercontent.com/2026/03/05/3dzK/image-20260305120246687.png)

## 原稿图示
