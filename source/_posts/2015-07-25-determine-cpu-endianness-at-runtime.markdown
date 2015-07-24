---
layout: post
title: "Determine CPU endianness at runtime"
date: 2015-07-25 00:33:49 +0800
comments: true
categories: 
---

之前工作上用的CPU由Big Endian改成了Little Endian,
同樣的code會在不同Endian的CPU上跑,
因此寫了段能在runtime偵測Endian的code,
這樣可以用來偵測程式跑在哪個Chip上.

``` c endianness.c
	
	#include <stdio.h>
	
	int main(void)
	{
	 unsigned int a = 1 ;
	 unsigned char *ptr ;
	 ptr = (unsigned char *)&a ;
	 if(ptr[0]==0)
	   {
	    printf("big endian\n");	
	   }
	  else
	   {
	    printf("little endian\n");	
	   } 
	 return 0 ;
	}
	
```

<font face="sans">
``` plain Terminal
	bramante@matrix:~/endianness$ gcc -o endianness endianness.c
	bramante@matrix:~/endianness$ ./endianness
	little endian
	bramante@matrix:~/endianness$
```
</font>
