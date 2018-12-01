---
id: 142
title: Smart formating source code using astyle in Scite
date: 2006-12-17T21:19:00+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=142
permalink: /smart-formating-source-code-using-astyle-in-scite/
categories:
  - Uncategorized
---
用 vc6 很喜欢 alt+F8 的功能. 能够格式化代码. scite 没有这个功能. 今天看它的配置文件发现这么两行:
command.name.0.*.cxx=Indentcommand.0.*.cxx=astyle -tapO $(FileNameExt)
用来缩进的? 查了一下 astyle, 原来就是这个功能. 下载了 astyle 1.19, 修改了一下配置文件 cpp.properties, 如下:
command.name.0.*.cpp=Indentcommand.0.*.cpp=astyle :style=ansi $(FileNameExt)
scite 更好用了.
