title: "Linux内核源码分析：转变！从16位实模式到32位保护模式"
date: 2019-12-27 18:56:05 +0800
update: 2019-12-27 18:56:05 +0800
author: me
cover: "-/images/os.jpg"
tags:
    - Linux
preview: 

---

## 回顾

从上文得知，在BIOS将Linux的磁盘引导程序`bootsect`加载到`0x07C00`之后，bootsect开始执行，其先是将自己移动到了`0x90000`处，然后设置了段寄存器ds、es、ss，后将setup、system程序加载至了指定位置，并确认了根设备号，最终通过段间跳转指令将CPU控制权交给了setup程序。

至此，操作系统的内核程序已经加载完成，但是计算机依旧运行在16位的实模式下，也就意味着只能利用20根地址总线(即`0 ~ 19`号地址线)，寻址空间仅1MB，也就是寻址范围为`0 ~ 0xFFFFF`。实模式下的特征是在1MB寻址空间内可以直接软件访问BIOS及周边硬件，但是没有硬件支持的分页机制和实时多任务概念。对于一个现代操作系统来说，这显然是不合适的。因此，从setup开始，做的至关重要的一件事情就是从实模式转变到保护模式下，成为一个真正的现代操作系统。

## 一、从BIOS中获取系统数据

setup做的第一件事就是从BIOS中获取系统数据，并将其保存到`0x90000 - 0x901FF`的位置。`0x90000`是bootsect的始址，并不超过`0x90200`，但是由于bootsect已经完成了任务，所以这段空间可以直接覆盖掉。

先是读取光标位置，通过BIOS中断`0x10`的`0x03`号功能来实现，代码如下

```
! ok, the read went well so we get current cursor position and save it for
! posterity.

    mov ax,#INITSEG ! this is done in bootsect already, but...
    mov ds,ax
    mov ah,#0x03    ! read cursor pos
    xor bh,bh
    int 0x10        ! save it in known place, con_init fetches
    mov [0],dx      ! it from 0x90000.
```

该功能号的入口参数为页号码，通过寄存器`bx`的高八位`bh`来传递，这里传入的是0，即通过`xor`运算将`bh`寄存器清零。

返回的参数包括

+ `ch`，扫描开始线；
+ `cl`，扫描结束线；
+ `dh`，行号(`0x00`是顶端)；
+ `dl`，列号(`0x00`是左边)。

`dx`寄存器总共两个字节，从`0x90000`开始保存，即占用`0x90000`和`0x90001`两个字节。

接下来获取拓展内存大小(即RAM中高于1MB的部分)，调用`0x15`中断的`0x88`功能号实现，代码如下

```
! Get memory size (extended mem, kB)

    mov ah,#0x88
    int 0x15
    mov [2],ax
```

返回值保存在`ax`寄存器中，共两个字节，保存在`0x90002`和`0x90003`中。

接下来读取显卡数据，通过`0x10`中断的`0x0f`功能实现，代码如下

```
! Get video-card data:

    mov ah,#0x0f
    int 0x10
    mov [4],bx      ! bh = display page
    mov [6],ax      ! al = video mode, ah = window width
```

返回参数为

+ `ah`，字符列数；
+ `al`，显示模式；
+ `bh`，当前显示页。

然后分别保存在偏移为`4`和`6`的位置。

接下来检查显示方式并取参数，分别通过`0x10`中断的`0x12`及`0x10`功能来实现，代码如下

```
! check for EGA/VGA and some config parameters

    mov ah,#0x12
    mov bl,#0x10
    int 0x10
    mov [8],ax
    mov [10],bx
    mov [12],cx
```

返回参数为

+ `bh`，显示状态(`0x00`-彩色模式, I/O端口 = `0x3dX`；`0x01`-单色模式, I/O端口=`0x3bX`)；
+ `bl`，安装的显示内存；
+ `cx`，显示卡特性参数。

然后分别保存在偏移为`8`, `10`, `12`的位置。

接着读取第一个硬盘的信息。需要注意的是，第一个硬盘参数表的首地址是中断向量`0x41`的向量值，该参数表的长度为16字节，保存的地址始址为`0x90080`，连续16个字节(`0x10`)。代码如下

```
! Get hd0 data

    mov ax,#0x0000
    mov ds,ax
    lds si,[4*0x41]
    mov ax,#INITSEG
    mov es,ax
    mov di,#0x0080
    mov cx,#0x10
    rep
    movsb
```

