---
layout: post
title: "Define Macro on Command Line"
date: 2015-09-09 09:30:10 +0800
comments: true
categories: 
---

之前曾經遇過在source file與header file裡找不到Macro定義的狀況,
最後才發現Macro被定義在command line上,
以下示範如何在command line上定義Macro.

範例程式:

``` c test.c
	#include <stdio.h>
	
	int main(void)
	{
	 int a = 1 ;
	 int b = 2 ;
	 int c ;
	 c = SUM(a,b);
	 printf("c = %d\n",c);
	 return 0 ;
	}
```

編譯與執行:

<font face="sans">
``` plain Terminal
	bramante@matrix:~/blog/define_macro_on_command_line$ gcc -o ./test "-DSUM(a,b)=(a+b)" ./test.c
	bramante@matrix:~/blog/define_macro_on_command_line$ ./test
	c = 3
	bramante@matrix:~/blog/define_macro_on_command_line$ gcc -E "-DSUM(a,b)=(a+b)" ./test.c | grep -A8 "int main(void)"
	int main(void)
	{
	 int a = 1 ;
	 int b = 2 ;
	 int c ;
	 c = (a+b);
	 printf("c = %d\n",c);
	 return 0 ;
	}
	bramante@matrix:~/blog/define_macro_on_command_line$
```
</font>

從執行的結果, 與展開Macro後的source code,
可以確定在command line上定義Macro是可行的.
