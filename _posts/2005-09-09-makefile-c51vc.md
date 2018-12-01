---
id: 20
title: makefile, c51@vc,
date: 2005-09-09T03:12:48+00:00
author: nick
layout: post
guid: http://nicoster.wordpress.com/2005/09/09/makefile-c51vc
permalink: /makefile-c51vc/
spaces_21ad9df954391fe1354efd1fc739c09a_permalink:
  - "http://cid-192788b236f6126b.users.api.live.net/Users(1812567674047566443)/Blogs('192788B236F6126B!102')/Entries('192788B236F6126B!164')?authkey=FlIl!wdwooA%24"
categories:
  - log
---

在 msdn 上看了一些 makefile 的写法. 实现了在 vc 的 ide 中写 c51 的代码. 并且可以编译. 这样的好处是, 可以使用高效的 vc ide 编辑环境. 并且可以快速定位错误. output 窗口. nice

编写的一些脚本:

cc.bat

	@echo off
	:begin
	if  "%1 " ==  " " goto end 
	c:\keil\bin\cx51 %1 > %temp%c51.tmp
	set errorcode=%ERRORLEVEL%
	type %temp%c51.tmp | awk  "/IN LINE/ 'sub(/:/,  " ", $8); printf  "%%s(%%s): %%s %%s %%sn ", $8, $6, $2, $3, substr($0, index($0,  ".C ") + 2, 255)&#125; "
	if %errorcode% geq 2 exit %errorcode%
	shift
	goto begin
	:end
	exit 0
	
link.cmd

	@echo off
	:begin
	if  "%1 " ==  " " goto end 
	c:\keil\bin\cx51 %1 > %temp%c51.tmp
	set errorcode=%ERRORLEVEL%
	type %temp%c51.tmp | awk  "/IN LINE/ 'sub(/:/,  " ", $8); printf  "%%s(%%s): %%s %%s %%sn ", $8, $6, $2, $3, substr($0, index($0,  ".C ") + 2, 255)&#125; "
	if %errorcode% geq 2 exit %errorcode%
	shift
	goto begin
	:end
	exit 0

使用 vc 建立一个 makefile 项目. 添加 c51 源文件, 编写一个简单的 makefile 即可.

