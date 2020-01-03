title: "Linux内核源码分析：适应32位保护模式并创建内核分页机制"
date: 2020-01-03 16:54:05 +0800
update: 2020-01-03 16:54:05 +0800
author: me
cover: "-/images/os.jpg"
tags:
    - Linux
preview: 

---

## 回顾

上文中，setup.s通过段间跳转指令`jmpi`跳转至`8:0`处执行，即开始执行system模块的head.s程序。由于setup.s将system模块移动到了内存始址处，而head.s程序在被编译后，会被连接成system模块的最前面开始部分，即head.s程序就是位于内存始址处。

head.s汇编程序与之前的汇编程序不同，其采用AT&T的汇编语言格式，并且使用GNU的gas和gld进行编译连接，最明显的特征是操作数的结合方向是自左向右，与Intel汇编更好相反。

## 一、设置数据段与栈段寄存器

head.s的开始要设置数据段寄存器，其本质是将数据段寄存器从实模式转向保护模式。实模式下代码段寄存器(cs)与数据段寄存器(ds、es、fs、gs)保存的就是代码段或数据段的基址，而保护模式下保存的则是代码段或数据段的选择符。cs寄存器在setup.s中已经完成了转变，因此这里需要对数据段寄存器进行设置，以使其适应32位保护模式。代码如下

```
startup_32:
    movl $0x10,%eax
    mov %ax,%ds
    mov %ax,%es
    mov %ax,%fs
    mov %ax,%gs
```

对于GNU汇编来说，每个直接数要以`$`开始，否则表示的是地址，而每个寄存器名都要以`%`开头，eax表示的是32位的ax寄存器。

这里的直接数`$0x10`依然要转化为二进制来看待，即`00010000`，那么第一第二位表示内核特权级，第三位表示的是GDT，第四和第五位为二进制`10`也就是十进制2，即GDT索引为2的那一项。这里使用的GDT依旧是定义在setup.s中的GDT，即上文中解释过的GDT项。也从侧面说明了ds、es、fs、gs的段基址、段限长、特权级都是相同的，段基址就是GDT索引为2的那一项指明的基址，即内存始址`0x0000000`(参见上文)，段限长为8MB，特权级为内核特权级。

上述通过eax寄存器设置了数据段寄存器，那么接下来设置栈段寄存器，代码如下

```
    lss _stack_start,%esp
```

`lss`指令，即Load Stack Segment Register，是80386及以后的CPU才有的指令，其指令格式如下

```
lss Mem, Reg
```

若`Reg`为16位寄存器，则`Mem`必须是32位指针(地址)；若`Reg`是32位寄存器，则`Mem`必须是48位指针，其低32位给指令中指定的寄存器，高16位给指令中的段寄存器。

stack_start定义在kernel/sched.c中，为

```
stack_start = { & user_stack[PAGE_SIZE>>2], 0x10}
```

上述代码将栈顶指针指向`user_stack`数据结构的最末位置，该数据结构定义在kernel/sched.c中，为

```
long user_stack [ PAGE_SIZE>>2 ]
```

大概位于`0x1E25C`处。这里的`0x10`将ss设置为与前面4个段选择符相同的值，即段基址都是`0x0000000`，段限长都是8MB，特权级都是内核特权级，其后的压栈动作就要在这里进行。

经过上述设置，代码段基址、数据段基址以及栈段基址都位于内存始址(`0x00000`处)。而栈顶指针则指向`0x1E25C`的位置，如下图所示

![代码段基址、数据段基址以及栈段基址,栈顶内存映像](/images/article/202001031656.png)

## 二、重新设置IDT与GDT

接下来要设置IDT，之前在setup.s中设置过IDT，但是只是将IDT的基地址传递给了IDTR，而IDT本身则没有设置，这里则进行设置，代码如下

```
    call setup_idt
```

通过`call`指令调用了`setup_idt`子程序，该子程序为

```
setup_idt:
    lea ignore_int,%edx
    movl $0x00080000,%eax
    movw %dx,%ax        /* selector = 0x0008 = cs */
    movw $0x8E00,%dx    /* interrupt gate - dpl=0, present */

    lea _idt,%edi
    mov $256,%ecx
rp_sidt:
    movl %eax,(%edi)
    movl %edx,4(%edi)
    addl $8,%edi
    dec %ecx
    jne rp_sidt
    lidt idt_descr
    ret
```