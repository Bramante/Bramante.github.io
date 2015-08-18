---
layout: post
title: "GDB Debugging Skills (6)"
date: 2015-08-19 00:53:37 +0800
comments: true
categories: 
---

假設在下面的程式裡, test()這個API是用來取得遙控器的key值的. 
正常狀況下test()只會送出系統所支援的A廠遙控器的key code (100~103).
目前拿到一支B廠的遙控器, 它的key code是0~3, 要怎樣在不改code的狀況下, 
利用gdb的batch mode去attach執行中的process, 
讓gdb幫忙做key code的轉換, 好讓系統能支援B廠的遙控器.

範例程式:

``` c test.c
	#include <stdio.h>
	#include <time.h>
	#include <stdlib.h>
	
	void test(unsigned int* getValue )
	{
	 static unsigned int count = 0 ;
	 *getValue = count % 4 ;
	 count += 1 ;
	 return ;
	}
	
	int main(int argc, char *argv[])
	{
	 if(argc == 2)
	   {
	    int i ;
	    int num = atoi(argv[1]);
	    unsigned int getValue ;
	    for( i = 0 ; i < num ; i ++ )
	       {
	        test(&getValue);
	        printf("[*] value: %d\n",getValue);
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
	drwxrwxr-x 2 bramante bramante 4096 Aug 19 01:08 ./
	drwxr-xr-x 5 bramante bramante 4096 Aug 11 23:32 ../
	-rw-rw-r-- 1 bramante bramante  385 Aug 19 00:50 gdb.cmds
	-rw-rw-r-- 1 bramante bramante  463 Aug 19 00:21 test.c
	bramante@matrix:~/test$ gcc -g -o ./test ./test.c
	bramante@matrix:~/test$ ./test 15 > /dev/null &
	[1] 3027
	bramante@matrix:~/test$ sudo gdb --batch --command=gdb.cmds --pid=3027
	
	warning: no loadable sections found in added symbol-file system-supplied DSO at 0x7fff23dfe000
	0x00007f25b2148090 in nanosleep () from /lib/x86_64-linux-gnu/libc.so.6
	Breakpoint 1 at 0x40058c: file ./test.c, line 8.
	
	Breakpoint 1, test (getValue=0x7fff23ca24a4) at ./test.c:8
	8        *getValue = count % 4 ;
	main (argc=2, argv=0x7fff23ca2598) at ./test.c:23
	23              printf("[*] value: %d\n",getValue);
	test (getValue=0x7fff23ca24a4) at ./test.c:11
	11      }
	9        count += 1 ;
	Because GDB is in replay mode, writing to memory will make the execution log unusable from this point onward.  Write memory at address 0x7fff23ca24a4?(y or n) [answered Y; input not from terminal]
	*getValue: 103
	Process record is stopped and all execution logs are deleted.
	
	Breakpoint 1, test (getValue=0x7fff23ca24a4) at ./test.c:8
	8        *getValue = count % 4 ;
	main (argc=2, argv=0x7fff23ca2598) at ./test.c:23
	23              printf("[*] value: %d\n",getValue);
	test (getValue=0x7fff23ca24a4) at ./test.c:11
	11      }
	9        count += 1 ;
	Because GDB is in replay mode, writing to memory will make the execution log unusable from this point onward.  Write memory at address 0x7fff23ca24a4?(y or n) [answered Y; input not from terminal]
	*getValue: 100
	Process record is stopped and all execution logs are deleted.
	
	Breakpoint 1, test (getValue=0x7fff23ca24a4) at ./test.c:8
	8        *getValue = count % 4 ;
	main (argc=2, argv=0x7fff23ca2598) at ./test.c:23
	23              printf("[*] value: %d\n",getValue);
	test (getValue=0x7fff23ca24a4) at ./test.c:11
	11      }
	9        count += 1 ;
	Because GDB is in replay mode, writing to memory will make the execution log unusable from this point onward.  Write memory at address 0x7fff23ca24a4?(y or n) [answered Y; input not from terminal]
	*getValue: 101
	Process record is stopped and all execution logs are deleted.
	
	Breakpoint 1, test (getValue=0x7fff23ca24a4) at ./test.c:8
	8        *getValue = count % 4 ;
	main (argc=2, argv=0x7fff23ca2598) at ./test.c:23
	23              printf("[*] value: %d\n",getValue);
	test (getValue=0x7fff23ca24a4) at ./test.c:11
	11      }
	9        count += 1 ;
	Because GDB is in replay mode, writing to memory will make the execution log unusable from this point onward.  Write memory at address 0x7fff23ca24a4?(y or n) [answered Y; input not from terminal]
	*getValue: 102
	Process record is stopped and all execution logs are deleted.
	[Inferior 1 (process 3027) exited normally]
	[1]+  Done                    ./test 15 > /dev/null
	bramante@matrix:~/test$
```
</font>

摘要出上面log裡的"*getValue: 10x", 可以知道gdb的確幫忙做了key code的轉換:

	*getValue: 103
	*getValue: 100
	*getValue: 101
	*getValue: 102

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
	  if *getValue == 0
	    set *getValue=100
	   else
	    if *getValue == 1
	      set *getValue=101
	     else
	      if *getValue == 2
	        set *getValue=102
	       else
	        set *getValue=103
	      end
	    end
	  end
	  printf "*getValue: %d\n", *getValue
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

神奇吧, gdb還能幫忙轉換key code ~
