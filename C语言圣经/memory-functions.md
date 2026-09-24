---
description: memset、memcmp、memcpy 和 memmove。
icon: code
---

# 内存操作函数

### 自定义`memset`

之前学习了无类型指针，在此处我们实现自定义`memset`时会用到。

```c
struct Student {
	char name[50];
	int age;
};

void mymemset(void* ptr, int value, size_t num) {
	unsigned char* p = (unsigned char*)ptr;
	for (size_t i = 0; i < num; i++) {
		p[i] = (unsigned char)value;
	}
}
```

```c
int main() {
	int iArr[5];
	double dArr[5];
	struct Student student;
	mymemset(iArr, 0, sizeof(iArr));
	mymemset(dArr, 0, sizeof(dArr));
	mymemset(&student, 0, sizeof(student));
	printf("Integer Array: ");
	for (int i = 0; i < 5; i++) {
		printf("%d ", iArr[i]);
	}
	printf("\nDouble Array: ");
	for (int i = 0; i < 5; i++) {
		printf("%.1f ", dArr[i]);
	}
	printf("\nStudent Name: '%s', Age: %d\n", student.name, student.age);
	return 0;
}
```

```
Integer Array: 0 0 0 0 0
Double Array: 0.0 0.0 0.0 0.0 0.0
Student Name: '', Age: 0
```

很容易的实现了一个根据不同类型来设置内存的自定义`memset`。

使用无类型指针的好处在于我们仅仅编写一个函数，就可以处理大部分的内存初始化。

> `memset`应用于字符数组的操作(`void *memset( void *dest, int ch, size_t count );`)，将值 `(unsigned char)ch` 复制到 `dest` 指向的对象的第一个 `count` 个字符中的每个字符。如果访问超出目标数组的末尾，则行为是未定义的。如果 `dest` 是空指针，则行为是未定义的。我们通过在这里类比字符串数组操作函数来编写内存初始化函数，显而易见的是，`value`仅仅能设置为0，若设置为其他值，将会导致不可预估的输出。

### `memcmp`

`int memcmp( const void* lhs, const void* rhs, size_t count );`

该函数比较 `lhs` 和 `rhs` 指向对象的首 `count` 个字节。比较按字典序进行。结果的符号是比较对象中第一对不同字节（均被解释为 `unsigned char`）值的差的符号。如果访问超出 `lhs` 和 `rhs` 指向的任一对象的末尾，则行为未定义。如果 `lhs` 或 `rhs` 是空指针，则行为未定义。

形参`lhs`, `rhs` 指向要比较的对象的指针，`count`表示要检查的字节数。如果 `lhs` 在字典序上出现在 `rhs` 之前，则返回负值；如果 `lhs` 和 `rhs `比较相等，或者 `count `为零，则返回零；如果 `lhs`在字典序上出现在 `rhs `之后，则返回正值。

```c
int main() {
	char arr1[] = "abcdef";
	char arr2[] = "abcdeF";
	int result = memcmp(arr1, arr2, 6);
	if (result < 0) printf("arr1 is less than arr2\n");
	else if (result > 0) printf("arr1 is greater than arr2\n");
	else printf("arr1 is equal to arr2\n");
	return 0;
}
```

### 自定义`memcmp`

```c
int mymemcmp(const void* ptr1, const void* ptr2, size_t num) {
	assert(ptr1 != NULL && ptr2 != NULL);
	if (num == 0) return 0;
	const unsigned char* p1 = (const unsigned char*)ptr1;
	const unsigned char* p2 = (const unsigned char*)ptr2;
	for (size_t i = 0; i < num; i++) {
		if (p1[i] != p2[i]) {
			return p1[i] - p2[i];
		}
	}
	return 0;
}
```

与库函数 `memcmp` 一样，自定义版本通过返回值的正负表示两段数据的先后关系；这里把返回值明确为首个不同字节的差值，读起来更直观。

