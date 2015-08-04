---
layout: post
title: "Getting the backtrace for all the threads in GDB"
date: 2015-08-04 23:57:27 +0800
comments: true
categories: 
---

在嵌入式系統上debug, 有時gdb的運作不是很可靠, 可能會遇到執行幾個指令之後就無法再繼續用的情形, 如果正在處理multi-thread hang住的bug, 那可麻煩了, 可能bt沒幾個thread的callstack, gdb就沒辦法再用了, 這時改用"thread apply all bt"是最佳的選擇, 使用一行command就可以把所有thread的callstack都給dump出來. 

``` c test.c
	#include <stdio.h>
	#include <pthread.h>
	#include <time.h>
	
	#define THREAD_SLEEP(ms) {\
	         struct timespec delay ;\
	         delay.tv_sec = ms / 1000 ;\
	         delay.tv_nsec = ((ms % 1000)*1000000) ;\
	         nanosleep(&delay,NULL) ;\
	        }
	        
	pthread_t th1 ;
	pthread_t th2 ;
	
	void* threadFunction1(void* arg)
	{
	 printf("\nthreadFunction1(): %lu\n", pthread_self());
	 while(1)
	      {
	       THREAD_SLEEP(10);
	      }
	 return NULL ;
	}
	
	void* threadFunction2(void* arg)
	{
	 printf("\nthreadFunction2(): %lu\n", pthread_self());
	 while(1)
	      {
	       THREAD_SLEEP(10);
	      }
	 return NULL ;
	}
	
	int main(void)
	{
	 printf("\nmain(): %lu\n", pthread_self());
	 pthread_create(&th1, NULL, &threadFunction1, NULL) ;
	 pthread_create(&th2, NULL, &threadFunction2, NULL) ;
	 while(1)
	      {
	       THREAD_SLEEP(10);
	      }
	 pthread_join(th1, NULL );
	 pthread_join(th2, NULL );
	 return 0 ;
	}
```

<font face="sans">
``` plain Terminal
	bramante@matrix:~/thread$ gcc -o ./test ./test.c -lpthread
	bramante@matrix:~/thread$ gdb ./test
	GNU gdb (Ubuntu/Linaro 7.4-2012.04-0ubuntu2.1) 7.4-2012.04
	Copyright (C) 2012 Free Software Foundation, Inc.
	License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
	This is free software: you are free to change and redistribute it.
	There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
	and "show warranty" for details.
	This GDB was configured as "x86_64-linux-gnu".
	For bug reporting instructions, please see:
	<http://bugs.launchpad.net/gdb-linaro/>...
	Reading symbols from /home/bramante/thread/test...(no debugging symbols found)...done.
	(gdb) run
	Starting program: /home/bramante/thread/test
	warning: no loadable sections found in added symbol-file system-supplied DSO at 0x7ffff7ffa000
	[Thread debugging using libthread_db enabled]
	Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".
	
	main(): 140737354028800
	[New Thread 0x7ffff77fe700 (LWP 3052)]
	
	threadFunction1(): 140737345742592
	[New Thread 0x7ffff6ffd700 (LWP 3053)]
	
	threadFunction2(): 140737337349888
	^C
	Program received signal SIGINT, Interrupt.
	0x00007ffff7bcc52d in nanosleep () from /lib/x86_64-linux-gnu/libpthread.so.0
	(gdb) thread apply all bt
	
	Thread 3 (Thread 0x7ffff6ffd700 (LWP 3053)):
	#0  0x00007ffff7bcc52d in nanosleep () from /lib/x86_64-linux-gnu/libpthread.so.0
	#1  0x00000000004006da in threadFunction2 ()
	#2  0x00007ffff7bc4e9a in start_thread () from /lib/x86_64-linux-gnu/libpthread.so.0
	#3  0x00007ffff78f238d in clone () from /lib/x86_64-linux-gnu/libc.so.6
	#4  0x0000000000000000 in ?? ()
	
	Thread 2 (Thread 0x7ffff77fe700 (LWP 3052)):
	#0  0x00007ffff7bcc52d in nanosleep () from /lib/x86_64-linux-gnu/libpthread.so.0
	#1  0x000000000040068e in threadFunction1 ()
	#2  0x00007ffff7bc4e9a in start_thread () from /lib/x86_64-linux-gnu/libpthread.so.0
	#3  0x00007ffff78f238d in clone () from /lib/x86_64-linux-gnu/libc.so.6
	#4  0x0000000000000000 in ?? ()
	
	Thread 1 (Thread 0x7ffff7fe5700 (LWP 3049)):
	#0  0x00007ffff7bcc52d in nanosleep () from /lib/x86_64-linux-gnu/libpthread.so.0
	#1  0x0000000000400754 in main ()
	(gdb)
```
</font>