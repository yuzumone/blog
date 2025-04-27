+++
title = "Try launchd on macOS"
author = ["yuzumone"]
date = 2024-10-10
slug = "try-launchd-on-macos"
tags = ["MacOS"]
draft = false
toc = false
image = "https://images.unsplash.com/photo-1583361703300-bf0a4dc1723c?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8NHx8ZGljdGlvbmFyeXxlbnwwfHwwfHx8Mg%3D%3D"
+++

{{< figure title="Photo by Joshua Hoehne on Unsplash" src="https://images.unsplash.com/photo-1583361703300-bf0a4dc1723c?q=80&w=2671&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D" >}}

macOS でのデーモン系の管理は launchd というものを利用するみたいですね． <br/>
早速 yaskkserv2 を launchd で管理してみました． <br/>

以下に記載の通りですが，ユーザ管理ものは `~/Library/LaunchAgents` に配置するようなのでそこに置きます． <br/>

[Script management with launched in Terminal on Mac](https://support.apple.com/guide/terminal/script-management-with-launchd-apdc6c1077b-5d5d-4d35-9c19-60f2397b2369/mac) <br/>

OS 起動時に立ち上がってほしいので RunAtLoad で起動するようにしました． <br/>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>net.yuzumone.yaskkserv</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/bin/yaskkserv2</string>
        <string>--no-daemonize</string>
        <string>--listen-address=127.0.0.1</string>
        <string>--google-japanese-input=notfound</string>
        <string>/usr/share/skk/dictionary.yaskkserv2</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
</dict>
</plist>
```

あとは有効化します． <br/>

```bash
$ launchctl load yaskkserv.plist
$ launchctl list | rg yuzumone
58020   0       net.yuzumone.yaskkserv
```

systemd.timer のように定期実行もできそうなので，crond よりもこっちを利用したほうが良さそうです． <br/>

