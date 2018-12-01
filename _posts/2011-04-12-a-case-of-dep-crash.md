---
id: 418
title: A Case of DEP Crash
date: 2011-04-12T18:14:13+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=418
permalink: /a-case-of-dep-crash/
tags:
  - dbg
  - atl thunk emulation
  - dep
  - windbg
---
bug 描述是这样的: client crashed when user tries to start it.
一启动程序就 crash. 这个 bug 有些奇怪, 只在 chellon 一台机器上出现, freq 100%. 用 windbg 查看 crash 生成的 dump 文件.
<pre>0:000> k
  *** Stack trace for last set context - .thread/.cxr resets it
ChildEBP RetAddr
0012f504 77d8867b apSMD!ObjectMap+0x30
0012f56c 77d8961e user32!UserCallWinProcCheckWow+0x150
0012f5c0 77d8e331 user32!DispatchClientMessage+0xa3
0012f5e8 77ccfb73 user32!__fnINOUTLPPOINT5+0x27
0012f630 77d907c6 ntdll!KiUserCallbackDispatcher+0x13
0012fad4 77d90887 user32!NtUserCreateWindowEx+0xc
0012fb80 77d9092b user32!_CreateWindowEx+0x1ed
0012fbbc 013a676f user32!CreateWindowExA+0x33
0012fc00 00a56060 apSMD!CTriSMD::CTriSMD+0xcf [E:\Workspace\TriMWM\windows\MWM\TriSMD\TriSMD.cpp @ 57]
0012fc28 004ce260 connect!TriSharing_ServiceImpl::TriSharing_ServiceImpl+0x50 [D:\workspace\src\UiMain\TriSharing_ServiceImpl.cpp @ 29]
0012fc2c 00a2f04a connect!TriSharing_GetService+0x20 [D:\workspace\src\UiMain\TriApp.cpp @ 71]
0012fc88 00521b2d connect!CTriRealSharingSessionMgr::OpenSession+0x10a [D:\workspace\src\UiMain\TriRealSharingSessionMgr.cpp @ 41]
0012fcd8 004d192b connect!CTriChatFrame::OnIdle+0xcd [D:\workspace\src\UiMain\TriChatFrame.cpp @ 130]
0012fce4 004d1862 connect!WTL::CMessageLoop::OnIdle+0x1b [..\h\atlapp.h @ 521]
0012fcfc 004d155a connect!WTL::CMessageLoop::Run+0x32 [..\h\atlapp.h @ 450]
0012fed4 0057c242 connect!CTriApp::InitApp+0x244a [D:\workspace\src\UiMain\TriApp.cpp @ 714]
0012ff24 00bc9a16 connect!WinMain+0x122 [D:\workspace\src\UiMain\TriConCli.cpp @ 183]
0012ff38 77cd1ca9 connect!WinMainCRTStartup+0x134
0012fff4 00bc98e2 ntdll!RtlFreeHeap+0x647
00000000 00000000 connect!WinMainCRTStartup</pre>
 
显示出错在 apSmd 中. 查看出错的指令:
<pre>0:000> .ecxr
eax=00000024 ebx=00050404 ecx=0012f604 edx=00000000 esi=013b9dd0 edi=013b9dc8
eip=013b9dd0 esp=0012f4bc ebp=0012f504 iopl=0         nv up ei pl nz na pe nc
cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00010206
apSMD!ObjectMap+0x30:
013b9dd0 c7442404c89d3b01 mov     dword ptr [esp+4],offset apSMD!ObjectMap+0x28 (013b9dc8) ss:0023:0012f4c0=00050404</pre>
 
查看一下前面的指令:
<pre>0:000> u eip - 10
apSMD!ObjectMap+0x20:
013b9dc0 0000            add     byte ptr [eax],al
013b9dc2 0000            add     byte ptr [eax],al
013b9dc4 0000            add     byte ptr [eax],al
013b9dc6 0000            add     byte ptr [eax],al
013b9dc8 c82d3b01        enter   3B2Dh,1
013b9dcc 0404            add     al,4
013b9dce 0500c74424      add     eax,2444C700h
013b9dd3 04c8            add     al,0C8h
0:000> u
apSMD!ObjectMap+0x35:
013b9dd5 9d              popfd
013b9dd6 3b01            cmp     eax,dword ptr [ecx]
013b9dd8 e953fdfeff      jmp     apSMD!ATL::CWindowImplBaseT<ATL::CWindow,ATL::CWinTraits<114229248,262400> >::WindowProc (013a9b30)
013b9ddd 0000            add     byte ptr [eax],al
013b9ddf 0000            add     byte ptr [eax],al
013b9de1 0000            add     byte ptr [eax],al
013b9de3 000f            add     byte ptr [edi],cl
013b9de5 af              scas    dword ptr es:[edi]</pre>
 
