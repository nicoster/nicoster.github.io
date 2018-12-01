---
id: 113
title: ida, ollydbg 使用 PDB. _NT_SYMBOL_PATH
date: 2009-08-20T10:44:00+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=113
permalink: /ida-ollydbg-%e4%bd%bf%e7%94%a8-pdb-_nt_symbol_path/
tags:
  - dbg
  - ida
  - ollydbg
  - pdb
  - windbg
  - _NT_SYMBOL_PATH
---
ida 5.2 自带的加载 pdb 的功能有问题. 于是找到了这个:
<a href="http://msmvps.com/blogs/v_scherbina/archive/2006/12/22/pdbext_5F00_v0_5F00_2.aspx">http://msmvps.com/blogs/v_scherbina/archive/2006/12/22/pdbext_5F00_v0_5F00_2.aspx</a>
ollydbg 只会在当前目录找 pdb. 这里的帖子上讲了如何修改 ollydbg.exe 来 fix 这个问题:

<a href="http://www.openrce.org/forums/posts/187">http://www.openrce.org/forums/posts/187</a>

	D:odbg110> fc /b ollydbg.exe ollydbgsym.exe
	Comparing files OLLYDBG.EXE and OLLYDBGSYM.EXE
	00090709: 10 37
	0009070A: 12 02
	0009070B: 00 03
	0009070C: 00 80
	000907EC: 74 EB
