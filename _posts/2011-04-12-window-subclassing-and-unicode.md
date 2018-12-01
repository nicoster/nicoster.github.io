---
id: 430
title: Window Subclassing and Unicode
date: 2011-04-12T18:31:14+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=430
permalink: /window-subclassing-and-unicode/
tags:
  - dbg
  - subclass
  - unicode
  - window
  - windbg
---
最近我把机器的 system locale 切换成英文. 运行 Connect® 的时候遇到了一些以前没有遇到过的问题. 比如这个 子类化和unicode 之间的联系问题.

Tony 在 TriManualNotificationDlg 中发送含有中文字符的 Notification 时出现乱码. 怀疑是与 WAPI 的交互有关. 让我看一下. 我调试了一下. 发现, 我这边居然在 GetWindowText 就出现的问题, GetWindowText 居然返回了 3F 00, 也就是 '?'. 但在 Tony 的机器上 GetWindowText() 表现正常. 我依稀记得 window 是有 unicode 和 ansi 之分的. 用 spy++ 查看 TriManualNotificationDlg 的 notification msg 编辑框, 如图:

![unicode_wnd_1.png]({{site.url}}/attachments/2011/04/unicode_wnd_1.png)

显示这个 edit 是 subclassed. 又查看了一下 OK 按钮的属性. 如图:

![unicode_wnd_2.png]({{site.url}}/attachments/2011/04/unicode_wnd_2.png)

对比就可以看出. edit 不是 unicode 窗口.
搜索一下 'windowproc unicode', 找到了这样一篇文章.

<http://www.ibm.com/developerworks/cn/linux/l-cn-guimigrt/index4.html>

其中有这样一段描述:

>由于兼容性的原因，Windows既支持ANSI窗口过程，也支持Unicode窗口过程，两者分别接受不同字符集编码的窗口消息（关于字符集的详细描述请参见'Unicode与国际化'一章）。如果应用程序调用RegisterClassA函数注册窗口类或者调用SetWindowLongA函数设置窗口过程，指定的窗口过程将只接受ANSI编码的窗口消息（如`WM_SETTEXT`）。如果应用程序调用RegisterClassW或 SetWindowLongW函数，则指定的窗口过程将只接受Unicode编码的窗口消息。系统必须跟踪特定窗口过程是ANSI或Unicode的，并且在必要的时候自动对消息参数进行编码转换。例如，如果一个窗口过程只接受Unicode版本的消息，那么当调用SendMessageA函数发送 ANSI编码的WM_SETTEXT消息时，系统就必须先将字符串参数（即lParam）转换为Unicode编码的字符串，再将转换后的Unicode 字符串传递给该窗口过程。反之亦然。


既然现在的 edit 是子类化过的. 那去掉子类化的代码看看如何. 遂注释掉 OnInitDialog() 中的:
	
	//	m_editNotificationMsg.SubclassWindow(GetDlgItem(IDC_NOTIFICATION_MESSAGE));
	
运行程序, 用 spy++ 查看, 居然显示还是 subclassed. 奇怪了. 既然是子类化, 而且现在是 ANSI 窗口. 根据上面文章的描述, 应该是调用了 SetWindowLongA, 用 windbg 加载 Connect.exe 调试
<pre>bp user32!SetWindowLongA "k;g"</pre>
在 SetWindowLongA 上设置断点. 运行程序, 发现了始作俑者是 coolsb. 调用栈如下:

	0012dcec 00bf59a2 USER32!SetWindowLongA
	0012dd54 00bf545d connect!CoolSBWndProc+0x5e2
	0012dd78 77291a10 connect!CoolSBWndProc+0x9d
	0012dda4 77293123 USER32!InternalCallWinProc+0x23
	0012de1c 77291c03 USER32!UserCallWinProcCheckWow+0xe0
	0012de78 77293656 USER32!DispatchClientMessage+0xda
	0012dea0 774c0e6e USER32!__fnDWORD+0x24
	0012decc 77292335 ntdll!KiUserCallbackDispatcher+0x2e
	0012ded0 7727f7f7 USER32!NtUserMessageCall+0xc
	0012df0c 77292bba USER32!SendMessageWorker+0x4d5
	0012df2c 003a177f USER32!SendMessageW+0x7c
	0012df94 0038caca apuidll!apuidll::CWbxEditBox::OnMouseHover+0x3f
	0012e004 003849f5 apuidll!apuidll::CWbxEditBox::ProcessWindowMessage+0x9a 
	0012e0a4 77291a10 apuidll!ATL::CWindowImplBaseT<WTL::CEditT<ATL::CWindow>,ATL::CWinTraits<1442840576,0> >::WindowProc+0x85 
	0012e0d0 77291ae8 USER32!InternalCallWinProc+0x23
	0012e148 77292d6e USER32!UserCallWinProcCheckWow+0x14b
	0012e178 7727991c USER32!CallWindowProcAorW+0x97
	0012e198 00bf55f6 USER32!CallWindowProcA+0x1b
	0012e1c0 77291a10 connect!CoolSBWndProc+0x236
	0012e1ec 77291ae8 USER32!InternalCallWinProc+0x23

以上结论还只是猜测, 没有经过论证.

James 将 coolsb 编译为 unicode 之后, spy++ 显示 edit 窗口已经是 unicode 版, `GetWindowText()`也能正确返回了. 问题解决.
至于 Tony 遇到的问题, 则是和 URLEncode 有关. 这里不再赘述了.
 
## References:
<http://www.codeproject.com/win32/safesubclassing.asp>
 
 
 