前后都是 0000, 这明显是在数据段啊. 怎么执行到这里来了. 第一个感觉就是程序跑飞了. 乱跳到这里. 在调用栈中下面的函数中放断点. 打算跟一下, 看看程序怎么跑到这里来的. 跟了好久, 也没有看出端倪(汇编功底太差!) 再回过头来看看出错的指令:
<pre>mov     dword ptr [esp+4],offset apSMD!ObjectMap+0x28 (013b9dc8) ss:0023:0012f4c0=00050404</pre>
 
将一个函数地址放到 esp+4 指向的地方. 而 esp+4 都是堆栈中的内存. 一般也就是临时变量, 调试器已经显示出其中的值. 也就是说, 这个地址肯定是可以访问的.这样一条指令怎么会出错呢? 不由得让我想起了前不久看到的一篇<a href="http://blogs.msdn.com/jeremyk/archive/2004/07/19/187696.aspx">文章</a>, crash 出现在
<pre>mov     eax,0x20</pre>
 
这样一条指令. 后来才发现是一个 hacker defender 的 rootkit 在捣乱. 难道我也有幸遇到这样的 rootkit?? 事实证明我没有这么幸运. &#8230; 在调试器上有浪费了一些时间之后, 可能是看到调用栈中的 user32!CreateWindowExA, 我灵光一闪, 突然想起了 atl thunk.
<div>简单的说 atl thunk 就是在创建窗口之后第一次收到消息时, atl 先初始化一个 thunk, 然后把 wndproc 指向这个 thunk, 之后的窗口消息都会先执行这个 thunk, 然后再去执行真正的窗口过程. 而 thunk 的作用只是把 wndproc 的 HWND 参数换成 pThis.</div>
查一下 ATL 的代码:
<pre>	void Init(WNDPROC proc, void* pThis)
	{
	#if defined (_M_IX86)
		thunk.m_mov = 0x042444C7;  //C7 44 24 0C
		thunk.m_this = (DWORD)pThis;
		thunk.m_jmp = 0xe9;
		thunk.m_relproc = (int)proc - ((int)this+sizeof(_WndProcThunk));

		...
	}</pre>
 
对比一下出错指令
<pre>013b9dd0 c7442404c89d3b01 mov     dword ptr [esp+4],offset apSMD!ObjectMap+0x28 (013b9dc8) ss:0023:0012f4c0=00050404
013b9dd8 e953fdfeff      jmp     apSMD!ATL::CWindowImplBaseT<ATL::CWindow,ATL::CWinTraits<114229248,262400> >::WindowProc (013a9b30)</pre>
 
My God! 看到 c7442404c89d3b01 没有? 真的就是这条指令. 那为什么这条指令会出错呢? 这就不得不提起 Data Execution Prevention(DEP).
<div>Data Execution Prevention (DEP) is a set of hardware and software technologies that perform additional checks on memory to help prevent malicious code from running on a system. In Microsoft Windows XP Service Pack 2 (SP2) and Microsoft Windows XP Tablet PC Edition 2005, DEP is enforced by hardware and by software.The primary benefit of DEP is to help prevent code execution from data pages. Typically, code is not executed from the default heap and the stack. Hardware-enforced DEP detects code that is running from these locations and raises an exception when execution occurs. Software-enforced DEP can help prevent malicious code from taking advantage of exception-handling mechanisms in Windows. 
</div>
 
