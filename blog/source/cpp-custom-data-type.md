title: "C++自定义数据类型"
date: 2019-08-09 20:29:42 +0800
update: 2019-08-09 20:29:42 +0800
author: me
cover: "-/images/c++.jpg"
tags:
    - C++
preview: C++自定义数据类型包括数组、结构体、共用体、枚举。写C/C++不用指针是没有灵魂的！

---

## 一、数组

C++数组的定义必须指明数组长度，通用格式如下

```
typeName arrayName[arraySize];
```

一维数组与二维数组的示例如下：

```
#include <iostream>

int main() {
	int arr[5];
	int arr_length = sizeof arr / sizeof arr[0];
	for (int i = 0; i < arr_length; i++) {
		arr[i] = i * i;
	}
	int index = 0;
	int* pArr = arr;
	while (index != arr_length) {
		std::cout << *(pArr + index) << "\t";
		++index;
	}
	int arrt[5][4];
	int arr_row_len = sizeof arrt / sizeof arrt[0];
	int arr_col_len = sizeof arrt[0] / sizeof arrt[0][0];
	for (int i = 0; i < arr_row_len; i++) {
		for (int j = 0; j < arr_col_len; j++) {
			arrt[i][j] = i * j;
		}
	}
	std::cout << std::endl;
	int(*pArrt)[4];
	pArrt = arrt;
	int row_index = 0, col_index = 0;
	for (int i = 0; i < arr_row_len; i++) {
		for (int j = 0; j < arr_col_len; j++) {
			std::cout << *(*(pArrt + i) + j) << "\t";
		}
		std::cout << std::endl;
	}
	std::cin.get();
	return 0;
}
```

输出为

```
0       1       4       9       16
0       0       0       0
0       1       2       3
0       2       4       6
0       3       6       9
0       4       8       12
```

上述程序分别演示了一维数组和二维数组，并且数组的输入是采用常规的方式，数组的输出是采用指针的方式。

一维数组的指针使用符合上文中的两个等式，如下

```
arr[i] = *(ar + i);
&arr[i] = ar + i; 
```

**而二维数组的指针则相对麻烦一点，解释如下**

一维数组中声明指针的方式如下

```
int* pArr = arr;
```

此时指针pArr中存储的是数组的首元素的地址，通过指针对元素的访问是加上元素偏移实现的，而指针的长度则可以防止访问越界。二维数组的指针我们换个方式看一下

仍然是基于上述程序，我们尝试输出一下pArrt

```
std::cout << pArrt;    // 0034FC2C
```

是一个地址，这很容易理解，毕竟指针存的就是地址，那么`*pArrt`以及`*(pArrt + 1)`呢，按照常理，如果pArrt是地址，那么pArrt + 1就是pArrt指针指向内存区的下一个内存区的地址，对二者取值必然是两个连续地址空间的值了，但是真的是这样吗？

```
std::cout << *pArrt << "\t" << *(pArrt + 1);    // 004BFB20        004BFB30
```

还是一个地址，但是观察一下这两个地址值相差16个字节，在本机int型实现为4个字节，那么按理来说每个int型元素只会占用4个字节，也就是`*(pArrt + 1)`应该是004BFB24，但是为什么是004BFB30呢？中间的12个字节去哪了？

不要忘了，我们二维数组的每列有4个int型元素，刚好16个字节，这样推论的话，就意味着`*pArrt`应该指的是二维数组第0行的首元素的地址，而`*(pArrt + 1)`指的就是二维数组第1行首元素的地址，因为中间12个字节刚好的第0行剩余三个元素的内存空间。那么相应的第i行的首元素的地址就是`*(pArrt + i)`。那么我们试一下第0行的值是否与上文程序输出一致



```
std::cout << *(*pArrt + 0) << "\t" << *(*pArrt + 1) << "\t" << *(*pArrt + 2) << "\t" << *(*pArrt + 3) << "\t";    // 0       0       0       0
```

