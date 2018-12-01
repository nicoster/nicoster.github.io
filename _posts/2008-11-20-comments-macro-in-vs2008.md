---
id: 120
title: comments macro in vs2008
date: 2008-11-20T14:39:00+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=120
permalink: /comments-macro-in-vs2008/
tags:
  - dbg
  - comments
  - macro
  - script
  - vs2008
---
    Sub AddCommentDateTime()        &#8216;DESCRIPTION: Add current date&&time as comment        
	    Dim nHour = DateAndTime.Now.Hour        
	    Dim strMinute = DateAndTime.Now.Minute
		If nHour > 9 And nHour < 16 And len(strMinute) = 1 Then            
			strMinute = '0&#8243; + strMinute        
		End If        
		Dim str As String        
		str = '/* nick ' & Right(CStr(Year(Now)), 2) & '-' & CStr(Month(Now)) & '-' & CStr(Day(Now)) & ' ' _         & CStr(nHour) & ':' & strMinute & '  */'
		Dim sel = CType(DTE.ActiveDocument.Selection(), EnvDTE.TextSelection)        
		sel.Text = str        
		sel.WordLeft(False, 1)        
		sel.CharLeft(False, 1)
	End Sub
