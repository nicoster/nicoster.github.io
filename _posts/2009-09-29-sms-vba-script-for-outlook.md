---
id: 111
title: 邮件短信提醒 vba script for outlook
date: 2009-09-29T13:16:00+00:00
author: nick
layout: post
guid: http://nick.luckygarden.org/?p=111
permalink: /sms-vba-script-for-outlook/
tags:
  - notification
  - outlook
  - sms
  - vba
---
使用 sms.api.bz 这个短信网关. 要求先开通飞信. 填好 URL 中的各个参数.
在 outlook 中, 用 alt+f11 打开 vba 编辑器. 保持好如下脚本.
	
	Sub SendSMS(Item As Outlook.MailItem)
	    Dim msg
	    Dim ret
	    msg = URLEncode(Item.Subject)
	    ret = GetDataFromURL("http://sms.api.bz/fetion.php?username=&#038;password=&#038;sendto=&#038;message=" &#038; msg, "GET", "")
	End Sub
	Public Function URLEncode(strURL)
	Dim I
	Dim tempStr
	For I = 1 To Len(strURL)
	    If Asc(Mid(strURL, I, 1)) < 0 Then
	       tempStr = "%" &#038; Right(CStr(Hex(Asc(Mid(strURL, I, 1)))), 2)
	       tempStr = "%" &#038; Left(CStr(Hex(Asc(Mid(strURL, I, 1)))), Len(CStr(Hex(Asc(Mid(strURL, I, 1))))) - 2) &#038; tempStr
	       URLEncode = URLEncode &#038; tempStr
	    ElseIf (Asc(Mid(strURL, I, 1)) >= 65 And Asc(Mid(strURL, I, 1)) <= 90) Or (Asc(Mid(strURL, I, 1)) >= 97 And Asc(Mid(strURL, I, 1)) <= 122) Then
	       URLEncode = URLEncode &#038; Mid(strURL, I, 1)
	    Else
	       URLEncode = URLEncode &#038; "%" &#038; Hex(Asc(Mid(strURL, I, 1)))
	    End If
	Next
	End Function
	Function GetDataFromURL(strURL, strMethod, strPostData)
	  Dim lngTimeout
	  Dim strUserAgentString
	  Dim intSslErrorIgnoreFlags
	  Dim blnEnableRedirects
	  Dim blnEnableHttpsToHttpRedirects
	  Dim strHostOverride
	  Dim strLogin
	  Dim strPassword
	  Dim strResponseText
	  Dim objWinHttp
	  lngTimeout = 59000
	  strUserAgentString = "http_requester/0.1"
	  intSslErrorIgnoreFlags = 13056 ' 13056: ignore all err, 0: accept no err
	  blnEnableRedirects = True
	  blnEnableHttpsToHttpRedirects = True
	  strHostOverride = ""
	  strLogin = ""
	  strPassword = ""
	  Set objWinHttp = CreateObject("WinHttp.WinHttpRequest.5.1")
	  objWinHttp.SetTimeouts lngTimeout, lngTimeout, lngTimeout, lngTimeout
	  objWinHttp.Open strMethod, strURL
	  If strMethod = "POST" Then
	    objWinHttp.SetRequestHeader "Content-type", _
	      "application/x-www-form-urlencoded"
	  End If
	  If strHostOverride <> "" Then
	    objWinHttp.SetRequestHeader "Host", strHostOverride
	  End If
	  objWinHttp.Option(0) = strUserAgentString
	  objWinHttp.Option(4) = intSslErrorIgnoreFlags
	  objWinHttp.Option(6) = blnEnableRedirects
	  objWinHttp.Option(12) = blnEnableHttpsToHttpRedirects
	  If (strLogin <> "") And (strPassword <> "") Then
	    objWinHttp.SetCredentials strLogin, strPassword, 0
	  End If
	  On Error Resume Next
	  objWinHttp.Send (strPostData)
	  If Err.Number = 0 Then
	    If objWinHttp.Status = "200" Then
	      GetDataFromURL = objWinHttp.ResponseText
	    Else
	      GetDataFromURL = "HTTP " &#038; objWinHttp.Status &#038; " " &#038; _
	        objWinHttp.StatusText
	    End If
	  Else
	    GetDataFromURL = "Error " &#038; Err.Number &#038; " " &#038; Err.Source &#038; " " &#038; _
	      Err.Description
	  End If
	  On Error GoTo 0
	  Set objWinHttp = Nothing
	End Function

然后创建一条规则, 当特定邮件到达时执行这个脚本.
由于安全性的问题, 还要给这个[脚本签名](nick.luckygarden.org/?p=370)