和前面不一样，这里使用`es:di`来指向传输的目的地址，而`ds:si`则指向参数表的地址，即源地址。

接下来要读取第二个硬盘的信息，代码逻辑和上述一致，其参数表的地址是中断向量`0x46`的地址值，由于第一个硬盘参数表刚好保存到`0x9008F`的位置，那么第二个硬盘的参数表就是从`0x90090`开始，连续16个字节(`0x10`)，代码如下

```
! Get hd1 data

    mov ax,#0x0000
    mov ds,ax
    lds si,[4*0x46]
    mov ax,#INITSEG
    mov es,ax
    mov di,#0x0090
    mov cx,#0x10
    rep
    movsb
```

最后要做的是检查系统是否存在第二个硬盘，如果不存在则将上述保存的第二个硬盘参数表清零，代码如下

```
! Check that there IS a hd1 :-)

    mov	ax,#0x01500
    mov	dl,#0x81
    int	0x13
    jc	no_disk1
    cmp	ah,#3
    je	is_disk1
no_disk1:
    mov	ax,#INITSEG
    mov	es,ax
    mov	di,#0x0090
    mov	cx,#0x10
    mov	ax,#0x00
    rep
    stosb
```

这个过程通过调用中断`0x13`的`0x15`号功能来实现。入口参数为`dl=驱动器号`，其中`0x8X`表示硬盘、`0x80`表示第一个硬盘、`0x81`表示第二个硬盘，那么自然这里必然是`0x81`。其出口参数为`ah=类型码`，`00`表示不存在此盘，并将`CF`置位；`01`表示软驱，没有`change-line`支持；`02`表示软驱或其它可移动设备，有`change-line`支持；`03`表示硬盘。

通过`jc`指令检查`CF`是否置位，如果置位即不存在第二个硬盘，那么就跳转至`no_disk1`处清零第二个参数表。如果存在第二个硬盘，那么`jc`指令自然不满足则执行`cmp`指令判断设备是否为硬盘，如果是则将标志寄存器ZF置位(即`ah == 03`)。再通过`je`指令判断`ZF`是否置位，如果置位那么代表设备为硬盘，那么就跳转至`is_disk1`处继续执行。即

```
is_disk1:
```

## 二、关中断！并将system移动至0x00000处

is_disk1第一句代码就是关中断，如下

```
is_disk1:

! now we want to move to protected mode ...

    cli         ! no interrupts allowed !
```

`cli`指令将CPU标志寄存器中中断允许标志`IF`置零，即不允许中断。关中断是16位实模式进入32位保护模式的标志性动作，这意味着接下来就可以废除16位实模式下的中断向量表，并初步打开32位寻址空间、建立保护模式下的中断响应机制等，这些都是与32位保护模式相配套的。


作为转变的开始，已经关闭了中断，那么接下来系统将不会响应中断，以便一心一意向保护模式转变。现在开始将system移动至内存始址处，代码如下

```
! first we move the system to it's rightful place

    mov	ax,#0x0000
    cld             ! 'direction'=0, movs moves forward
do_move:
    mov	es,ax       ! destination segment
    add	ax,#0x1000
    cmp	ax,#0x9000
    jz	end_move
    mov	ds,ax       ! source segment
    sub	di,di
    sub	si,si
    mov cx,#0x8000
    rep
    movsw
    jmp	do_move
```

其中`es:di`指向目的地址`0x0000:0x0`处，`ds:si`指向源地址`0x10000:0x0`处，由于起初假设system模块不会超过`0x80000`，即`512KB`，那么就不会超过`0x90000`,即system最初不会覆盖bootsect。那么这段程序就是将`[0x10000, 0x10000 + 512KB)`的内存数据块移动`[0x00000, 0x00000 + 512KB)`处，移动的数据块长度为`0x8000`节，即`512KB`。也就是说将每个源地址字节向内存低端移动`0x10000`个位置最终到达目标位置，上述汇编代码可以用如下伪码描述

```
ax = 0x0000
while truees = ax
    ax += 0x1000
    ds = ax
    di = si = 0x0
    ds:si to es:di, moving one seg continuously, i.e. 64KB
    if ax + 0x1000 == 0x9000
        break
```

再进一步地说明，上述伪码中各参数变动情况如下所示(注：下述`while`循环用来解释上述中的`ds:si to es:di, moving one seg continuously, i.e. 64KB`)

