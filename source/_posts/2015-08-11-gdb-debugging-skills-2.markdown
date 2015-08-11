---
layout: post
title: "GDB Debugging Skills (2)"
date: 2015-08-11 23:56:31 +0800
comments: true
categories: 
---

假設有一支客戶寫的程式, 
執行檔有debug symbol, 
在客戶不給source code的狀況下,
該如何知道某個API被call的情形?

範例程式:

``` c test.c
	#include <stdio.h>
	#include <time.h>
	#include <stdlib.h>
	
	void test(int arg)
	{
	 printf("@test(arg:%d)\n",arg);
	}
	
	int main(int argc, char *argv[])
	{
	 if(argc == 2)
	   {
	    int num = atoi(argv[1]);
	    int i ;
	    for( i = 0 ; i < num ; i ++ )
	       {
	        test(i);
	        sleep(1);
	       }
	   }
	 return 0 ;
	}
```

編譯與執行:

<font face="sans">
``` plain Terminal
	bramante@matrix:~/test$ gcc -g -o ./test ./test.c
	bramante@matrix:~/test$ ./test 5
	@test(arg:0)
	@test(arg:1)
	@test(arg:2)
	@test(arg:3)
	@test(arg:4)
	bramante@matrix:~/test$
```
</font>

先在有source code的狀況下, 讓gdb自動印出API被call時的參數:

<font face="sans">
``` plain Terminal
	bramante@matrix:~/test$ gdb ./test
	GNU gdb (Ubuntu/Linaro 7.4-2012.04-0ubuntu2.1) 7.4-2012.04
	Copyright (C) 2012 Free Software Foundation, Inc.
	License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
	This is free software: you are free to change and redistribute it.
	There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
	and "show warranty" for details.
	This GDB was configured as "x86_64-linux-gnu".
	For bug reporting instructions, please see:
	<http://bugs.launchpad.net/gdb-linaro/>...
	Reading symbols from /home/bramante/test/test...done.
	(gdb) b test
	Breakpoint 1 at 0x40058f: file ./test.c, line 7.
	(gdb) commands 1
	Type commands for breakpoint(s) 1, one per line.
	End with a line saying just "end".
	>info args
	>continue
	>end
	(gdb) run 5
	Starting program: /home/bramante/test/test 5
	warning: no loadable sections found in added symbol-file system-supplied DSO at 0x7ffff7ffa000
	
	Breakpoint 1, test (arg=0) at ./test.c:7
	7        printf("@test(arg:%d)\n",arg);
	arg = 0
	@test(arg:0)
	
	Breakpoint 1, test (arg=1) at ./test.c:7
	7        printf("@test(arg:%d)\n",arg);
	arg = 1
	@test(arg:1)
	
	Breakpoint 1, test (arg=2) at ./test.c:7
	7        printf("@test(arg:%d)\n",arg);
	arg = 2
	@test(arg:2)
	
	Breakpoint 1, test (arg=3) at ./test.c:7
	7        printf("@test(arg:%d)\n",arg);
	arg = 3
	@test(arg:3)
	
	Breakpoint 1, test (arg=4) at ./test.c:7
	7        printf("@test(arg:%d)\n",arg);
	arg = 4
	@test(arg:4)
	[Inferior 1 (process 3035) exited normally]
	(gdb)
```
</font>

接下來, 改用gdb的batch mode, 
這樣就可以在target上進行測試後, 
再透過console log來了解API被call的情形:

<font face="sans">
``` plain Terminal
	bramante@matrix:~/test$ gdb --batch --command=gdb.cmds --args ./test 5
	Breakpoint 1 at 0x40058f: file ./test.c, line 7.
	warning: no loadable sections found in added symbol-file system-supplied DSO at 0x7ffff7ffa000
	
	Breakpoint 1, test (arg=0) at ./test.c:7
	7        printf("@test(arg:%d)\n",arg);
	arg = 0
	@test(arg:0)
	
	Breakpoint 1, test (arg=1) at ./test.c:7
	7        printf("@test(arg:%d)\n",arg);
	arg = 1
	@test(arg:1)
	
	Breakpoint 1, test (arg=2) at ./test.c:7
	7        printf("@test(arg:%d)\n",arg);
	arg = 2
	@test(arg:2)
	
	Breakpoint 1, test (arg=3) at ./test.c:7
	7        printf("@test(arg:%d)\n",arg);
	arg = 3
	@test(arg:3)
	
	Breakpoint 1, test (arg=4) at ./test.c:7
	7        printf("@test(arg:%d)\n",arg);
	arg = 4
	@test(arg:4)
	[Inferior 1 (process 3048) exited normally]
	bramante@matrix:~/test$
```
</font>

