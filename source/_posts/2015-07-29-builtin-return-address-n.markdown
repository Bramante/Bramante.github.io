---
layout: post
title: "Builtin Return Address"
date: 2015-07-29 23:43:20 +0800
comments: true
categories: 
---

在debug時, 若想知道是哪裡的code來call了某個API,
可以把\_\_builtin_return_address(0)的value給印出來,
然後再透過addr2line查出line number.

下面的範例是追查哪裡的code去call了funcA().

``` c ret.c
	     1  #include <stdio.h>
	     2
	     3  void funcA(void)
	     4  {
	     5   printf("ret:0x%08x\n",__builtin_return_address(0));
	     6  }
	     7
	     8  int main(void)
	     9  {
	    10   funcA();
	    11   return 0 ;
	    12  }
```

<font face="sans">
``` plain Terminal
	bramante@matrix:~/test$ gcc -g -o ret ret.c
	bramante@matrix:~/test$ ./ret
	ret:0x0040051f
	bramante@matrix:~/test$ addr2line -Cfe ./ret 40051f
	main
	/home/bramante/test/ret.c:11
	bramante@matrix:~/test$
```
</font>

addr2line查到的line number是執行完該API後的下一行,
如果想得到call API的那行的line number,
可以把address減1之後再查line number.

例如, 0x40051f - 1 = 0x40051e, 然後查0x40051e的line number.

<font face="sans">
``` plain Terminal
	bramante@matrix:~/test$ addr2line -Cfe ./ret 40051e
	main
	/home/bramante/test/ret.c:10
	bramante@matrix:~/test$
```
</font>

x86對 \_\_builtin_return_address(n)的支援相當完整,
幾乎可以當成backtrace()來用,
MIPS和ARM對 \_\_builtin_return_address()的支援則相當有限,
只支援了 \__builtin_return_address(0),
而且還必需在API內的第一行就取值,
不然有可能會出錯.

在MIPS或ARM上, \_\_builtin_return_address(0)要放在funcA()開頭的第一行,
才能確保得到的return address的value是正確的.

``` c funcA()
	void funcA(void)
	{
	 unsigned int ret = (unsigned int) __builtin_return_address(0) ;//必須在API開頭的第一行就取值
	 .... 
	 printf("ret:0x%08x\n",ret);
	}
```