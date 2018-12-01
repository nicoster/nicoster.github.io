---
id: 125
title: Remove annoying AD window when PopKart exits
date: 2007-07-31T23:50:00+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=125
permalink: /remove-annoying-ad-window-when-popkart-exits/
tags:
  - dbg
  - ifeo
  - popkart
---
跑跑卡丁车退出的时候会弹出一个广告窗口. 从进程列表中可看出有一个 AdBalloonExt.exe. 直接删除这个文件. 卡丁车似乎不能运行. 网上也有方法去掉这个. 原理就是不让它运行. 一般使用组策略. 不过我还是喜欢用 ifeo. 在注册表中加入简单的一项. 一劳永逸的解决这个问题. 
注册表文件如下:
Windows Registry Editor Version 5.00
[HKEY_LOCAL_MACHINESOFTWAREMicrosoftWindows NTCurrentVersionImage File Execution OptionsAdBalloonExt.exe] "debugger "= "c:\windows\system32\conime.exe "
 也就是运行 AdBalloonExt 的时候偷梁换柱, 运行 conime. 而 conime 一运行就退出了. 
<a href="http://dl2.csdn.net/down4/20070731/31235500514.rar">点击下载</a>
