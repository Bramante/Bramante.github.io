---
layout: post
title: "fallocate"
date: 2015-09-04 00:10:01 +0800
comments: true
categories: 
---

fallocate command可以用來快速地建立1個file, 以下是操作範例:

<font face="sans">
``` plain Terminal
	bramante@matrix:~/test$ fallocate -o 0 -l 96 data
	bramante@matrix:~/test$ ll
	total 12
	drwxrwxr-x 2 bramante bramante 4096 Sep  4 00:10 ./
	drwxr-xr-x 6 bramante bramante 4096 Sep  2 23:36 ../
	-rw-r--r-- 1 bramante bramante   96 Sep  4 00:10 data
	bramante@matrix:~/test$ hexdump -Cv ./data
	00000000  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
	00000010  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
	00000020  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
	00000030  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
	00000040  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
	00000050  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
	00000060
	bramante@matrix:~/test$
```
</font>

fallocate command也可以用來擴充file的size, 以下是操作範例:

<font face="sans">
``` plain Terminal
	bramante@matrix:~/test$ dd if=/dev/zero bs=1 count=96 | tr '\000' '\377' > data
	96+0 records in
	96+0 records out
	96 bytes (96 B) copied, 0.000357439 s, 269 kB/s
	bramante@matrix:~/test$ hexdump -Cv ./data
	00000000  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
	00000010  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
	00000020  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
	00000030  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
	00000040  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
	00000050  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
	00000060
	bramante@matrix:~/test$ fallocate -o 80 -l 32 data
	bramante@matrix:~/test$ hexdump -Cv ./data
	00000000  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
	00000010  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
	00000020  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
	00000030  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
	00000040  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
	00000050  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
	00000060  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
	00000070
	bramante@matrix:~/test$
```
</font>

"fallocate -o 80 -l 32 data"的意思是, 從data這個檔案的offset 80 byte的位置開始, 
要有32 byte的空間, 由於原本在offset 80之後就有16個byte, 因此檔案最後只擴充了16 byte的size.
 