前面所用到的gdb.cmds的內容如下:

<font face="sans">
``` plain gdb.cmds
	b test
	commands 1
	  info args
	  continue
	end
	run
```
</font>

回到本文最前面所提到的"沒有source code"的狀況, 
我把test.c給砍掉, 看會發生怎樣的狀況:

<font face="sans">
``` plain Terminal
	bramante@matrix:~/test$ ll
	total 28
	drwxrwxr-x 2 bramante bramante  4096 Aug 12 00:10 ./
	drwxr-xr-x 5 bramante bramante  4096 Aug 11 23:32 ../
	-rw-rw-r-- 1 bramante bramante    50 Aug 11 23:44 gdb.cmds
	-rwxrwxr-x 1 bramante bramante 10084 Aug 11 23:57 test*
	-rw-rw-r-- 1 bramante bramante   316 Aug 11 23:54 test.c
	bramante@matrix:~/test$ rm test.c
	bramante@matrix:~/test$ gdb --batch --command=gdb.cmds --args ./test 5
	Breakpoint 1 at 0x40058f: file ./test.c, line 7.
	warning: no loadable sections found in added symbol-file system-supplied DSO at 0x7ffff7ffa000
	
	Breakpoint 1, test (arg=0) at ./test.c:7
	7       ./test.c: No such file or directory.
	arg = 0
	@test(arg:0)
	
	Breakpoint 1, test (arg=1) at ./test.c:7
	7       in ./test.c
	arg = 1
	@test(arg:1)
	
	Breakpoint 1, test (arg=2) at ./test.c:7
	7       in ./test.c
	arg = 2
	@test(arg:2)
	
	Breakpoint 1, test (arg=3) at ./test.c:7
	7       in ./test.c
	arg = 3
	@test(arg:3)
	
	Breakpoint 1, test (arg=4) at ./test.c:7
	7       in ./test.c
	arg = 4
	@test(arg:4)
	[Inferior 1 (process 3060) exited normally]
	bramante@matrix:~/test$
```
</font>

gdb依然可以印出API的參數.

將gdb.cmds的內容稍做修改:

<font face="sans">
``` plain gdb.cmds
	b test
	commands 1
	  bt
	  continue
	end
	run
```
</font>

很不錯, 在修改gdb.cmds後可以印出callstack, 這樣就可以知道API是怎麼被call的:

<font face="sans">
``` plain Terminal
	bramante@matrix:~/test$ gdb --batch --command=gdb.cmds --args ./test 2
	Breakpoint 1 at 0x40058f: file ./test.c, line 7.
	warning: no loadable sections found in added symbol-file system-supplied DSO at 0x7ffff7ffa000
	
	Breakpoint 1, test (arg=0) at ./test.c:7
	7       ./test.c: No such file or directory.
	#0  test (arg=0) at ./test.c:7
	#1  0x00000000004005e6 in main (argc=2, argv=0x7fffffffe5b8) at ./test.c:18
	@test(arg:0)
	
	Breakpoint 1, test (arg=1) at ./test.c:7
	7       in ./test.c
	#0  test (arg=1) at ./test.c:7
	#1  0x00000000004005e6 in main (argc=2, argv=0x7fffffffe5b8) at ./test.c:18
	@test(arg:1)
	[Inferior 1 (process 3091) exited normally]
	bramante@matrix:~/test$
```
</font>

透過gdb的這招, 可以在沒source code的狀況下, 深入了解third-party程式的運作方式.
