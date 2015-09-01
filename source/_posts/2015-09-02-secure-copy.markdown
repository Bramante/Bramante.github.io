---
layout: post
title: "secure copy"
date: 2015-09-02 00:00:46 +0800
comments: true
categories: 
---

在2台Linux電腦之間copy檔案, 使用scp command (secure copy, remote file copy program)
是相當方便的, 以下是使用範例:

<font face="sans">
``` plain Terminal 192.168.1.2
	bramante@matrix:~$ ll ./test.bin
	-rwxr--r-- 1 bramante bramante 72087592 Sep  1 15:46 ./test.bin*
	bramante@matrix:~$     
	bramante@matrix:~$ scp ./test.bin minions@192.168.1.3:~/
	The authenticity of host '192.168.1.3 (192.168.1.3)' can't be established.
	ECDSA key fingerprint is a1:3e:45:35:f5:23:e2:f3:56:c8:50:bf:22:05:4b:9b.
	Are you sure you want to continue connecting (yes/no)? yes
	Warning: Permanently added '192.168.1.3' (ECDSA) to the list of known hosts.
	minions@192.168.1.3's password:
	test.bin                                                                                                                100%   69MB  11.5MB/s   00:06
	bramante@matrix:~$
```
</font>

<font face="sans">
``` plain Terminal 192.168.1.3
	minions@zion:~$ ll ./test.bin
	-rwxr--r-- 1 minions minions 72087592 Sep  1 15:50 ./test.bin*
	minions@zion:~$
```
</font>

scp也有Windows的版本, 所以在Linux電腦與Windows電腦間copy檔案, 也可以使用scp.