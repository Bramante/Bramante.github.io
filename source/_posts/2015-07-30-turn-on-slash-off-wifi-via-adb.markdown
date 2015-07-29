---
layout: post
title: "Turn ON/OFF WIFI via ADB"
date: 2015-07-30 00:29:22 +0800
comments: true
categories: 
---


在Android平台上, 可以透過在command line執行svc command來開關WIFI.

	adb shell svc wifi enable
	adb shell svc wifi disable

執行command後, setting APK上的UI也會同步反應WIFI的開關狀態.
