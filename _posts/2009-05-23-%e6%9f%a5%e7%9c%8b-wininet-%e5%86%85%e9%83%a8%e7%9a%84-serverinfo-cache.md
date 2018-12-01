---
id: 119
title: 查看 wininet 内部的 serverinfo cache
date: 2009-05-23T09:24:00+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=119
permalink: /%e6%9f%a5%e7%9c%8b-wininet-%e5%86%85%e9%83%a8%e7%9a%84-serverinfo-cache/
tags:
  - dbg
  - dns
  - serverinfo cache
  - windbg
  - wininet
---
	
	0:030>as dx !list '-t ntdll!_LIST_ENTRY.Flink -e -x 'dpa poi(@$extret)+0n20 l1;dd poi(@$extret)+10 l1&#8243; WININET!GlobalServerInfoList'
	0:030> dx
	dpa poi(@$extret)+0n20 l1;dd poi(@$extret)+10 l1 
	04141dcc  04779fb0 'www.flash.cn' 04141dc8  00000001
	dpa poi(@$extret)+0n20 l1;dd poi(@$extret)+10 l1 
	0412f98c  047f72a8 'toolbarqueries.google.com' 0412f988  00000001
	dpa poi(@$extret)+0n20 l1;dd poi(@$extret)+10 l1 
	041427b4  047f3310 'www.hi-pda.com' 041427b0  00000001
	
第一行是域名, 第二行是 ref count. 清 wininet dns cache 时, 需要清除这个 serverinfo cache