注意：即使比较的两个内容完全一样，也仍然有可能出现返回结果和预期不符的情况，请看以下代码：

```c
struct Student {
	char a;
	int num;
	char b;
};

int main() {
	struct Student s1;
	struct Student s2;
	s1.a = 'A';
	s1.num = 100;
	s1.b = 'B';
	s2.a = 'A';
	s2.num = 100;
	s2.b = 'B';
	int result = mymemcmp(&s1, &s2, sizeof(s1));
	printf("Comparison result: %d\n", result);
}
#if 0
int main() {
	int arr1[] = { 1,4,3,4,5 };
	int arr2[] = { 1,2,3,4,5 };
	printf("%d\n", mymemcmp(arr1, arr2, sizeof(arr1)));
}
```

比较函数不做改变，当我们运行代码，结果打印为0，这没有问题。倘若我将s1定义在全局呢，会发生什么？

```
Comparison result: -204
```

结果不在是预料之中的0，反而是令人摸不着头脑的数字，这是为什么？

C 语言编译器为了提升内存访问效率，会对结构体进行**内存对齐**—— 在结构体成员之间或末尾填充额外的空字节（padding bytes），这些字节并不存储结构体的有效成员数据，但会占用内存空间。

以`Student`结构体为例（以常见的 32/64 位系统、默认对齐数为 4 为例），其内存布局如下：

```c
struct Student {
    char a;      // 占1字节（偏移0）
    // 填充3字节（偏移1-3）：因为int需要对齐到4字节边界
    int num;     // 占4字节（偏移4-7）
    char b;      // 占1字节（偏移8）
    // 填充3字节（偏移9-11）：结构体整体需对齐到最大成员（int）的大小（4）
};
// 总大小：1（a）+3（填充）+4（num）+1（b）+3（填充）= 12字节
```

也就是说，这个结构体实际占用 12 字节，但只有 3 个字节是有效成员（a、num 的 4 字节、b），剩下的 8 字节都是**填充字节**（无效但占内存）。

在不同存储区域的变量，未显式初始化的字节（包括填充字节）值不同：

- **全局变量**：存储在静态存储区，编译器会自动将未显式初始化的字节（包括填充字节）初始化为`0`。
- **局部变量**：存储在栈区，未显式初始化的字节是栈上的随机垃圾值（残留数据）。

`mymemcmp`会逐字节比较两个内存块的**全部字节**（包括填充字节），当`s1`是全局、`s2`是局部时，虽然`a`、`num`、`b`的有效字节值完全相同，但填充字节的值不同（s1 的填充是 0，s2 的填充是随机值），这就导致了函数认定为两个结构体并不相同，从而返回填充值之差，也就是这里看到的204。

> 如果想比较两个结构体是否 “逻辑上相等”，**不能直接比较内存块**，而应该逐个比较结构体的有效成员。

### `memcpy`

`void* memcpy( void *dest, const void *src, size_t count );`

函数从 `src` 指向的对象复制 `count` 个字符到 `dest` 指向的对象。两个对象都被解释为 unsigned char 数组。如果访问超出 `dest` 数组的末尾，或 `dest` 或 `src` 是无效或空指针，则行为未定义。

其中dest指向要复制到的对象的指针，src指向要从中复制的对象的指针，count表示要复制的字节数。

函数返回 `dest` 的副本。成功时返回零，错误时返回非零值。

```c
int main() {
	char arr1[] = "abcdef";
	char arr2[] = "xxxxxx";
	printf("Before memcpy, arr2: %s\n", arr2);
	memcpy(arr2, arr1, 6);
	printf("After memcpy, arr2: %s\n", arr2);
	return 0;
}
```

```
Before memcpy, arr2: xxxxxx
After memcpy, arr2: abcdef
```

### 自定义`memcpy`

```c
void* mymemcpy(void* destination, const void* source, size_t num) {
	assert(destination != NULL && source != NULL);
	unsigned char* dest = (unsigned char*)destination;
	const unsigned char* src = (const unsigned char*)source;
	for (size_t i = 0; i < num; i++) {
		dest[i] = src[i];
	}
	return destination;
}
```