```
The first cycle is as follows:
  es = 0x0000, ax = 0x1000, ds = 0x1000, di = 0x0, si = 0x0, cx = 0x8000
    while cx != 0x0000
      ds:si to es:di, one word at a time, i.e. movsw
        cx -= 0x0001

The second cycle is as follows:
  es = 0x1000, ax = 0x2000, ds = 0x2000, di = 0x0, si = 0x0, cx = 0x8000
    while cx != 0x0000
      ds:si to es:di, one word at a time, i.e. movsw
        cx -= 0x0001
...
```

这就很容易理解了。那么该段汇编就可以整体上用如下伪码描述

```
ax = 0x0000
while truees = ax
    ax += 0x1000
    ds = ax
    di = si = 0x0
    cx = 0x8000
    while cx != 0x0000
        ds:si to es:di, one word at a time, i.e. movsw
        cx -= 0x0001
    if ax + 0x1000 == 0x9000
        break
```

至此，就完成了对system模块的移动。对system模块的移动起到了如下的效果

+ 废除了BIOS中断向量表，等同于废除了BIOS所提供的实模式下的中断服务程序；
+ 回收已经无用的内存空间，因为要向保护模式转变，BIOS中断向量表所占空间自然无用，应当回收；
+ 让system模块占据内存物理地址最天然、有利的位置。

## 三、设置IDT与GDT

与实模式不同的是，保护模式下的段寻址是通过GDT(Global Descriptor Table, 全局描述符表)完成的，GDT中存放的是段寄存器内容，其数据结构为数组。GDT在操作系统进程切换中意义重大，其中存放了每个任务的LDT(Local Descriptor Table, 局部描述符表)地址和TSS(Task Structure Segment, 任务状态段)地址，以完成进程中各段的寻址、现场保护与现场恢复。

GDT的初始内容已经写在了setup程序中，如下

```
gdt:
    .word   0,0,0,0     ! dummy

    .word   0x07FF      ! 8Mb - limit=2047 (2048*4096=8Mb)
    .word   0x0000      ! base address=0
    .word   0x9A00      ! code read/exec
    .word   0x00C0      ! granularity=4096, 386

    .word   0x07FF      ! 8Mb - limit=2047 (2048*4096=8Mb)
    .word   0x0000      ! base address=0
    .word   0x9200      ! data read/write
    .word   0x00C0      ! granularity=4096, 386
```

可以整理为如下一张表

```
 index |    GDT
------------------
   2   | 00C0 9200
       | 0000 07FF
------------------
   1   | 00C0 9A00
       | 0000 07FF
------------------
   0   | 0000 0000
       | 0000 0000
```

其中每个GDT表项共64位，即8字节，结构如下

```
31------------------------------16-15-----------------------0
             Base 0:15            |         Limit 0:15
63--------56-55---52-51---------48-47---------40-39--------32
 Base 24:31 | Flags | Limit 16:19 | Access Byte | Base 16:23
```

Access Byte的结构如下

```
8---7-------5---4----3----2----1---0
 Pr | Privl | 1 | Ex | Dc | RW | Ac
```

Flags的结构如下

```
8---7----6---5--4
 Gr | Sz | 0 | 0
```

特别说明的`Privl`为特权级，如果为`00`表示内核特权级，如果为`11`，则表明用户特权级；`Gr`标志位为颗粒度标志，如果为1，那么段限长的单位为4KB，如果为0，那么就是1B。可见，GDT将段基址与段限长拆分保存在不连续的bit位中。这是为了兼容286架构。那么现在GDT表项就很容易理解了。

以第一项GDT为例，其内容为

```
00C0 9A00
0000 07FF
```

第`0-15bit`与`48-51bit`构成段限长，内容为

```
007FF
```

将其转换为10进制就是2047bit，即2KB；再看一下颗粒度标志，其包含在最后一个字节，即

```
00C0    // 00000000 11000000
```

可见，颗粒度标志位的值为1，那么也就意味着段限长实际上为`0x007FF * 4KB = 8MB`。确定了段限长，我们再确定一下段基址，`16-31bit`, `32-39bit`, `56-63bit`构成了段基址，那么合起来就是

```
0x0000000
```

即内存始址。

现在我们知道了GDT的内容与含义，那么设置GDT就是将GDT表的始址保存在GDTR(Global Descriptor Table Register, GDT基地址寄存器)中，通过下述指令完成

```
lidt    idt_48      ! load idt with 0,0
```