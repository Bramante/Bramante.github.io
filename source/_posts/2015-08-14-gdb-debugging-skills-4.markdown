---
layout: post
title: "GDB Debugging Skills (4)"
date: 2015-08-14 00:20:40 +0800
comments: true
categories: 
---

在複雜的系統裡, 要用gdb把程式帶(跑)起來, 有時是比較麻煩的, 
這時用attach process的方式會比較方便. 
下面示範如何在gdb的batch mode去attach執行中的process, 
並印出API的參數與它的return value.

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
	bramante@matrix:~/test$ ll
	total 28
	drwxrwxr-x 2 bramante bramante  4096 Aug 14 00:24 ./
	drwxr-xr-x 5 bramante bramante  4096 Aug 11 23:32 ../
	-rw-rw-r-- 1 bramante bramante   107 Aug 14 00:19 gdb.cmds
	-rwxrwxr-x 1 bramante bramante 10068 Aug 14 00:24 test*
	-rw-rw-r-- 1 bramante bramante   330 Aug 13 00:04 test.c
	bramante@matrix:~/test$ ./test 20 > /dev/null &
	[4] 2896
	bramante@matrix:~/test$ gdb --batch --command=gdb.cmds --pid=2896
	Could not attach to process.  If your uid matches the uid of the target
	process, check the setting of /proc/sys/kernel/yama/ptrace_scope, or try
	again as the root user.  For more details, see /etc/sysctl.d/10-ptrace.conf
	ptrace: Operation not permitted.
	No symbol table is loaded.  Use the "file" command.
	Make breakpoint pending on future shared library load? (y or [n]) [answered N; input not from terminal]
	No breakpoint number 1.
	gdb.cmds:9: Error in sourced command file:
	No breakpoints specified.
	bramante@matrix:~/test$ sudo gdb --batch --command=gdb.cmds --pid=2896
	
	warning: no loadable sections found in added symbol-file system-supplied DSO at 0x7fff8c5fe000
	0x00007f8021bd5090 in nanosleep () from /lib/x86_64-linux-gnu/libc.so.6
	Breakpoint 1 at 0x40058f: file ./test.c, line 7.
	
	Breakpoint 1, test (arg=15) at ./test.c:7
	7        printf("@test(arg:%d)\n",arg);
	arg = 15
	main (argc=2, argv=0x7fff8c4736a8) at ./test.c:20
	20              sleep(1);
	Value returned is $1 = 15
	
	Breakpoint 1, test (arg=16) at ./test.c:7
	7        printf("@test(arg:%d)\n",arg);
	arg = 16
	main (argc=2, argv=0x7fff8c4736a8) at ./test.c:20
	20              sleep(1);
	Value returned is $2 = 16
	
	Breakpoint 1, test (arg=17) at ./test.c:7
	7        printf("@test(arg:%d)\n",arg);
	arg = 17
	main (argc=2, argv=0x7fff8c4736a8) at ./test.c:20
	20              sleep(1);
	Value returned is $3 = 17
	
	Breakpoint 1, test (arg=18) at ./test.c:7
	7        printf("@test(arg:%d)\n",arg);
	arg = 18
	main (argc=2, argv=0x7fff8c4736a8) at ./test.c:20
	20              sleep(1);
	Value returned is $4 = 18
	
	Breakpoint 1, test (arg=19) at ./test.c:7
	7        printf("@test(arg:%d)\n",arg);
	arg = 19
	main (argc=2, argv=0x7fff8c4736a8) at ./test.c:20
	20              sleep(1);
	Value returned is $5 = 19
	[Inferior 1 (process 2896) exited normally]
	[4]+  Done                    ./test 20 > /dev/null
	bramante@matrix:~/test$
```
</font>

gdb順利印出API的參數與它的return value.
請留意, 需要有root的權限才能成功的attach process.

前面所用到的gdb.cmds檔案的內容如下, 請留意, attach之後是用continue而不是run:

<font face="sans">
``` plain gdb.cmds
	bramante@matrix:~/test$ cat gdb.cmds
	b test
	
	define printFuncInfo
	  info args
	  finish
	  continue
	end
	
	commands 1
	  printFuncInfo
	end
	
	continue
	bramante@matrix:~/test$
```
</font>

在沒有source code的狀況下, gdb一樣是可以印值的(重要的是debug symbol而不是source code):

<font face="sans">
``` plain Terminal
	bramante@matrix:~/test$ rm test.c
	bramante@matrix:~/test$ ll
	total 24
	drwxrwxr-x 2 bramante bramante  4096 Aug 14 00:39 ./
	drwxr-xr-x 5 bramante bramante  4096 Aug 11 23:32 ../
	-rw-rw-r-- 1 bramante bramante   107 Aug 14 00:19 gdb.cmds
	-rwxrwxr-x 1 bramante bramante 10068 Aug 14 00:28 test*
	bramante@matrix:~/test$ ./test 15 > /dev/null &
	[3] 2933
	bramante@matrix:~/test$ sudo gdb --batch --command=gdb.cmds --pid=2933
	
	warning: no loadable sections found in added symbol-file system-supplied DSO at 0x7fff965d7000
	0x00007f9148f53090 in nanosleep () from /lib/x86_64-linux-gnu/libc.so.6
	Breakpoint 1 at 0x40058f: file ./test.c, line 7.
	
	Breakpoint 1, test (arg=9) at ./test.c:7
	7       ./test.c: No such file or directory.
	arg = 9
	main (argc=2, argv=0x7fff9642daf8) at ./test.c:20
	20      in ./test.c
	Value returned is $1 = 9
	
	Breakpoint 1, test (arg=10) at ./test.c:7
	7       in ./test.c
	arg = 10
	main (argc=2, argv=0x7fff9642daf8) at ./test.c:20
	20      in ./test.c
	Value returned is $2 = 10
	
	Breakpoint 1, test (arg=11) at ./test.c:7
	7       in ./test.c
	arg = 11
	main (argc=2, argv=0x7fff9642daf8) at ./test.c:20
	20      in ./test.c
	Value returned is $3 = 11
	
	Breakpoint 1, test (arg=12) at ./test.c:7
	7       in ./test.c
	arg = 12
	main (argc=2, argv=0x7fff9642daf8) at ./test.c:20
	20      in ./test.c
	Value returned is $4 = 12
	
	Breakpoint 1, test (arg=13) at ./test.c:7
	7       in ./test.c
	arg = 13
	main (argc=2, argv=0x7fff9642daf8) at ./test.c:20
	20      in ./test.c
	Value returned is $5 = 13
	
	Breakpoint 1, test (arg=14) at ./test.c:7
	7       in ./test.c
	arg = 14
	main (argc=2, argv=0x7fff9642daf8) at ./test.c:20
	20      in ./test.c
	Value returned is $6 = 14
	[Inferior 1 (process 2933) exited normally]
	[3]+  Done                    ./test 15 > /dev/null
	bramante@matrix:~/test$
```
</font>
