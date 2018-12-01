---
id: 26
title: Checked build version of Windows
date: 2005-08-10T07:02:02+00:00
author: nick
layout: post
guid: http://nicoster.wordpress.com/2005/08/10/checked-build-version-of-windows
permalink: /checked-build-version-of-windows/
spaces_21ad9df954391fe1354efd1fc739c09a_permalink:
  - "http://cid-192788b236f6126b.users.api.live.net/Users(1812567674047566443)/Blogs('192788B236F6126B!102')/Entries('192788B236F6126B!156')?authkey=FlIl!wdwooA%24"
categories:
  - 计算机与 Internet
---
<div id="msgcns!192788B236F6126B!156" class="bvMsg">
<div></div>
You don't have to install the entire checked build to take advantage of the debug version of the operating system. You can just copy the checked version of the kernel image (Ntoskrnl.exe) and the appropriate HAL (Hal.dll) to a normal retail installation. The advantage of this approach is that device drivers and other kernel code get the rigorous checking of the checked build without having to run the slower debug versions of all components in the system. For detailed instructions on how to do this, see the section  "Installing Just the Checked Operating System and HAL " in the Windows DDK documentation. Because Microsoft doesn't supply a checked build version of Windows 2000 Server, you can also apply this technique to run the checked version of the kernel on a Windows 2000 Server system.</div>
