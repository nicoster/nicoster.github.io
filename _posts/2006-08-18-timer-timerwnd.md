---
id: 156
title: Timer, TimerWnd
date: 2006-08-18T11:29:00+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=156
permalink: /timer-timerwnd/
categories:
  - Uncategorized
---
有时候, 你需要一个定时器. 可能你在这个线程里还没有属于自己的窗口. 你当然也可以不创建窗口而直接使用线程里的定时器消息. 就象这样(以 CWinThread 的一个派生类为例)

	CYourThread::InitInstance()
	{
	 m_nTimerID = ::SetTimer(NULL, NULL, 1000/*ms*/,  NULL);
	 …
	}
	
	BEGIN_MESSAGE_MAP()
	 ON_WM_TIMER()
	 …
	END_MESSAGE_MAP
	
	CYourThread::OnTimer(UINT nID)
	{
	 if (nID == m_nTimerID)
	 {
	  // do your work here
	 }
	}

这个方法的缺点在于. 如果你不用 CWinThread 的派生类. 可能添加 WM_TIMER 的消息处理函数比较麻烦. 甚至不能实现
所以, 还是回到老办法, 创建一个辅助窗口. 写自己的 WM_TIMER 处理函数. 但如果使用 MFC 的 CWnd 的派生类. 会遇到这样的问题. 你可能不能跨线程来关闭这个窗口. 或者一些其他的窗口操作. 又是由于 MFC 里臭名昭著的大量的 map, 这些 map 都是线程相关的. 既然我们这里的应用只是一个定时器的辅助窗口. 我们完全可以自己实现一个 TimerWnd. 就像这样:


	// TimerWnd.h start here ————————————————-
	
	/* 06-8-14 1124 
	 a simple HWND encapsulation for timer 
	*/
	
	#ifndef _TIMERWND_H_
	#define _TIMERWND_H_
	
	class TimerWnd
	{
	public:
	 TimerWnd(){Initialize();}
	
	private:
	    static  LRESULT CALLBACK
	    WndProc(HWND hwnd, UINT uiMsg, WPARAM wParam, LPARAM lParam)
	    {
	  if (uiMsg == WM_TIMER)
	  {
	   TimerWnd *that = (TimerWnd *)GetWindowLong(hwnd, GWL_USERDATA);
	   that->OnTimer(wParam);
	   return 0;
	  }
	        return DefWindowProc(hwnd, uiMsg, wParam, lParam);
	    }
	
	 virtual void OnTimer(UINT uID)
	 {
	  OutputDebugString("TimerWnd::OnTimer  ");
	 }
	    
	    static BOOL Initialize()
	    {
	        WNDCLASSW wc = {0, WndProc, 0, 0, 0, 0, 0, 0, 0, L"TimerWndClass"};
	   
	        if (!RegisterClassW(&wc)) return FALSE;
	  return TRUE;
	    }
	    
	public:
	    BOOL Create()
	    {
	        HWND hwnd = CreateWindowW(
	            L"TimerWndClass",                      /* Class Name */
	            L"TimerWnd",                      /* Title */
	            WS_OVERLAPPED,            /* Style */
	            CW_USEDEFAULT, CW_USEDEFAULT,   /* Position */
	            CW_USEDEFAULT, CW_USEDEFAULT,   /* Size */
	            NULL,                           /* Parent */
	            NULL,                           /* No menu */
	            NULL,                          /* Instance */
	            0);                             /* No special parameters */
	
	  if (hwnd == NULL) return FALSE;
	  SetWindowLong(hwnd, GWL_USERDATA, (long)this);
	        m_hwnd = hwnd;
	  return TRUE;
	    }
	
	 BOOL DestroyWindow()
	 {
	  return ::DestroyWindow(m_hwnd);
	 }
	
	 UINT_PTR SetTimer(UINT_PTR nIDEvent, UINT uElapse)
	 {
	  return ::SetTimer(m_hwnd, nIDEvent, uElapse, NULL);
	 }
	
	 BOOL KillTimer(UINT_PTR uIDEvent)
	 {
	  return ::KillTimer(m_hwnd, uIDEvent);
	 }
	    
	    HWND m_hwnd;
	};
	
	#endif // _TIMERWND_H_
	
	// TimerWnd.h end here ————————————————-


使用起来很简单, 从 TimerWnd 派生一个窗口类. 重载 OnTimer(), 在初始化的时候调用 Create() 创建即可.. 使用简单, 不用记住繁琐的 Create 参数. 没有 MFC 的烦人的 map. 可以跨线程操作.
说说实现上的几个细节.. 使用了宽字符版本的 API, 从理论上讲, 效率能高一点.. 从窗口句柄转换为对象, 使用的是简单的 Set/GetWindowLong, 因为只处理 WM_TIMER, 效率不成问题, 没有使用 MFC 繁琐的 map, 也没有用 ATL 的高深的 thunk.. 没有使用 .cpp, 直接 include 即可. 
