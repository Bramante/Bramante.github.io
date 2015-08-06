---
layout: post
title: "Send signal to thread"
date: 2015-08-07 00:39:25 +0800
comments: true
categories: 
---

使用pthread_kill()發signal給thread, 可以做到在run-time去調整thread的行為, 下面這個範例是發signal給thread, 把thread給結束掉.

``` c test.c
	#include <stdio.h>
	#include <pthread.h>
	#include <unistd.h>
	#include <stdlib.h>
	#include <signal.h>
	#include <time.h>
	#include <errno.h>
	        
	#define THREAD_SLEEP(ms) {\
	         struct timespec delay, rem ;\
	         delay.tv_sec = ms / 1000 ;\
	         delay.tv_nsec = ((ms % 1000)*1000000) ;\
	         while(nanosleep(&delay, &rem) == -1 && errno == EINTR) delay = rem;\
	        }         
	        
	pthread_t th1 ;
	pthread_t th2 ;
	
	void signalHandler(int arg)
	{
	 printf("\nsignalHandler\n");
	 printf("\nsignalHandler(): %lu\n", pthread_self());
	 pthread_exit(0);
	}
	
	void* threadFunction1(void* arg)
	{
	 printf("\nthreadFunction1(): %lu\n", pthread_self());
	 printf("\nthreadFunction1\n");
	 while(1)
	      {
	       printf("1");
	       THREAD_SLEEP(10);
	      }
	 return NULL ;
	}
	
	void* threadFunction2(void* arg)
	{
	 printf("\nthreadFunction2(): %lu\n", pthread_self());
	 printf("\nthreadFunction2\n");
	 while(1)
	      {
	       printf("2");
	       THREAD_SLEEP(10);
	      }
	 return NULL ;
	}
	
	int main(void)
	{
	 printf("\nmain(): %lu\n", pthread_self());
	 struct sigaction newact ;
	 newact.sa_handler = signalHandler ;
	 sigaction(SIGALRM, &newact, NULL);
	 pthread_create(&th1, NULL, &threadFunction1, NULL) ;
	 pthread_create(&th2, NULL, &threadFunction2, NULL) ;
	 sleep(1);
	 pthread_kill( th1, SIGALRM );
	 pthread_join(th1, NULL );
	 printf("\njoin1\n");
	 sleep(1);
	 pthread_kill( th2, SIGALRM );
	 pthread_join(th2, NULL );
	 printf("\njoin2\n");
	 return 0 ;
	}
```


<font face="sans">
``` plain Terminal
	bramante@matrix:~/thread$ gcc -o ./test ./test.c -lpthread
	bramante@matrix:~/thread$ ./test
	
	main(): 140158652921600
	
	threadFunction2(): 140158636234496
	
	threadFunction2
	2
	threadFunction1(): 140158644627200
	
	threadFunction1
	121212121211212122121121221121221121221121221122121121221122112211221122112212112122112212112122112122112211221122112122112212112211221122112211221122112211221122112211221122112211212
	signalHandler
	
	signalHandler(): 140158644627200
	
	join1
	2222222222222222222222222222222222222222222222222222222222222222222222222222222222222222222
	signalHandler
	
	signalHandler(): 140158636234496
	
	join2
	bramante@matrix:~/thread$
```
</font>