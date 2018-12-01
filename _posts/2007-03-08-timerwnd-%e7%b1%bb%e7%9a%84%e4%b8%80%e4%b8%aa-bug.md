---
id: 135
title: TimerWnd 类的一个 bug
date: 2007-03-08T17:22:00+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=135
permalink: /timerwnd-%e7%b1%bb%e7%9a%84%e4%b8%80%e4%b8%aa-bug/
categories:
  - Uncategorized
---
今天准备修改一下服务器程序关于 socket, 线程这一块的代码. 将 CObjectThread 的一个成员 m_pPCLayer 变成了父类. 发现程序在 TimerWnd 的代码出现问题. 分析发现, TimerWnd 在 PCLayer 和 CCAMThread 中被使用. 代码改完之后, 程序会先创建 PCLayer 的 3 个 TimerWnd 的实例. 之后又创建 CCAMThread 的一个 TimerWnd 实例. 但是, CCAMThread 的实例却调用了 PCLayer 的 TimerWnd 的 WndProc. 导致出错. 调试的大部分时间其实被用来得出这些结论. 
有了这些, 后面的工作就好做了. 原来是 CCAMThread 创建 TimerWnd 窗口之前, 先注册窗口类. 但是注册失败了. 使用了 PCLayer 注册的窗口类创建了窗口. 自然, 调用的是 PCLayer 的 WndProc. TimerWnd 的设计之初的想法是, 反正 RegisterClass 函数在第二次注册时会失败. 没有多加考虑. 当一个程序中有两个不同的类继承了 TimerWnd, 今天这样的问题就出现了. 因为以前的程序一般都只有一个类使用了 TimerWnd, 所以问题一直没有发现. 
最后一个问题是, 为什么修改代码之前服务器能正常运行呢? 现在就是体现 SVN  的优势的时候了. 改代码之前我已经将代码做了一个 branch, 修改的是 branch 的代码. 现在我从 svn 上取服务器的主干代码到另一个工作拷贝. 编译, 加断点, 调试. 很快就得到了一个令人瞠目结舌的答案. 原来在服务器中 PCLayer 的定时器一直没有用过! 由于 PCLayer 中的定时器是一个冗余容错的设计. 不执行一般也不会有问题. 所以问题一直没有暴露. 
由于使用模版的缘故, 使得 TimerWnd 如果被多个类使用, 则需要注册不同的窗口类. 在这里. 模版就不是那么的合适了. 使用虚函数则不会有这样的问题: 即使被多个类使用. 也只需要一个窗口类, 一个 WndProc, 只是调用不同的子类中的 OnTimer 即可.  
