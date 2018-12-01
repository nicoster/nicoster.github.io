---
id: 414
title: A Case of Heap Corruption
date: 2011-04-12T18:09:13+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=414
permalink: /a-case-of-heap-corruption/
tags:
  - dbg
  - heap corruption
  - windbg
---
At the very beginning, as I mentioned in the bug comments:
'dump reveals that crash occurred when try to allocate memory. it's a indication of heap corruption. Enable the heap option in application verifier, reproduce this bug, crash occurs earlier. I found that the CWclManager object gets deallocated when handling mouse move message. Refer to dump2 for more details. I hope this would be helpful when you diagnose this bug. &#8212; nickx, 5/15/2008 1:32:35 AM'
After reproducing the bug several times, I found it crashes if you click the cancel button, but not if you click the 'X' in the setting dialog. Checking the code, both call the same function of CSettingMainDlg::OnCancel(). That's quite peculiar. Then I changed the step.
Instead of clicking the cancel, I use the keyboard shortcut key to dismiss the dialog. the program survives. The crash has something to do with the mouse click. I set a breakpoint in the CSettingMainDlg::OnCancel(). Nothing special when click the 'X'. But when I click the cancel button, something different shows up, see the callstack:
<pre>0:000> kL100
ChildEBP RetAddr  
0012e828 0213b581 apExtCmp!CSettingMainDlg::OnCancel
0012e854 0210579d apExtCmp!CSettingMainDlg::ProcessWindowMessage+0x103
0012e87c 7e418734 apExtCmp!AT::CWclManager::DftWindowProc+0x2c
0012e8ac 7e418816 USER32!InternalCallWinProc+0x28
0012e914 7e41b4c0 USER32!UserCallWinProcCheckWow+0x150
0012e968 7e41b50c USER32!DispatchClientMessage+0xa3
0012e990 7c90eae3 USER32!__fnDWORD+0x24
0012e9b4 7e4194be ntdll!KiUserCallbackDispatcher+0x13
0012e9f0 7e41b903 USER32!NtUserMessageCall+0xc
0012ea10 02105fcf USER32!SendMessageW+0x7f
0012ea3c 02104a00 apExtCmp!AT::CWclManager::ProcessWindowMessage+0x99
0012ea68 02104786 apExtCmp!AT::CWclWin::SendMessageW+0x3a
0012ea9c 0211b043 apExtCmp!AT::CWclWin::ProcessWindowMessage+0x3bf
0012eacc 02104a00 apExtCmp!AT::CWclStaticImpl::ProcessWindowMessage+0x75
0012eaf8 02104aa9 apExtCmp!AT::CWclWin::SendMessageW+0x3a
0012eb0c 02106fe4 apExtCmp!AT::CWclWin::SendNotifyMessageW+0x32
0012eb1c 02107065 apExtCmp!AT::CWclButtonImpl::SendNotify+0x1a
0012eb28 02106cab apExtCmp!AT::CWclButtonImpl::OnLButtonUp+0x48
0012eb50 021072ef apExtCmp!AT::CWclButtonImpl::ProcessWindowMessage+0x6a
0012eb70 02104a00 apExtCmp!AT::CWclPushButtonImpl::ProcessWindowMessage+0x44
0012eb9c 02106453 apExtCmp!AT::CWclWin::SendMessageW+0x3a
0012ebb0 021065d0 apExtCmp!AT::CWclWinHelper::SendMouseMessage+0x6a  
0012ec04 02105de1 apExtCmp!AT::CWclManager::OnMouse+0x152            // here it is
0012ec38 021057ba apExtCmp!AT::CWclManager::ProcessWindowMessage+0x2e5
0012ec60 7e418734 apExtCmp!AT::CWclManager::DftWindowProc+0x49
0012ec90 7e418816 USER32!InternalCallWinProc+0x28
0012ecf8 7e4189cd USER32!UserCallWinProcCheckWow+0x150
0012ed58 7e418a10 USER32!DispatchMessageWorker+0x306
0012ed68 0052b5c4 USER32!DispatchMessageW+0xf
0012edd4 0052b35a connect!WTL::CMessageLoop::Run+0x124
0012fe20 005c164e connect!CTriApp::InitApp+0x1b2a
0012ff20 009cb0a3 connect!WinMain+0x24e
0012ffc0 7c816fd7 connect!WinMainCRTStartup+0x1b3
0012fff0 00000000 kernel32!BaseProcessStart+0x23</pre>
 
A SendMouseMessage was called in the CWclManager::OnMouse which finally called the OnCancel. Let's see the source code:
<pre>         ...
    if ( pNew )
    {
        CWclWinHelper::SendMouseMessage ( m_dwPriStyle & MGR_PRI_NCHITID, pNew, uMsg, wParam, lParam );
    }

    if ( WM_LBUTTONUP == uMsg )
        m_pDown = 0;          // crash here</pre>
 
Now we know what's really going on behind the scene. SendMouseMessage() call OnCancel which destroy the window and the whole object. Then SendMouseMessage returns and the code after the call gets executed. while at this moment, m_pDown is invalid since the whole object gets deallocated.
I'd like to share some knowledge about heap corruption.  m_pDown points to some memory not belonging to you, now gets cleared. If you're lucky enough, the program crashes immediately, thus provides you a chance to track down the bug. But the most common situation is, the program runs smoothly but crashes on a later occasion when try to allocate/deallocate memory, just like CrystalJ sees, when apAOLSdk try to new some memory. That's why the Application Verifier is so useful when diagnosing this kind of bugs on heap. It enables the program to crash immediately when something goes wrong.
After I comment out the code after SendMouseMessage, the program doesn't crash any more when I open/close the settings dialog.
 
