---
layout: post
title: "nanosleep"
date: 2015-08-06 00:16:39 +0800
comments: true
categories: 
---

在nanosleep()的說明裡, 有提到它可能會被signal handler所打斷.

<font face="sans">
``` plain Terminal
	NANOSLEEP(2)                                                         Linux Programmer's Manual                                                         NANOSLEEP(2)
	
	NAME
	       nanosleep - high-resolution sleep
	
	SYNOPSIS
	       #include <time.h>
	
	       int nanosleep(const struct timespec *req, struct timespec *rem);
	
	   Feature Test Macro Requirements for glibc (see feature_test_macros(7)):
	
	       nanosleep(): _POSIX_C_SOURCE >= 199309L
	
	DESCRIPTION
	       nanosleep()  suspends  the  execution  of  the calling thread until either at least the time specified in *req has elapsed, or the delivery of a signal that
	       triggers the invocation of a handler in the calling thread or that terminates the process.
	
	       If the call is interrupted by a signal handler, nanosleep() returns -1, sets errno to EINTR, and writes the remaining time into the structure pointed to  by
	       rem unless rem is NULL.  The value of *rem can then be used to call nanosleep() again and complete the specified pause (but see NOTES).
	
	       The structure timespec is used to specify intervals of time with nanosecond precision.  It is defined as follows:
	
	           struct timespec {
	               time_t tv_sec;        /* seconds */
	               long   tv_nsec;       /* nanoseconds */
	           };
	
	       The value of the nanoseconds field must be in the range 0 to 999999999.
	
	       Compared  to  sleep(3)  and  usleep(3), nanosleep() has the following advantages: it provides a higher resolution for specifying the sleep interval; POSIX.1
	       explicitly specifies that it does not interact with signals; and it makes the task of resuming a sleep that has been interrupted by a signal handler easier.
	
	RETURN VALUE
	       On successfully sleeping for the requested interval, nanosleep() returns 0.  If the call is interrupted by a signal handler or encounters an error, then  it
	       returns -1, with errno set to indicate the error.
```
</font>

如果沒有考量到nanosleep()被signal handler打斷的狀況, sleep的時間有可能會達不到預期的長度. Random發生沒有sleep到充足時間, 有可能會造成程式發生random的bug, 因此不要小看這個問題.

``` c 錯誤的用法
	#define MY_SLEEP(ms) {\
	         struct timespec delay ;\
	         delay.tv_sec = ms / 1000 ;\
	         delay.tv_nsec = ((ms % 1000)*1000000) ;\
	         nanosleep(&delay,NULL) ;\
	        }
```

正確的用法應該如下, 才能確保sleep的時間達到預期的要求.

``` c 正確的用法
	#define MY_SLEEP(ms) {\
	         struct timespec delay, rem ;\
	         delay.tv_sec = ms / 1000 ;\
	         delay.tv_nsec = ((ms % 1000)*1000000) ;\
	         while(nanosleep(&delay, &rem) == -1 && errno == EINTR) delay = rem;\
	        } 
```
