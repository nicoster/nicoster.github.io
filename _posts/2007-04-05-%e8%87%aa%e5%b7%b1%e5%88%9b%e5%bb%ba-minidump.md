---
id: 133
title: 自己创建 minidump
date: 2007-04-05T15:49:00+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=133
permalink: /%e8%87%aa%e5%b7%b1%e5%88%9b%e5%bb%ba-minidump/
tags:
  - dbg
  - dump
  - windbg
---
以前曾写过一个 bugslayer.dll 的介绍. 在程序出错时将调用栈写到文件. 觉得不错. 后来开始用 windbg. 知道了 userdump. 知道了如何调试 dump&#8230; 才知道程序崩溃的时候写 dump 文件其实可以获取比调用栈多得多的信息.  
如果你的程序什么都不干. 那么在程序出错的时候. drwtsn32.exe 会写一个 userdump. 但 drwtsn32 有些缺点. 比如只能写一个 dump 文件. 后面的崩溃写 dump 文件时会覆盖前面的. win2000 下的 drwtsn32 只能写旧式的 dump 文件(往往尺寸比较大). 有一篇文章论述的比较清楚: <a href="http://www.debuginfo.com/articles/ntsdwatson.html">http://www.debuginfo.com/articles/ntsdwatson.html</a>
建议使用 ntsd 代替 drwtsn32. 但 ntsd 的缺点就是需要安装最新的 windbg. 这是一个硬伤. 在看了 debuginfo.com 的另一篇文章: <a href="http://www.debuginfo.com/articles/effminidumps.html">http://www.debuginfo.com/articles/effminidumps.html</a> 之后, 我选择的是在程序出错的时候调用 api 自己写 minidump. 克服了 drwtsn32, ntsd 的缺点. 将封装好的函数放到了一个头文件中, 包含即可. 使用的方法很简单:
#include <windows.h>#include  "minidump.h "LONG __stdcall MyUnhandledExceptionFilter(PEXCEPTION_POINTERS pExceptionInfo){ CreateMiniDump(pExceptionInfo,  "c:\user.dmp "); return EXCEPTION_EXECUTE_HANDLER;}
void main(){ SetUnhandledExceptionFilter(MyUnhandledExceptionFilter); *(int*)0=0; // AV}
这样就好了. 注意安装至少 xp 以上的 sdk.这里用到了一个 api SetUnhandledExceptionFilter(), 如果不明白可以搜一下 msdn.运行例子程序出错退出之后, 就得到了 c:\user.dmp. 可以用 windbg 等调试器来分析了.  
代码从这里下载 <a href="http://nicoster.googlepages.com/minidump.rar">http://nicoster.googlepages.com/minidump.rar</a>
