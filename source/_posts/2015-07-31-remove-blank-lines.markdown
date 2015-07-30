---
layout: post
title: "Remove Blank Lines"
date: 2015-07-31 00:12:12 +0800
comments: true
categories: 
---

使用UltraEdit要怎麼刪除空行呢?

在Replace時, 勾選Regular Expressions的功能, 照下圖這樣輸入, 便可刪除空行.

{% img left /images/2015-07-31-remove-blank-lines/replace.jpg replace.jpg %}

	%[ ^t]++^p

這個功能我通常用在寫工作日誌的時候, 這樣可以讓日誌的source code內容看起來比較緊湊.
