title: "Linux内核源码分析：从开机加电到磁盘引导程序的执行"
date: 2019-09-13 16:01:05 +0800
update: 2019-09-13 16:01:05 +0800
author: me
cover: "-/images/os.jpg"
tags:
    - Linux
preview: 常量与变量、常用数据类型。

---

## 引言

我们知道，计算机的运行主要是以RAM为对象的CPU运算，即程序执行的前提是被调入RAM当中。操作系统作为底层软件，理所当然地需要被加载至RAM中通过CPU运算构建整个操作系统体系以接管、分配和调度系统资源。

由于CPU无法直接与外设进行数据交互，那么位于软盘/硬盘中的操作系统程序就必须首先调入RAM中。但是现在RAM中空空如也，既没有数据，也没有指令，那么是谁通过CPU将操作系统程序调入RAM中的呢？

答案是`BIOS`！

## 一、BIOS的启动原理

既然是BIOS将操作系统程序调入RAM当中，那么问题来了，BIOS又是如何启动的？

答案是`0xFFFF0`！

PC机加电伊始，基于80x86结构的CPU将会自动进入实模式(实模式下CPU的地址总线为16根，其寻址能力为2<sup>16</sup>)，并且CPU将会强制将CS寄存器的值设置为`0xF000`、IP寄存器的值设置为`0xFFF0`，即CPU将会指向`0xFFFF0`地址处，这个地址通常是ROM-BIOS的入口地址(并不是BIOS程序的始址)，即BIOS程序的第一条指令就在这个位置。等等，不是说RAM中没有任何数据和指令吗？那么CPU指向这个位置不也是空的吗？有什么意义？要注意的是CPU此时指向的是由主存地址空间(RAM)、显存地址空间(RAM)、显卡BIOS地址空间(ROM)、网卡BIOS地址空间(ROM)、系统BIOS地址空间(ROM)等物理存储器编址而成的逻辑存储器，被称为内存地址空间(以下简称内存)。所以CPU此时指向的地址是装有系统BIOS程序的构成内存地址空间的ROM地址空间，而非空无一物的主存地址空间。BIOS程序被固化在ROM芯片上，由于ROM的非易失性使得其内部并非空无一物。

那么此刻CPU已经指向`0xFFFF0`的位置，意味着BIOS开始启动，紧接着将是BIOS程序在CPU中执行，屏幕上会显示出显卡信息、内存信息等，这说明CPU在进行自检，期间一项非常重要的工作就是BIOS在内存中构建中断向量表和中断处理程序。BIOS会在内存(主存RAM)的开始位置(`0x00000`)用1KB的内存空间(`0x00000` ～ `0x003FF`)构建BIOS中断向量表，在紧挨着它的位置用256字节分内存空间构建BIOS数据区(`0x00400` ～ `0x004FF`)，并在大约57KB以后的位置(`0x0E05B`)加载大约8KB的与中断向量表对应的若干中断处理程序。如下图所示

![构建BIOS之后的内存布局](-/images/article/ram-bios.png)

中断向量表中有256个中断向量，每个中断向量占4字节，刚好是1KB(256 × 4Byte = 1KB)。其中两个字节是该中断向量对应的处理程序的CS的值，剩下的两个字节是对应IP的值。

BIOS构建完中断服务体系之后，CPU会接收到一个`int 0x19`中断(该中断的作用是把可启动设备的第一个扇区的程序加载至内存的指定位置)，然后CPU在中断向量表中查找该中断对应的中断处理程序地址，随后执行该中断的中断处理程序将启动设备(这里是软盘)的第一个扇区中的程序加载到内存的`0x07C00`(31KB)处。

那么软盘第一个扇区中的程序是什么呢？就是Linux的磁盘引导程序`bootsect`。

## 二、bootsect对内存的规划

bootsect是完全用汇编语言写成的，其首先要对实模式下可寻址(最大范围1KB,即0x00000 ～ 0xFFFFF)的内存进行规划。代码如下所示

反映到实模式下的内存上，如下图所示

接下来bootsect将会按照上述内存布局进行相应的操作。

## 三、bootsect对自身的复制

bootsect先将自身的全部内容从内存0x07C00(BOOTSEG)处复制到内存0x90000(INITSEG)处,对应的汇编代码如下

```
...
SYSSIZE = 0x3000
...
SETUPLEN = 4					! nr of setup-sectors
BOOTSEG  = 0x07c0				! original address of boot-sector
INITSEG  = 0x9000				! we move boot here - out of the way
SETUPSEG = 0x9020				! setup starts here
SYSSEG   = 0x1000				! system loaded at 0x10000 (65536).
ENDSEG   = SYSSEG + SYSSIZE		! where to stop loading
```

其中`SYSSIZE`是system模块的节数，由于16字节为1节，所以system模块共`0x30000Byte`(对应的十进制为`196608Byte`)，即`192KB (196608 / 1024)`。

`SETUPLEN`为setup程序的扇区数，即4个扇区，由于bootsect为第1扇区，那么setup必然就是第2扇区到第5扇区。

`BOOTSEG`为bootsect程序段的初始位置，也就是BIOS加在到内存中的位置，即`0x07c0`。

`INITSEG`为bootsect将要移动的新位置，为`0x9000`。

`SETUPSEG`为setup程序的始址，由于bootsect大小为1个扇区，且新始址为`0x9000`，也就是在`576KB`处(`0x9000`节，即`0x90000Byte`，转化为十进制为`589824Byte`，即`589824 / 1024 = 576KB`),由于第一扇区大小为`512Byte`,即`0.5KB`，那么bootsect的末端就在`576.5KB`处，即十进制`590336Byte`处，对应的十六进制为`0x90200Byte`，也就是`0x9020`(节)处，也就是setup程序紧跟着bootsect，可见Linus对内存的精妙控制。

`SYSSEG`为system模块的始址，即地址`0x10000`处，也就是64KB(对应十进制`65536KB / 1024`)处。

`ENDSEG`自然为system模块的地址末端。

BIOS已经将bootsect加载到内存的0x07C00处了，接下来就轮到bootsect执行了


虚拟地址空间，编址

Intel 80x86系列的CPU均可在16位实模式和32位保护模式下运行。为了向后兼容，Intel将所有80x86系列的CPU都设计为加电后自动进入16位实模式状态运行。