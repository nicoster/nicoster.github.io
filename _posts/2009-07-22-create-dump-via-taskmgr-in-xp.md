---
id: 117
title: Create dump via taskmgr in XP
date: 2009-07-22T15:54:00+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=117
permalink: /create-dump-via-taskmgr-in-xp/
tags:
  - dbg
  - context menu
  - detour
  - dump
  - taskmgr
  - windbg
---
You can create a dump file in Taskmgr in Vista. now I’m gonna add this feature to Taskmgr in XP.

说一些细节.

* makedump.exe 用于创建 dump. 实现很简单. 只是调用了 MiniDumpWriteDump.
  makedump.exe 可被单独使用.
  
* makedmp.dll 用于 hook taskmgr.exe

  hook 一个 exe 有很多办法. 看看 核心编程 就知道. 我这里使用的 setdll.exe 更 taskmgr 增加 makedmp.dll 的依赖.
  
  makedmp.dll 的实现参考了 codeproject 上的 TaskEx 项目. 实际上也就是做了大量的精简. 因为我的需求也很简单, 只是需要给 Process list 的右键菜单增加两个菜单项.
  
  为了避免修改 system32 下的 taskmgr.exe, 我将 taskmgr.exe 复制到安装目录的taskmgrex.exe, 用 setdll 修改了本地拷贝. 然后又修改了 IFEO 键值. 每次 taskmgr.exe 被启动时实际是执行了 taskmgrex.exe.
* 编译出来的文件很小. makedump.exe (6k), makedmp.dll (3k)

[Download MakeDump]({{site.url}}/attachments/2009/07/mkdmp.zip)

![makedump.png]({{site.url}}/attachments/2009/07/makedump.png)
