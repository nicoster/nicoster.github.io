---
id: 403
title: UnhandledExceptionFilter 未被调用分析一例
date: 2011-04-12T17:55:31+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=403
permalink: /unhandledexceptionfilter-not-called
tags:
  - dbg
  - dump
  - unhandledexceptionfilter
  - windbg
---
有时候会遇到 crash 的时候没有生成 dump file. 因为 unhandledexceptionfilter 没有被调用. 而没有被调用的原因, 文档上说有时候的确不会调用. 比如在处理异常的时候再次发生异常.
最近发现在 space 打开的情况下, unhandledexceptionfilter 不被调用, 此时, 会出现系统默认的 crash 对话框. 这个对话框的出现意味着 kernel32!UnhandledExceptionFilter 被调用了, 虽然在 2000, xp, vista 里, 出现对话框的具体实现都不一样, 但都是从 kernel32!UnhandledExceptionFilter 开始的.
这个函数并不长. 它会调用 IsDebugPortPresent(), 如果存在调试器, 则直接返回, 让调试器处理异常. 否则调用用户通过 SetUnhandledExceptionFilter() 注册的处理函数. 如果要调试自己的处理函数,可以在 IsDebugPortPresent() 函数返回之后, 将 eax 置 0.
<pre>bp 772b5a38 "r eax=0;g";bp connect!_UnhandledExceptionFilter</pre>
 
地址 772b5a38 是在 vista enterprise build 6000 中反汇编 kernel32!UnhandledExceptionFilter 得到的. 当然, 随着 windows 版本的不同地址是不一样的.
在 space 打开时, 触发异常, 的确没有进到 connect!_UnhandledExceptionFilter 中. 怀疑是不是我们的处理函数被替换了. SetUnhandledExceptionFilter() 将用户注册的函数地址存到全局变量 kernel32!BasepCurrentTopLevelFilter 中, 查看该变量的值, 却发现不是一个函数地址. 查看 SetUnhandledExceptionFilter() 的代码, 才知道它会将函数地址用 RtlEncodePointer() 加密. 而在调用这个函数时会使用 RtlDecodePointer() 解密. 既然每次调用 SetUnhandledExceptionFilter() 都会返回之前的处理函数的地址. 于是在手动触发异常之前先来一个 ASSERT:
<pre> 	ATLASSERT(_UnhandledExceptionFilter == SetUnhandledExceptionFilter(_UnhandledExceptionFilter));</pre>
 
运行程序, 居然真的 ASSERT 了. 有其他的代码在调用 SetUnhandledExceptionFilter(), 遂在其上加断点, 不看不知道, 一看吓一跳. 原来很多的 dll 都会设置自己的异常处理函数.
为了确保我们的处理函数能够调用, 过河拆桥, 在我们调用 SetUnhandledExceptionFilter() 之后, 给 SetUnhandledExceptionFilter() 打一个补丁, 以后再有调用, 则直接返回. 打补丁的代码很简单:
<pre>void PatchSetUnhandledExceptionFilter()
{
	static BYTE RETURN_CODE[] = { 0xc2, 0x04, 0x00}; // __asm ret 4
	MEMORY_BASIC_INFORMATION   mbi;
	DWORD dwOldProtect = 0;
	DWORD pfnSetFilter =(DWORD)GetProcAddress(GetModuleHandleW(L"kernel32.dll"), "SetUnhandledExceptionFilter");
	VirtualQuery((void *)pfnSetFilter, &mbi, sizeof(mbi) );
	VirtualProtect( (void *)pfnSetFilter, sizeof(RETURN_CODE), PAGE_READWRITE, &dwOldProtect);

	WriteProcessMemory(GetCurrentProcess(),	(void *)pfnSetFilter,	RETURN_CODE,	sizeof(RETURN_CODE), NULL);
	VirtualProtect((void *)pfnSetFilter, sizeof(RETURN_CODE), mbi.Protect, 0);
	FlushInstructionCache(GetCurrentProcess(), (void *)pfnSetFilter, sizeof(RETURN_CODE));
}</pre>
 
在 kernel32!UnhandledExceptionFilter() 的起始处直接写上 __asm ret 4.
测试通过.
<hr />
参考文献:
<ol>
<li><a href="http://www.microsoft.com/msj/0197/exception/exception.aspx">A Crash Course on the Depths of Win32™ Structured Exception Handling</a><img src="http://tricon.sz.webex.com/jspwiki/images/out.png" alt="" /></li>
<li><a href="http://www.dumpanalysis.org/blog/index.php/2007/05/19/inside-vista-error-reporting-part-1/">Inside Vista Error Reporting (Part 1)</a><img src="http://tricon.sz.webex.com/jspwiki/images/out.png" alt="" /></li>
<li><a href="http://eparg.spaces.live.com/blog/cns!59BFC22C0E7E1A76!2650.entry?wa=wsignin1.0">简单Access Violation的异常派发，Longhorn Server</a><img src="http://tricon.sz.webex.com/jspwiki/images/out.png" alt="" /></li>
<li><a href="http://www.donews.com/content/200602/22bf40febd8f414681aae5bcc1fa9650.shtm">Windows XP SP2溢出保护</a><img src="http://tricon.sz.webex.com/jspwiki/images/out.png" alt="" /></li>
</ol>
 
 
