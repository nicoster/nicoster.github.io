---
id: 15
title: Send To Mail Recipient option does nothing
date: 2005-09-21T01:47:15+00:00
author: nick
layout: post
guid: http://nicoster.wordpress.com/2005/09/21/send-to-mail-recipient-option-does-nothing
permalink: /send-to-mail-recipient-option-does-nothing/
spaces_21ad9df954391fe1354efd1fc739c09a_permalink:
  - "http://cid-192788b236f6126b.users.api.live.net/Users(1812567674047566443)/Blogs('192788B236F6126B!102')/Entries('192788B236F6126B!174')?authkey=FlIl!wdwooA%24"
categories:
  - 
---
<div id="msgcns!192788B236F6126B!174" class="bvMsg">
<div>
<hr />
When you right-click a file and choose  "Mail Recipient " in the SendTo option, the email window does not open / nothing happens. There are two known causes for this:
<ul>
<li>
The default mail client is not set
<li>
The DLLPath in the registry is missing for your mail client
<li>
Missing or invalid Registry keys related to .MAPIMAIL
</li>
</ul>
First, set the default mail client in the Internet Options, Programs tab. For Outlook Express, type these commands in the RUN box. (The DLLPath will be recreated):
MSIMN.EXE /REGregsvr32  "%ProgramFiles%Outlook Expressmsoe.dll "
For Case 3: Apply this registry fix. Copy the contents (from Windows Registry Editor&#8230;&#8230;. line) and save the contents to a file named  "mapi.reg " using Notepad. Use quotes while saving the file, to avoid double-extensions. Leave an blank line at the end of the file.
<blockquote>
<p style="text-align:left;"><font face="Tahoma" size="1">&#8212;&#8212;&#8212;Cut&#8212;&#8212;&#8212;</font>
Windows Registry Editor Version 5.00
[HKEY_CLASSES_ROOT.MAPIMail]
@= "CLSID\'9E56BE60-C50F-11CF-9A2C-00A0C90A90CE&#125; "
 
[-HKEY_CURRENT_USERSoftwareMicrosoftWindowsCurrentVersionExplorerFileExts.MAPIMail]
 
<p style="text-align:left;"><font face="Tahoma" size="1">&#8212;&#8212;&#8212;Cut&#8212;&#8212;&#8212;</font>
</blockquote>
If this does not help, type <b>REGSVR32 SENDMAIL</b> in the RUN box and press OK.
</div>
</div>
