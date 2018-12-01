---
id: 107
title: Check ShowWindow API
date: 2009-12-10T13:53:00+00:00
author: nick
layout: post
permalink: /check-showwindow-api/
tags:
  - dbg
  - script
  - SetWindowsPos
  - ShowWindow
  - windbg
---
在 windows 下调试窗口相关的代码时, 有时候需要检查一个窗口为什么被显示/隐藏. 归根结底显示/隐藏窗口都是通过调用 `ShowWindow`/`SetWindowPos` 这两个 API 来实现的. 这里提供的脚本本质上就是在这两个 API 上放断点. trace 出窗口句柄以及参数. 看脚本:

	.if (not(${/d:$arg1}))
	{
		.echo Now you're monitoring all windows for show/hide events in current process.
		.echo
		.echo If you just want to monitor one window, do it this way:
		.echo  " $$>a<${$arg0} [hwnd] "
		.echo
	
		bp USER32!NtUserShowWindow  ".printf \ "ShowWindow(%N, %d)\\n\ ", poi(@esp+4), @@(!!@@(poi(@esp+8)));g "
		bp USER32!NtUserSetWindowPos  ".printf \ "SetWindowPos(%N, %d)\\n\ ", poi(@esp+4), @@(!!(@@(poi(@esp+1c))&0x40));g "
	}
	.else
	{
		bp USER32!NtUserShowWindow  "j (poi(@esp+4) == ${$arg1}) '.echo;.printf \ "ShowWindow(%N, %d) \\n\ ", poi(@esp+4), @@(!!@@(poi(@esp+8)));kL;g';'g' "
		bp USER32!NtUserSetWindowPos  "j (poi(@esp+4) == ${$arg1}) '.echo;.printf \ "SetWindowPos(%N, %d)\\n\ ", poi(@esp+4), @@(!!(@@(poi(@esp+1c))&0x40));kL;g';'g' "
	}
	
	bl

将文件保存为 chkshowwnd 到 windbg 安装目录. 在 windbg 中这样调用:

	$$>a<chkshowwnd [hwnd]
	
如果不指定窗口句柄, 则会显示所有的 ShowWindow/SetWindowPos 调用. 一般你会发现这两个 API 调用得太多了. 因此, 你可以指定你想关注的窗口句柄, 这样, 在这个窗口被显示/隐藏时, windbg 会停下来, 你就可以检查调用栈了.
