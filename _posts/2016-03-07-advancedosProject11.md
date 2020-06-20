---
layout: post
title: 实验1.1 在v9-cpu中实现时钟中断
categories:
- life
tags:
- technology
- 
---

清华大学高级操作系统课程作业[项目文件地址](https://github.com/dengguang2012/advancedosProject)

在v9-cpu中编译运行方式如下:
在V9-CPU中程序执行的方式是使用的shell来执行程序

	gcc -o xc -O3 -m32 -Ilinux -Iroot/lib root/bin/c.c

表示编译root/bin/c.c生成一个自定义的编译器
-o表示输出可执行程序名称为xc
-m32：产生32位程序

	gcc -o xem -O3 -m32 -Ilinux -Iroot/lib root/bin/em.c -lm

编译生成自定义的cpu模拟器

	./xc -o os0 -Iroot/lib root/usr/os/os3.c
编译生成机器码

	./xem os3

模拟器中运行os3机器码
时钟中断使用TIME命令来设置时钟到时值

	TIME	0xiiiiiia7	if operand0 is 0: timeout = a -- set current timeout from a; else: printk("timer%d=%u timeout=%u", operand0, timer, timeout)

在em.c的cpu方法中有timer和timeout两个函数
-timer: 当前时钟计时（和时间时间中断有关）
-timeout: 时钟周期，当timer>timeout时，产生时钟中断
设置中断向量的方法是使用ivec(alltraps);
 使用指令IVEC的时候如果在内核态，设置中断向量的地址ivec为a; 如果在用户态，产生FPRIV异常

在os3.c中有了时间中断才能实现两个任务的切换和键盘中断响应。
在中断服务程序trap()函数中可以看到

	trap(int *sp, int c, int b, int a, int fc, unsigned *pc)
	{
	  switch (fc) {
	  case FSYS + USER: // syscall
		switch (pc[-1] >> 8) {
		case S_write: a = sys_write(a, b, c); break;
		default: sys_write(1, "panic! unknown syscall\n", 23); asm(HALT);
		}
		break;
	 
		
	  case FTIMER:  // timer
	  case FTIMER + USER:
		out(1,'x');
		if (++current & 1)
		  swtch(&task0_sp, task1_sp);
		else
		  swtch(&task1_sp, task0_sp);
		break;
	  case FKEYBD
	  case FKEYBD + USER:
		sys_write(1, "keyboard\n", 25);asm(HALT);
		break;
	  default:
		default: sys_write(1, "panic! unknown interrupt\n", 25); asm(HALT);  
	  }
	}

在时钟中断发生的时候task0和task1的堆栈指针交换，任务交替执行，程序执行的结果就是交替输出0序列和1序列

<img src="http://7xrmn9.com1.z0.glb.clouddn.com/project111.png" style="width: 50%; height: 50%"/>​

在键盘中断发生的时候触发了FKEYBD，输出keyboard.

<img src="http://7xrmn9.com1.z0.glb.clouddn.com/project112.png" style="width: 50%; height: 50%"/>​


#此外我还尝试了ucore的lab1实验

练习1:理解通过make生成执行文件的过程。
make "V=" :

<img src="http://7xrmn9.com1.z0.glb.clouddn.com/project113.png" style="width: 50%; height: 50%"/>​

	gcc -Ikern/init/ -fno-builtin -Wall -ggdb -m32 -gstabs -nostdinc  -fno-stack-protector -Ilibs/ -Ikern/debug/ -Ikern/driver/ -Ikern/trap/ -Ikern/mm/ -c kern/init/init.c -o obj/kern/init/init.o
	-fno-builtin：禁用了内建函数，
	-nostdinc：不在标准系统目录中搜索头文件
	-fno-stack-protector：禁用堆栈保护
	-I：指定链接的库
	-c：只编译不链接
	-m32：产生32位程序
	-ggdb、-gstabs：与gdb调试相关
	
<img src="http://7xrmn9.com1.z0.glb.clouddn.com/project114.png" style="width: 50%; height: 50%"/>​

m：指定运行程序格式（此处使用elf_i386格式）
-nostdlib：不链接标准库
-T：指定链接脚本（此处使用tools/kernel.ld）

<img src="http://7xrmn9.com1.z0.glb.clouddn.com/project115.png" style="width: 50%; height: 50%"/>​
编译boot/bootasm.S、boot/bootmain.c生成目标文件

<img src="http://7xrmn9.com1.z0.glb.clouddn.com/project116.png" style="width: 50%; height: 50%"/>​

编译tools/sign.c生成目标文件，并最终生成运行程序bin/sign

<img src="http://7xrmn9.com1.z0.glb.clouddn.com/project117.png" style="width: 50%; height: 50%"/>​

	ld -m    elf_i386 -nostdlib -N -e start -Ttext 0x7C00 obj/boot/bootasm.o obj/boot/bootmain.o -o obj/bootblock.o
	-N：设置text和data段可读写
	-e：指定程序入口点
	-Ttext：指定链接时初始重定向地址（此处指定为0x7C00）
链接obj/boot/bootasm.o obj/boot/bootmain.o生成目标文件obj/bootblock.o

通过obj/bootblock.o生成obj/bootblock.out
通过刚才编译生成的bin/sign将obj/bootblock.out生成bin/bootblock

dd if=/dev/zero of=bin/ucore.img count=10000
生成10000个块大小，即5120000字节的bin/ucore.img，内容全为0

dd if=bootblock of=bin/ucore.img conv=notrunc
conv=notrunc意味着不缩减输出文件,也就是说,如果输出文件已经存在,只改变指定的字节，然后退出，并保留输出文件的剩余部分
将bin/bootblock拷贝到bin/ucore.img的前512个字节

dd if=bin/kernel of=bin/ucore.img seek=1 conv=notrunc
seek=1意味着跳过输出文件的一个块后开始复制
将bin/kernel拷贝到bin/ucore.img第513个字节开始的位置，实际拷贝大小为70783字节
根据上面对make "V="的分析，可以看出主引导扇区大小为512个字节，同时查看sign.c可以看出，主引导扇区要以0x55AA结尾
                                                                                                                                                                                   

###练习2:使用qemu执行并调试lab1中的软件。

由于已知第一条指令在0xfffffff0处，所以将tools/gdbinit修改为：
	file bin/kernel
	target remote :1234
	break kern_init
即删除continue，再用make debug启动，会发现程序停在0xf000:0xffff，由于还在8086模式下，故0xfffffff0即相当于是0xfffff，此处命令为：
   0xfffffff0:  ljmp   $0xf000,$0xe05b

<img src="http://7xrmn9.com1.z0.glb.clouddn.com/project118.png" style="width: 50%; height: 50%"/>​

然后si，会跳转到0xf000:0xe05b，此处即为BIOS

<img src="http://7xrmn9.com1.z0.glb.clouddn.com/project119.png" style="width: 50%; height: 50%"/>​

然后再break *0x7c00并continue，会发现程序暂停在0x7c00，查看0x100000处的代码：
   0x100000 <kern_init>:        add    %al,(%eax)
   0x100002 <kern_init+2>:      add    %al,(%eax)
   0x100004 <kern_init+4>:      add    %al,(%eax)
即由0x7c00处的bootloader首先执行，载入ucore到0x100000，然后跳转到0x100000执行启动操作系统
在gdb中，b *0x7c00，然后c：
	(gdb) c
	Continuing.

<img src="http://7xrmn9.com1.z0.glb.clouddn.com/project1110.png" style="width: 50%; height: 50%"/>​

Breakpoint 2, 0x00007c00 in ?? ()
程序成功暂停在0x7c00处

<img src="http://7xrmn9.com1.z0.glb.clouddn.com/project1111.png" style="width: 50%; height: 50%"/>​

通过单步跟踪以及x/i、x/x发现，反汇编得到的结果与bootblock.S、bootblock.asm的代码基本一致

<img src="http://7xrmn9.com1.z0.glb.clouddn.com/project1112.png" style="width: 50%; height: 50%"/>​

###练习3:分析bootloader进入保护模式的过程

bootloader首先关中断、初始化段寄存器
然等待8042 input buffer empty后，向0x64端口写入0xd1，然后再等待8042 input buffer empty后，向0x60端口写入0xdf打开A20 Gate)
接着将全局描述符表入口加载到寄存器后，修改控制寄存器CR0的第0位（PE位）转换模式（为0表示实模式，为1表示保护模式）
最后通过ljmp以32位的格式跳转到下一条命令完成实模式到保护模式的切换