完全一致！

那么我们再验证一下第0行的第四个元素的最后一个元素是否是第1行的首元素，如下

```
std::cout << *(pArrt + 0) << "\t" << *(pArrt + 1) << std::endl;
for (int i = 0; i < 5; i++) {
    std::cout << *(pArrt + 0) + i << "\t";
}
```

输出为

```
002EF700        002EF710
002EF700        002EF704        002EF708        002EF70C        002EF710
```

> 注意：由于每次运行都会产生不一样的结果，所以地址是有变化的。

观察一下输出结果的第一行和第二行的最后的地址，一模一样，这也就证明了`*(pArrt + 0)`(也就是`*(pArrt)`)存放的是第0行的首元素的地址，之后跟随的是第0行的剩余的三个元素，由于数组存储在连续地址空间，那么第1行的第一个元素必然紧接着第0行的最后一个元素，在上述代码中，也得到了证明。那么这里都得到了每个元素的地址，在取值一次不就得到了值了嘛！，对的


```
for (int i = 0; i < 5; i++) {
    std::cout << *(*(pArrt + 0) + i) << "\t";
}
```

输出为

```
0       0       0       0       0
```

没问题，综上，我们得出以下结论

+ `*(pArrt + i)`代表的是二维数组第i行的首元素的**地址（注意，这里是地址，不是值）**
+ `*(pArrt + i) + j`代表的就是二维数组第i行第j列的**地址（注意，这里是地址，不是值）**
+ `*(*(pArrt + i) + j)`代表的就是二维数组第i行第j列的**值（现在是值了，因为通过`*`再取值了一次）**

那么pArrt自然代表的就是`*pArrt`的地址，也就是地址的地址，即二维数组首行首元素的地址。

**还有，二维数组的指针这样声明**

```
int(*pArrt)[4];
```

为什么？不能是`int * ar2[4]`吗？

不行，因为`int * ar2[4]`代表的是组织了4个int型指针的数组，其本质是int型指针，而不是数组，数组只是将其组织了起来而已。而`int(*pArrt)[4]`则代表的是二维数组的一行，这一行由4个int型元素组成，其本质是数组，指针是指向该一维数组（就是二维数组的行）的首元素地址，即指向由4个int组成的数组的指针。

## 二、结构体

C++中结构体的定义如下所示

```
struct s_h {
	int id;
	char name[12];
	char sex[4];
};
```

结构体本身就是一种数据类型，因此上述定义算是一种数据类型的定义，那么声明该种数据类型的变量如下

```
s_h stu;
```

对其初始化的方式为

```
stu = { 0, "Feily Zhang", "man" };
```

当然也可以直接声明的时候就初始化

```
s_h stu = { 0, "Feily Zhang", "man" };
```

而C++11中支持将列表初始化用于结构，这种方式下等号是可选的，即

```
s_h stu { 0, "Feily Zhang", "man" };
```

当然，定义结构体的时候直接声明变量也是可以的，如下

```
struct s_h {
	int id;
	char name[12];
	char sex[4];
} stu;
```

在上述基础之上直接初始化也是可以的，如下

```
struct s_h {
	int id;
	char name[12];
	char sex[4];
} stu { 0, "Feily Zhang", "man" };
```

当然，也可以定义结构体数组，如下

```
s_h stu[3]{
    {0, "Feily Zhang", "man"},
    {1, "Itsing Lin", "man"},
    {2, "Me", "man"}
};
for (int i = 0; i < 3; i++) {
    std::cout << stu[i].id << "\t" << stu[i].name << "\t" << stu[i].sex << std::endl;
}
```

**指针对结构体的操作如下**

需要注意的是指针无法对结构体中的char数组赋值，char数组作为字符串来使用是可以定义的时候直接赋值的，但是单独无法赋值，只能一个一个字符赋值，也很容易理解，Java中数组也不可能直接赋值给数组名，解决办法之一是使用string替代char数组，如下

