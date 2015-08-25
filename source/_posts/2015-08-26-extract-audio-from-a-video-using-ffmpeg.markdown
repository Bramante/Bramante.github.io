---
layout: post
title: "Extract Audio From a Video Using FFmpeg"
date: 2015-08-26 00:02:31 +0800
comments: true
categories: 
---

使用FFmpeg可以很容易地把Audio由TS file裡頭給分離出來, 
如果Audio是使用MPEG Audio來Encode的, 
那麼把分離出的Audio另存成*.mp3的file,
就可以直接用一般的軟體來播放.

以下是操作範例:

<font face="sans">
``` plain Terminal
	bramante@matrix:~/test/audio$ ll
	total 10592
	drwxrwxr-x 2 bramante bramante     4096 Aug 24 17:52 ./
	drwxrwxr-x 4 bramante bramante     4096 Aug 24 17:51 ../
	-rwxrw-rw- 1 bramante bramante 10834252 Aug 24 17:25 demo.ts*
	bramante@matrix:~/test/audio$ ffmpeg -i demo.ts -acodec copy demo.mp3
	ffmpeg version 0.8.17-4:0.8.17-0ubuntu0.12.04.1, Copyright (c) 2000-2014 the Libav developers
	  built on Mar 16 2015 13:26:50 with gcc 4.6.3
	The ffmpeg program is only provided for script compatibility and will be removed
	in a future release. It has been deprecated in the Libav project to allow for
	incompatible command line syntax improvements in its replacement called avconv
	(see Changelog for details). Please use avconv instead.
	[mp3 @ 0x86dd60] Header missing
	[mpegts @ 0x8337a0] max_analyze_duration reached
	[NULL @ 0x878c40] start time is not set in estimate_timings_from_pts
	[mpegts @ 0x8337a0] Continuity check failed for pid 18 expected 9 got 7
	[mpegts @ 0x8337a0] Continuity check failed for pid 1601 expected 13 got 2
	Input #0, mpegts, from 'demo.ts':
	  Duration: 26:30:41.26, start: 50944.346833, bitrate: 0 kb/s
	  Program 25664
	  Program 25728
	  Program 25792
	  Program 25856
	  Program 25920
	  Program 26240
	  Program 26368
	  Program 26560
	  Program 27136
	  Program 27168
	  Program 27360
	  Program 27456
	  Program 27520
	  Program 27584
	  Program 27648
	  Program 27680
	  Program 27712
	  Program 27744
	  Program 27840
	  Program 27872
	  Program 27904
	  Program 28032
	  Program 28096
	  Program 28160
	    Stream #0.0[0x641](eng): Audio: mp2, 48000 Hz, mono, s16, 64 kb/s
	    Stream #0.1[0x642]: Data: [11][0][0][0] / 0x000B
	  Program 28288
	  Program 28320
	  Program 28352
	  Program 28384
	  Program 28416
	  Program 28480
	  Program 28512
	Output #0, mp3, to 'demo.mp3':
	  Metadata:
	    TSSE            : Lavf53.21.1
	    Stream #0.0(eng): Audio: mp2, 48000 Hz, mono, 64 kb/s
	Stream mapping:
	  Stream #0.0 -> #0.0
	Press ctrl-c to stop encoding
	[mpegts @ 0x8337a0] Continuity check failed for pid 20 expected 15 got 3
	[mpegts @ 0x8337a0] Continuity check failed for pid 20 expected 5 got 9
	[mpegts @ 0x8337a0] Continuity check failed for pid 20 expected 11 got 15
	[mpegts @ 0x8337a0] Continuity check failed for pid 20 expected 1 got 5
	[mpegts @ 0x8337a0] Continuity check failed for pid 0 expected 10 got 11
	[mpegts @ 0x8337a0] Continuity check failed for pid 1601 expected 9 got 6
	[mpegts @ 0x8337a0] Continuity check failed for pid 18 expected 3 got 13
	[mpegts @ 0x8337a0] Continuity check failed for pid 18 expected 14 got 1
	[mpegts @ 0x8337a0] Continuity check failed for pid 1602 expected 12 got 10
	[mpegts @ 0x8337a0] Continuity check failed for pid 16 expected 11 got 5
	[mpegts @ 0x8337a0] Continuity check failed for pid 20 expected 7 got 13
	[mpegts @ 0x8337a0] Continuity check failed for pid 20 expected 15 got 3
	[mpegts @ 0x8337a0] Continuity check failed for pid 20 expected 5 got 9
	[mpegts @ 0x8337a0] Continuity check failed for pid 20 expected 11 got 15
	[mpegts @ 0x8337a0] Continuity check failed for pid 20 expected 1 got 5
	[mpegts @ 0x8337a0] Continuity check failed for pid 18 expected 9 got 7
	[mpegts @ 0x8337a0] Continuity check failed for pid 1601 expected 13 got 2
	size=     865kB time=110.69 bitrate=  64.0kbits/s
	video:0kB audio:865kB global headers:0kB muxing overhead 0.014569%
	bramante@matrix:~/test/audio$ ll
	total 11460
	drwxrwxr-x 2 bramante bramante     4096 Aug 24 17:53 ./
	drwxrwxr-x 4 bramante bramante     4096 Aug 24 17:51 ../
	-rw-rw-r-- 1 bramante bramante   885585 Aug 24 17:53 demo.mp3
	-rwxrw-rw- 1 bramante bramante 10834252 Aug 24 17:25 demo.ts*
	bramante@matrix:~/test/audio$ mediainfo demo.mp3
	General
	Complete name                            : demo.mp3
	Format                                   : MPEG Audio
	File size                                : 865 KiB
	Duration                                 : 1mn 50s
	Overall bit rate mode                    : Constant
	Overall bit rate                         : 64.0 Kbps
	Encoding settings                        : Lavf53.21.1
	
	Audio
	Format                                   : MPEG Audio
	Format version                           : Version 1
	Format profile                           : Layer 2
	Duration                                 : 1mn 50s
	Bit rate mode                            : Constant
	Bit rate                                 : 64.0 Kbps
	Channel(s)                               : 1 channel
	Sampling rate                            : 48.0 KHz
	Compression mode                         : Lossy
	Stream size                              : 864 KiB (100%)
	
	
	bramante@matrix:~/test/audio$
```
</font>