###练习4:分析bootloader加载ELF格式的OS的过程

读取扇区：readsect首先等待硬盘就位，然后向IO地址为0x1F2-0x1F7写入对应参数预备读取，然后再等待硬盘就位后，就可以读取对应扇区（读取扇区采取LBA模式）。bootloader通过调用readseg读入一个页作为elf header，readseg通过调用readsect读入特定编号的扇区
然后验证elf header的e_magic为ELF_MAGIC，接着按照elf header中的e_phoff获取program header指针，并按照program header得到程序的各个段信息，通过readseg读入各个段
最后依照elf header的e_entry最后跳转到程序的入口地址处，将运行权交给载入的ELF格式的OS


###练习5:实现函数调用堆栈跟踪函数	

最后一行为：
	ebp:0x00007bf8 eip:0x00007d68 args:0xc031fcfa 0xc08ed88e 0x64e4d08e 0xfa7502a8
各数据分别意味着：
由bootblock的protcseg调用进入bootmain后，bootmain的栈桢为0x00007bf8
而bootmain调用调用kernel init启动ucore时，应该是运行到了0x00007d68的前面一条命令处，即0x00007d66处的call   *%eax
而args对应的4个参数并不是真正的意义上的参数，因为bootmain调用kernel init的时候是没有参数的，这4个参数实际是只是栈顶之上的内容，而栈顶之上为0x7c00，也就是bootloader的代码，也就是说这16个字节是bootloader的代码的前16个字节：

	(gdb) x/16bx 0x7c00
	0x7c00: 0xfa    0xfc    0x31    0xc0    0x8e    0xd8    0x8e    0xc0
	0x7c08: 0x8e    0xd0    0xe4    0x64    0xa8    0x02    0x75    0xfa

