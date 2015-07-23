---
layout: post
title: "How to backup an old computer"
date: 2015-07-23 23:33:04 +0800
comments: true
categories: 
---

電腦老舊之後, 會買新的電腦取代,
有些資料很容易從舊電腦搬到新電腦,
像影音檔之類的檔案, copy是很容易的,
但是有些像email之類的資料,
可能會因為email軟體的不同而需要轉檔,
才能搬到新電腦, 這就很麻煩.
如果遇到無法轉檔, 或新電腦與舊有軟體存在相容性的問題,
那該怎麼辦?

有個方法可以解決這個問題,
就是把舊電腦給虛擬化,
把它轉成vmware image,
讓它擺脫老舊的硬體,
在新電腦裡繼續存活下去,
這樣還有個好處, 就是不用擔心有資料忘了copy出來,
因為所有的內容都還在vmware image裡.
     
Windows可以使用 VMware vCenter Converter 直接把電腦虛擬化,
Linux則可以使用 MondoRescue, 先把電腦轉換成Rescue Image,
然後再把Rescue Image "還原"到vmware image上進行虛擬化,
這2套軟體都是免費的, create vmware image有免費軟體可用,
vmware player也是免費的, 所以把舊電腦虛擬化是可以不花錢的.