```

struct s_h {
	int id;
	std::string name;
	std::string sex;
};

void say() {
	using namespace std;
	std::cout << "hello, world!" << endl;
	s_h* pt = new s_h;
	pt->id = 0;
	pt->name = "Feily Zhang";
	pt->sex = "man";
    std:cout << pt->id << "\t" << pt->name << "\t" << pt->sex;
	delete pt;
	std::cin.get();
}
```

另一种解决办法就是使用`strcpy()`函数将字符串复制到char数组的地址上，如下

```
struct s_h {
	int id;
	char name[20];
	char sex[4];
};

void say() {
	using namespace std;
	std::cout << "hello, world!" << endl;
	s_h* pt = new s_h;
	pt->id = 0;
	strcpy(pt->name, "Feily Zhang");
	strcpy(pt->sex, "man");
    std:cout << pt->id << "\t" << pt->name << "\t" << pt->sex;
	delete pt;
	std::cin.get();
}
```


对于结构体数组仍然可以声明指针，此处不再赘述。

## 三、共用体

共用体中的诸数据成员共用一块数据空间。需要注意的是

1. 共用体中不能使用构造函数，即string等是无法使用的；
2. 不能对共用体变量名赋值，也不能在定义一个共用体变量名的时候对其赋值，也就意味着只能单独对内部成员赋值；
3. 不能把共用体当做函数参数也不能作为返回值返回，但是可以使用指向共用体变量的指针；
4. 共用体可以在结构体中定义，也可以定义共用体指针。

```
de <iostream>

union id {
	int id_num;
	int name_num;
};

int main() {
	id id1;
	id1.id_num = 4311;
	id1.name_num = 4312;
	id* pt = &id1;
	std::cout << pt->id_num << "\t" << id1.name_num;
	std::cin.get();
	return 0;
}
```

输出为

```
4312    4312
```

可见，不同的数据成员的值是一样的，也就证明了数据成员使用一块内存空间。

## 四、枚举

枚举是一种创建符号常量的方式，这种方式可以替代const。其通用声明格式如下

```
enum enumName {};
```

其中`{}`中为该枚举类型的所有可能的取值范围，为符号常量，并且编译器会其该符号常量指定一个整数值，缺省状态下从0开始，依次递增，当然也可以通过`=`修改。示例如下

```
<iostream>
 
enum spe {red, orange, yellow, green, blue};

int main() {
	spe s1, s2, s3;
	s1 = spe(2);
	s2 = red;
	s3 = s1;
	spe* s4;
	s4 = &s1;
	std::cout << (s1 == s2) << "\t" << (s1 == s3) << "\t" << *s4;
	std::cin.get();
	return 0;
}
```

输出为

```
0       1       2
```

## 五、string类

与Java中String类没什么区别，string对象的声明、初始化、赋值以及拼接操作如下所示

```
#include <iostream>
#include <string>

int main() {
	using namespace std;
	string s1 = "hello, world";
	cout << s1 << "\t" << s1.size() << endl;
	string s2 = s1;
	s2 += ", this is s2 not s1";
	cout << s2 << "\t" << s2.size() << endl;
	return 0;
}
```

输出为

```
hello, world    12
hello, world, this is s2 not s1 31
```

在C++新增string类之前，想要完成对字符串（或者说对于C风格的字符串）的赋值工作，必须借助于C语言库函数来完成，比如`strcpy(char[], char[])`和`strcat(char[], char[])`，但是在新增string类之后就不必了，很容易通过面向对象的方式来实现。

对比如下:

```
#include <iostream>
#include <string>

int main() {
	using namespace std;
	char c1[20];
	char c2[20] {"jaguar"};
	strcpy(c1, c2);
	cout << "now c1 is " << c1 << " and length is " << strlen(c1) << endl;
	strcat(strcat(c1, c2), "c1 + c2");
	cout << "now c1 is " << c1 << " and length is " << strlen(c1) << endl;
	string s1 = "hello, world";
	cout << s1 << "\t" << s1.size() << endl;
	string s2 = s1;
	s2 += ", this is s2 not s1";
	cout << s2 << "\t" << s2.size() << endl;
	return 0;
}
```

