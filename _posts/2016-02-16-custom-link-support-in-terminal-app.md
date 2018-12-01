---
id: 721
title: Custom Link Support in Terminal.app
date: 2016-02-16T00:10:02+00:00
author: nick
layout: post
permalink: /custom-link-support-in-terminal-app/
tags:
  - terminal
  - hyperlink
  - osx
---
一直很喜欢 Windbg 里的 DML 的用法. 也就是点击一个 link, 可以在 windbg 中执行一条命令. 于是想把它搬到 terminal 里. 实现点击一个 link 就能在 terminal 里执行一条命令. 

在 mac 给 app 写插件, 有一个框架可以使用. 那就是 SIMBL. 它可以将 dylib 加载到目标进程. 再使用 objc 的 method swizzling, 可以很容易的实现 API hook 的效果. 

接下来就要找出从哪里入手了.
Terminal.app 里本来就支持点击 `cmd + dbl-click`普通的的 URL. 用 classdump 找到了好些 url 相关的 method, property, member 等. 比如 `-[TTView openURL:]` 和 `-[TTApplication openURL:]`.

先试试在这两个函数上放断点.
先解决调试的问题. 找了一个现成的 Terminal.app 的 plugin - [mouseterm](https://bitheap.org/mouseterm/), 创建一个 Xcode 项目, 稍作调整, 设置好 debugee 为 Terminal.app 就可以在 Xcode 里调试了.
怎么设断点呢? classdump 是通过 objc 的 API 来遍历一个 binary 里的所有 class, method 等信息. lldb 却没有利用这些信息. (有时间的话, 可以考虑写一个 python 脚本将所有的符号导入到 lldb, 再加上 auto completion, 实现类似 cycript 的效果, 那就方便了)
比如上面提到的两个 openURL 方法. lldb 里看到的是 

	___lldb_unnamed_function1412$$Terminal
	___lldb_unnamed_function1784$$Terminal

在 [StackOverflow](http://stackoverflow.com/questions/24230475/import-class-dump-info-into-lldb) 找到了一个 break_message.py 导入到 lldb 里之后, 就可以通过 

	break_message -[TTView openURL:] 
	break_message -[TTApplication openURL:]

来设置断点了. 很方便!
随意在 bash 里输入一个 URL. cmd+双击, 发现断点并没有到 :( 
看看这两个函数的代码吧. 

	(lldb) disas -n ___lldb_unnamed_function1412$$Terminal
	Terminal`___lldb_unnamed_function1412$$Terminal:
	0x1000623e5 <+0>:   push   rbp
	0x1000623e6 <+1>:   mov    rbp, rsp
	0x1000623e9 <+4>:   push   r15
	0x1000623eb <+6>:   push   r14
	0x1000623ed <+8>:   push   rbx
	0x1000623ee <+9>:   push   rax
	0x1000623ef <+10>:  mov    rbx, rdi
	0x1000623f2 <+13>:  mov    rdi, qword ptr [rip + 0x744b7] ; (void *)0x00007fff7c3ca6e0: NSWorkspace
	0x1000623f9 <+20>:  mov    rsi, qword ptr [rip + 0x70520] ; "sharedWorkspace"
	0x100062400 <+27>:  mov    r15, qword ptr [rip + 0x4e089] ; (void *)0x00007fff8d0190c0: objc_msgSend
	0x100062407 <+34>:  call   r15
	0x10006240a <+37>:  mov    r14, qword ptr [rip + 0x753ef]
	0x100062411 <+44>:  mov    rdx, qword ptr [rbx + r14]
	0x100062415 <+48>:  mov    rsi, qword ptr [rip + 0x7239c] ; "openURL:"
	0x10006241c <+55>:  mov    rdi, rax
	0x10006241f <+58>:  call   r15
	0x100062422 <+61>:  test   al, al
	0x100062424 <+63>:  jne    0x10006242b               ; <+70>
	0x100062426 <+65>:  call   0x10008eb46               ; symbol stub for: NSBeep
	0x10006242b <+70>:  mov    rdi, qword ptr [rbx + r14]
	0x10006242f <+74>:  mov    rsi, qword ptr [rip + 0x6feda] ; "release"
	0x100062436 <+81>:  call   qword ptr [rip + 0x4e054] ; (void *)0x00007fff8d0190c0: objc_msgSend
	0x10006243c <+87>:  mov    qword ptr [rbx + r14], 0x0
	0x100062444 <+95>:  add    rsp, 0x8
	0x100062448 <+99>:  pop    rbx
	0x100062449 <+100>: pop    r14
	0x10006244b <+102>: pop    r15
	0x10006244d <+104>: pop    rbp
	0x10006244e <+105>: ret    

看起来很简单, 就是调用了一下 `-[NSWorkspace openURL:]`. 
另一个函数代码比较多. 这里就不贴出来了.
那就在 `-[NSWorkspace openURL:]` 放断点试试吧. 这次一下子就断下来了 :)

原来在 `-[TTView mouseUp:]` 里会直接调用 `-[NSWorkspace openURL:]`. 在 lldb 里检查一些变量(其中的 \_clickedURL, selectedText 都是通过 classdump 发现的, $rdi 是 self 指针. 可以查看 x86_64 abi 文档) 

	(lldb) p (TTView*)$rdi
	(TTView *) $33 = 0x00000001003608d0
	(lldb) p $33.selectedText
	(__NSCFString *) $34 = 0x000000010020f4e0 @"http://baidu.com"
	(lldb) p $33->_clickedURL
	(NSURL *) $35 = 0x00000001004aafd0 @"http://baidu.com"

在折腾 openURL 之前, 发琢磨了一下重载 NSURLProtocol 处理自定义 URL scheme, 发现通过 `-[WebFrame loadRequest:]` 是起作用的. 但 `-[NSWorkspace openURL:]` 发起的请求不会加载自定义的 URL scheme. 看了很久才发现, 文档里写着:

	/* Open a URL, using the default handler for the URL's scheme. */
	- (BOOL)openURL:(NSURL *)url;

既然如此, 那就简单粗暴的 hook 一下处理自己扩展的 scheme 吧.
在调试的过程中发现, 我处理了 sh:// 这个 scheme 之后, 进入默认的处理流程, 也就是弹出对话框说无法处理 sh:// 协议. 跟踪了一下代码, 发现默认处理时会判断 `TTView->_clickedURL`, 如果不为空则弹出对话框. 但这个 ivar 并没有对应的 property. 
那么这个变量是在哪里被清空的呢:

	(lldb) p $0->_clickedURL
	(NSURL *) $1 = 0x000000010029b310 @"sh://ls%20"
	(lldb) p &($0->_clickedURL)
	(id *) $2 = 0x0000000100267438
	(lldb) watch set e $2
	Watchpoint created: Watchpoint 1: addr = 0x100267438 size = 8 state = enabled type = w
	    new value: 0x000000010029b310

通过 watchpoint 很容易就发现原来就是在 `-[TTView mouseUp:]` 方法里. 所以打消了我想直接调用某个方法清除这个 ivar 的想法.

其实 lldb 是可以直接访问到这个 ivar 的. 所以想到通过 objc runtime 应该是可以访问到的. 代码如下:

	NSURL* clickedURL = nil;
	object_getInstanceVariable(self, "_clickedURL", (void**)&clickedURL);

处理完成后, 将其置空:

	object_setInstanceVariable(self, "_clickedURL", nil);
	[clickedURL release];
 
至此, 大功告成. 代码很简单, 不到 100 行. 见 [github](https://github.com/nicoster/termlinkexec)
