---
layout: post
title: "Program Crashes On Exit"
date: 2015-09-04 23:09:08 +0800
comments: true
categories: 
---

之前有遇過程式在執行exit()之後, 竟然發生crash的狀況. 
這個問題罩因於, 在執行exit()時, 會對C++的static/global Object進行de-init, 
若C++ class的destructor沒有寫好, 就會在執行destructor時發生crash, 
下面的程式故意在destructor裡製造crash, 重製這樣的bug:

``` cpp crash.cpp
	#include <stdlib.h>
	#include <stdio.h>
	
	static char* ptr ;
	
	class PARENT {
	      public:
	              PARENT()
	              {
	               if(ptr)
	                 {
	                  ptr = (char*) malloc(100);
	                 }
	              }
	              ~PARENT()
	              {
	               free(ptr);
	               ptr = (char*) 0xDEADBEEF ;
	              }
	              void test(void)
	              {
	               printf("test()\n");
	              }
	};
	
	int main(void)
	{
	 static PARENT parentObj1, parentObj2 ;
	 parentObj1.test();
	 parentObj2.test();
	 exit(0);
	 return 0 ;
	}
```

編譯與執行:

<font face="sans">
``` plain Terminal
	bramante@matrix:~/test$ g++ -o ./crash ./crash.cpp
	bramante@matrix:~/test$ gdb ./crash
	GNU gdb (Ubuntu/Linaro 7.4-2012.04-0ubuntu2.1) 7.4-2012.04
	Copyright (C) 2012 Free Software Foundation, Inc.
	License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
	This is free software: you are free to change and redistribute it.
	There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
	and "show warranty" for details.
	This GDB was configured as "x86_64-linux-gnu".
	For bug reporting instructions, please see:
	<http://bugs.launchpad.net/gdb-linaro/>...
	Reading symbols from /home/bramante/test/crash...(no debugging symbols found)...done.
	(gdb) run
	Starting program: /home/bramante/test/crash
	test()
	test()
	
	Program received signal SIGSEGV, Segmentation fault.
	__GI___libc_free (mem=0xdeadbeef) at malloc.c:2970
	2970    malloc.c: No such file or directory.
	(gdb) bt
	#0  __GI___libc_free (mem=0xdeadbeef) at malloc.c:2970
	#1  0x0000000000400831 in PARENT::~PARENT() ()
	#2  0x00007ffff77575b1 in __run_exit_handlers (status=0, listp=0x7ffff7ad3688, run_list_atexit=true) at exit.c:78
	#3  0x00007ffff7757635 in __GI_exit (status=<optimized out>) at exit.c:100
	#4  0x00000000004007ea in main ()
	(gdb)
```
</font>

從callstack可以看出是main()執行exit()時,
在class PARENT的destructor發生crash.
crash的原因是程式去踩到一個不合法的address 0xDEADBEEF.

這個bug, 在glibc的callstack看到的是:

	__run_exit_handlers()
	__GI_exit()
	
在bionic libc看到的則是:

	__cxa_finalize()
	exit()

不同的libc在這部份的實作上, 似乎有點不一樣.

C++的程式在一跑起來時, 會利用__cxa_atexit()去註冊destructor, 
然後在程式結束執行時, 便會去執行這些登記在案的destructor.

work around bug的方法很簡單, 如果只是單純要程式結束, 並且不想看到crash的log,
可以把exit()換成kill(getpid(), SIGKILL),
或是把exit()換成_exit(), 讓程式刻意不去執行destructor, 
便可以讓crash不要發生.

