---
layout: post
title: "Kill Process by Name"
date: 2015-07-31 00:35:50 +0800
comments: true
categories: 
---

在使用busybox的嵌入式系統上, 若因bug導致log狂印, 有時會想砍掉這個狂印log的process, 以便後續能操作console進行debug. 此時由於log處於狂印的狀態, 若使用ps查詢kill所需的pid時, 往往會很難看到想看的內容, 此時若能kill process by name把狂印log的process給砍掉會很方便.

假設要砍掉的process name叫做Application的話, 我都是用下面這樣的一串command來做到kill process by name的:

	kill -9 $(/bin/busybox ps -eo pid,comm | grep Application | sed 's/Application//g')

