---
id: 421
title: Yet Another Crash Analysis
date: 2011-04-12T18:19:27+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=421
permalink: /yet-another-crash-analysis/
tags:
  - dbg
  - application verifier
  - bp
  - gflag
  - windbg
---
手头有这么一个 bug，当启用 application verifier 之后，用键盘 navigate 菜单时会 crash。我在自己的机器上没能重现。 今天终于找到机会登陆到 qa 的机器上查这个问题。不错，在她的机器上能够很稳定的重现。
先把 pdb 下载下来添加到本地 symbol server, 在 qa 的机器上设置好 `_NT_SYMBOL_PATH` 环境变量。直接把 windbg 打开，开始调试。
这是 crash 的时候的调用栈等信息：

	apMenu!CSkinMenu::GetCurSel+0x4:
	22ec57df ff761c          push    dword ptr [esi+1Ch]  ds:0023:28b7afc4=????????
	
	0:000> kL
	ChildEBP RetAddr  
	0012f224 22ec6772 apMenu!CSkinMenu::GetCurSel+0x4
	0012f2d4 22ec9154 apMenu!CSkinMenu::WindowProc+0x282
	0012f304 76e11a10 apMenu!CSubclassWnd::HookWndProc+0x90
	0012f330 76e11ae8 USER32!InternalCallWinProc+0x23
	0012f3a8 76e11c03 USER32!UserCallWinProcCheckWow+0x14b</pre>
	查看一下GetCurSel 函数的代码：
	<pre>0:000> uf apMenu!CSkinMenu::GetCurSel
	apMenu!CSkinMenu::GetCurSel:
	22ec57db 56              push    esi
	22ec57dc 57              push    edi
	22ec57dd 8bf1            mov     esi,ecx
	22ec57df ff761c          push    dword ptr [esi+1Ch]	// crash 在这里
	22ec57e2 ff158cb2ec22    call    dword ptr [apMenu!_imp__GetMenuItemCount (22ecb28c)]
	22ec57e8 8bf8            mov     edi,eax
	22ec57ea eb14            jmp     apMenu!CSkinMenu::GetCurSel+0x25 (22ec5800)
	
	apMenu!CSkinMenu::GetCurSel+0x11:
	22ec57ec 6800040000      push    400h
	22ec57f1 4f              dec     edi
	22ec57f2 57              push    edi
	...
	
this 指针有问题. 既然只有加载 appverifier 才能重现，十有八九是 这个对象被删除了. 看看这个地址的属性:

	0:000> !address @esi
	 ProcessParametrs 0015a808 in range 0015a000 0015b000
	 Environment 26f3c448 in range 26f3c000 26f3d000
	    28aa0000 : 28b79000 - 00003000
	                    Type     00020000 MEM_PRIVATE
	                    State    00002000 MEM_RESERVE	// 的确不是 committed 的状态. 应该是被删除了.
	                    Usage    RegionUsagePageHeap
	                    Handle   04721000
                    
通过 review 代码应该能找出来是在哪里被删除的，但我对这块逻辑不熟悉，还是让 windbg 来找找吧.
这里我们需要使用 gflags 启用这个进程的 stack trace.
	
	C:\Program Files\Debugging Tools for Windows (x86)>gflags /i app.exe +ust
	Current Registry Settings for app.exe executable are: 00001100
	    vrf - Enable application verifier
	    ust - Create user mode stack trace database

