---
layout: post
title: "GNU Make print commands before executing"
date: 2015-10-09 00:27:29 +0800
comments: true
categories: 
---

複雜的大型程式有時很難搞清楚它們是怎麼被build出來的,
如果我們能看到GNU Make在執行Makefile時做了些什麼,
那麼狀況就會變得很清楚.

另外, 像是Android.mk, 
最終負責執行它的內容的程式,
其實也是GNU Make,
所以如果我們能看到GNU Make執行了哪些command,
那我們就可以知道在build Android時,
Android到底用了哪些特別的compiler flag.

我改了一版GNU Make來取得make執行了哪些command的資訊,
build code的方法如下:

<font face="sans">
``` plain Terminal
	bramante@matrix:~$ git clone https://github.com/Bramante/debugging-tools.git
	bramante@matrix:~$ cd debugging-tools/make-3.81/
	bramante@matrix:~/debugging-tools/make-3.81$ ./configure
	bramante@matrix:~/debugging-tools/make-3.81$ make
```
</font>

我改動的code其實就只有下面這幾行:

<font face="sans">
``` plain Terminal
	bramante@matrix:~/debugging-tools/make-3.81$ git log job.c
	commit 127676f45707a6bef7361c146b9b106d4302393a
	Author: bramante <bramante@matrix.(none)>
	Date:   Thu Oct 8 00:04:48 2015 +0800
	
	    +: Making make print commands before executing.
	
	commit d6ded34ecab13b1569186fd4b879fdee98b49f65
	Author: bramante <bramante@matrix.(none)>
	Date:   Wed Oct 7 09:38:08 2015 +0800
	
	    +: make-3.81
	bramante@matrix:~/debugging-tools/make-3.81$ git diff d6ded34ecab13b1569186fd4b879fdee98b49f65 job.c
	diff --git a/make-3.81/job.c b/make-3.81/job.c
	old mode 100644
	new mode 100755
	index a81cd81..a912b36
	--- a/make-3.81/job.c
	+++ b/make-3.81/job.c
	@@ -2073,6 +2073,32 @@ exec_command (char **argv, char **envp)
	
	 # else
	
	+  char* makeLogFileName = getenv("MAKE_LOG_FILE_NAME");
	+  if( makeLogFileName )
	+    {
	+     int i ;
	+     FILE * fh = NULL ;
	+     fh = fopen( makeLogFileName, "a" );
	+     if(fh)
	+       {
	+        fprintf(fh,"[*][argv] ");
	+        for(i=0;argv[i]!=NULL;i++)
	+           {
	+            fprintf(fh,"%s ",argv[i]);
	+           }
	+        fprintf(fh,"\n");
	+        fprintf(fh,"[*][envp] ");
	+        for(i=0;envp[i]!=NULL;i++)
	+           {
	+            fprintf(fh,"%s ",envp[i]);
	+           }
	+        fprintf(fh,"\n");
	+        fflush(fh);
	+        fsync(fileno(fh));
	+        fclose(fh);
	+       }
	+   }
	+
	   /* Run the program.  */
	   environ = envp;
	   execvp (argv[0], argv);
	bramante@matrix:~/debugging-tools/make-3.81$
```
</font>

替換掉原本系統裡的GNU Make, 並設定log file的名稱:

<font face="sans">
``` plain Terminal
	bramante@matrix:~/debugging-tools/make-3.81$ which make
	/usr/bin/make
	bramante@matrix:~/debugging-tools/make-3.81$ ll make
	-rwxrwxr-x 1 bramante bramante 610769 Oct  8 17:10 make*
	bramante@matrix:~/debugging-tools/make-3.81$ cp ./make ../
	bramante@matrix:~/debugging-tools/make-3.81$ cd ..
	bramante@matrix:~/debugging-tools$ ll
	total 616
	drwxr-xr-x  4 bramante bramante   4096 Oct  8 18:20 ./
	drwxr-xr-x 19 bramante bramante   4096 Oct  8 17:09 ../
	drwxrwxr-x  8 bramante bramante   4096 Oct  8 17:09 .git/
	-rwxrwxr-x  1 bramante bramante 610769 Oct  8 18:20 make*
	drwxrwxr-x 10 bramante bramante   4096 Oct  8 17:10 make-3.81/
	bramante@matrix:~/debugging-tools$ pwd
	/home/bramante/debugging-tools
	bramante@matrix:~/debugging-tools$ export PATH="/home/bramante/debugging-tools:$PATH"
	bramante@matrix:~/debugging-tools$ which make
	/home/bramante/debugging-tools/make
	bramante@matrix:~/debugging-tools$ export MAKE_LOG_FILE_NAME="make_log.txt"
```
</font>