###练习6:完善中断初始化和处理

中断向量表一个表项占8个字节，0-15位代表offset的0-15位，48-63位代表offset的16-31位，16-31位为段选择子，通过段选择子从GDT从取得段基址，段基址加上偏移offset才是中断处理代码的入口


首先extern声明__vectors
然后查看SETGATE宏的定义，发现注释：

	/* *
	 * Set up a normal interrupt/trap gate descriptor
	 *   - istrap: 1 for a trap (= exception) gate, 0 for an interrupt gate
	 *   - sel: Code segment selector for interrupt/trap handler
	 *   - off: Offset in code segment for interrupt/trap handler
	 *   - dpl: Descriptor Privilege Level - the privilege level required
	 *          for software to invoke this interrupt/trap gate explicitly
	 *          using an int instruction.
	 * */
	#define SETGATE(gate, istrap, sel, off, dpl)
	
那么我们就应该是SETGATE(idt[i], 0, sel, __vectors[i], 0)，只差sel了
查看GDT的初始化：

	static struct segdesc gdt[] = {
		SEG_NULL,
		[SEG_KTEXT] = SEG(STA_X | STA_R, 0x0, 0xFFFFFFFF, DPL_KERNEL),
		[SEG_KDATA] = SEG(STA_W, 0x0, 0xFFFFFFFF, DPL_KERNEL),
		[SEG_UTEXT] = SEG(STA_X | STA_R, 0x0, 0xFFFFFFFF, DPL_USER),
		[SEG_UDATA] = SEG(STA_W, 0x0, 0xFFFFFFFF, DPL_USER),
		[SEG_TSS]    = SEG_NULL,
	};

那么此处sel应该对应于[SEG_KTEXT]这个段，那么sel的值就应该是SEG_KTEXT << 3
于是乎，初始化idt，并单独设置idt[T_SYSCALL]，最后再lidt一下即可
至此程序已经完成，但查找了一下SEG_KTEXT的定义，发现：

	/* global segment number */
	#define SEG_KTEXT    1
	#define SEG_KDATA    2
	#define SEG_UTEXT    3
	#define SEG_UDATA    4
	#define SEG_TSS        5

	/* global descrptor numbers */
	#define GD_KTEXT    ((SEG_KTEXT) << 3)        // kernel text
	#define GD_KDATA    ((SEG_KDATA) << 3)        // kernel data
	#define GD_UTEXT    ((SEG_UTEXT) << 3)        // user text
	#define GD_UDATA    ((SEG_UDATA) << 3)        // user data
	#define GD_TSS        ((SEG_TSS) << 3)        // task segment selector

	#define DPL_KERNEL    (0)
	#define DPL_USER    (3)

	#define KERNEL_CS    ((GD_KTEXT) | DPL_KERNEL)
	#define KERNEL_DS    ((GD_KDATA) | DPL_KERNEL)
	#define USER_CS        ((GD_UTEXT) | DPL_USER)
	#define USER_DS        ((GD_UDATA) | DPL_USER)
	
故我们可以将SEG_KTEXT << 3替换成KERNEL_CS，将dpl使用DPL_KERNEL


ticks在include的clock.h中已经声明，故不用再声明
然后，每次运行到该case中就意味着一次时钟中断，于是++ticks，然后判断ticks为TICK_NUM的整数倍意味着一轮输出

	ticks ++;
	if (ticks % TICK_NUM == 0) {
		print_ticks();
	}
	break;
