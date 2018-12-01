---
id: 13
title: PrintScreen 2.41 released.
date: 2005-09-26T23:02:49+00:00
author: nick
layout: post
guid: http://nicoster.wordpress.com/2005/09/26/printscreen-2-41-released
permalink: /printscreen-2-41-released/
spaces_21ad9df954391fe1354efd1fc739c09a_permalink:
  - "http://cid-192788b236f6126b.users.api.live.net/Users(1812567674047566443)/Blogs('192788B236F6126B!102')/Entries('192788B236F6126B!184')?authkey=FlIl!wdwooA%24"
categories:
  - 计算机与 Internet
---
<div id="msgcns!192788B236F6126B!184" class="bvMsg">
<blockquote>
<div><img src="http://storage.msn.com/x1pbglk-vqL4Bv8-1BOciVATn-ZnHjwHyV5kKYjE-hpnG_hyA1gk1deHi5WluGJcYG4pLjZWK-P9mVSKd8WzEHVKEev_2ofaBtEJDelc4HyBQF4SFHPnNEAxrwCMYiqrS_qyzJ8z0mAJdayW0Br72TZ0A" border="0" /> <img src="http://storage.msn.com/x1pbglk-vqL4Bv8-1BOciVATn-ZnHjwHyV5kKYjE-hpnG-Ktv_1PaZczBLL625enIFojqmDvI5yA3kLRHKbiyBSvj3XVTJic-vIi6gj-YOPcJ-vVyyek5LR0OivPicQob_2LGD0dGG2Srv-hcT5y9l8FQ" /> <img src="http://storage.msn.com/x1pbglk-vqL4Bv8-1BOciVATn-ZnHjwHyV5kKYjE-hpnG_4jZulrsSe07-gHqqROE2u1YPkClaKqqzW9qBH9lhTcVdvGvY0gwtrXq8yU0nxHJ7PslxCinWJG54qju2E_kGtJd3uXSP0qyptJSArDfpvbA" /> <img src="ftp://222.90.231.60/pic/printscreen1.png" /> </div>
<div>/* Nico 05-9-5 1405  使用 WTL 的第一个程序, 体会了 mix-in class 的优越性, 并写了一个 CTrayIcon 的 mix-in class. WTL 的消息处理灵活性. </div>
<div> 感受到 MFC 的单继承是多么大的一个限制. */</div>
<div>/* Nico 05-9-5 168  大家是否还记得论坛里面提到过的一个程序 capture? 它是一个抓图程序, 可以将抓图放到 c:capture 目录. 但是他的功能比较弱. 只能抓全屏, 且只能存为 jpg 格式.</div>
<div> PrintScreen.exe 就是为了弥补这两个缺点而写的.  PrintScreen: . 支持 bmp, jpg, png. (gif 现在还不支持). . 支持 alt + PrntScrn, 抓取当前窗口. . 支持抓图后自动打开图片查看 &#8230;</div>
<div> */</div>
<div> </div>
<div>/* Nico 05-9-19 1117 v2.1 released. .BUGFIX: DrawTipWindow() 里有严重的 GDI 泄漏. 导致程序运行一段时间 GDI 耗尽.  .ADD: 可以用 ctrl + ↑, Ctrl + ↓ 改变放大倍数. (Ctrl + mouseroll 也可以改变放大倍数). */</div>
<div>/* Nico 05-9-19 1444 v2.2 released. .IMPROVED: 在配置比较低的机器上测试发现, tipwindow 的边框部分有些闪烁. 现在把整个   tipwindow 都用 memdc 缓冲, 然后再显示出来. 解决了闪烁的问题. */</div>
<div>/* Nico 05-9-19 184 v2.3 released. */ .IMPROVED: 支持命令行参数:  /ONCE 程序运行即进入区域选定模式. 用户可以选定区域, 放入剪贴板, 或者存为文件,   之后, 程序即退出. 这种方式的优点是不常驻内存. 只是在需要的时候才运行程序.   使用这个命令行参数, 可以使用第三方热键管理工具, 或者只是在快速启动里面建立   一个快捷方式来运行本程序.  /SETACCESSIBM 为了支持 /ONCE 命令行参数, 对于从来不使用 Access IBM 键的黑友,    可以把 Access IBM 键作为 PrintScreen 的快捷键. 带这个参数执行一次程序, 即可   设置 Access IBM 为 PrintScreen 的启动热键.  /RESTOREACCESSIBM 恢复 Access IBM 键的以前的设置.  */</div>
<div>/* Nico 05-9-20 949  .BUGFIX: 修正了放大非整数倍时放大图形的畸变问题. 现在确保了整数倍. 放大的效果  和 windows 自带的放大镜一致. */</div>
<div>/* Nico 05-9-20 181 v2.4 released. .ADD: 在选中的区域中, 添加右键菜单.  .ADD: 新增  "Make this area always on top " 的功能.  */</div>
<div>/* Nico 05-9-21 1524  .BUGFIX: 去除了 imageWnd 多余的窗口菜单项 */</div>
<div>/* Nico 05-9-23 1044  .ADD 给 imagewnd 添加了一个 trackbar (slider), 用来调节窗口的透明度. just for fun:) */</div>
<div>/* Nico 05-9-23 1350 v2.41 released. .ADD tipwindow 同样实现了透明处理. 用鼠标滚轮或者 ↑,↓ 来调节透明度. */</div>
</blockquote>
<div> </div>
<div>下载: <a href="ftp://222.90.231.60/setup/ps2.41.rar">ftp://222.90.231.60/setup/ps2.41.rar</a></div>
<div>讨论: <a href="http://www.51nb.com/forum/viewthread.php?tid=280843">http://www.51nb.com/forum/viewthread.php?tid=280843</a></div>
</div>
