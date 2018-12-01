---
id: 146
title: SHUT UP, GreatNews
date: 2006-12-03T21:56:00+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=146
permalink: /shut-up-greatnews/
categories:
  - dbg
tags:
  - detour
  - MessageBox
  - windbg
---
一直都用 GreatNews 看 RSS, 虽然不是那么令人满意. 但它还是是我用起来最习惯的. 但有一点实在 annoying. 每次我启动程序的时候就提醒我. 我已经有 30 天没有更新程序了. 我已经有 216 天没有清理旧的消息了. 而且还一次弹出来 3 个. 在点击 RSS 的时候还得弹出来两三个. 本来也是作者善意的提醒. 但是,. 每 30 天一次的更新并不是那样的激动人心. 作为一个普通用户. 我看不出更新前后有什么区别. 那我为什么还有花时间去做这样无意义的事情?. 我喜欢把看过的 feeds 在本地保留下来. 因为以后我要找的时候不用在全球范围内搜索, 只要找我机器里的记录就好了. 况且用来保存 feeds 的数据库文件并不大 &#8212; 即使我 216 天没有清空. 文件也只要 24M. 其实以前我也清理过. 但我也没有觉得清理之后程序跑起来就轻快一些了. 每次我打开程序, 它开始自动更新的时候. 程序的主界面就死了好一会儿. 更糟的是. 我的任务栏窗口都不响应了. 不知道作者是怎么实现的. 
其实就这两个问题我也给作者写过邮件. 但却没有任何回复. 也许是太忙了吧.作者回复不回复, 那是遥远的事情, 仿佛可以不管, 但每次要点击五六次确定. 却实在令人难受. 怎么办?  自己动手, 丰衣足食. 我的要求也不高. 只要不弹出 msgbox 就成. 还好这很容易实现. 祭出我们的终极武器: DETOUR! 玩 API 劫持. 改了改 detour 的例子. 编译成一个 simple.dll. 执行 detour 自带的 setdll.exe, 让 simple.dll 作为 greatnews.exe 的一个 dependency. 再次打开程序. 哈. 整个世界清净了.
附件中是源码和编译好的 dll. 将他们解压到 greatnews 所在目录. 执行 setdll /d:simple.dll greatnews.exe 即可.
<u><font color="#0000ff"><a href="http://nicoster.googlepages.com/shutup_greatnews.rar">http://nicoster.googlepages.com/shutup_greatnews.rar</a></font></u>
另外, 导出我的 RSS 频道和大家共享:
<u><font color="#0000ff"><a href="http://nicoster.googlepages.com/myfeeds.opml">http://nicoster.googlepages.com/myfeeds.opml</a></font></u><a href="http://nicoster.googlepages.com/shutup_greatnews.rar"></a>
