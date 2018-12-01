---
id: 572
title: AppleScript 与自动连接 VPN
date: 2012-03-17T23:44:21+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=572
permalink: /applescript-%e4%b8%8e%e8%87%aa%e5%8a%a8%e8%bf%9e%e6%8e%a5-vpn/
tags:
  - vpn
  - applescript
  - SofToken II
  - Accessibility
---
下载 [vpnauto.applescript]({{site.url}}/attachments/2012/03/vpnauto.applescript.zip)

公司使用的 VPN 使用 SofToken 生成的 token 来认证，所以每次连接都很繁琐。今天终于打算花点时间解决这个问题。
之前有同事在 OSX 上使用 AppleScript 来做 Test Automation。所以先找找 AppleScript 的文档来看看，看了好一会儿，一头雾水。于是搜索 'applescript vpn', 找到不少例子，但也只是简单的连接 vpn，不能满足我的需要。此路不通。还是搜搜 'AppleScript Tutorial', 找到几个 blog 讲的不错。于是自己写了一段代码可以让 softoken 生成 token 了

	tell application "SofToken II" to activate
	tell application "System Events"
		tell process "SofToken II"
			keystroke "<YOUR_PIN>"
			key code 36
		end tell
	end tell
	
但这离一个完整的脚本还差点。进一步的搜索让我找到这个 [blog](
http://coreygilmore.com/projects/automated-securid-token-generation-and-vpn-login-applescript/)
原来早有人已经这么干了，把他的脚本拿过来跑了一跑，不错。但有一些小问题比如: 没有错误处理，连接失败的时候不能自动重连。于是花了一些时间完善了一下. 效果很不错. 如果你也要用这个脚本的话需要修改几个地方：

	set theConnection to "ShangHai"
	set thePin to "" -- set to "" to prompt every time (recommended)
	
脚本开头的两行，第一行的 "ShangHai"，需要改为你的 VPN 连接的名字, 如图：
![Screen-Shot-2012-03-17-at-11.22.30-PM.png]({{site.url}}/attachments/2012/03/Screen-Shot-2012-03-17-at-11.22.30-PM.png)

第二行的 thePin, 默认是空。每次脚本执行的时候会让你输入 softoken 的 pin. 如果想偷懒，可以把 pin 保存在脚本里。当然，安全性就差点了。脚本要能执行，还需要启用 OSX 里的一个 Accessibility 的设置:

![Screen-Shot-2012-03-17-at-11.18.11-PM.png]({{site.url}}/attachments/2012/03/Screen-Shot-2012-03-17-at-11.18.11-PM.png)

给 Enable access for assistive devices 打上勾。
关于 AppleScript 的执行，还需要提醒一下. 你可以用系统自带的 AppleScript Editor 来编辑，执行。但为了方便起见。把脚本在 AppleScript Editor 里另存为一个 Application 更好.

![Screen-Shot-2012-03-17-at-11.30.48-PM.png]({{site.url}}/attachments/2012/03/Screen-Shot-2012-03-17-at-11.30.48-PM.png)

该脚本在 OSX10.7 locale 为 English 上测试通过. 最新的代码在 
<git://github.com/nicoster/vpnauto.git>
fork me.
