---
layout: post
title: "HTTP Redirect"
date: 2015-08-27 00:23:32 +0800
comments: true
categories: 
---

我的NAS之前改成NAS裡的每個帳號都可以有自己的web page之後, 
就一直沒特別去處理NAS的home page, 
以至於連到home page時, 
畫面上什麼也沒有, 
目前索性把它redirect到github pages的blog, 
修改方法很簡單, 只要把.htaccess修改如下就可以了:

<font face="sans">
``` plain Terminal
	DiskStation> cat /volume1/web/.htaccess
	
	# BEGIN WordPress
	RewriteEngine On
	RewriteCond %{HTTPS} off
	RewriteRule index.htm https://bramante.github.io/
	# END WordPress
	DiskStation>
```
</font>
