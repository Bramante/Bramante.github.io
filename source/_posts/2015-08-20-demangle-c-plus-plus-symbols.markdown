---
layout: post
title: "Demangle C++ Symbols"
date: 2015-08-20 09:26:40 +0800
comments: true
categories: 
---

當我們在debug時, 有時候會遇到未經demangle過的C++ symbol name, 
乍看之下, 常常不太能確定這到底是哪個API, 
這時可以用c++filt把這樣的symbol name轉換(demangle)成我們看得懂的symbol name.

<font face="sans">
``` plain Terminal
	C++FILT(1)                                                             GNU Development Tools                                                             C++FILT(1)
	
	
	
	NAME
	       c++filt - Demangle C++ and Java symbols.
	
	SYNOPSIS
	       c++filt [-_|--strip-underscores]
	               [-n|--no-strip-underscores]
	               [-p|--no-params]
	               [-t|--types]
	               [-i|--no-verbose]
	               [-s format|--format=format]
	               [--help]  [--version]  [symbol...]
	
	DESCRIPTION
	       The C++ and Java languages provide function overloading, which means that you can write many functions with the same name, providing that each function
	       takes parameters of different types.  In order to be able to distinguish these similarly named functions C++ and Java encode them into a low-level assembler
	       name which uniquely identifies each different version.  This process is known as mangling. The c++filt [1] program does the inverse mapping: it decodes
	       (demangles) low-level names into user-level names so that they can be read.
	
	       Every alphanumeric word (consisting of letters, digits, underscores, dollars, or periods) seen in the input is a potential mangled name.  If the name
	       decodes into a C++ name, the C++ name replaces the low-level name in the output, otherwise the original word is output.  In this way you can pass an entire
	       assembler source file, containing mangled names, through c++filt and see the same source file containing demangled names.
	
	       You can also use c++filt to decipher individual symbols by passing them on the command line:
	
	               c++filt <symbol>
	
	       If no symbol arguments are given, c++filt reads symbol names from the standard input instead.  All the results are printed on the standard output.  The
	       difference between reading names from the command line versus reading names from the standard input is that command line arguments are expected to be just
	       mangled names and no checking is performed to separate them from surrounding text.  Thus for example:
	
	               c++filt -n _Z1fv
	
	       will work and demangle the name to "f()" whereas:
	
	               c++filt -n _Z1fv,
	
	       will not work.  (Note the extra comma at the end of the mangled name which makes it invalid).  This command however will work:
	
	               echo _Z1fv, | c++filt -n
	
	       and will display "f(),", i.e., the demangled name followed by a trailing comma.  This behaviour is because when the names are read from the standard input
	       it is expected that they might be part of an assembler source file where there might be extra, extraneous characters trailing after a mangled name.  For
	       example:
	
	                   .type   _Z1fv, @function
	
	OPTIONS
	       -_
	       --strip-underscores
	           On some systems, both the C and C++ compilers put an underscore in front of every name.  For example, the C name "foo" gets the low-level name "_foo".
	           This option removes the initial underscore.  Whether c++filt removes the underscore by default is target dependent.
	
	       -n
	       --no-strip-underscores
	           Do not remove the initial underscore.
	....
```
</font>

找幾個C++的symbol name來示範一下:

<font face="sans">
``` plain Terminal
	bramante@matrix:~$ objdump -R /usr/lib/x86_64-linux-gnu/libQtXml.so.4.8 | tail -n 11
	0000000000244368 R_X86_64_JUMP_SLOT  _ZN11QTextStreamC1EP7QString6QFlagsIN9QIODevice12OpenModeFlagEE
	0000000000244370 R_X86_64_JUMP_SLOT  _ZNK7QString10startsWithERKS_N2Qt15CaseSensitivityE
	0000000000244378 R_X86_64_JUMP_SLOT  _Znwm
	0000000000244380 R_X86_64_JUMP_SLOT  _ZNK10QTextCodec9canEncodeE5QChar
	0000000000244388 R_X86_64_JUMP_SLOT  _Unwind_Resume
	0000000000244390 R_X86_64_JUMP_SLOT  _ZNK7QString10simplifiedEv
	0000000000244398 R_X86_64_JUMP_SLOT  memcpy
	00000000002443a0 R_X86_64_JUMP_SLOT  _ZN7QString7reallocEi
	00000000002443a8 R_X86_64_JUMP_SLOT  _ZN10QByteArray6appendERKS_
	
	
	bramante@matrix:~$ c++filt -n _ZN11QTextStreamC1EP7QString6QFlagsIN9QIODevice12OpenModeFlagEE
	QTextStream::QTextStream(QString*, QFlags<QIODevice::OpenModeFlag>)
	bramante@matrix:~$ c++filt -n _ZNK7QString10startsWithERKS_N2Qt15CaseSensitivityE
	QString::startsWith(QString const&, Qt::CaseSensitivity) const
	bramante@matrix:~$ c++filt -n _Znwm
	operator new(unsigned long)
	bramante@matrix:~$ c++filt -n _ZNK10QTextCodec9canEncodeE5QChar
	QTextCodec::canEncode(QChar) const
	bramante@matrix:~$ c++filt -n _ZNK7QString10simplifiedEv
	QString::simplified() const
	bramante@matrix:~$ c++filt -n _ZN7QString7reallocEi
	QString::realloc(int)
	bramante@matrix:~$ c++filt -n _ZN10QByteArray6appendERKS_
	QByteArray::append(QByteArray const&)
	bramante@matrix:~$
```
</font>

請注意, 過長的symbol name會被readelf截去尾巴, 
導致c++filt無法demangle出正確的symbol name, 
這時請在readelf使用"-W"參數, 讓readelf印出完整的symbol name, 
以下是操作範例:

<font face="sans">
``` plain Terminal
	bramante@matrix:~$ readelf -a /usr/lib/x86_64-linux-gnu/libQtXml.so.4.8 | grep 244368
	000000244368  007a00000007 R_X86_64_JUMP_SLO 0000000000000000 _ZN11QTextStreamC1EP7Q + 0
	bramante@matrix:~$ c++filt -n _ZN11QTextStreamC1EP7Q
	_ZN11QTextStreamC1EP7Q
	bramante@matrix:~$ readelf -Wa /usr/lib/x86_64-linux-gnu/libQtXml.so.4.8 | grep 244368
	0000000000244368  0000007a00000007 R_X86_64_JUMP_SLOT     0000000000000000 _ZN11QTextStreamC1EP7QString6QFlagsIN9QIODevice12OpenModeFlagEE + 0
	bramante@matrix:~$ c++filt -n _ZN11QTextStreamC1EP7QString6QFlagsIN9QIODevice12OpenModeFlagEE
	QTextStream::QTextStream(QString*, QFlags<QIODevice::OpenModeFlag>)
	bramante@matrix:~$
```
</font>
