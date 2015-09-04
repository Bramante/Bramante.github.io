---
layout: post
title: "Program Hangs On Exit"
date: 2015-09-04 23:09:43 +0800
comments: true
categories: 
---

之前有遇過程式在執行exit()之後, 竟然發生程式hang住, 無法結束執行的狀況. 
這個問題罩因於, 在執行exit()時, 會對C++的static/global Object進行de-init, 
若C++ class的destructor沒有寫好, 例如卡在destructor裡的永久迴圈, 或是發生了dead lock,
便會造成這個異常的現象, 下面的程式刻意在destructor裡加入永久迴圈, 
故意製造hang住的狀態, 重製這樣的bug:

``` cpp hang.cpp
	#include <stdlib.h>
	#include <stdio.h>
	#include <unistd.h>
	
	static char* ptr ;
	
	class PARENT {
	      public:
	              PARENT()
	              {
	              }
	              ~PARENT()
	              {
	               while(1)
	                    {
	                     printf("block...\n");
	                     sleep(1);
	                    }
	              }
	              void test(void)
	              {
	               printf("test()\n");
	              }
	};
	
	int main(void)
	{
	 static PARENT parentObj ;
	 parentObj.test();
	 exit(0);
	 return 0 ;
	}
```

編譯與執行:

<font face="sans">
``` plain Terminal
	bramante@matrix:~/test$ g++ -o ./hang ./hang.cpp
	bramante@matrix:~/test$ gdb ./hang
	GNU gdb (Ubuntu/Linaro 7.4-2012.04-0ubuntu2.1) 7.4-2012.04
	Copyright (C) 2012 Free Software Foundation, Inc.
	License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
	This is free software: you are free to change and redistribute it.
	There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
	and "show warranty" for details.
	This GDB was configured as "x86_64-linux-gnu".
	For bug reporting instructions, please see:
	<http://bugs.launchpad.net/gdb-linaro/>...
	Reading symbols from /home/bramante/test/hang...(no debugging symbols found)...done.
	(gdb) run
	Starting program: /home/bramante/test/hang
	test()
	block...
	block...
	block...
	block...
	block...
	block...
	block...
	^C
	Program received signal SIGINT, Interrupt.
	0x00007ffff77db090 in __nanosleep_nocancel () at ../sysdeps/unix/syscall-template.S:82
	82      ../sysdeps/unix/syscall-template.S: No such file or directory.
	(gdb) bt
	#0  0x00007ffff77db090 in __nanosleep_nocancel () at ../sysdeps/unix/syscall-template.S:82
	#1  0x00007ffff77daf4c in __sleep (seconds=0) at ../sysdeps/unix/sysv/linux/sleep.c:138
	#2  0x0000000000400770 in PARENT::~PARENT() ()
	#3  0x00007ffff77575b1 in __run_exit_handlers (status=0, listp=0x7ffff7ad3688, run_list_atexit=true) at exit.c:78
	#4  0x00007ffff7757635 in __GI_exit (status=<optimized out>) at exit.c:100
	#5  0x0000000000400746 in main ()
	(gdb)
```
</font>

從callstack可以看出是main()執行exit()時,
在class PARENT的destructor發生hang住的狀況.

這個bug, 在glibc的callstack看到的是:

	__run_exit_handlers()
	__GI_exit()
	
在bionic libc看到的則是:

	__cxa_finalize()
	exit()

不同的libc在這部份的實作上, 似乎有點不一樣.

C++的程式在一跑起來時, 會利用__cxa_atexit()去註冊destructor, 
然後在程式結束執行時, 便會去執行這些登記在案的destructor.

work around bug的方法很簡單,
可以把exit()換成kill(getpid(), SIGKILL),
或是把exit()換成_exit(), 讓程式刻意不去執行destructor, 
便可以讓hang住不要發生.
