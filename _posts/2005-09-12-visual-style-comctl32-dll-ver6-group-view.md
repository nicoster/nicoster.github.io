---
id: 16
title: Visual style, comctl32.dll ver6, group view
date: 2005-09-12T21:36:34+00:00
author: nick
layout: post
guid: http://nicoster.wordpress.com/2005/09/12/visual-style-comctl32-dll-ver6-group-view
permalink: /visual-style-comctl32-dll-ver6-group-view/
spaces_21ad9df954391fe1354efd1fc739c09a_permalink:
  - "http://cid-192788b236f6126b.users.api.live.net/Users(1812567674047566443)/Blogs('192788B236F6126B!102')/Entries('192788B236F6126B!173')?authkey=FlIl!wdwooA%24"
categories:
  - 
---
<div id="msgcns!192788B236F6126B!173" class="bvMsg">
<div>想在自己的程序里面使用  "show in groups " 的属性. 在 msdn 上找到了一篇文章.</div>
<div><a href="http://msdn.microsoft.com/library/default.asp?url=/library/en-us/shellcc/platform/commctls/userex/cookbook.asp">http://msdn.microsoft.com/library/default.asp?url=/library/en-us/shellcc/platform/commctls/userex/cookbook.asp</a></div>
<div> </div>
<div>其中提到需要使用 comctl32.dll ver6, 这个版本的 comctl 是随 xp 发布的, 并且,  "ComCtl32.dll version 6 is not redistributable " 这就意味着, 早一些的版本比如 win2000 上是不能使用的.</div>
<div> </div>
<div>的确是一个坏消息.</div>
<div> </div>
<div>文章还介绍了一些自绘制控件如何使用 visual style 的 API:</div>
<div> </div>
<div>InitCommonControls();</div>
<div>OpenThemeData();</div>
<div>DrawThemeBackground();</div>
<div>SetWindowTheme();</div>
<div>..</div>
<div>还有一段代码:</div>
<div>
<blockquote><pre style="display:block;"><hr />HTHEME hTheme = NULL;

hTheme = OpenThemeData(hwndButton, L "Button ");
...
DrawMyControl(hDC, hwndButton, hTheme, iState);
...
if (hTheme)
'
    CloseTheme(hTheme);
&#125;

void DrawMyControl(HDC hDC, HWND hwndButton, HTHEME hTheme, int iState)
'
    RECT rc, rcContent;
    TCHAR szButtonText[255];
    HRESULT hr;
    size_t cch;

    GetWindowRect(hwndButton, &rc);
    GetWindowText(hwndButton, szButtonText,
	              (sizeof(szButtonText)/sizeof(szButtonText[0])+1));
	hr = StringCchLength(szButtonText,
         (sizeof(szButtonText)/sizeof(szButtonText[0])), &cch);
    if (hTheme)
    '
        hr = DrawThemeBackground(hTheme, hDC, BP_BUTTON,
                iState, &rc, 0);
        <font color="blue">//</font><font color="green"> Always check your result codes.</font>

        hr = GetThemeBackgroundContentRect(hTheme, hDC,
                BP_BUTTON, iState, &rc, &rcContent);
        hr = DrawThemeText(hTheme, hDC, BP_BUTTON, iState,
                szButtonText, cch,
                DT_CENTER | DT_VCENTER | DT_SINGLELINE,
                0, &rcContent);
    &#125;
    else
    '
        <font color="blue">//</font><font color="green"> Draw the control without using visual styles.</font>
    &#125;
&#125;
</pre>
</blockquote>


使用抗锯齿的32bit图标.
调用 <a href="http://spaces.msn.com/library/en-us/shellcc/platform/commctls/imagelist/functions/imagelist_create.asp"><u><font color="#0000ff">ImageList_Create</font></u></a> (), 使用 ILC_COLOR32 标志.
调用 LoadImage(), 使用 LR_CREATEDIBSECTION 标志.
32bit 图标带有 alpha 通道. 可以用第三方工具来制作图标. 
一个图标文件中包含多个图标, 通常为 48&#215;48, 32&#215;32, 16&#215;16 象素.
他们需要按顺序排列. 
 
找到了这么些文档. 还没有找到如何在自己的程序中实现 Show in gruops.
 
另外, google 到了第三方的类, 据说可以实现.
NSElib, MegaPack 不过都是要钱的. 还没有找到破解.:(
 
</div>
