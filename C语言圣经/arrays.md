---
description: 一维数组、二维数组和字符串数组。
icon: code
---

# 数组

数组是包含给定类型的一组数据，并将这些数据依次存储在连续的内存空间中。每个独立的数据被称为数组的元素(element)。

元素的类型可以是任意类型。数组本身也是一个结构，其类型由它的元素类型延伸而来。更具体地说数组的类型由元素的类型和数量所决定。如果一个数组的元素是 T 类型，那么该数组就称为"T数组"。

例如，如果元素类型为 int，那么该数组的类型就是“int 数组”。然而，int 数组类型是不完整的类型，除非指定了数组元素的数量。如果一个 int 数组有 16 个元素，那么它就是一个完整的对象类型，即“16 个 int 元素数组”。

### 一维数组

数组的定义决定了数组名、元素类型以及元素个数。

`<类型> 数组名[<元素数量>];`

其语法如下:
元素数量在方括号 ( [ ] ) 之间，它必须是大于 0 的整数常量表达式

```c
int main()
{
    const int n = 10;
    int arr[n] = {12,23,34,45,56,67,78,89,90,100};//sizeof(arr) ?
    for(int i = 0;i<n;i++)
    { 
        printf("%d",arr[i]);
    }
    printf("\n");
    return 0;
}
```

应用示例：查表法

将事先计算好的结果存储在数组中，使用时直接按下标取数据，以节省运行时的计算时间，用空间换时间

```c
int Get_Day(int year,int month)
{
    int day = 0;
    switch(month)
    {
        case 1:case 3:case 5:case 7:case 8:case 10:case 12:
            day = 31;
            break;
        case 4:case 6:case 9:case 11:
            day = 30;
            break;
        case 2:
            day = IsLeap(year)?29:28;
            break;
    }
    return day;
}
//31 28/29  31  30  31  30  31  31  30  31  30  31
//1  2	    3 	4	5	6	7	8	9	10	11	12
int Get_Day_ARR(int year,int month)
{
    static const int days[] = {29,31,28,31,30,31,30,31,31,30,31,30,31};
    if(month == 2&&IsLeap(year))
    {
        month = 0;
    }
    return days[month]; 
} 
```

#### 杨辉三角

在不使用二维数组的情况下该如何打印杨辉三角呢？我们能注意到杨辉三角的特性很符合数组的递推更新，并且当我们从后向前进行更新时，可以很好的避免前一个值被覆盖的情况。因此可以利用前一行的数据，对下一行进行更新。

