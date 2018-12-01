---
id: 114
title: skip running a func using windbg
date: 2009-08-06T09:56:00+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=114
permalink: /skip-running-a-func-using-windbg/
tags:
  - dbg
  - script
  - skip
  - windbg
---
	.if (not(${/d:$arg1}))
	{
	.echo usage:
	.echo "  ${$arg0} func-name [arg-count]"
	.echo "  if arg-count is omitted, 0 is the default value."
	}
	.else
	{
	.if (${/d:$arg2})
	{
	bp ${$arg1} "reip=poi(@esp); resp=@esp+ ${$arg2}*4+4;.echo #skip ${$arg1};g"
	}
	.else
	{
	bp ${$arg1} "reip=poi(@esp); resp=@esp+4;.echo #skip ${$arg1};g"
	}
	bl

使用起来很简单. 把上面的代码另存为 skipfunc 到 windbg 的安装目录.
调试一个程序的时候, 用这样的命令来取消一个函数(比如 MessageBoxW) 的调用.
`$$>a<skipfunc user32!messageboxw 4`
第一个参数执行要 skip 的函数名, 第二个参数指定这个函数的参数个数. 如果没有指定, 则认为没有参数.
