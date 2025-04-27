+++
title = "Get RedirectUri on PowerShell"
author = ["yuzumone"]
date = 2023-11-03
slug = "get-redirecturi-on-powershell"
tags = ["Powershell"]
draft = false
toc = false
image = "https://images.unsplash.com/photo-1545262810-77515befe149?q=80&w=2670&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
+++

{{< figure title="Photo by Marc Rentschler on Unsplash" src="https://images.unsplash.com/photo-1545262810-77515befe149?q=80&w=2670&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D" >}}

PowerShell で curl -w “%{redirect_url}” 相当がしたくなったのでメモ． <br/>

```powershell
Param(
    [String]$Uri
)
$request = Invoke-WebRequest -Method Head -Uri $Uri
$redirectedUri = $request.BaseResponse.ResponseUri.AbsoluteUri
echo $redirectedUri
```

