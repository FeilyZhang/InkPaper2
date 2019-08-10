title: "函数关于指针"
date: 2019-08-10 19:48:42 +0800
update: 2019-08-10 19:48:42 +0800
author: me
cover: "-/images/c++.jpg"
tags:
    - C++
preview: 指针在函数中的应用。

---

## 一、函数和一维数组

有如下函数

```
int sum_arr(int arr[], int n);

int sum_arr(int arr[], int n) {
	int total = 0;
	for (int i = 0; i < n; i ++) {
		total += arr[i];
	}
	return total;
}
```

其中函数`sum_arr(int arr[], int n)`接受一个数组，由于数组名代表的是数组首元素地址，也就是说这个函数实质上接受的是一个指针，正确的函数头应该是`int sum_arr(int * arr, int n)`，那么正确的函数定义就是

```

int sum_arr(int * arr, int n);

int sum_arr(int* arr, int n) {
	int total = 0;
	for (int i = 0; i < n; i ++) {
		total += arr[i];
	}
	return total;
}
```

指向数组的指针访问数组元素仍然可以使用下标方式，指针方式也是适用的，如下

```
int sum_arr(int * arr, int n);

int sum_arr(int* arr, int n) {
	int total = 0;
	for (int i = 0; i < n; i ++) {
		total += *(arr + i);
	}
	return total;
}
```

调用该函数直接将数组传入即可，因为归根结底依旧是传入地址，如下

```
int sum_arr(int * arr, int n);

int main() {
	using namespace std;
	int arr[] {5, 4, 3, 2, 1};
	int len = sizeof arr / sizeof arr[0];
	cout << "array's sum is " << sum_arr(arr, len);
	return 0;
}

int sum_arr(int* arr, int n) {
	int total = 0;
	for (int i = 0; i < n; i ++) {
		total += *(arr + i);
	}
	return total;
}
```

## 二、函数与二维数组

依旧传的是地址，由于之前解释过二维数组指针，所以此处不赘述，直接上代码，如下

```
#include <iostream>

int sum_arr(int (*arr)[4], int n);

int main() {
	using namespace std;
	int arr[2][4] {{4, 3, 2, 1}, {1, 2, 3, 4}};
	int len = sizeof arr / sizeof arr[0];
	cout << "array's sum is " << sum_arr(arr, len);
	return 0;
}

int sum_arr(int(*arr)[4], int n) {
	int total = 0;
	for (int i = 0; i < n; i ++) {
		for (int j = 0; j < 4; j++) {
			total += *(*(arr + i) + j);
		}
	}
	return total;
}
```

输出为

```
array's sum is 20
```

## 三、函数与指针

上述函数依旧可以获取地址然后交给一个指针，这样就可以通过指针来引用该函数。

### 3.1 函数指针的声明

函数指针的声明同函数的声明一样，必须指定返回值类型和参数列表，，只不过把函数名变成了指向函数的指针，也就是`*XXX`的形式，如下声明一个函数和与函数同类型的指针，只不过是还没有指向该函数

```
int sum_arr(int (*arr)[4], int n);

int (*p) (int(*arr)[4], int n)
```

可见，只不过是函数名换成了指针，通过获取函数的地址将其赋给指针就实现了指针向该函数的指向。

### 3.2 获取函数地址并赋值给指针并调用函数

函数名就代表函数的地址，注意仅仅是函数名，不包括参数列表，赋值如下

```
int (*p) (int(*arr)[4], int n) = sum_arr;
```

这样，指针p就指向了函数，也就可以通过指针p来调用函数。

由于函数名就是地址，那么每次函数调用也就是通过函数地址来调用函数的，可以理解为该地址就是函数的入口地址，只需要将参数列表传入即可。那么现在指针p指向函数地址，也就是指针内存中存储的就是函数的入口地址，那么通过`*`取值就可以得到函数的地址，也就意味着，通过指针调用函数就是如下形式

```
(*p)(arr, len)
```

等同于

```
sum_arr(arr, len)
```

很明显，指针p的取值也就是指针p内存空间中存储的就是函数地址，也就是(*p)就是sum_arr。完整的示例如下

```
#include <iostream>

int sum_arr(int (*arr)[4], int n);

int main() {
	using namespace std;
	int arr[2][4]{ {4, 3, 2, 1}, {1, 2, 3, 4}};
	int len = sizeof arr / sizeof arr[0];
	int (*p) (int(*arr)[4], int n) = sum_arr;
	cout << "array's sum is " << (*p)(arr, len);
	return 0;
}

int sum_arr(int(*arr)[4], int n) {
	int total = 0;
	for (int i = 0; i < n; i ++) {
		for (int j = 0; j < 4; j++) {
			total += *(*(arr + i) + j);
		}
	}
	return total;
}
```

### 3.3 auto关键字

也可以使用C++11的新特性，auto关键字来自动推导类型，那么下述两条语句是等价的

```
int (*p) (int(*arr)[4], int n) = sum_arr;
auto p2 = sum_arr;
```

也就是p2中存放的还是sum_arr的地址，自然而言，对p2取值就可以获取sum_arr的地址也就意味着可以调用函数sum_arr，如下依旧等价

```
cout << "array's sum is " << (*p)(arr, len);
cout << "array's sum is " << (*p2)(arr, len);
```

可见，auto关键字在这里不过是让声明指向函数的指针变的简单而优雅了许多。

### 3.4 typedef关键字

typedef关键字可以定义类型的别名，如下

```
#include <iostream>
#include <string>

typedef struct s_h {
	int id;
	std::string name;
} stu;

int main() {
	using namespace std;
	typedef double dou;
	dou d = 100.0;
	stu student = { 1, "Feily Zhang" };
	cout << "d = " << d << endl;
	cout << "student = " << student.id << "  " << student.name << endl;
	return 0;
}
```