---
layout: post
title: "GDB Debugging Skills (5)"
date: 2015-08-18 09:24:13 +0800
comments: true
categories: 
---

debug時必需要能夠掌握程式的flow,
API如果有多個return點,
必須要知道程式到底從哪一個位置return,
若要透過gdb取得這方面的資訊,
手動加break point是比較麻煩的,
較為自動化的方法可以使用gdb的reverse commands.

下面示範如何在gdb的batch mode去attach執行中的process, 
並印出API在哪一行return的.

範例程式:

``` c test.c
	#include <stdio.h>
	#include <time.h>
	#include <stdlib.h>
	
	unsigned int test(unsigned int arg)
	{
	 arg %= 4 ;
	 switch(arg)
	   {
	    case 0:
	           return arg ;
	           break;
	    case 1:
	           return arg ;
	           break;
	    case 2:
	           return arg ;
	           break;
	    case 3:
	    default:
	           return arg ;
	           break;
	   }
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
	bramante@matrix:~/test$ ll
	total 16
	drwxrwxr-x 2 bramante bramante 4096 Aug 18 09:37 ./
	drwxr-xr-x 5 bramante bramante 4096 Aug 11 23:32 ../
	-rw-rw-r-- 1 bramante bramante  154 Aug 18 09:20 gdb.cmds
	-rw-rw-r-- 1 bramante bramante  581 Aug 18 09:18 test.c
	bramante@matrix:~/test$ gcc -g -o ./test ./test.c
	bramante@matrix:~/test$ ./test 15 > /dev/null &
	[1] 2927
	bramante@matrix:~/test$ sudo gdb --batch --command=gdb.cmds --pid=2927
	[sudo] password for bramante:
	0x00007f287fe2a090 in nanosleep () from /lib/x86_64-linux-gnu/libc.so.6
	Breakpoint 1 at 0x40054b: file ./test.c, line 7.
	
	Breakpoint 1, test (arg=10) at ./test.c:7
	7        arg %= 4 ;
	main (argc=2, argv=0x7fff02d8e6b8) at ./test.c:36
	36              sleep(1);
	Value returned is $1 = 2
	test (arg=2) at ./test.c:25
	25      }
	17                 return arg ;
	arg: 2
	Process record is stopped and all execution logs are deleted.
	
	Breakpoint 1, test (arg=11) at ./test.c:7
	7        arg %= 4 ;
	main (argc=2, argv=0x7fff02d8e6b8) at ./test.c:36
	36              sleep(1);
	Value returned is $2 = 3
	test (arg=3) at ./test.c:25
	25      }
	21                 return arg ;
	arg: 3
	Process record is stopped and all execution logs are deleted.
	
	Breakpoint 1, test (arg=12) at ./test.c:7
	7        arg %= 4 ;
	main (argc=2, argv=0x7fff02d8e6b8) at ./test.c:36
	36              sleep(1);
	Value returned is $3 = 0
	test (arg=0) at ./test.c:25
	25      }
	11                 return arg ;
	arg: 0
	Process record is stopped and all execution logs are deleted.
	
	Breakpoint 1, test (arg=13) at ./test.c:7
	7        arg %= 4 ;
	main (argc=2, argv=0x7fff02d8e6b8) at ./test.c:36
	36              sleep(1);
	Value returned is $4 = 1
	test (arg=1) at ./test.c:25
	25      }
	14                 return arg ;
	arg: 1
	Process record is stopped and all execution logs are deleted.
	
	Breakpoint 1, test (arg=14) at ./test.c:7
	7        arg %= 4 ;
	main (argc=2, argv=0x7fff02d8e6b8) at ./test.c:36
	36              sleep(1);
	Value returned is $5 = 2
	test (arg=2) at ./test.c:25
	25      }
	17                 return arg ;
	arg: 2
	Process record is stopped and all execution logs are deleted.
	[Inferior 1 (process 2927) exited normally]
	[1]+  Done                    ./test 15 > /dev/null
	bramante@matrix:~/test$
```
</font>

摘要出上面log裡的"return arg", 可以知道return點有4個, 
分別在 11, 14, 17, 21 行:

	17                 return arg ;
	21                 return arg ;
	11                 return arg ;
	14                 return arg ;
	17                 return arg ;


前面所用到的gdb.cmds檔案的內容如下:

<font face="sans">
``` plain gdb.cmds
	bramante@matrix:~/test$ cat gdb.cmds
	b test
	
	define printFuncInfo
	  record
	  finish
	  rs
	  rn
	  printf "arg: %d\n", arg
	  record stop
	  continue
	end
	
	commands 1
	  printFuncInfo
	end
	
	continue
	bramante@matrix:~/test$
```
</font>

要使用reverse command前要先record, 
使用後要record stop,
其中的rs是reverse-step,
rn則是reverse-next,
reverse command的執行方向與正常的command相反.
由於finish時已經回到main(), 因此需要先執行rs回到test(), 
再執行rn找到return點.

沒有source code時的執行狀況如下:

<font face="sans">
``` plain Terminal
	bramante@matrix:~/test$ ./test 10 > /dev/null &
	[1] 2968
	bramante@matrix:~/test$ sudo gdb --batch --command=gdb.cmds --pid=2968
	0x00007f4f518f3090 in nanosleep () from /lib/x86_64-linux-gnu/libc.so.6
	Breakpoint 1 at 0x40054b: file ./test.c, line 7.
	
	Breakpoint 1, test (arg=5) at ./test.c:7
	7       ./test.c: No such file or directory.
	main (argc=2, argv=0x7fff9f6e2d98) at ./test.c:36
	36      in ./test.c
	Value returned is $1 = 1
	test (arg=1) at ./test.c:25
	25      in ./test.c
	14      in ./test.c
	arg: 1
	Process record is stopped and all execution logs are deleted.
	
	Breakpoint 1, test (arg=6) at ./test.c:7
	7       in ./test.c
	main (argc=2, argv=0x7fff9f6e2d98) at ./test.c:36
	36      in ./test.c
	Value returned is $2 = 2
	test (arg=2) at ./test.c:25
	25      in ./test.c
	17      in ./test.c
	arg: 2
	Process record is stopped and all execution logs are deleted.
	
	Breakpoint 1, test (arg=7) at ./test.c:7
	7       in ./test.c
	main (argc=2, argv=0x7fff9f6e2d98) at ./test.c:36
	36      in ./test.c
	Value returned is $3 = 3
	test (arg=3) at ./test.c:25
	25      in ./test.c
	21      in ./test.c
	arg: 3
	Process record is stopped and all execution logs are deleted.
	
	Breakpoint 1, test (arg=8) at ./test.c:7
	7       in ./test.c
	main (argc=2, argv=0x7fff9f6e2d98) at ./test.c:36
	36      in ./test.c
	Value returned is $4 = 0
	test (arg=0) at ./test.c:25
	25      in ./test.c
	11      in ./test.c
	arg: 0
	Process record is stopped and all execution logs are deleted.
	
	Breakpoint 1, test (arg=9) at ./test.c:7
	7       in ./test.c
	main (argc=2, argv=0x7fff9f6e2d98) at ./test.c:36
	36      in ./test.c
	Value returned is $5 = 1
	test (arg=1) at ./test.c:25
	25      in ./test.c
	14      in ./test.c
	arg: 1
	Process record is stopped and all execution logs are deleted.
	[Inferior 1 (process 2968) exited normally]
	[1]+  Done                    ./test 10 > /dev/null
	bramante@matrix:~/test$
```
</font>


在 http://www.gnu.org/software/gdb/news
 有提到, GDB 7.5之後才支援ARM.
