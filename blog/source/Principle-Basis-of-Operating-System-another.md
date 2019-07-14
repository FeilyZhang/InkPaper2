title: "计算机操作系统原理总结(二)——存储管理"
date: 2019-07-06 08:00:14 +0800
update: 2019-07-06 08:00:14 +0800
author: me
cover: "-/images/os.jpg"
tags:
    - OS
preview: 地址重定位、存储保护与存储共享、连续存储管理（分区存储管理）、分页存储管理、分段存储管理、虚拟内存管理——请求分页存储管理。

---

## 一、地址重定位、存储保护与存储共享

#### 1.1 逻辑地址与物理地址、地址重定位

+ 逻辑地址：是与程序在内存中的物理位置无关的访问地址。在执行对内存的访问之前，必须把逻辑地址转化为物理地址；
+ 相对地址：是逻辑地址的一个特例，是相对于已知点的存储单元；
+ 物理地址：是程序运行时中央处理器实际访问的内存单元地址；

地址重定位或地址变换：在执行程序时，将其中的逻辑地址变为物理地址的过程。分为两种，如下：

+ 静态重定位：是指在程序装入内存时一次性将程序中所有的逻辑地址全部转化为物理地址，然后程序开始执行。优点是无须硬件支持易于实现，缺点是不允许程序在内存中移动位置，这就给内存碎片的合并带来挑战；
+ 动态重定位：地址转换工作穿插在指令执行过程中，每执行一条指令，CPU对指令中涉及的逻辑地址进行转换。优点是允许程序在内存中移动位置，缺点是必须借助硬件地址转换机构来实现。其地址转换公式为：物理地址 = 逻辑地址 + 内存始址。

#### 1.2 存储保护

目的是访问访问地址越界和控制正确存取。每道程序的地址空间限定了自己的合法访问范围，若无特别许可，则一个进程也不能访问其他进程的数据区。越界保护依赖于硬件设施，常用的有界地址与存储键。

进程访问分配给自己的主存区时，要对访问权限进行检查，从而确保数据的安全性与完整性，防止有意或无意的误操作而破坏主存信息，这就是信息存取保护。

#### 1.3 存储共享

当多个进程执行一个程序，或者多个进程合作完成同一个任务需要访问相同的数据结构时，内存管理系统都要提供存储共享机制，以对内存共享区域进行受控访问。

## 二、连续存储管理（分区存储管理）

连续存储管理对每个进程分配一个连续的存储区域，连续存储管理分为固定分区存储管理和可变分区存储管理。

#### 2.1 固定分区存储管理

固定分区存储管理将内存空间划分为若干个位置和大小固定的连续区域，每一个连续区域称为一个分区，各分区大小可以相同，也可以不同。该存储管理方案存在以下缺点：

+ 分区数目和大小在系统启动阶段就已经确定，限制了系统中活动进程的数目，也限制了当前分区方案下可运行的最大进程；
+ 当分区长度大于其中进程长度时，造成存储空间浪费。由于进程所在分区大于进程大小而造成的分区内部浪费部分称为内部碎片。

固定分区存储管理通过内存分配表完成对存储的管理，各项分别为分区号、起始地址、长度以及占用标志，进程申请内存时需要在该表上进行登记然后在运行时或者装入内存前需要进程地址重定位。

该种存储管理模式下，作业的排队策略为：

+ 将每个进程分配到能够容纳它的最小分区中；
+ 把每个分区的集合看作一种共享资源，设置一个队列，无空闲分区可用的所有进程排成一个队列，每当需要装入进程到内存中时，选择可以容纳该进程的最小可用分区，如果所有分区均被占据，则进行交换。

#### 2.2 可变分区存储管理

相对于固定分区存储管理而言，可变分区存储管理指的是分区的位置和大小是动态的。缺点是随着时间的推移，内存中会产生许多小到难以利用的分区，称之为外部碎片。克服外部碎片的技术是压缩：操作系统移动进程，将空闲分区连成一片。但是压缩非常耗时，浪费处理器时间，而且系统需要具备动态重定位能力。

可变分区存储管理通过分区表来管理内存，分区表分为已分配分区表与空闲分区表。作业的装入与撤销过程如下

+ 装入新作业时，从空闲分区表中找出足够容纳它的空闲区，将该区一分为二，一部分用来装入作业称为已分配区，并将其大小和起始地址登记在已分配分区表中；另一部分作为空闲分区，修改空闲分区表中原分区的大小和起始地址；
+ 作业运行完撤离时，将作业所在分区作为空闲区登记在空闲分区表中，并考虑该空闲分区与相邻分区的合并问题，同时从已分配表中删除该区对应的表项。

