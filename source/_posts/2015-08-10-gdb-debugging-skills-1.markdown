---
layout: post
title: "GDB Debugging Skills (1)"
date: 2015-08-10 23:33:42 +0800
comments: true
categories: 
---

gdb以Hex格式印出100個byte的array內容: x/100ubx array

gdb以二進制印出變數的value: p /t var

gdb印出function prototype: call function_name

以下是實際操作的範例:

``` c test.c
	#include <stdio.h>
	
	int main(void)
	{
	 unsigned char var = 0x5A ; 
	 unsigned char array[100];
	 int i ;
	 for( i = 0 ; i < 100 ; i ++ )
	    {
	     array[i] = i + 100 ;
	    }
	 return 0 ;
	}
```

<font face="sans">
``` plain Terminal
	bramante@matrix:~/test$ gcc -g -o ./test ./test.c
	bramante@matrix:~/test$ cat -n test.c
	     1  #include <stdio.h>
	     2
	     3  int main(void)
	     4  {
	     5   unsigned char var = 0x5A ;
	     6   unsigned char array[100];
	     7   int i ;
	     8   for( i = 0 ; i < 100 ; i ++ )
	     9      {
	    10       array[i] = i + 100 ;
	    11      }
	    12   return 0 ;
	    13  }
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
	(gdb) b test.c:12
	Breakpoint 1 at 0x400561: file ./test.c, line 12.
	(gdb) run
	Starting program: /home/bramante/test/test
	warning: no loadable sections found in added symbol-file system-supplied DSO at 0x7ffff7ffa000
	
	Breakpoint 1, main () at ./test.c:12
	12       return 0 ;
	(gdb) x/100ubx array
	0x7fffffffe460: 0x64    0x65    0x66    0x67    0x68    0x69    0x6a    0x6b
	0x7fffffffe468: 0x6c    0x6d    0x6e    0x6f    0x70    0x71    0x72    0x73
	0x7fffffffe470: 0x74    0x75    0x76    0x77    0x78    0x79    0x7a    0x7b
	0x7fffffffe478: 0x7c    0x7d    0x7e    0x7f    0x80    0x81    0x82    0x83
	0x7fffffffe480: 0x84    0x85    0x86    0x87    0x88    0x89    0x8a    0x8b
	0x7fffffffe488: 0x8c    0x8d    0x8e    0x8f    0x90    0x91    0x92    0x93
	0x7fffffffe490: 0x94    0x95    0x96    0x97    0x98    0x99    0x9a    0x9b
	0x7fffffffe498: 0x9c    0x9d    0x9e    0x9f    0xa0    0xa1    0xa2    0xa3
	0x7fffffffe4a0: 0xa4    0xa5    0xa6    0xa7    0xa8    0xa9    0xaa    0xab
	0x7fffffffe4a8: 0xac    0xad    0xae    0xaf    0xb0    0xb1    0xb2    0xb3
	0x7fffffffe4b0: 0xb4    0xb5    0xb6    0xb7    0xb8    0xb9    0xba    0xbb
	0x7fffffffe4b8: 0xbc    0xbd    0xbe    0xbf    0xc0    0xc1    0xc2    0xc3
	0x7fffffffe4c0: 0xc4    0xc5    0xc6    0xc7
	(gdb) p /t var
	$1 = 1011010
	(gdb) call main
	$2 = {int (void)} 0x400524 <main>
	(gdb)
```
</font>


在得知function prototype後, 
就能利用LD_PRELOAD來hook function, 
在整合沒有source code的third-party的library時,
這是一個重要的debug技巧.
