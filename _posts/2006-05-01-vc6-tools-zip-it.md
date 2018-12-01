---
id: 164
title: 'vc6 Tools: &Zip it'
date: 2006-05-01T00:13:00+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=164
permalink: /vc6-tools-zip-it/
categories:
  - Uncategorized
---
cmd:
c:program fileswinrarwinrar.exe
param:
a -ag "YYYY-MM-DD HH.MM' " -r -o- -xdebug -xrelease -idp -m5 -ed -isnd -x*.rar -x*.exe -x*.ilk -x*.ncb -x*.scc -x*.obj -x*.pch -x*.res -x*.pdb  "..$(WkspName)  " 
