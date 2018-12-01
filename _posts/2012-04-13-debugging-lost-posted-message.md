---
id: 603
title: Debugging Lost Posted Message
date: 2012-04-13T11:10:41+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=603
permalink: /debugging-lost-posted-message/
tags:
  - dbg
  - GetMessage
  - lost
  - Message loop
  - Message queue
  - PeekMessage
  - PostMessage
  - script
  - wds
  - windbg
---
The message you posted through PostMessage() is lost? Believe me, it wasn't a bug in Windows. Check your own code. You've already done that? Okay, I have some advices on solving this problem. 

1. Check the return value of `PostMessage()`.
`PostMessage()` might fail. It return false in the case. call `GetLastError()` for the detailed error information.
2. Check the message loop.
I'd like to present a windbg script to make life easier.
The idea behind it is straightforward. All message are retrieved by `GetMessage()` or `PeekMessage()`, then delivered to corresponding window proc by `DispatchMessage()`. So we could set some checkpoints on these 3 APIs, tracing the message we're interested.

To use this script, please setup the `_NT_SYMBOL_PATH`, and compile your project with symbols. Here we go.


	.if (not(${/d:$arg1}))
	{
		.echo Usage:
		.echo  " $$>a<${$arg0} msg [hwnd] "
		.echo  " Specify the msg you want to check. You could specify the hwnd as well "
		.echo  " "
		.echo Example:
		.echo  " $$>a<${$arg0} 400 1a0396			# monitor msg WM_USER (0x400) for window 0x1a0396 "
		.echo  " $$>a<${$arg0} 1					# monitor msg WM_CREATE (0x1) for all windows in current process "
	}
	.else
	{
		.if (${/d:$arg2})
		{
			bp USER32!NtUserGetMessage+0xc  "j(poi(poi(esp+4)+4)==${$arg1} & (poi(poi(esp+4))==${$arg2})) '.echo;kL;g';'g' "
			bp USER32!NtUserPeekMessage+0xc  "j(poi(poi(esp+4)+4)==${$arg1} & (poi(esp+14)&1) & (poi(poi(esp+4))==${$arg2})) '.echo;kL;g';'g' "
			bp USER32!DispatchMessageW+0xc  "j(poi(poi(esp+4)+4)==${$arg1} & (poi(poi(esp+4))==${$arg2})) '.echo;kL;g';'g' "
		}
		.else
		{
			bp USER32!NtUserGetMessage+0xc  "j(poi(poi(esp+4)+4)==${$arg1}) '.echo;kL;g';'g' "
			bp USER32!NtUserPeekMessage+0xc  "j(poi(poi(esp+4)+4)==${$arg1} & (poi(esp+14)&1)) '.echo;kL;g';'g' "
			bp user32!DispatchMessageW  "j(poi(poi(esp+4)+4)==${$arg1}) '.echo;kL;g';'g' "
		}
		
		bl
	}

Say you save it as c:\msgmon.wds, call it in windbg this way:

	$$>a<c:\msgmon.wds messge-id [hwnd]

Hope you solve your problem.
/nick
