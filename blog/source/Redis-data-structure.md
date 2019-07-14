title: "Redis数据结构"
date: 2019-07-07 09:53:49 +0800
update: 2019-07-07 09:53:49 +0800
author: me
cover: "-/images/redis.jpg"
tags:
    - Redis
preview: 简单动态字符串(SDS的结构、SDS与C字符串的区别)、链表(链表与链表节点的实现、Redis链表的特性)。

---

## 一、简单动态字符串（SDS）

Redis没有直接使用C语言传统的字符串表示（即以空字符`\0`结尾的字符数组），而是直接自己构建了一种称为简单动态字符串的抽象类型，并作为Redis中的默认字符串表示。

当Redis需要的不仅仅是一个字符串字面量，而是一个可以被修改的字符串值时，Redis就会使用SDS来表示字符串值，比如客户端执行如下命令

```
redis> SET msg "hello, world"
OK
```

那么Redis就会在底层创建一个新的键值对，包含：

+ 键值对的键是一个字符串对象，对象的底层实现是一个保存着字符串“msg”的SDS;
+ 键值对的值也是一个字符串对象，对象的底层实现是一个保存着字符串“hello, world”的SDS。

#### 1.1 SDS的结构（或实现）

SDS通过一个结构体来实现，每个`sds.h/sdshdr`结构表示一个SDS值，如下

```
struct sdshdr {
    int len;
    int free;
    char buf[];
}
```

各属性解释如下：

+ `len`：记录buf数组中已经使用的字节的数量；
+ `free`：记录buf数组中未使用的字节数量；
+ `buf[]`：字节**数组**，用于保存字符串。

由于SDS遵循了C语言字符串以`\0`结尾的惯例，那么buf数组的实际长度就是` len + free + 1`个字节，加一是因为`\0`占一个字节。更直观的看，一个存放了字符串`Redis`的SDS结构如下

![SDS结构](/images/article/redis-sds.png)

#### 1.2 SDS与C字符串的区别

1. **常数复杂度获取字符串长度**：由于SDS结构内维护了一个`len`属性，因此可以直接获取字符串长度；但是C字符串则要从头到尾遍历才可以获得字符串长度；获取字符串长度的时间复杂度从O(N)降低到了0(1);
2. **杜绝缓冲区溢出**：C字符串不记录自身长度，在执行字符串拼接操作时它假定用户已经分配好了足够的空间，但是当这个假设不成立时，拼接字符串后形成的新字符串就会溢出到当前字符串之外的空间；而SDS对字符串进行修改（拼接是修改的一种）时，则会检查`free`属性的值是否能够容纳要拼接的字符串，如果能够容纳则直接拼接，如果不够则先扩容再拼接；
3. **减少修改字符串时带来的内存重分配次数**：在拼接操作时，如果`free`属性指示的剩余字节空间不够，那么要先进行内存重分配操作然后再执行拼接操作；在缩短字符串时，也要通过内存重分配来释放不用的那一部分空间；对应这两种情况，分别有**两种内存重分配策略**：
    1. 空间预分配：针对字符串增长操作时剩余空间可能不够的情况，SDS采用空间预分配策略来保证：当SDS的API对一个SDS进行修改并且需要对SDS空间进行拓展的时候，程序不仅会为SDS分配修改所必需的空间，还会为SDS分配额外的未使用空间，交由`free`属性指示，具体的额外分配的未使用空间由以下公式决定：
        + 如果对SDS进行修改后，SDS的长度（即`len`属性）小于1MB，那么程序分配与`len`属性同样大小的未使用空间。也就是说如果当前`len`属性为13byte，那么空间预分配后SDS的buf数组的实际长度就是`13byte + 13byte + 1byte`（额外的一个字节用于保存空字符）；
        + 如果对SDS进行修改后，SDS的长度大于等于1MB，那么程序会分配1MB的未使用空间。也就是说如果当前`len`属性值为30MB，那么空间预分配后SDS的buf数组的实际长度就是`30MB + 1MB + 1byte`（额外的一个字节用于保存空字符）。
    2. 惰性空间释放：针对字符串的缩短操作：当SDS的一个API对其进行缩短操作时，程序不立即使用内存重分配来回收缩短后多出来的字节，而是使用`free`属性将这些字节记录起来，以备未来使用。
4. **二进制安全**：C字符串中的字符必须符合某种编码（比如ASCII码），并且除了字符串的末尾之外，字符串里面不能包含空字符（即`\0`），但是SDS的buf属性保存的实际上是二进制数据，所以可以保存任意格式的数据，因为SDS是用`len`属性的值而不是`\0`来判断字符串结尾的，之所以保留字符串末尾的`\0`是因为这样做可以兼容一部分C语言字符串函数；
5. **兼容部分C字符串函数**：SDS遵循C字符串以空字符结尾的惯例，可以重用部分C字符串函数。

## 二、链表

链表提供了高效的节点重排能力，以及顺序性的节点访问方式，并且可以通过增删节点来灵活地调整链表的长度。

在Redis中，当一个列表键包含了数量比较多的元素或者列表中包含的元素都是比较长的字符串时，Redis就会使用链表作为列表键的底层实现。

#### 2.1 链表与链表节点的实现

每个链表由一个节点构成，使用`adlist.h/listNode`结构来表示：

```
typedef struct listNode {
    struct listNode *prev;
    struct listNode *next;
    void *value;
} listNode;
```

各属性解释如下：

+ `*prev`：指针，指向链表中当前节点的上一个节点；
+ `*next`：指针，指向链表中当前节点的下一个节点；
+ `*value`：节点的实际数据。

多个节点可以连接成一个链表，可以交由list结构来维护，使用`adlist.h/list`结构来表示，如下

```
typedef struct list {
    listNode *head;
    listNode *tail;
    unsigned long len;
    void *(*dup) (void *ptr);
    void (*free) (void *ptr);
    int (*match) (void *ptr, void *key);
} list;
```

各属性解释如下：

+ `*head`：链表的头结点；
+ `*tail`：链表的尾结点；
+ `len`：链表的节点数目；
+ `*dup`：该函数对链表进行操作，用于复制链表节点所保存的值；
+ `*free`：该函数对链表进行操作，用于释放链表节点所保存的值；
+ `*match`：该函数对链表进行操作，用于对比链表节点所保存的值和另一个输入值是否相等。

#### 2.2 Redis链表的特性

+ 双端：链表节点带有prev和next指针；
+ 无环：表头节点的prev指针和表尾节点的next指针都指向NULL，对链表的访问以NULL为终点；
+ 带表头指针和表尾指针：获取链表头部节点与尾部节点的时间复杂度都是O(1)；
+ 带链表长度计数器，获取链表长度的时间复杂度为O(1)；
+ 多态：链表节点使用`void*`指针来保存节点值，并且可以通过list结构的`dup``free``match`三个属性为节点值设置类型特定函数，所以链表可以用于保存各种不同类型的值。