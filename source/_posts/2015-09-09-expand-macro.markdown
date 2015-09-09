---
layout: post
title: "Expand Macro"
date: 2015-09-09 09:22:22 +0800
comments: true
categories: 
---

閱讀別人所寫的程式, 如果遇到複雜的Macro,
不太能確定執行後會有什麼結果時,
可以使用Expand Macro的技巧,
把Macro給展開, 這樣程式應該會變得比較容易了解一點.

範例程式:

``` c test.c
	#include <stdio.h>
	
	#define SUM( a, b ) (a+b)
	
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

利用gcc的"-E"參數, 可以把Macro給展開:

<font face="sans">
``` plain Terminal
	bramante@matrix:~/test/Expand_Macro$ gcc -o ./test ./test.c
	bramante@matrix:~/test/Expand_Macro$ ./test
	c = 3
	bramante@matrix:~/test/Expand_Macro$ gcc -E ./test.c | grep -A8 "int main(void)"
	int main(void)
	{
	 int a = 1 ;
	 int b = 2 ;
	 int c ;
	 c = (a+b);
	 printf("c = %d\n",c);
	 return 0 ;
	}
	bramante@matrix:~/test/Expand_Macro$
```
</font>
