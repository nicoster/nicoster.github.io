---
id: 123
title: 实现 UnhandledExceptionFilter() 需要的几个问题
date: 2007-12-12T19:42:00+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=123
permalink: /%e5%ae%9e%e7%8e%b0-unhandledexceptionfilter-%e9%9c%80%e8%a6%81%e7%9a%84%e5%87%a0%e4%b8%aa%e9%97%ae%e9%a2%98/
tags:
  - dbg
  - dump
  - unhandledexceptionfilter
  - windbg
---
dump file 是分析程序 crash 的利器, 在程序 crash 时写 dump 文件就是很自然的了. 而想要在程序 crash 的时候写 dump, 就不得不提 UnhandledExceptionFilter() 函数. 通过 API SetUnhandledExceptionFilter() 将自己写的 UnhandledExceptionFilter() 告诉操作系统, 系统在程序 crash 的时候就会调用这个函数. 用户在其中就可以完成想要的功能.
首先要解决的是 UnhandledExceptionFilter() 被多次调用的问题.
试想, 程序在一个线程中 crash, 执行 UnhandledExceptionFilter(), 在执行过程中, 另一个线程又出现 crash, 再次进入 UnhandledExceptionFilter() , 一般的这不是期望的结果. 所以, 在执行 UnhandledExceptionFilter() 时, 应该确保进程中所有其他的线程都被挂起. 就像当一个进程被调试器断点时, 被调试进程所有线程都被挂起一样. 似乎没有线程的 API 比如 SuspendAllThreads() 可以用, 但自己实现也很简单. 用 toolhelp 枚举所有线程, 逐个挂起即可, 注意不要把自己挂起.
UnhandledExceptionFilter() 被多次执行的另一种情况是, 其他线程都挂起了, 出错的线程执行完 UnhandledExceptionFilter() 之后退出, 进程还需要做一些清理工作, 在这个时候再次 crash, 又一次执行 UnhandledExceptionFilter(). 我在 UnhandledExceptionFilter() 的开始处设了一个标志, 如果不是第一次执行, 则直接返回.
另一个需要注意的是, 其他的 dll 有可能会调用 SetUnhandledExceptionFilter() 函数设置自己的异常处理函数. 如果发现有时候出现异常, 却没有 dump 文件, 可以先查看一下是不是被其他的 dll 替换了异常处理函数. 当然, 还有其他一些原因也会导致没有 dump 文件, 比如发生异常时调用栈等数据结构被严重破坏, 异常处理函数中又抛出未处理的异常等.
<未完待续>
参考文献:
http://www.microsoft.com/msj/0197/exception/exception.aspx
http://www.dumpanalysis.org/blog/index.php/2007/05/19/inside-vista-error-reporting-part-1/
http://eparg.spaces.live.com/blog/cns!59BFC22C0E7E1A76!2650.entry?wa=wsignin1.0