![image-20260219121951980](https://raw.githubusercontent.com/shanchuann/TheGitbookLibrary/main/C%E8%AF%AD%E8%A8%80%E5%9C%A3%E7%BB%8F/.gitbook/assets/book-images/typora/image-20260219121951980.png)

我们通过设定`arr[i] = 1`来保证每一层的结尾都为1，然后判断是否应该向前进行下一步，如果`i - 1>0`则说明前方仍有数据则进行覆盖若没有则退出二层循环，进行数据的打印，将下一个空间设置为1，重复判断是否应该从后向前覆盖，按同样的规则继续。

![image-20260219121918719](https://raw.githubusercontent.com/shanchuann/TheGitbookLibrary/main/C%E8%AF%AD%E8%A8%80%E5%9C%A3%E7%BB%8F/.gitbook/assets/book-images/typora/image-20260219121918719.png)

```c
#define N 20
int main() {
	int arr[N] = { 0 };
	for (int i = 0; i < N; i++) {
		arr[i] = 1;
		for (int j = i - 1; j > 0; j--) {
			arr[j] += arr[j - 1];
		}
		for (int k = 0; k <= i; k++) {
			printf("%-6d ", arr[k]);
		}
		printf("\n");
	}
}
```

打印结果如上图。

#### 二分查找

当我们遇见数组元素从小到大排列的数组，想从中找到某一具体数值时，若是从数组首元素逐个查找将会很浪费时间，这时就需要引入一个新的效率高的查找方式：二分查找。

二分查找主要操作是**每次都砍掉一半的查找范围**，在**有序**数组中，先取数组中间位置的元素和目标值比较，如果中间元素等于目标值则直接找到；如果目标值更小，就只在数组左半部分继续重复这个 “找中间、做比较、缩范围” 的操作；如果目标值更大，就只在右半部分重复，直到找到目标值。当查找范围缩小到空则说明目标值不存在，这种方式时间复杂度为 O (log n)，快于直接遍历。

![image-20260219123637233](https://raw.githubusercontent.com/shanchuann/TheGitbookLibrary/main/C%E8%AF%AD%E8%A8%80%E5%9C%A3%E7%BB%8F/.gitbook/assets/book-images/typora/image-20260219123637233.png)

```c
int binarySearch(const int* arr, int n, int val) {
	int left = 0;
	int right = n - 1;
	while (left <= right) {
		int mid = left + (right - left) / 2;
		if (arr[mid] == val) {
			return mid;			// 找到目标元素，返回索引
		}
		else if (arr[mid] < val) {
			left = mid + 1;		// 目标元素在右半部分
		}
		else {
			right = mid - 1;	// 目标元素在左半部分
		}
	}
	return -1;					// 没有找到目标元素
}
```

通过调用函数即可查询数组元素的下标位置：

```c
int main() {
	int arr[] = { 1, 2, 3, 4, 5, 6, 7, 8, 9 };
	int n = sizeof(arr) / sizeof(arr[0]);
	int val = 5;
	int index = binarySearch(arr, n, val);
	if (index != -1) {
		printf("元素 %d 在数组中的索引为: %d\n", val, index);
	}
	else {
		printf("元素 %d 不在数组中\n", val);
	}
	return 0;
} // 元素 5 在数组中的索引为: 4
```

#### 数组的移动

如何将一个一维数组左移、右移一位或K位呢？

基础移动（1 位）

- 左移 1 位：`{2, 3, 4, 5, 6, 7, 8, 9, 10, 1}`（第一个元素 1 移到末尾，其余元素整体左移）
- 右移 1 位：`{10, 1, 2, 3, 4, 5, 6, 7, 8, 9}`（最后一个元素 10 移到开头，其余元素整体右移）

任意 K 位移动

- 左移 3 位（K=3）：`{4, 5, 6, 7, 8, 9, 10, 1, 2, 3}`（前 3 个元素 1、2、3 移到末尾，其余元素左移）
- 左移 5 位（K=5）：`{6, 7, 8, 9, 10, 1, 2, 3, 4, 5}`
- 右移 2 位（K=2）：`{9, 10, 1, 2, 3, 4, 5, 6, 7, 8}`（最后 2 个元素 9、10 移到开头，其余元素右移）
- 右移 4 位（K=4）：`{7, 8, 9, 10, 1, 2, 3, 4, 5, 6}`

K 等于数组长度

- 左移 10 位（K=10）：`{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}`（回到原数组）
- 右移 10 位（K=10）：`{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}`（回到原数组）

那么我们该如何实现？

```c
void rightMovearray(int arr[], int n) { // 右移一个数据元素
    if (n <= 1) return; // 边界处理
    int temp = arr[n - 1]; // 保存最后一个元素
    for (int i = n - 1; i > 0; i--) arr[i] = arr[i - 1]; // 元素依次后移
    arr[0] = temp; // 最后一个元素放到开头
}
void rightMovearray_K(int arr[], int n, int k) { // 右移k个数据元素
    k = k % n; // 处理k大于n的情况
    if (k == 0) return;
    for (int i = 0; i < k; i++) rightMovearray(arr, n); // 调用右移一位函数k次
}
void leftMovearray(int arr[], int n) { // 左移一个数据元素
    if (n <= 1) return; // 边界处理
    int temp = arr[0]; // 保存第一个元素
    for (int i = 0; i < n - 1; i++) arr[i] = arr[i + 1]; // 元素依次前移
    arr[n - 1] = temp; // 第一个元素放到末尾
}
void leftMovearray_K(int arr[], int n, int k) { // 左移k个数据元素
    k = k % n; // 处理k大于n的情况
    if (k == 0) return;
    for (int i = 0; i < k; i++) leftMovearray(arr, n); // 调用左移一位函数k次
}
void printArray(int arr[], int n) {
    for (int i = 0; i < n; i++) printf("%d ", arr[i]);
    printf("\n");
}
```

`rightMovearray()` 和 `leftMovearray()` 实现了数组循环移动的基础函数，分别对应数组右移一位和左移一位的操作，当数组长度小于等于 1 时直接返回，避免无效操作和数组越界问题。其中右移一位的函数会先暂存数组最后一个元素，防止后续元素移位时被覆盖丢失，再从数组末尾开始倒序遍历，把每个元素依次向后挪动一位，最后把提前暂存的末尾元素放到数组的起始位置，完成所有元素的循环右移；左移一位的函数则是完全反向的逻辑，先暂存数组的第一个元素，再从数组头部开始正序遍历，让每个元素依次向前挪动一位，最后把暂存的起始元素放到数组的末尾，完成循环左移。

`rightMovearray_K()` 和 `leftMovearray_K()` 在单步移动函数的基础上，实现了数组任意位数的循环移动，通过循环调用对应方向的单步移动函数，重复执行 K 次来完成批量移位，同时做了两处关键的优化和容错处理。首先会通过 `k % n` 的取模运算计算出有效移动位数，因为对于长度为 n 的数组来说，移动 n 位就相当于回到了初始状态，没有任何变化，比如长度为 10 的数组移动 13 位，实际有效移位只有 3 位，取模操作可以避免大量重复的无效循环；如果取模后的结果为 0，说明移动后数组和原数组完全一致，函数会直接返回，不执行多余的操作。

```c
int main() {
    int arr[10] = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
    int n = 10;
    printf("原数组: ");
    printArray(arr, n);
    // 测试右移一位
    rightMovearray(arr, n);
    printf("右移一位后: ");
    printArray(arr, n);
    // 测试右移k=3位
    rightMovearray_K(arr, n, 3);
    printf("右移3位后: ");
    printArray(arr, n);
    // 测试左移一位
    leftMovearray(arr, n);
    printf("左移一位后: ");
    printArray(arr, n);
    // 测试左移k=2位
    leftMovearray_K(arr, n, 2);
    printf("左移2位后: ");
    printArray(arr, n);
	// 测试k等于n的情况
	rightMovearray_K(arr, n, n);
    printf("K等于n（右移）：");
	printArray(arr, n);
	// 测试k大于n的情况
	leftMovearray_K(arr, n, n + 2);
	printf("K大于n（左移n+2）：");
	printArray(arr, n);
    return 0;
}
```

```
原数组: 1 2 3 4 5 6 7 8 9 10
右移一位后: 10 1 2 3 4 5 6 7 8 9
右移3位后: 7 8 9 10 1 2 3 4 5 6
左移一位后: 8 9 10 1 2 3 4 5 6 7
左移2位后: 10 1 2 3 4 5 6 7 8 9
K等于n（右移）：10 1 2 3 4 5 6 7 8 9
K大于n（左移n+2）：2 3 4 5 6 7 8 9 10 1
```

这样一来就实现了函数移动的功能，通过模块化功能设计大大减少了开发工作量。但是由于采用暴力解方法，时间复杂度**O(k\*n)**。在最坏情况下（如 `k = n-1`）会退化为 **O(n²)**。那么有没有什么方法能解决这一问题，降低时间复杂度呢？

**三次翻转法**是一种更优雅的数组移动方案，它通过三次局部反转来实现整体循环移动，时间复杂度优化为 O (n)，空间复杂度仍为 O (1)。这种方法避免了重复遍历数组的低效操作，仅通过有限次的元素交换就能完成目标，在处理大规模数组时性能优势尤为明显，既不需要额外的辅助数组占用内存，也能保证线性的时间效率。

对于长度为 n 的数组，右移 k 位可以拆解为三步：首先翻转整个数组，让原本位于数组末尾的 k 个元素 “移动” 到数组的前半部分，但此时这两部分元素的内部顺序是颠倒的；接下来翻转前 k 个元素，把这部分元素的顺序调整回来，让它们恢复原本的相对位置；最后翻转后 n-k 个元素，同样恢复这部分元素的原有顺序，经过这三次局部反转，就能实现数组整体右移 k 位的效果，整个过程只需要遍历数组两次（三次翻转的总操作次数为 n 次）。

如果是左移 k 位，逻辑类似但步骤顺序有所调整。需要先翻转前 k 个元素，再翻转后 n-k 个元素，最后翻转整个数组，通过这样的顺序变化，就能精准实现左移的效果，无论左移还是右移，三次翻转法都能在保持 O (1) 空间复杂度的前提下，以 O (n) 的时间高效完成，是处理数组循环移动问题的经典最优解。

以右移3位为例：

![image-20260219151101499](https://raw.githubusercontent.com/shanchuann/TheGitbookLibrary/main/C%E8%AF%AD%E8%A8%80%E5%9C%A3%E7%BB%8F/.gitbook/assets/book-images/typora/image-20260219151101499.png)

左移则是：

原数组：`[1, 2, 3, 4, 5, 6, 7, 8, 9, 10]`（n=10，k=2）

1. **翻转前 k 个元素：**翻转范围：`[0, 1]`，结果：`[2, 1, 3, 4, 5, 6, 7, 8, 9, 10]`
2. **翻转后 n-k 个元素：**翻转范围：`[2, 9]`，结果：`[2, 1, 10, 9, 8, 7, 6, 5, 4, 3]`
3. **翻转整个数组：**翻转范围：`[0, 9]`，结果：`[3, 4, 5, 6, 7, 8, 9, 10, 1, 2]`（完成左移 2 位）

```c
void reverse(int arr[], int start, int end) { // 辅助函数：翻转数组中从 start 到 end 的元素
    while (start < end) {
        int temp = arr[start];
        arr[start] = arr[end];
        arr[end] = temp;
        start++;
        end--;
    }
}
void rightMoveArrayK(int arr[], int n, int k) { // 右移 k 个数据元素
    k = k % n; // 处理 k 大于 n 的情况
    if (k == 0) return;
    reverse(arr, 0, n - 1);      // 1. 翻转整个数组
    reverse(arr, 0, k - 1);      // 2. 翻转前 k 个元素
    reverse(arr, k, n - 1);      // 3. 翻转后 n-k 个元素
}
void leftMoveArrayK(int arr[], int n, int k) { // 左移 k 个数据元素
    k = k % n;
    if (k == 0) return;
    reverse(arr, 0, k - 1);      // 1. 翻转前 k 个元素
    reverse(arr, k, n - 1);      // 2. 翻转后 n-k 个元素
    reverse(arr, 0, n - 1);      // 3. 翻转整个数组
}
void printArray(int arr[], int n) {
    printf("[");
    for (int i = 0; i < n; i++) {
        printf("%d", arr[i]);
        if (i < n - 1) printf(", ");
    }
    printf("]\n");
}
```

通过这一方法就可以在不使用过长时间的情况下进行数组的移动（测试用例基本不变）。

```c
原数组: [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
右移3位后: [8, 9, 10, 1, 2, 3, 4, 5, 6, 7]
左移2位后: [10, 1, 2, 3, 4, 5, 6, 7, 8, 9]
K 等于 n（右移）: [10, 1, 2, 3, 4, 5, 6, 7, 8, 9]
K 大于 n（左移 n + 2）: [2, 3, 4, 5, 6, 7, 8, 9, 10, 1]
```

### 字符串与字符数组 

```c
int main(){
    char stra[]={"hello"};//5
    char strb={'h','e','l','l','o'};//15?
	int lena, lenb;
	lena = strlen(stra);
    lenb = strlen(strb);//?
	printf("lena :%d\n",lena);
    printf("lenb :%d\n",lenb);
	return 0;
}
```

由于编译器不同,在`strlen()`字符数组时,倘若结尾未有`'\0'`,则继续读取直到读取到'\0'结束,因为'\0'在编译器中位置不可知,则导致了strlen()函数输出错误的结果。

### 指针数组和数组指针

```c
int main(){
    int a = 1,b = 2,c = 3;
    int* ar[3] = {&a,&b,&c};   //指针数组
    int (*pa)[5];  //数组指针
}
```

#### 数组指针
数组指针：本质是一个指针，指向了一个数组，数组中的每个元素都是某种数据类型的值（比如 int 类型）。

`int (*p)[n];//定义了一个数组指针，指向一个大小为n的数组,数组中的每个元素都是int类型`

数组指针也称行指针，也就是说，当指针p执行p+1时，指针会指向数组的下一行，如：

```cint a[3][4];
int a[3][4];
int (*p)[4];//p是一个数组指针，指向了一个包含4个int型元素的数组
p=a;		//将二维数组的首地址赋给p，即a[0]或a[0][0]
p++;		//跨过第一行，p指向了a[1][0]
```

#### 指针数组
指针数组：本质是一个数组，该数组中的每个元素都是一个指针。

`int *p[n];	//定义了一个指针数组，数组大小为n，数组中的每个元素都是一个int*指针`

指针数组是一个包含若干个指针的数组，p是数组名，当执行p+1时，则p会指向数组中的下一个元素。

```cint a[3][4];
int a[3][4];
int *p[3];		//定义了一个数组，该数组中有3个int*指针变量，分别为p[0]、p[1]、p[2]
//p++;			//若执行此语句，则数组p指向下一个数组元素
for(int i=0;i<3;i++){
    p[i]=a[i];	//数组p中有3个指针，分别指向二维数组a的每一行
}
```

### 二维数组

定义二维数组很简单，只需要`类型名 数组名[行表达式][列表达式];`，行与列用常量表达式。

```c
int main(){
	//未初始化的二维数组
	int iar[3][4]; // 3行4列
	char car[3][4];
	double dar[3][4];
	return 0;
}
```

二维数组的初始化可以省略最高维的大小，如`int arr[][5];`但不能省略最低维的大小。

```c
int main()
{
	int ar[][4] = { 1,2,3,4,5,6,7,8,9,10,11,12 };
	int total = sizeof(ar);		// 计算整个二维数组的总字节数
	int size = sizeof(ar[0]);	// 计算数组第一行的字节数
	int n = total / size;		// 计算数组的行数
	return 0;
}
```



他的逻辑表示相当于一个n*m的表格：

![image-20260215161140080](https://raw.githubusercontent.com/shanchuann/TheGitbookLibrary/main/C%E8%AF%AD%E8%A8%80%E5%9C%A3%E7%BB%8F/.gitbook/assets/book-images/typora/image-20260215161140080.png)

二维数组在本质上由多个一维数组构成(每个一维数的大小必须相同)。如定义`int ar[3][4]`的二维数组，它是由3个一维数组组成，每个一维数组的大小是4个整型元素。

数组可以只对部分元素赋值，未赋值的元素自动取0值。

```c
int main() {
	int arr[3][4] = { 1,2,3,4,5,6,7,8,9,10,11,12 }; // 3行4列
	int brr[3][4] = { 1,2,3,4,5,6,7};
	int crr[3][4] = { {1,2},{3,4},{5,6} };
	return 0;
}
```

![image-20260215163510987](https://raw.githubusercontent.com/shanchuann/TheGitbookLibrary/main/C%E8%AF%AD%E8%A8%80%E5%9C%A3%E7%BB%8F/.gitbook/assets/book-images/typora/image-20260215163510987.png)
在C语言中，二维数组存放形式是按行优先存储，也就是物理表现形式如图：

![image-20260215195306920](https://raw.githubusercontent.com/shanchuann/TheGitbookLibrary/main/C%E8%AF%AD%E8%A8%80%E5%9C%A3%E7%BB%8F/.gitbook/assets/book-images/typora/image-20260215195306920.png)

| 元素       | 起始地址   | 占用的字节地址范围      | 十进制地址范围      |
| ---------- | ---------- | ----------------------- | ------------------- |
| `ar[0][0]` | 0x00b3fbac | 0x00b3fbac ~ 0x00b3fabf | 11786156 ~ 11786159 |
| `ar[0][1]` | 0x00b3fbb0 | 0x00b3fbb0 ~ 0x00b3fbb3 | 11786160 ~ 11786163 |
| `ar[0][2]` | 0x00b3fbb4 | 0x00b3fbb4 ~ 0x00b3fbb7 | 11786164 ~ 11786167 |
| `ar[0][3]` | 0x00b3fbb8 | 0x00b3fbb8 ~ 0x00b3fbbb | 11786168 ~ 11786171 |
| `ar[1][0]` | 0x00b3fbbc | 0x00b3fbbc ~ 0x00b3fbbf | 11786172 ~ 11786175 |

可以看到随着数组位置的增加，数组元素的地址每次增长一个类型大小，等到了三维数组时，就将是立体的概念。

### 二维数组与指针

在一维数组的指针关系中：

```c
int main(){
	int ar[4] = { 1,2,3,4 };
	sizeof(ar); // 16，计算整个数组的大小
	int* p = ar; //ar; 首元素地址 // int *p = &ar[0];
	int (*s)[4] = &ar; // int size
	return 0;
}
```

`ar` 是**首元素地址**，类型为 `int*`，指向 `ar[0]`，`&ar` 是**整个数组的地址**，类型为 `int(*)[4]`（指向包含4个int的数组的指针），两者虽然地址数值相同，但含义不同，步长不同：`ar+1` 跳过4字节，`&ar+1` 则跳过16字节。因此想让指针指向数组的地址，还需要对其大小进行约束。

同样的在二维数组的指针中：

```c
int main() {
	int dx[3][4] = { 1,2,3,4,5,6,7,8,9,10,11,12 };
	int size = sizeof(dx); // 计算整个二维数组大小：3*4*4=48字节
	int (*p)[4] = dx; // 首元素地址
	int (*s)[3][4] = &dx; // 整个二维数组的地址
	return 0;
}
```

由于二维数组的本质是“**数组的数组**”，因此它的指针关系有三层含义：

- `dx` 的“首元素”是第一行的一维数组 `dx[0]`，因此 `dx` 隐式转换为**指向一维数组的指针**，类型为 `int(*)[4]`。
- `&dx` 是**整个二维数组的地址**，类型为 `int(*)[3][4]`。

可以用一张表格清晰对比这三层含义：

| 表达式     | 类型           | 指向的内容         | 步长（字节） |
| ---------- | -------------- | ------------------ | ------------ |
| `dx[0][0]` | `int`          | 第一个整型元素     | 4            |
| `dx[0]`    | `int*`         | 第一行首元素地址   | 4            |
| `dx`       | `int(*)[4]`    | 第一行一维数组地址 | 16 (4×4)     |
| `&dx`      | `int(*)[3][4]` | 整个二维数组地址   | 48 (3×4×4)   |

由此可以总结出等价关系（详细的转换见后续二级指针中指针与数组的应用部分）：

```c
dx[i][j] == *(*(dx + i) + j)
```

```c
int main() {
	int dx[3][4] = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12 }; 
	int size = sizeof(dx);	// int, row col
	int (*s)[3][4] = &dx;	// int row col
	int (*p)[4] = dx;		//首元素地址
	printf("%d \n", sizeof(p));		// 4
	printf("%d \n",sizeof(*p));		// 16
	printf("%d \n",sizeof(s));		// 4
	printf ("%d \n",sizeof (*s));	// 48
	printf("%d \n", sizeof(**s));	// 16
    printf("%d \n", sizeof(***s));	// 4
	return 0;
}
```

需要注意`sizeof(s)`表示这是一个指针变量，大小为4，而`sizeof(***s)`表示这是一个int型数据，这两个并不是同样的含义。

经过以上内容的学习，想必对于以下内容已经胸有成竹了吧。

```c
int main()
{
	int dx[5][2] = { 1,2,3,4,5,6,7,8,9,10 };
	int(*s)[2] = &dx[1];
	int* p = dx[1];
	printf("%d \n", s[1][3]);
	printf("%d \n", p[3]);
	return 0;
}
```

这里的printf会打印哪两个数字呢？

![image-20260215210435109](https://raw.githubusercontent.com/shanchuann/TheGitbookLibrary/main/C%E8%AF%AD%E8%A8%80%E5%9C%A3%E7%BB%8F/.gitbook/assets/book-images/typora/image-20260215210435109.png)

可以看见，本书将下标访问转化为指针访问的形式之后，第一个数字将输出8。

`dx[1]`等同于`*(dx + 1)`，也就是`p`指向的是3的地址，而p[3]就是*(p + 3)，p是一个指向整型的指针，因此p将偏移3个整型的偏移量指向6。

这里需要提及的是：

```c
int main()
{
	char dxstr[3][10] = { "shanchuan","hello","c" }; //1
	char* pstr[3] = { "shanchuan","hello","c" };	 //2
	return 0;
}
```

这是两个不同的概念，1是常规的二维数组，在栈区开辟30字节的空间后进行部分初始化，其余补0，2存储的是指向字符串常量的指针，只开辟12字节大小，为了保证能编译通过，最好加上const来保证不能改变其内容。
