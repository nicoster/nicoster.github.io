---
id: 482
title: mac 下编译 chromium
date: 2011-06-19T02:21:52+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=482
permalink: /mac-%e4%b8%8b%e7%bc%96%e8%af%91-chromium/
tags:
  - mac
  - chromium
  - compile
  - xcodebuild
  - ld
---
![Screen-shot-2011-06-19-at-2.21.11-AM.png]({{site.url}}/attachments/2011/06/Screen-shot-2011-06-19-at-2.21.11-AM.png)
最近在写一个 chrome 的扩展, 但发现有问题, 行为和文档表述的不一样, 所以想调试一下 chromium 看问题出在哪里. 所以在 mac 下试着编译了一下 chromium.
首先是把代码弄下来. 我按照文档的建议, 下载了一个 tar 包, 解压之后用 gclient 更新代码到最新的 green revision (也就是没有编译错误 基本上没有问题的版本)
<blockquote>gclient config http://src.chromium.org/svn/trunk/src http://chromium-status.appspot.com/lkgr
gclient sync</blockquote>
gclient sync 可能花了将近 1 个小时的时间.
然后就是编译. 之前看文档说 xcode 似乎会创建多个 ld 进程进行链接从而导致整个机器都没有响应. 所以我也按照文档的建议用了一个脚本来限制 ld 的数量 (因为我的 mbp 是 4 核, 所以限制为 4 个), 细节在这里:
<pre>http://groups.google.com/a/chromium.org/group/chromium-dev/browse_thread/thread/54cf7662fc3ea251/a782d309b03eed79
</pre>
但即使是这样, 在 xcode 里编译的时候还是遇到 ld hang 的问题, 4 个 ld 进程在那里一动不动. 无奈, 试着一个一个项目编译吧. 就这么编译了下来 最后还是卡在一些项目上, 一看是测试项目, 倒也无所谓, 至少 chromium.app 已经编译好了. 运行/调试都 ok.
本着 discover 的精神, 我又试了试在命令行下用 xcodebuild 编译, 才发现原来这才是最简单的方式. 直接用这个命令:
<blockquote>xcodebuild -project chrome.xcodeproj -configuration Debug -target chrome</blockquote>
先是编译的 chrome 这个 target, 一次通过. 后来又编译了 all, 也顺利完成.
<blockquote>xcodebuild -project chrome.xcodeproj -configuration Debug -target All<a href="http://nick.luckygarden.org/wp-content/uploads/2011/06/Screen-shot-2011-06-19-at-2.19.16-AM.png">
</a></blockquote>
分析之前编译出错可能的原因是, 因为单个 ld 链接要占用很大的内存(峰值大概 1.4G). 在 xcode 下编译时, xcode 本身就要占大概 800M, 加上要同时起多个 ld 进程, 导致内存不够用从而死锁. 用 xcodebuild 命令行编译时, xcodebuild 占的内存不太多, 而且不管编译/链接都是单进程的, 加上我把多余的程序都关掉了, 编译下来就没有遇到问题了. 说一下我的 mbp 配置: 2.4G Core i5, 4G DDR3, 512G HD.
文档里说要准备 10G 的空间. 编译下来我建议硬盘剩余空间不小于 40G

![xcode]({{site.url}}/attachments/2011/06/Screen-shot-2011-06-19-at-2.19.16-AM.png)