简单的说, 就是为了增加病毒攻击的难度, 阻止执行位于数据段的代码. 而我们的 thunk 就位于数据段. DEP 分为两种, 硬件的和软件的. 真正起作用的是硬件 DEP.
<div>Hardware-enforced DEP Hardware-enforced DEP marks all memory locations in a process as non-executable unless the location explicitly contains executable code. A class of attacks exists that tries to insert and run code from non-executable memory locations. DEP helps prevent these attacks by intercepting them and raising an exception.Hardware-enforced DEP relies on processor hardware to mark memory with an attribute that indicates that code should not be executed from that memory. DEP functions on a per-virtual memory page basis, and DEP typically changes a bit in the page table entry (PTE) to mark the memory page. 
Software-enforced DEP An additional set of Data Execution Prevention security checks have been added to Windows XP SP2. These checks, known as software-enforced DEP, are designed to block malicious code that takes advantage of exception-handling mechanisms in Windows. Software-enforced DEP runs on any processor that can run Windows XP SP2. By default, software-enforced DEP helps protect only limited system binaries, regardless of the hardware-enforced DEP capabilities of the processor.
</div>
 
这个 bug 在其他的一些机器上不能重现, 就是因为哪些机器不支持 hardware DEP. 查看一下 chellon 的机器, 的确就是支持硬件 DEP 的. (ps: 只有 p4 以后的 CPU 才支持 hardware DEP). 禁用 DEP 之后, chellon 机器上的程序运行正常了.
但事情还没有完. 有几个问题还没有解决. 这个 crash 的故障现象是, 主窗口出现之后, 界面死掉几秒钟, 然后程序就消失了. 界面死掉的几秒是在写 dump 文件(100M+), 但注意, crash 之前主窗口已经显示了. 也就是说, thunk 这样的代码肯定早就执行过了. 但为什么没有 crash 呢? 答案可能是因为 ATL thunk emulation. 在最新的 MSDN 中有这样一个函数的介绍: <a href="http://msdn2.microsoft.com/en-us/library/bb736299.aspx">SetProcessDEPPolicy</a><img src="http://tricon.sz.webex.com/jspwiki/images/out.png" alt="" />
<div>Applications written to ATL 7.1 and earlier can attempt to execute code on pages marked as non-executable, which triggers an NX fault and terminates the application. DEP-ATL thunk emulation allows an application that would otherwise trigger an NX fault to run with DEP enabled. For information about ATL versions, see ATL and MFC Version Numbers.If DEP-ATL thunk emulation is enabled, the system intercepts NX faults, emulates the instructions, and handles the exceptions so the application can continue to run. If DEP-ATL thunk emulation is disabled by setting PROCESS_DEP_DISABLE_ATL_THUNK_EMULATION for the process, NX faults are not intercepted, which is useful when testing applications for compatibility with DEP. 
</div>
 
可以在程序中调用该函数禁用 ATL thunk emulation, 看程序会不会 crash. 郁闷的是这个函数要在 vista sp1 或者 windows server 2008 中才提供. 这个测试暂时没法做.
上面提到的 ATL7.1 是 vs2003 自带的, 之后的版本是 vs2005 自带的 ATL8.0. 据说已经解决了 DEP 这个问题. 主窗口没有 crash 是因为 ATL thunk emulation, 但为什么 apSMD.dll 中创建的一个窗口就会 crash 呢?我写了一个例子程序(见附件), 主程序调用 dll 的一个导出函数, 其中会创建一个窗口. 但程序没有 crash. 现在需要查看一下 apSMD.dll 中的代码了..(bug 已转给 york)
<hr />
some links on DEP:
<ul>
<li><a href="http://support.microsoft.com/kb/912923/en-us">How to determine that hardware DEP is available and configured on your computer</a></li>
<li><a href="http://blog.surfulater.com/2006/05/22/data-execution-protection-rex-winn/">Data Execution Protection & Rex Winn</a></li>
<li><a href="http://www.codeproject.com/useritems/thunk32.asp?df=100&forumid=367982&exp=0&select=1809849#xx1809849xx">Thunking in Win32: Simplifying callbacks to non-static member functions</a></li>
<li><a href="http://technet.microsoft.com/en-us/library/bb457155.aspx">Changes to Functionality in Microsoft Windows XP Service Pack 2: Part 3: Memory Protection Technologies</a></li>
<li><a href="http://discuss.develop.com/archives/wa.exe?A2=ind0402D&L=ATL&P=R355&I=-3">Discussion related to the Active Template Library and COM development</a></li>
</ul>
 
