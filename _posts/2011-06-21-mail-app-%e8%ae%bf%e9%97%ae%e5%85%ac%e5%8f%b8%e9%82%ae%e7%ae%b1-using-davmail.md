---
id: 508
title: Mail.app 访问公司邮箱 using davmail
date: 2011-06-21T21:22:49+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=508
permalink: /mail-app-%e8%ae%bf%e9%97%ae%e5%85%ac%e5%8f%b8%e9%82%ae%e7%ae%b1-using-davmail/
tags:
  - davmail
  - mail.app
  - imap
  - owa
---
![Screen-shot-2011-06-21-at-9.07.05-PM.png]({{site.url}}/attachments/2011/06/Screen-shot-2011-06-21-at-9.07.05-PM.png)

邮件迁到 email.cisco.com 之后, 在 mac 上还是只能用 Entourage 来收发邮件. 向来对 Entourage 没有好感, 所以还是想用 mac 自带的 mail.app 来试试. 虽然可以 Mail.app 设置 IMAP 来收邮件. 但仍然无法发送(SMTP被block). 这里介绍通过使用 davmail 这个程序中转来曲线救国.
davmail 的详细介绍可以看这里:
<http://davmail.sourceforge.net/index.html>

由于 exchange 2003 还在广泛使用, 而很多 client 程序(比如 mail.app, outlook 2010)都不支持 exchange 2003 的协议, davmail 就应运而生了. 只要 exchange server 支持 OWA, 也就是通过网页来访问邮箱. davmail 就可以将 OWA 协议转成 SMTP/POP/IMAP 等协议来和 client 交互.
cisco 使用的就是 exchange2003, 在内网支持 OWA 和 IMAP. 但 smtp 由于防火墙的原因在国内无法访问. 这里我们就用 davmail 来本机上模拟出一个 smtp 服务.
davmail 是用 java 写的. 使用很简单. 提供了 client 和 server 版, 也就是GUI和命令行的版本. 如果下载的是client版. 双击图标稍作设置就可以了.

![Screen-shot-2011-06-21-at-9.10.48-PM.png]({{site.url}}/attachments/2011/06/Screen-shot-2011-06-21-at-9.10.48-PM.png)

OWA 填 `https://email.cisco.com/exchange`
只启用 Local SMTP port, 因为 IMAP 可以直接访问, 而其他几个暂时还没用到.
mail.app 的设置细节看这里:

<http://davmail.sourceforge.net/osximapmailsetup.html>

最重要的是设置 smtp:

![smtp]({{site.url}}/attachments/2011/06/Screen-shot-2011-06-21-at-9.13.36-PM.png)

![Screen-shot-2011-06-21-at-9.13.42-PM.png]({{site.url}}/attachments/2011/06/Screen-shot-2011-06-21-at-9.13.42-PM.png)

使用 client 版的问题是会在 dock 里留下一个图标. 不爽. 所以我建议是用 server 版, 作为一个服务在后台运行. server 版的具体配置看这里:

<http://davmail.sourceforge.net/serversetup.html>

我已经写好 davmail.properties, 直接解压附件到比如 /Applications/davmail 目录. 添加一个 StartupItem 重启机器即可. 如何添加 StartupItem 可以看着这篇帖子:
<http://nick.luckygarden.org/?p=104>

注意目录和文件的权限的设置.
 
	sh-3.2# cd /Library/StartupItems/
	sh-3.2# ls -l
	total 0
	drwxr-xr-x   4 root  wheel  136 Jun 21 20:36 davmail
	sh-3.2# cd davmail/
	sh-3.2# ls -l
	total 16
	-rwxr-xr-x  1 root  wheel  104 Jun 21 20:36 StartupParameters.plist
	-rwxr-xr-x  1 root  wheel  210 Jun 21 20:35 davmail
	sh-3.2# cat StartupParameters.plist
	{
	Description = "Davmail";
	Provides = ("davmail");
	Requires = ("Disks");
	OrderPreference = "None";
	}
	sh-3.2# cat davmail
	#!/bin/sh
	 
	. /etc/rc.common
	StartService ()
	{
	cd /Applications/davmail
	nohup ./davmail.sh davmail.properties &
	}
	 
	StopService ()
	{
	return 0
	}
	RestartService ()
	{
	return 0
	}
 
	RunService "$1"

davmail 的 log 是这样子的:

![Screen-shot-2011-06-21-at-9.18.07-PM.png]({{site.url}}/attachments/2011/06/Screen-shot-2011-06-21-at-9.18.07-PM.png)

