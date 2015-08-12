---
layout: post
title: "GDB Debugging Skills (3)"
date: 2015-08-13 00:47:52 +0800
comments: true
categories: 
---

在gdb batch mode要怎麼印出每次執行某API的return value, 方法如下.

範例程式:

``` c test.c
	#include <stdio.h>
	#include <time.h>
	#include <stdlib.h>
	
	int test(int arg)
	{
	 printf("@test(arg:%d)\n",arg);
	 return arg ;
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
	bramante@matrix:~/test$ gdb --batch --command=gdb.cmds --args ./test 3
	Breakpoint 1 at 0x40058f: file ./test.c, line 7.
	
	Breakpoint 1, test (arg=0) at ./test.c:7
	7        printf("@test(arg:%d)\n",arg);
	arg = 0
	@test(arg:0)
	main (argc=2, argv=0x7fffffffe5b8) at ./test.c:20
	20              sleep(1);
	Value returned is $1 = 0
	
	Breakpoint 1, test (arg=1) at ./test.c:7
	7        printf("@test(arg:%d)\n",arg);
	arg = 1
	@test(arg:1)
	main (argc=2, argv=0x7fffffffe5b8) at ./test.c:20
	20              sleep(1);
	Value returned is $2 = 1
	
	Breakpoint 1, test (arg=2) at ./test.c:7
	7        printf("@test(arg:%d)\n",arg);
	arg = 2
	@test(arg:2)
	main (argc=2, argv=0x7fffffffe5b8) at ./test.c:20
	20              sleep(1);
	Value returned is $3 = 2
	[Inferior 1 (process 2986) exited normally]
	bramante@matrix:~/test$
```
</font>

可以看到test()被執行了3次, 它的return value都被gdb給印了出來.

前面所用到的gdb.cmds檔案的內容如下:

<font face="sans">
``` plain gdb.cmds
	b test
	
	define printFuncInfo
	  info args
	  finish
	  continue
	end
	
	commands 1
	  printFuncInfo
	end
	
	run
```
</font>

要注意, gdb.cmds的內容不能寫成下面這樣, 不然commands在執行到finish後, 在finish後面的command都不會被執行, 需要寫成像前面那樣, 用個user-defined commands把這些command多包一層, 才能避開這個問題.

<font face="sans">
``` plain 錯誤的gdb.cmds
	b test
	
	commands 1
	  info args
	  finish
	  continue
	end
	
	run
```
</font>