重build GNU Make 3.81來取得make執行了哪些command的資訊:

<font face="sans">
``` plain Terminal
	bramante@matrix:~/debugging-tools$ cd -
	/home/bramante/debugging-tools/make-3.81
	bramante@matrix:~/debugging-tools/make-3.81$ make clean ; make
	bramante@matrix:~/debugging-tools/make-3.81$ ll make_log.txt
	-rw-rw-r-- 1 bramante bramante 99665 Oct  8 18:27 make_log.txt
	bramante@matrix:~/debugging-tools/make-3.81$
```
</font>

從log file可以看出make執行過哪些command:

<font face="sans">
``` plain Terminal
	bramante@matrix:~/debugging-tools/make-3.81$ grep "\[argv]" make_log.txt
	[*][argv] /bin/sh -c if test ! -f config.h; then \
	[*][argv] make all-recursive
	[*][argv] /bin/sh -c failcom='exit 1'; \
	[*][argv] /bin/sh -c if test ! -f config.h; then \
	[*][argv] /bin/sh -c failcom='exit 1'; \
	[*][argv] /bin/sh -c test -z "make" || rm -f make
	[*][argv] /bin/sh -c test -z "loadavg" || rm -f loadavg
	[*][argv] rm -f ansi2knr
	[*][argv] /bin/sh -c rm -f *.o
	[*][argv] /bin/sh -c test "" = "" || rm -f *_.c
	[*][argv] /bin/sh -c if test ! -f config.h; then \
	[*][argv] make all-recursive
	[*][argv] /bin/sh -c failcom='exit 1'; \
	[*][argv] /bin/sh -c if test ! -f config.h; then \
	[*][argv] /bin/sh -c if gcc -DLOCALEDIR=\"/usr/local/share/locale\" -DLIBDIR=\"/usr/local/lib\" -DINCLUDEDIR=\"/usr/local/include\" -DHAVE_CONFIG_H -I. -I. -I.      -g -O2 -MT ar.o -MD -MP -MF ".deps/ar.Tpo" -c -o ar.o ar.c; \
	[*][argv] /bin/sh -c if gcc -DLOCALEDIR=\"/usr/local/share/locale\" -DLIBDIR=\"/usr/local/lib\" -DINCLUDEDIR=\"/usr/local/include\" -DHAVE_CONFIG_H -I. -I. -I.      -g -O2 -MT arscan.o -MD -MP -MF ".deps/arscan.Tpo" -c -o arscan.o arscan.c; \
	[*][argv] /bin/sh -c if gcc -DLOCALEDIR=\"/usr/local/share/locale\" -DLIBDIR=\"/usr/local/lib\" -DINCLUDEDIR=\"/usr/local/include\" -DHAVE_CONFIG_H -I. -I. -I.      -g -O2 -MT commands.o -MD -MP -MF ".deps/commands.Tpo" -c -o commands.o commands.c; \
	[*][argv] /bin/sh -c if gcc -DLOCALEDIR=\"/usr/local/share/locale\" -DLIBDIR=\"/usr/local/lib\" -DINCLUDEDIR=\"/usr/local/include\" -DHAVE_CONFIG_H -I. -I. -I.      -g -O2 -MT default.o -MD -MP -MF ".deps/default.Tpo" -c -o default.o default.c; \
	[*][argv] /bin/sh -c if gcc -DLOCALEDIR=\"/usr/local/share/locale\" -DLIBDIR=\"/usr/local/lib\" -DINCLUDEDIR=\"/usr/local/include\" -DHAVE_CONFIG_H -I. -I. -I.      -g -O2 -MT dir.o -MD -MP -MF ".deps/dir.Tpo" -c -o dir.o dir.c; \
	[*][argv] /bin/sh -c if gcc -DLOCALEDIR=\"/usr/local/share/locale\" -DLIBDIR=\"/usr/local/lib\" -DINCLUDEDIR=\"/usr/local/include\" -DHAVE_CONFIG_H -I. -I. -I.      -g -O2 -MT expand.o -MD -MP -MF ".deps/expand.Tpo" -c -o expand.o expand.c; \
	[*][argv] /bin/sh -c if gcc -DLOCALEDIR=\"/usr/local/share/locale\" -DLIBDIR=\"/usr/local/lib\" -DINCLUDEDIR=\"/usr/local/include\" -DHAVE_CONFIG_H -I. -I. -I.      -g -O2 -MT file.o -MD -MP -MF ".deps/file.Tpo" -c -o file.o file.c; \
	[*][argv] /bin/sh -c if gcc -DLOCALEDIR=\"/usr/local/share/locale\" -DLIBDIR=\"/usr/local/lib\" -DINCLUDEDIR=\"/usr/local/include\" -DHAVE_CONFIG_H -I. -I. -I.      -g -O2 -MT function.o -MD -MP -MF ".deps/function.Tpo" -c -o function.o function.c; \
	[*][argv] /bin/sh -c if gcc -DLOCALEDIR=\"/usr/local/share/locale\" -DLIBDIR=\"/usr/local/lib\" -DINCLUDEDIR=\"/usr/local/include\" -DHAVE_CONFIG_H -I. -I. -I.      -g -O2 -MT getopt.o -MD -MP -MF ".deps/getopt.Tpo" -c -o getopt.o getopt.c; \
	[*][argv] /bin/sh -c if gcc -DLOCALEDIR=\"/usr/local/share/locale\" -DLIBDIR=\"/usr/local/lib\" -DINCLUDEDIR=\"/usr/local/include\" -DHAVE_CONFIG_H -I. -I. -I.      -g -O2 -MT getopt1.o -MD -MP -MF ".deps/getopt1.Tpo" -c -o getopt1.o getopt1.c; \
	[*][argv] /bin/sh -c if gcc -DLOCALEDIR=\"/usr/local/share/locale\" -DLIBDIR=\"/usr/local/lib\" -DINCLUDEDIR=\"/usr/local/include\" -DHAVE_CONFIG_H -I. -I. -I.      -g -O2 -MT implicit.o -MD -MP -MF ".deps/implicit.Tpo" -c -o implicit.o implicit.c; \
	[*][argv] /bin/sh -c if gcc -DLOCALEDIR=\"/usr/local/share/locale\" -DLIBDIR=\"/usr/local/lib\" -DINCLUDEDIR=\"/usr/local/include\" -DHAVE_CONFIG_H -I. -I. -I.      -g -O2 -MT job.o -MD -MP -MF ".deps/job.Tpo" -c -o job.o job.c; \
	[*][argv] /bin/sh -c if gcc -DLOCALEDIR=\"/usr/local/share/locale\" -DLIBDIR=\"/usr/local/lib\" -DINCLUDEDIR=\"/usr/local/include\" -DHAVE_CONFIG_H -I. -I. -I.      -g -O2 -MT main.o -MD -MP -MF ".deps/main.Tpo" -c -o main.o main.c; \
	[*][argv] /bin/sh -c if gcc -DLOCALEDIR=\"/usr/local/share/locale\" -DLIBDIR=\"/usr/local/lib\" -DINCLUDEDIR=\"/usr/local/include\" -DHAVE_CONFIG_H -I. -I. -I.      -g -O2 -MT misc.o -MD -MP -MF ".deps/misc.Tpo" -c -o misc.o misc.c; \
	[*][argv] /bin/sh -c if gcc -DLOCALEDIR=\"/usr/local/share/locale\" -DLIBDIR=\"/usr/local/lib\" -DINCLUDEDIR=\"/usr/local/include\" -DHAVE_CONFIG_H -I. -I. -I.      -g -O2 -MT read.o -MD -MP -MF ".deps/read.Tpo" -c -o read.o read.c; \
	[*][argv] /bin/sh -c if gcc -DLOCALEDIR=\"/usr/local/share/locale\" -DLIBDIR=\"/usr/local/lib\" -DINCLUDEDIR=\"/usr/local/include\" -DHAVE_CONFIG_H -I. -I. -I.      -g -O2 -MT remake.o -MD -MP -MF ".deps/remake.Tpo" -c -o remake.o remake.c; \
	[*][argv] /bin/sh -c if gcc -DLOCALEDIR=\"/usr/local/share/locale\" -DLIBDIR=\"/usr/local/lib\" -DINCLUDEDIR=\"/usr/local/include\" -DHAVE_CONFIG_H -I. -I. -I.      -g -O2 -MT remote-stub.o -MD -MP -MF ".deps/remote-stub.Tpo" -c -o remote-stub.o remote-stub.c; \
	[*][argv] /bin/sh -c if gcc -DLOCALEDIR=\"/usr/local/share/locale\" -DLIBDIR=\"/usr/local/lib\" -DINCLUDEDIR=\"/usr/local/include\" -DHAVE_CONFIG_H -I. -I. -I.      -g -O2 -MT rule.o -MD -MP -MF ".deps/rule.Tpo" -c -o rule.o rule.c; \
	[*][argv] /bin/sh -c if gcc -DLOCALEDIR=\"/usr/local/share/locale\" -DLIBDIR=\"/usr/local/lib\" -DINCLUDEDIR=\"/usr/local/include\" -DHAVE_CONFIG_H -I. -I. -I.      -g -O2 -MT signame.o -MD -MP -MF ".deps/signame.Tpo" -c -o signame.o signame.c; \
	[*][argv] /bin/sh -c if gcc -DLOCALEDIR=\"/usr/local/share/locale\" -DLIBDIR=\"/usr/local/lib\" -DINCLUDEDIR=\"/usr/local/include\" -DHAVE_CONFIG_H -I. -I. -I.      -g -O2 -MT strcache.o -MD -MP -MF ".deps/strcache.Tpo" -c -o strcache.o strcache.c; \
	[*][argv] /bin/sh -c if gcc -DLOCALEDIR=\"/usr/local/share/locale\" -DLIBDIR=\"/usr/local/lib\" -DINCLUDEDIR=\"/usr/local/include\" -DHAVE_CONFIG_H -I. -I. -I.      -g -O2 -MT variable.o -MD -MP -MF ".deps/variable.Tpo" -c -o variable.o variable.c; \
	[*][argv] /bin/sh -c if gcc -DLOCALEDIR=\"/usr/local/share/locale\" -DLIBDIR=\"/usr/local/lib\" -DINCLUDEDIR=\"/usr/local/include\" -DHAVE_CONFIG_H -I. -I. -I.      -g -O2 -MT version.o -MD -MP -MF ".deps/version.Tpo" -c -o version.o version.c; \
	[*][argv] /bin/sh -c if gcc -DLOCALEDIR=\"/usr/local/share/locale\" -DLIBDIR=\"/usr/local/lib\" -DINCLUDEDIR=\"/usr/local/include\" -DHAVE_CONFIG_H -I. -I. -I.      -g -O2 -MT vpath.o -MD -MP -MF ".deps/vpath.Tpo" -c -o vpath.o vpath.c; \
	[*][argv] /bin/sh -c if gcc -DLOCALEDIR=\"/usr/local/share/locale\" -DLIBDIR=\"/usr/local/lib\" -DINCLUDEDIR=\"/usr/local/include\" -DHAVE_CONFIG_H -I. -I. -I.      -g -O2 -MT hash.o -MD -MP -MF ".deps/hash.Tpo" -c -o hash.o hash.c; \
	[*][argv] rm -f make
	[*][argv] gcc -g -O2 -o make ar.o arscan.o commands.o default.o dir.o expand.o file.o function.o getopt.o getopt1.o implicit.o job.o main.o misc.o read.o remake.o remote-stub.o rule.o signame.o strcache.o variable.o version.o vpath.o hash.o -lrt
	bramante@matrix:~/debugging-tools/make-3.81$
```
</font>