重起程序, 重现 crash，用 !heap 命令就可以看到这个地址到底是在什么时候被删除的。


	0:000> !heap -p -a @esi
    address 28b7afa8 found in
    _DPH_HEAP_ROOT @ 4721000
    in free-ed allocation (  DPH_HEAP_BLOCK:         VirtAddr         VirtSize)
                                   28a2b9c0:         28b7a000             2000
    774fd9fa ntdll!RtlpFreeHeap+0x0000005f
    774e1c21 ntdll!RtlFreeHeap+0x0000014e
    72d6bb0b vfbasics!AVrfpRtlFreeHeap+0x0000016b
    75df7a7e kernel32!HeapFree+0x00000014
    705d38bb MSVCR90!free+0x000000cd
    22ec5ed2 apMenu!CSkinMenu::`vector deleting destructor'+0x00000017
    22ec81fc apMenu!CSkinMenuMgr::Unskin+0x00000093
    22ec86c8 apMenu!CSkinMenuMgr::OnCallWndProc+0x000000fe
    22ec8807 apMenu!CHookMgr<CSkinMenuMgr>::CallWndProc+0x00000056
    76e03617 USER32!DispatchHookW+0x00000033
    76df4e8b USER32!fnHkINLPCWPSTRUCTW+0x0000004f
    76e0933f USER32!__fnINOUTLPWINDOWPOS+0x00000027
	...
	
原来是在调用一个 Unskin() 函数的时候删除的。看一下源文件，还有一个对应的 Skin()， 其中会 new 出这个对象，分别在这两个函数放两个断点， 反汇编一下这个函数，就知道应该在哪里放断点了.

	0:000> uf apMenu!CSkinMenuMgr::Skin
	apMenu!CSkinMenuMgr::Skin:
	22ec84ac 6a04            push    4
	22ec84ae b889a5ec22      mov     eax,offset apMenu!_clean_type_info_names_internal+0x4f1 (22eca589)
	22ec84b3 e820100000      call    apMenu!_EH_prolog3 (22ec94d8)
	22ec84b8 8bf1            mov     esi,ecx
	22ec84ba ff7508          push    dword ptr [ebp+8]
	22ec84bd e854d2ffff      call    apMenu!CSkinMenu::IsMenuWnd (22ec5716)
	22ec84c2 59              pop     ecx
	22ec84c3 85c0            test    eax,eax
	22ec84c5 0f84a1000000    je      apMenu!CSkinMenuMgr::Skin+0xc0 (22ec856c)
	... ...
	apMenu!CSkinMenuMgr::Skin+0x35:
	22ec84e1 6a58            push    58h
	22ec84e3 e82e120000      call    apMenu!operator new (22ec9716)	// 这里是 new， eax 就是返回值
	22ec84e8 59              pop     ecx
	22ec84e9 8bc8            mov     ecx,eax
	22ec84eb 894df0          mov     dword ptr [ebp-10h],ecx
	...

那就在 22ec84e8 这一行放断点吧：

	0:000>bp 22ec84e8 '.printf \'skin \';r eax;g'
	类似的， 在 unskin 函数 delete 之前也放一个
	0:000>bp 22ec819f '.printf \'unskin\';r eax;g'

放好之后是这样的：

	0:000> bl
	 0 e 22ec819f     0001 (0001)  0:**** apMenu!CSkinMenuMgr::Unskin+0x36 ".printf \"unskin \";r eax;g"
	 1 e 22ec84e8     0001 (0001)  0:**** apMenu!CSkinMenuMgr::Skin+0x3c ".printf \"skin \";r eax;g"

重启进程，重现:
	
	skin eax=2d3ddfa8	// 第一次 skin new 了一个对象
	unskin eax=2d3ddfa8  // 第一次 unskin 删除了这个对象
	unskin eax=00000000  // 这次 unskin 没有作用，因为 对象已经清空了
	unskin eax=00000000  // 同上
	skin eax=2d40dfa8  // 第二次 skin
	(e40.e74): Access violation - code c0000005 (first chance)
	First chance exceptions are reported before any exception handling.
	This exception may be expected and handled.
	eax=00000000 ebx=00000027 ecx=2d3ddfa8 edx=00000030 esi=2d3ddfa8 edi=00000000
	eip=22ec57df esp=0012f220 ebp=0012f2d4 iopl=0         nv up ei pl zr na pe nc
	cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00210246
	apMenu!CSkinMenu::GetCurSel+0x4:
	22ec57df ff761c          push    dword ptr [esi+1Ch]  ds:0023:2d3ddfc4=????????  // 这里试图访问第一次 new 出来的那个对象
	 
可以看到问题就在第二次的时候访问第一次已经删除了的对象。 分析到这里也差不多了 让负责这个模块的 dev 去查吧。
 
