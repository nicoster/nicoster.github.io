---
id: 109
title: 显示源码相关命令 lsa lsc lsp lsf
date: 2009-11-27T09:25:00+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=109
permalink: /%e6%98%be%e7%a4%ba%e6%ba%90%e7%a0%81%e7%9b%b8%e5%85%b3%e5%91%bd%e4%bb%a4-lsa-lsc-lsp-lsf/
tags:
  - dbg
  - lsa
  - lsc
  - lsf
  - lsp
  - windbg
---
首先保证 .srcpath 设置正确
lsa address 列出 address 所在的代码块.如果你觉得显示的不够多, 可以用 lsp -a 30 来设置 lsa 命令显示的行数.
lsc 列出当前显示的文件名和行号
lsf filename 加载特定的源文件, 但此命令不理会 srcpath, 所以你必须指定相对当前目录的路径或者绝对路径. 所以, 这个命令不太实用.
