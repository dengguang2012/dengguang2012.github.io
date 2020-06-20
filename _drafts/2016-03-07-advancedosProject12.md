---
layout: post
title: 实验1.2 在v9-cpu中实现建立页表操作
categories:
- life
tags:
- technology
- 
---

##实验要求，完成建立页表的实验lab2.c
	
清华大学高级操作系统课程作业[项目文件地址](https://github.com/dengguang2012/advancedosProject)
	
在v9-cpu中编译运行方式如下:

	.xc -o os2 -Iroot/lib root/usr/os/os2.c
	./xem os2

<img src="http://7xrmn9.com1.z0.glb.clouddn.com/project121.png" style="width: 50%; height: 50%"/>​

setup_paging函数是用来建立页表
下面是页表进入的标志位

	enum { // page table entry flags
	  PTE_P   = 0x001,       // Present
	  PTE_W   = 0x002,       // Writeable
	  PTE_U   = 0x004,       // User
	//PTE_PWT = 0x008,       // Write-Through
	//PTE_PCD = 0x010,       // Cache-Disable
	  PTE_A   = 0x020,       // Accessed
	  PTE_D   = 0x040,       // Dirty
	//PTE_PS  = 0x080,       // Page Size
	//PTE_MBZ = 0x180,       // Bits must be zero
	};

pg_mem是实际使用的物理内存数组，其中前面的1024个字节为页目录，后面的4096个字节分别为4个页，再后面1024个字节是对齐字节
	
	char pg_mem[6 * 4096]; // page dir + 4 entries + alignment
	setup_paging函数用来描述建立页目录页表的过程，其中可以看到pg0,pg1,pg2,pg3分别指向了四个页表起点，而pg_dir[0],pg_dir[1],pg_dir[2],pg_dir[3]是页目录分别指向三个页表
	setup_paging()
	{
	  int i;
	  
	  pg_dir = (int *)((((int)&pg_mem) + 4095) & -4096);
	  pg0 = pg_dir + 1024;
	  pg1 = pg0 + 1024;
	  pg2 = pg1 + 1024;
	  pg3 = pg2 + 1024;
	  printf("pg:%d..%d..%d..%d...ok\n",pg0,pg1,pg2,pg3);
	  pg_dir[0] = (int)pg0 | PTE_P | PTE_W | PTE_U;  // identity map 16M
	  pg_dir[1] = (int)pg1 | PTE_P | PTE_W | PTE_U;
	  pg_dir[2] = (int)pg2 | PTE_P | PTE_W | PTE_U;
	  pg_dir[3] = (int)pg3 | PTE_P | PTE_W | PTE_U;
	  printf("pgdir:%d..%d..%d..%d...ok\n",pg_dir[0],pg_dir[1],pg_dir[2],pg_dir[3]);
	  for (i=4;i<1024;i++) pg_dir[i] = 0;
	  
	  for (i=0;i<4096;i++) pg0[i] = (i<<12) | PTE_P | PTE_W | PTE_U;  // trick to write all 4 contiguous pages
	  
	  pdir(pg_dir);
	  spage(1);
	}
	
在主函数中对于建立的页表进行了测试，能够看到对于(int *)(50<<12)地址进行读写都触发了中断。

	printf("test paging...");
	// reposition stack within first 16M
	asm(LI, 4*1024*1024); // a = 4M
	asm(SSP); // sp = a
	setup_paging();
	printf("identity map...ok\n");

	printf("test page fault read...");
	pg0[50] = 0;
	pdir(pg_dir);
	t = *(int *)(50<<12);
	printf("%d...ok\n",t);

	printf("test page fault write...");
	*(int *)(50<<12) = 5;
	printf("...ok\n");
