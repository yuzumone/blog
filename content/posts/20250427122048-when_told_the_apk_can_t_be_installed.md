+++
title = "When told the APK can't be installed"
author = ["yuzumone"]
date = 2018-03-30
slug = "when-told-the-apk-cant-be-installed"
tags = ["Android"]
draft = false
toc = false
image = "https://images.unsplash.com/photo-1630442923896-244dd3717b35?q=80&w=2670&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
+++

{{< figure title="Photo by Fahim Muntashir on Unsplash" src="https://images.unsplash.com/photo-1630442923896-244dd3717b35?q=80&w=2670&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D" >}}

最近 issue がとんできてて，そのときになんじゃこれ～と思ったのでメモ． <br/>
きた issue は以下． <br/>

[Parsing error · Issue #9 · yuzumone/Recordachi](https://github.com/yuzumone/Recordachi/issues/9) <br/>


## 解決策 {#解決策}

APK を作成する際，Signature Versions を V1 と V2 にチェックを入れる． <br/>


## Signature Versions とは {#signature-versions-とは}

Android Studio2.3 から選択するようになった． <br/>
Android7.0 から新しい署名スキームである V2 が導入され，以前のスキームは V1 となった． <br/>
これは何となく知ってたんだけど，Android7.0 未満の端末では V2 でしか署名がないとインストールに失敗するのは知らなかった． <br/>


## 参考 {#参考}

['App not Installed' Error on Android](https://stackoverflow.com/questions/4226132/app-not-installed-error-on-android) <br/>

