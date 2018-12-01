---
id: 110
title: UniqueList excel
date: 2009-10-22T14:41:00+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=110
permalink: /uniquelist-excel/
tags:
  - excel
  - script
  - unique
  - vba
---
对 excel 表中的列作 unique 操作. 要求先排好序.
	
	Sub UniqueList()    
	On Error Resume Next    
		For col = 65 To 65 + 23        
			Range(Chr(col) & “1″, Range(Chr(col) & “65536″).End(xlUp)).AdvancedFilter 
			Action:=xlFilterCopy, _        
			Unique:=True, _        
			CopyToRange:=Range(“A” & Chr(col) & “1″, “A” & Chr(col) & “1″)    
		Next col
	End Sub