---
id: 106
title: win64 debugging
date: 2009-12-28T14:00:00+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=106
permalink: /win64-debugging/
tags:
  - dbg
  - win64
  - windbg
  - x64
---
32位程序在 win64 下运行 crash, 用 taskmgr 创建一个 dump. 用 32位 windbg 打开这个 dump, 用如下命令切换到 32位 模式之后, 调试就和普通 32位 dump 一样了:

	0:000> .load wow64exts
	0:000> !sw
	Switched to 32bit mode
