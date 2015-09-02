---
layout: post
title: "Download the source code of the Ginga"
date: 2015-09-03 00:01:01 +0800
comments: true
categories: 
---

Ginga是巴西DTV上的Middleware, 它的source code是open source的, 可以用下面的command把source code給抓回來:

<font face="sans">
``` plain Terminal
	bramante@matrix:~$ git clone http://git.telemidia.puc-rio.br/ginga.git
	Cloning into 'ginga'...
	bramante@matrix:~$ cd ginga/
	bramante@matrix:~/ginga$ ll
	total 184
	drwxr-xr-x 25 bramante bramante  4096 Sep  2 23:52 ./
	drwxr-xr-x  6 bramante bramante  4096 Sep  2 23:36 ../
	-rw-rw-r--  1 bramante bramante    96 Sep  2 23:52 AUTHORS
	-rwxrwxr-x  1 bramante bramante   292 Sep  2 23:52 bootstrap*
	drwxrwxr-x  2 bramante bramante  4096 Sep  2 23:52 build-aux/
	-rw-rw-r--  1 bramante bramante     0 Sep  2 23:52 ChangeLog
	drwxrwxr-x  2 bramante bramante  4096 Sep  2 23:52 config/
	-rw-rw-r--  1 bramante bramante 17007 Sep  2 23:52 configure.ac
	drwxrwxr-x  2 bramante bramante  4096 Sep  2 23:52 contrib/
	-rw-rw-r--  1 bramante bramante 18009 Sep  2 23:52 COPYING
	drwxrwxr-x  4 bramante bramante  4096 Sep  2 23:52 debian/
	drwxrwxr-x  3 bramante bramante  4096 Sep  2 23:52 ginga/
	drwxrwxr-x  6 bramante bramante  4096 Sep  2 23:52 gingacc-cm/
	drwxrwxr-x  6 bramante bramante  4096 Sep  2 23:52 gingacc-contextmanager/
	drwxrwxr-x  5 bramante bramante  4096 Sep  2 23:52 gingacc-dataprocessing/
	drwxrwxr-x  5 bramante bramante  4096 Sep  2 23:52 gingacc-ic/
	drwxrwxr-x  6 bramante bramante  4096 Sep  2 23:52 gingacc-mb/
	drwxrwxr-x  5 bramante bramante  4096 Sep  2 23:52 gingacc-multidevice/
	drwxrwxr-x  6 bramante bramante  4096 Sep  2 23:52 gingacc-player/
	drwxrwxr-x  4 bramante bramante  4096 Sep  2 23:52 gingacc-system/
	drwxrwxr-x  5 bramante bramante  4096 Sep  2 23:52 gingacc-tsparser/
	drwxrwxr-x  6 bramante bramante  4096 Sep  2 23:52 gingacc-tuner/
	drwxrwxr-x  5 bramante bramante  4096 Sep  2 23:52 gingacc-um/
	drwxrwxr-x  5 bramante bramante  4096 Sep  2 23:52 gingalssm/
	drwxrwxr-x  6 bramante bramante  4096 Sep  2 23:52 gingancl/
	drwxrwxr-x  6 bramante bramante  4096 Sep  2 23:52 ginga-vs2010-solution/
	drwxrwxr-x  8 bramante bramante  4096 Sep  2 23:52 .git/
	-rw-rw-r--  1 bramante bramante  1406 Sep  2 23:52 .gitignore
	-rw-rw-r--  1 bramante bramante  9152 Sep  2 23:52 LICENSING
	-rw-rw-r--  1 bramante bramante   183 Sep  2 23:52 maint.mk
	-rw-rw-r--  1 bramante bramante  2066 Sep  2 23:52 Makefile.am
	drwxrwxr-x  4 bramante bramante  4096 Sep  2 23:52 ncl30/
	drwxrwxr-x  4 bramante bramante  4096 Sep  2 23:52 ncl30-converter/
	-rw-rw-r--  1 bramante bramante     0 Sep  2 23:52 NEWS
	-rw-rw-r--  1 bramante bramante   233 Sep  2 23:52 README
	-rw-rw-r--  1 bramante bramante  2066 Sep  2 23:52 README.linux
	-rw-rw-r--  1 bramante bramante   188 Sep  2 23:52 README.windows
	drwxrwxr-x  5 bramante bramante  4096 Sep  2 23:52 telemidia-util/
	bramante@matrix:~/ginga$
```
</font>