输出为

```
now c1 is jaguar and length is 6
now c1 is jaguarjaguarc1 + c2 and length is 19
hello, world    12
hello, world, this is s2 not s1 31
```

利用C语言库函数的缺点在于不安全，如果目标字符数组的大小不够，那么必然会产生数据错误，但是string类具有自动调整大小的功能，更加的安全。

## 六、模板类vector与array

模板类vector类似于string，也是一种动态数组，可以在运行阶段设置vector对象的长度，也可以在末尾和中间插入数据。基本上，它是利用new创建动态数组的替代品（前文的`getName()`函数），实际上，vector类确实使用new和delete来管理内存。

对于vector的使用，第一其包含在std命名空间中；第二，在声明时要指出数据类型。如下

```
#include <iostream>
#include <string>
#include <vector>

int main() {
	using namespace std;
	vector<string> v1;
	v1.resize(2);
	v1[0] = "Feily Zhang";
	v1[1] = "Itsing Lin";
	/*
	v1.push_back("Feily Zhang");
	v1.push_back("Itsing Lin");*/
	for (int i = 0; i < v1.size(); i++) {
		cout << v1[i] << endl;
	}

	vector<string> v2(2);
	v2[0] = "Feily Zhang";
	v2[1] = "Itsing Lin";
	for (int i = 0; i < v2.size(); i++) {
		cout << v2[i] << endl;
	}
	return 0;
}
```

有两种使用方式，第一种是直接指明长度，另一种是通过`.resize()`方法来指明长度，vector调整长度也是通过这个方法，如下

```
#include <iostream>
#include <string>
#include <vector>

int main() {
	using namespace std;
	vector<string> v1;
	v1.resize(2);
	v1[0] = "Feily Zhang";
	v1[1] = "Itsing Lin";
	/*
	v1.push_back("Feily Zhang");
	v1.push_back("Itsing Lin");*/
	for (int i = 0; i < v1.size(); i++) {
		cout << v1[i] << endl;
	}

	vector<string> v2(2);
	v2[0] = "Feily Zhang";
	v2[1] = "Itsing Lin";
	for (int i = 0; i < v2.size(); i++) {
		cout << v2[i] << endl;
	}

	v1.resize(4);
	for (int i = 0; i < v1.size(); i++) {
		cout << i << " = " << v1[i] << endl;
	}
	return 0;
}
```

输出为

```
Feily Zhang
Itsing Lin
Feily Zhang
Itsing Lin
0 = Feily Zhang
1 = Itsing Lin
2 =
3 =
```

另外，也可以使用`.push_back()`方法来添加元素。


vector的功能比数组强大，比如说可以动态扩容，但是付出的代价是效率稍低。如果需要的是固定长度的数组，使用数组无疑是最方便且高效的，但是却并不安全。C++11新增的模板类array，其对象长度固定，也使用栈（静态内存分配）而不是自由存储区，因此效率与数组相同，且更加安全.

array同样位于std命名空间，使用时需要指明长度和数据类型，用法如下

```
#include <iostream>
#include <string>
#include <array>

int main() {
	using namespace std;
	array<string, 2> a1 {"Feily Zhang", "Itsing Lin"};
	for (int i = 0; i < a1.size(); i++) {
		cout << i << " = " << a1[i] << endl;
	}
	array<string, 2> a2;
	a2[0] = "hello, world";
	a2[1] = "world, hello";
	for (int i = 0; i < a2.size(); i++) {
		cout << i << " = " << a2[i] << endl;
	}
	return 0;
}
```

输出为

```
0 = Feily Zhang
1 = Itsing Lin
0 = hello, world
1 = world, hello
```