这里是打包好的 
[davmail]({{site.url}}/attachments/2011/06/davmail.zip), 

其中包含了我写好的 davmail.properties, 可以根据需要修改。
另一个是
[davmail.tar]({{site.url}}/attachments/2011/06/davmail.tar.gz)
解压到 /Library/StartupItems 下省的自己写了. 如果你也是把上面的包解压到  /Applications/davmail 下就可以不用修改的使用了. 注意修改权限:
	
	nickx:/Applications/davmail $ ls -l
	total 1000
	-rwx&#8212;&#8212;   1 nickx  staff  495392 Jun 21 20:01 davmail.jar
	-rwx&#8212;&#8212;   1 nickx  staff    1298 Jun 21 20:01 davmail.log
	-rwx&#8212;&#8212;   1 nickx  staff     940 Jun 21 20:08 davmail.properties
	-rwx&#8212;&#8212;   1 nickx  staff     243 Jun 21 20:01 davmail.sh
	drwx&#8212;&#8212;  21 nickx  staff     714 Jun 21 20:01 lib
	-rw&#8212;&#8212;-   1 root   staff    3762 Jun 24 14:25 nohup.out
	</blockquote>
	<blockquote>root:/Library/StartupItems $ chown -R root:wheel davmail/
	root:/Library/StartupItems $ chmod 755 davmail/*
	
PS: 可以用这个命令查看一下 StartupItems 是否设置正确. 下面是我运行这个命令时的情形: 

	sh-3.2# SystemStarter -n -D
	SystemStarter[547]: Found item: davmail
	SystemStarter[547]: Requires: Evaluating Disks
	SystemStarter[547]: Found item: DoubleCommand
	SystemStarter[547]: Requires: Evaluating Disks
	SystemStarter[547]: Found item: CiscoVPN
	SystemStarter[547]: Requires: Evaluating Network
	SystemStarter[547]: Checking Davmail
	SystemStarter[547]: Antecedents: <CFArray 0x10010ad10 [0x7fff70b26ee0]>{type = mutable-small, count = 0, values = ()}
	SystemStarter[547]: No soft dependancies
	SystemStarter[547]: Best pick so far, based on failed dependancies (2147483647->0)
	SystemStarter[547]: Checking DoubleCommand
	SystemStarter[547]: Antecedents: <CFArray 0x10010ac50 [0x7fff70b26ee0]>{type = mutable-small, count = 0, values = ()}
	SystemStarter[547]: No soft dependancies
	SystemStarter[547]: Checking Cisco Systems VPN Client Kernel Extension
	SystemStarter[547]: Antecedents: <CFArray 0x10010ae90 [0x7fff70b26ee0]>{type = mutable-small, count = 0, values = ()}
	SystemStarter[547]: No soft dependancies
	SystemStarter[547]: Running command (549): /Library/StartupItems/davmail/davmail start
	SystemStarter[547]: Checking DoubleCommand
	SystemStarter[547]: Antecedents: <CFArray 0x10010ac50 [0x7fff70b26ee0]>{type = mutable-small, count = 0, values = ()}
	SystemStarter[547]: No soft dependancies
	SystemStarter[547]: Best pick so far, based on failed dependancies (2147483647->0)
	SystemStarter[547]: Checking Cisco Systems VPN Client Kernel Extension
	SystemStarter[547]: Antecedents: <CFArray 0x10010ae90 [0x7fff70b26ee0]>{type = mutable-small, count = 0, values = ()}
	SystemStarter[547]: No soft dependancies
	SystemStarter[547]: Running command (550): /Library/StartupItems/DoubleCommand/DoubleCommand start
	SystemStarter[547]: Checking Cisco Systems VPN Client Kernel Extension
	SystemStarter[547]: Antecedents: <CFArray 0x10010ae90 [0x7fff70b26ee0]>{type = mutable-small, count = 0, values = ()}
	SystemStarter[547]: No soft dependancies
	SystemStarter[547]: Best pick so far, based on failed dependancies (2147483647->0)
	SystemStarter[547]: Running command (551): /System/Library/StartupItems/CiscoVPN/CiscoVPN start
	Start DavMail
	SystemStarter[547]: Finished Davmail (549)
	appending output to nohup.out
	Starting Cisco Systems VPN Driver
	Loading DoubleCommand
	SystemStarter[547]: Finished Cisco Systems VPN Client Kernel Extension (551)
	dc.config: 1 -> 1
	SystemStarter[547]: Finished DoubleCommand (550)
	SystemStarter[547]: none left

