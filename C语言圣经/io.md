---
description: 标准输入输出与格式化。
icon: code
---

# C语言输入输出

### 1. printf

格式输出函数,末尾字母"f"为format(格式)之意

```c
int a = 10;
float b = 1.5;
char ch = 'a';
printf("浮点型:%f 字符型:%c",b,ch);
printf("十进制:%d 八进制:%o 十六进制:%x",a,a,a);
//printf("格式控制字符串",输出表列);
```

| 格式字符串 | 输出格式     |
| ---------- | ------------ |
| %d         | 十进制整型   |
| %o         | 八进制整型   |
| %x         | 十六进制整型 |
| %ld        | 十进制长整型 |
| %c         | 字符型       |
| %f         | 浮点型       |

### 2. scanf

格式输入函数,用户从键盘把数据输入到指定变量中

```c
scanf("%d",&a);//scanf("格式控制字符串",地址表列);
```

**注意:**从键盘输入时,应匹配scanf读取格式

函数将用户输入存放进缓冲区,又回显到控制台。

### 3. 例子

海伦公式求三角形面积

```c
#include <stdio.h>
#include <math.h>
int main()
{
	float a,b,c,p,area;
	printf("请输入三角形的三边长\n");
	scanf("%f %f %f",&a,&b,&c);
	p=1.0/2*(a+b+c);
	if(a+b>c&&b+c>a&&a+c>b)
    {
		area=sqrt(p*(p-a)*(p-b)*(p-c));
		printf("三角形的面积为：%.2f\n",area);
	}
	else printf("不能构成三角形\n");
	return 0;
}
```
