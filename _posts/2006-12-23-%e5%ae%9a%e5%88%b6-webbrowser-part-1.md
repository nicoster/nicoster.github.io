---
id: 139
title: 定制 WebBrowser (Part 1)
date: 2006-12-23T16:45:00+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=139
permalink: /%e5%ae%9a%e5%88%b6-webbrowser-part-1/
categories:
  - Uncategorized
tags:
  - atl
  - COM
  - webbrowser
---
用 WTL 向导创建一个程序 WtlHtml, 显示 HTML page. 其实也就是 host 一个 webBrowser 控件. 打算做一些定制的工作. 
1. 首先需要定制的是指定 external IDispatch 接口. 我们可以向项目中增加一个 ATL Object. 或者, 简单一点, 直接在现有的类上实现. 我打算采用第二种方法. 在 CWtlHtmlView 上实现. 给 CWtlHtmlView 增加两个父类:class CWtlHtmlView :  public CWindowImpl<CWtlHtmlView, CAxWindow>, public CComObjectRootEx<CComSingleThreadModel>, // 提供 IUnknown 接口 public IDispatchImpl<IWtlHtml, &IID_IWtlHtml, &LIBID_WTLHTMLLib> // 实现 IDispatch{..}
既然用 IDispatchImpl, 那得有自己的类型库. 创建一个 idl. 从别的项目中复制一份. 将名字. GUID 换了就可以了. 编译之, 就生成了 tlb (LIBID_WTLHTMLLib). 还得在类的声明中加上 COM map: BEGIN_COM_MAP(CWtlHtmlView)  COM_INTERFACE_ENTRY(IDispatch) END_COM_MAP() 现在 CWtlHtmlView 成了纯虚类. 不能实例化了. 所以, 修改 CMainFrame 中 m_view 的定义: CComObject<CWtlHtmlView> m_view; 由于 m_view 没有在堆上创建, 所以不用考虑生命期的问题, 当然也不能被 Release() 里的 delete 掉, 我们可以先在 CMainFrame::OnCreate() AddRef() 一下, 再设置 external. 
 m_view.AddRef(); m_view.SetExternalDispatch(static_cast<IWtlHtml*>(&m_view)); 现在已经可以在 html 中调用 IWtlHtml 接口中的方法了.  
2. 然后为了让网页上的按钮能有 visual style, 需要设置 DOCHOSTUIFLAG_THEME. 一般的方法就是实现 IDocHostUIHandler, 在 ATL 中有一个现成的 IDocHostUIHandlerDispatch 接口. 实现这个比较简单, 继续给 CWtlHtmlView 增加父类. class CWtlHtmlView :  .. public IDispatchImpl<IDocHostUIHandlerDispatch, &IID_IDocHostUIHandlerDispatch, &LIBID_ATLLib>{..}这里的 LIBID_ATLLib 是 ATL.dll 自带的 tlb, 用 oleview 可以看到, 里面只有两个接口. IDocHostUIHandlerDispatch.IAxWinAmbientDispatch;加入继承了 IID_IDocHostUIHandlerDispatch. 我们不得不实现它的方法. 好在可以直接返回 E_NOTIMPL; 一个比较简单的方法就是从 atliface.h 文件中将方法声明都粘过来. 实现都写成 {return E_NOTIMPL;} 不要忘了实现 GetHostInfo(), 这是我们实现 IDocHostUIHandlerDispatch 的目的:
    virtual HRESULT STDMETHODCALLTYPE GetHostInfo(         /* [out][in] */ DWORD __RPC_FAR *pdwFlags,        /* [out][in] */ DWORD __RPC_FAR *pdwDoubleClick) {  ATLTRACE( "called GetHostInfon ");     *pdwFlags = DOCHOSTUIFLAG_THEME | DOCHOSTUIFLAG_SCROLL_NO | DOCHOSTUIFLAG_NO3DOUTERBORDER;  return S_OK; } 
我们在这里还设置了 DOCHOSTUIFLAG_SCROLL_NO 标志位, 用于去掉右边的滚动条. DOCHOSTUIFLAG_NO3DOUTERBORDER 用于去掉多余的 border. 在 CMainFrame::OnCreate() 中加入: m_view.SetExternalUIHandler(static_cast<IDocHostUIHandlerDispatch*>(&m_view)); 应该差不多了, 编译. 没有起作用? 而且第一步已经实现的 external 也不可用了. 查看了一下 IDocHostUIHandlerDispatch, 发现有个 GetExternal 方法. 一查, 原来 webBrowser 发现你实现了 IDocHostUIHandlerDispatch, 它会在需要的时候调用 GetExternal() 获得这个 IDispatch. 因为我们直接返回了 E_NOTIMPL.  所以导致 external 不可用了. 修改也很简单:    virtual HRESULT STDMETHODCALLTYPE GetExternal(         /* [out] */ IDispatch __RPC_FAR *__RPC_FAR *ppDispatch) {  *ppDispatch = static_cast<IWtlHtml*>(this);   return S_OK; } 
新问题解决了. 但我们要的 visual style 还是没有. 没有办法. google 吧. 找到了 codeguru 的一个帖子:<a href="http://www.codeguru.com/forum/archive/index.php/t-164752.html">http://www.codeguru.com/forum/archive/index.php/t-164752.html</a>  原来是因为在调用 m_view.Create() 的时候就创建了控件, 加载了 HTML 页面. 但我们的 SetExternalUIHandler() 调用是在这之后的. 所以当前页面没有 visual style. 但现在如果调用 Navigate2() 到一个新页面就可以看到设置的那些 flag 就生效了. 最直接的办法就是在 Create() 的时候先到 about:blank, 设置属性之后在 Navigate2 到我们要的页面. 不过 codeguru 的帖子里提到了另一种方法: 调用 IOleControl::OnAmbientPropertyChange() 通知 webBrowser 属性改变了. webBrowser 得到通知后自然就会重新应用新的属性. 代码就象这样:  CComPtr<IOleControl> spOC;  HRESULT hr = m_view.QueryControl(IID_IOleControl, (void**)&spOC);  if (SUCCEEDED(hr))   spOC->OnAmbientPropertyChange(DISPID_UNKNOWN); DISPID_UNKNOWN 表示告诉 webBrowser 所有的属性都要重新应用一遍. 现在再运行程序. 已经是预期的效果了.  3. 我们常常还要将右键菜单去掉. 不过现在已经是举手之劳了. IDocHostUIHandlerDispatch 还有一个 ShowContextMenu(), 重载这个方法就可以了:    virtual HRESULT STDMETHODCALLTYPE ShowContextMenu(         /* [in] */ DWORD dwID,        /* [in] */ DWORD x,        /* [in] */ DWORD y,        /* [in] */ IUnknown __RPC_FAR *pcmdtReserved,        /* [in] */ IDispatch __RPC_FAR *pdispReserved,        /* [retval][out] */ HRESULT __RPC_FAR *dwRetVal)  {*dwRetVal = S_OK; return S_OK;}     *dwRetVal = S_OK 是告诉 webBrowser 菜单不用它操心了. 我们自己搞定. 
