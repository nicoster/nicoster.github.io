---
id: 33
title: set DEVMGR_SHOW_NONPRESENT_DEVICES=1
date: 2005-07-25T21:18:16+00:00
author: nick
layout: post
guid: http://nicoster.wordpress.com/2005/07/25/set-devmgr_show_nonpresent_devices1
permalink: /set-devmgr_show_nonpresent_devices1/
spaces_21ad9df954391fe1354efd1fc739c09a_permalink:
  - "http://cid-192788b236f6126b.users.api.live.net/Users(1812567674047566443)/Blogs('192788B236F6126B!102')/Entries('192788B236F6126B!145')?authkey=FlIl!wdwooA%24"
categories:
  - 计算机与 Internet
---
<div id="msgcns!192788B236F6126B!145" class="bvMsg">
<div>
由於windows莫名其妙的registry原因,很多用户发现,在安装蓝牙管理软件之後,对应的蓝牙虚拟端口居然变成COM9,甚至是COM13或更高,导致与许多应用程序,如手机同步软件,PDA同步软件无法使用 多次重新安装蓝牙管理软件只会让状况更恶化.. 
解决方案如下(执行下列步骤之前,请务必卸载蓝牙管理程序,并重新开机) 1.在Windows系统,按开始>执行>输入cmd,按回车 2.出现命令字符视窗之後,输入 set DEVMGR_SHOW_NONPRESENT_DEVICES=1 按回车 devmgmt.msc 按回车 3.然后在设备管理器点击>查看>显示隐藏的设备 您现在能能删除多余的端口了,删除完毕之後,请务必重新开机 
4.重新安装蓝牙管理软件 
5.如果您想永久性投入这个环境变量到XP, 到我的电脑>点选之後,按鼠标右键>属性>高级>环境变量 在系统变量里面,按 "新建 ",在 "变量名 "里面填入 " DEVMGR_SHOW_NONPRESENT_DEVICES " ,变量值填入 "1 "
</div>
</div>
