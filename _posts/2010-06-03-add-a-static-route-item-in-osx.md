---
id: 104
title: Add a static route item in OSX
date: 2010-06-03T14:10:00+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=104
permalink: /add-a-static-route-item-in-osx/
tags:
  - osx
  - route
  - startup item
---
<blockquote>nickx:~ $ cd /Library/StartupItems/
nickx:/Library/StartupItems $ ls
AddRoutes
nickx:/Library/StartupItems $ cd AddRoutes/
nickx:/Library/StartupItems/AddRoutes $ ls
AddRoutes
StartupParameters.plist
nickx:/Library/StartupItems/AddRoutes $ cat AddRoutes
#!/bin/sh
# Set up static routing tables
# Roark Holz, Thursday, April 6, 2006
. /etc/rc.common
StartService ()
{
ConsoleMessage 'Adding Static Routing Tables'
sudo route add 10.224.0.0/16 10.224.104.129
}
StopService ()
{return 0}
RestartService ()
{return 0}
RunService '$1&#8243;
nickx:/Library/StartupItems/AddRoutes $ cat StartupParameters.plist
{
Description = 'Add static routing tables';
Provides = ('AddRoutes');
Requires = ('Network');
OrderPreference = 'None';
}
nickx:/Library/StartupItems/AddRoutes $ cd ..
nickx:/Library/StartupItems $ chown -R root:wheel AddRoutes/
nickx:/Library/StartupItems $ cd AddRoutes/
nickx:/Library/StartupItems/AddRoutes $ chmod 755 *
nickx:/Library/StartupItems/AddRoutes $ ls -l
total 16
-rwxr-xr-x@ 1 root  wheel  313 Jun  3 10:26 AddRoutes
-rwxr-xr-x@ 1 root  wheel  123 Jun  3 10:26 StartupParameters.plist</blockquote>
 
