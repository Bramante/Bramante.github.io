---
layout: post
title: "Create video with audio and still image"
date: 2015-07-28 09:40:15 +0800
comments: true
categories: 
---

FFmpeg可以把一個jpeg file的image, 轉成靜態畫面的video file, 並配上聲音, 此外bitrate還可以設得很低.

<font face="sans">
``` plain Terminal
	bramante@matrix:~/video$ ffmpeg -loop 1 -i photo.jpg -i free.mp3 -vcodec mpeg4 -acodec aac -strict experimental -s 640x480 -pix_fmt yuv420p -vb 16k -r 1 -ab 64k -shortest -f mpegts demo.ts
```
</font>

免費配樂可以到YouTube Audio Library去找, 就我的理解, 應該可以免費使用, 而且不限定是製做YouTube Video.