### `memmove`

`void* memmove( void* dest, const void* src, size_t count );`

函数从 `src` 指向的对象复制 `count` 个字符到 `dest` 指向的对象。两个对象都被解释为 `unsigned char` 数组。对象可以重叠：复制发生的方式，就好像字符被复制到一个临时字符数组，然后字符从该数组复制到 `dest`。如果访问超出 `dest` 数组的末尾，则行为未定义。如果 `dest` 或 `src` 是无效指针或空指针，则行为未定义。

形参dest指向要复制到的对象的指针，src 指向要从中复制的对象的指针，count表示要复制的字节数。函数返回 dest 的副本。成功时返回零，错误时返回非零值。

相比于memcpy，menmove具有自拷贝的能力，例如：

假设有一个数组 `int arr[] = {1,2,3,4,5}`，我们想把前 3 个元素（1,2,3）拷贝到从第 2 个位置开始的区域（原本的 2,3,4 位置），期望得到 `{1,1,2,3,5}`：

- 用`memcpy`：会先把`arr[0]`（1）拷贝到`arr[1]`，此时数组变成`{1,1,3,4,5}`；接着拷贝`arr[1]`（已经被改成 1 了）到`arr[2]`，数组变成`{1,1,1,4,5}`；最后拷贝`arr[2]`（1）到`arr[3]`，最终得到错误结果`{1,1,1,4,5}`。
- 用`memmove`：会先判断 “目标地址在源地址后面且重叠”，于是**从后往前拷贝**：先拷贝`arr[2]`（3）到`arr[3]`，再拷贝`arr[1]`（2）到`arr[2]`，最后拷贝`arr[0]`（1）到`arr[1]`，最终得到正确结果`{1,1,2,3,5}`。

不过现在无论是memcpy还是memmove，进行自拷贝后得出的均为正确的结果：

```c
int main() {
	// 测试自拷贝（内存重叠）
	int arr_memcpy[5] = { 1,2,3,4,5 };
	int arr_memmove[5] = { 1,2,3,4,5 };
	printf("原始数组：");
	for (int i = 0; i < 5; i++) printf("%d ", arr_memcpy[i]); // 1 2 3 4 5
	printf("\n自拷贝：把前3个元素拷贝到从第2个位置开始\n");
	// memcpy自拷贝（结果错误）
	memcpy(arr_memcpy + 1, arr_memcpy, 3 * sizeof(int));
	printf("memcpy自拷贝：");
	for (int i = 0; i < 5; i++) printf("%d ", arr_memcpy[i]); // 1 1 1 4 5
	// memmove自拷贝（结果正确）
	memmove(arr_memmove + 1, arr_memmove, 3 * sizeof(int));
	printf("\nmemmove自拷贝：");
	for (int i = 0; i < 5; i++) printf("%d ", arr_memmove[i]); // 1 1 2 3 5
	return 0;
}
```

```
原始数组：1 2 3 4 5
自拷贝：把前3个元素拷贝到从第2个位置开始
memcpy自拷贝：1 1 2 3 5
memmove自拷贝：1 1 2 3 5
```

因为编译器的优化，**即使标准规定`memcpy`不处理重叠，实际却和 `memmove` 一样支持安全拷贝**。

### 自定义`memmove`

```c
void* mymemmove(void* destination, const void* source, size_t num) {
	assert(destination != NULL && source != NULL);
	unsigned char* dest = (unsigned char*)destination;
	const unsigned char* src = (const unsigned char*)source;
	if (dest < src) {
		for (size_t i = 0; i < num; i++) {
			dest[i] = src[i];
		}
	} else if (dest > src) {
		for (size_t i = num; i > 0; i--) {
			dest[i - 1] = src[i - 1];
		}
	}
	return destination;
}
```
