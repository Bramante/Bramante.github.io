---
layout: post
title: "Fallocate System Call"
date: 2015-09-11 23:39:45 +0800
comments: true
categories: 
---

嵌入式系統的儲存空間相當有限, 如果在下載檔案前, 
先把檔案按預期的大小給建立起來, 下載時再來寫入真正的檔案內容, 
這樣便能保證不會發生下載到一半卻出現Disk Full的狀況.

Linux系統有個fallocate system call可以讓我們快速建立內容全為0x00的檔案,
相當適合用來處理這樣的問題.

以下的程式會利用fallocate system call, 建立多個1GB大小的file, 
直到無法再寫檔為止:

``` c test.c
	#include <sys/stat.h>
	#include <sys/syscall.h>
	#include <sys/types.h>
	#include <fcntl.h>
	#include <stdio.h>
	#include <stdlib.h>
	#include <unistd.h>
	#include <ctype.h>
	#include <linux/falloc.h>
	
	int main(int argc, char **argv)
	{
	 FILE* fh ;
	 char fname[128] ;
	 loff_t  length = 1024*1024*1024 ;
	 loff_t  offset = 0 ;
	 int falloc_mode = 0 ;
	 int error;
	 int i ;
	 for( i = 0 ; i < 200 ; i ++ )
	    {
	     sprintf(fname,"./data%04d",i);
	     fh = fopen( fname,"w+");
	     if(fh)
	       {
	        error = syscall(SYS_fallocate, fileno(fh), falloc_mode, offset, length);
	        if(error < 0)
	          {
	           printf("[*][error] no space !!!\n");
	           perror("fallocate failed");
	           exit(EXIT_FAILURE);
	          }
	        fclose(fh);
	       }
	    }
	 return 0;
	}
```

編譯與執行:

<font face="sans">
``` plain Terminal
	bramante@matrix:~/test/fallocate$ gcc -o ./test ./test.c
	bramante@matrix:~/test/fallocate$ time ./test
	[*][error] no space !!!
	fallocate failed: Disk quota exceeded
	
	real    0m0.897s
	user    0m0.000s
	sys     0m0.100s
	bramante@matrix:~/test/fallocate$ ll
	total 92309096
	drwxrwxr-x 2 bramante bramante       4096 Sep  8 19:04 ./
	drwxr-xr-x 6 bramante bramante       4096 Sep  8 18:16 ../
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0000
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0001
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0002
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0003
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0004
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0005
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0006
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0007
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0008
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0009
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0010
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0011
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0012
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0013
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0014
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0015
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0016
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0017
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0018
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0019
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0020
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0021
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0022
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0023
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0024
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0025
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0026
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0027
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0028
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0029
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0030
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0031
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0032
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0033
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0034
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0035
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0036
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0037
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0038
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0039
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0040
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0041
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0042
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0043
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0044
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0045
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0046
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0047
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0048
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0049
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0050
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0051
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0052
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0053
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0054
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0055
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0056
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0057
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0058
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0059
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0060
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0061
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0062
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0063
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0064
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0065
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0066
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0067
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0068
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0069
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0070
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0071
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0072
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0073
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0074
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0075
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0076
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0077
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0078
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0079
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0080
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0081
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0082
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0083
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0084
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0085
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0086
	-rw-rw-r-- 1 bramante bramante 1073741824 Sep  8 19:04 data0087
	-rw-rw-r-- 1 bramante bramante   34848768 Sep  8 19:04 data0088
	-rwxrwxr-x 1 bramante bramante       8799 Sep  8 19:04 test*
	-rw-rw-r-- 1 bramante bramante        721 Sep  8 19:04 test.c
	bramante@matrix:~/test/fallocate$
```
</font>

不到1 sec可以建立出超過88 GB的檔案, 真的是好快,
而且disk quota用盡時, 程式還能順利停下來,
實在是很不錯.

從以下的執行範例, 可以看出建檔與砍檔都很快, 
都不用1 sec, 而且檔案的確有內容, 內容都為0x00 :

<font face="sans">
``` plain Terminal
	bramante@matrix:~/test/fallocate$ time ./test
	[*][error] no space !!!
	fallocate failed: Disk quota exceeded
	
	real    0m0.741s
	user    0m0.000s
	sys     0m0.100s
	bramante@matrix:~/test/fallocate$ hexdump -C data0087
	00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
	*
	40000000
	bramante@matrix:~/test/fallocate$ hexdump -C data0088
	00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
	*
	0213a000
	bramante@matrix:~/test/fallocate$ hexdump -C data0001
	00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
	*
	40000000
	bramante@matrix:~/test/fallocate$ time rm data0*
	
	real    0m0.864s
	user    0m0.004s
	sys     0m0.820s
	bramante@matrix:~/test/fallocate$
```
</font>

另外要提一下, fallocate其實也可以用來把file裡的內容給刪掉一段,
例如, 刪除file開頭的4096 byte:

	syscall(SYS_fallocate, fileno(fh), FALLOC_FL_COLLAPSE_RANGE, 0, 4096);
	
這個功能滿強大的, 如果能用的話, 刪除檔案裡一些不需要的內容會很快, 
只可惜我的Linux server太舊了, 這個功能沒有試成功.