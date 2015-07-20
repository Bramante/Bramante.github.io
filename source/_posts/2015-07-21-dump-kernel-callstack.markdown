---
layout: post
title: "Dump Kernel Callstack"
date: 2015-07-21 00:45:38 +0800
comments: true
categories: 
---

在Console Dump Kernel的Callstack很簡單,
只要執行下列的Command即可印出所有Task的Callstack:

	echo 1 > /proc/sys/kernel/sysrq
	echo 9 > /proc/sys/kernel/printk
	echo "t" > /proc/sysrq-trigger

如果Hot Key有用的話, 也可以用Hot Key觸發:

	echo 1 > /proc/sys/kernel/sysrq
	echo 9 > /proc/sys/kernel/printk
	Ctrl + break  放開再按  t

如果想在Kernel的code裡印出Callstack, 了解Function怎麼被Call到的,
可以call dump_stack(), 如果想一併了解Task Name與PID, 
可以印出current的comm與pid:

``` c Dump Stack
	printk("task name: %s, pid: %d \n", current->comm, current->pid);
	dump_stack();
```