可变分区的分配算法如下：

+ 最先适应分配算法：从链首顺序查找，找到第一个满足要求的分区即可开始分配；
+ 下次适应分配算法：每次不从链首顺序查找，而是上次找到的空闲分区的下一个空闲分区开始查找；
+ 最优适应分配算法：每次从链首查找，直到找到一个能够满足要求的最小分区为止；
+ 最坏适应分配算法：扫描整个空闲分区表或空闲分区链，总是挑选一个最大的空闲分区分割给作业使用；
+ 快速适应分配算法：为那些经常用到的长度的空闲区设立单独的空闲分区链表。

可变分区存储管理可以采用静态或动态重定位实施地址变换。静态重定位时，由加载程序检查地址是否越界；动态地址重定位需要硬件支持。

## 三、分页存储管理

允许每个进程占用多个位置不相邻的物理内存区，避免了内存块的合并和内存作业的移动。

该种存储管理方案下，将全部内存划分为长度相等的若干份，每一份称为一个物理块或者页框。作业也自动被系统划分为与每个物理块相等的若干份，每一份称为一页。

分页存储管理通过页表来管理内存，当进程装入内存时，进程的每一页会与内存中的每一个物理块相对应，从而可以整理出一个页表，页表每个进程唯一。分页存储管理采用动态重定位。当CPU访问内存时将访存逻辑地址（该逻辑地址由页号与页内偏移组成）的页号根据重定位寄存器载入的页表转化为对应的物理块号，然后加上页内偏移，从而实现访问主存。

页表放在内存中降低了程序执行速度，CPU每执行一个指令/数据，就需要两次访问内存，第一次访问内存取得物理块号以形成物理地址，第二次根据物理地址存取指令/数据，速度降低了一半。为了提高速度，在存储管理部件中增设了一个专用的高速缓冲存储器，用来存放最近访问过的部分页表项，称之为快表。

## 四、分段存储管理

以程序段为单位进行内存分配，以段表来进行地址映射，由于程序段的大小不一，那么不能通过分页式存储管理的页表结构来映射地址，段表的结构如下（示例）

段号 | 长度 | 段起始地址
:-: |:-: |:-: 
0 | k | 3200
1 | p | 1500
2 | l | 6000
3 | n | 8000
4 | s | 5000

当CPU访存时，根据程序段号从段表中找到对应的段起始地址，然后加上段内偏移形成物理地址。长度起到了越界保护的作用。

## 五、虚拟内存管理——请求分页存储管理

在进程开始运行之前，不是装入全部的页面，而是装入一个或几个页面，当进程运行过程中，访问的页面不存在时，再装入所需页面；若内存空间已满，而又需要装入新的页面，则根据某种算法淘汰某个页面，以便装入新的页面。因此，请求分页系统的页表机制需要记住页面是否在内存中，若不在内存中，则需记住位于外存的位置。

请求分页的内存管理由外页表和内页表组成，外页表是页面与磁盘物理地址的对应表，由操作系统管理，进程启动前，系统为其建立外页表，并把进程程序页面装入外存，该表按进程也好的程序排列。为节省主存，外页表可以放在磁盘中，当发生缺页中断时再调入内存，结构如下

页号 | 外存地址
:-: | :-: 

可见，缺哪一页可以直接根据对应的外存地址调入主存。

而内页表的结构如下

页号 | 物理块号 | 驻留标志位 | 引用位 | 修改位 | 访问权限位
:-: | :-: | :-: | :-: | :-: | :-: 

+ 驻留标志位：指示页面是否在内存中；
+ 引用位：指示页面最近是否被访问过，以便页面淘汰；
+ 修改位：指示页面是否最近被修改过，以决定页面调出内存时是否写回外存；
+ 访问权限位：规定页面的访问权限。

**可见，内页表是外存具体页面与主存页面的对应关系，访问外存的地址在当前页存在的情况下需要通过内页表映射物理块号然后根据页内偏移访存，而其余字段是对该页的控制，当当前页不存在时，则根据外页表查找对应的磁盘位置，然后调入。**

页面替换算法包括如下几种：

+ 最佳页面淘汰算法：所淘汰的页是以后不再访问或距离现在最长时间后再访问的页；
+ 先进先出页面淘汰算法；
+ 最近最久使用页面淘汰算法；
+ 第二次机会页面淘汰算法；
+ 时钟页面替换算法。

> 参考《操作系统原理与Linux实践教程》 / 申丰山 王黎明 编著