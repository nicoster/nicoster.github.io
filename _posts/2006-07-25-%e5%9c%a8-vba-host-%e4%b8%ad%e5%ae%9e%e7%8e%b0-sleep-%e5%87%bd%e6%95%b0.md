---
id: 157
title: 在 VBA host 中实现 Sleep() 函数
date: 2006-07-25T18:06:00+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=157
permalink: /%e5%9c%a8-vba-host-%e4%b8%ad%e5%ae%9e%e7%8e%b0-sleep-%e5%87%bd%e6%95%b0/
categories:
  - Uncategorized
tags:
  - atl
  - sleep
  - vba
---
在 vba 中需要使用 sleep() 函数. 想当然, 一开始是这样实现的:STDMETHODIMP CShell::Sleep(int nMillSeconds){ ::Sleep(nMillSeconds); return S_OK;} 但很快就发现这样不行.  Sleep 的时候, 程序主界面死了. 应该用 GetMessage() 来实现. 但 GetMessage() 是阻塞的. 不拿到一条消息不会返回. 我需要的是 GetMessageTimeOut(), 可以指定一个超时. 超时后不管有没有消息都返回. 可惜 MSDN 里没有这样的函数. 那就只好在循环里面 PeekMessage(), 发现有消息再 GetMessage() 了. 象这样:
STDMETHODIMP CShell::Sleep(int nMillSeconds){ for (int n = 0; n < nMillSeconds; n += 50) {  MSG msg;  while (PeekMessage(&msg, NULL, NULL, NULL, PM_NOREMOVE))  {   if (GetMessage(&msg, NULL, NULL, NULL))   {    TranslateMessage(&msg);    DispatchMessage(&msg);   }  }  ::Sleep(50); } return S_OK;}
这样子能够满足需要. 但还是不够完美. 每 50ms 醒来一次. 醒来只是为了继续睡觉.我需要的是一直睡, 直到指定的时间再醒来. 并且还得在睡觉的时候有消息还得处理消息.还是 GetMessage(), 如果在指定的时间到了之后收到一条不管什么消息. 让 GetMessage() 返回即可.既然是指定的时间之后的消息, 那 WM_TIMER 就很自然了. 最后的实现是这样的:
STDMETHODIMP CShell::Sleep(int nMillSeconds){ MSG msg; int nTimerID = SetTimer(NULL, 0, nMillSeconds, NULL); while (GetMessage(&msg, NULL, NULL, NULL)) {  if (msg.message == WM_TIMER && msg.wParam == nTimerID)   return S_OK;  TranslateMessage(&msg);  DispatchMessage(&msg); }
 return S_FALSE;} 
