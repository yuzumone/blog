+++
title = "Created an Android library for multi-line ellipsizing"
author = ["yuzumone"]
date = 2017-06-07
slug = "created-an-anroid-library-for-multi-line-ellipsizing"
tags = ["Android"]
draft = false
toc = false
image = "https://images.unsplash.com/photo-1709547228697-fa1f424a3f39?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8NTF8fHByb2dyYW1pbmd8ZW58MHx8MHx8fDI%3D"
+++

{{< figure title="Photo by Volodymyr Dobrovolskyy on Unsplash" src="https://images.unsplash.com/photo-1709547228697-fa1f424a3f39?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8NTF8fHByb2dyYW1pbmd8ZW58MHx8MHx8fDI%3D" >}}

Android の TextView で複数行 Ellipsize してタップすると全文表示するようなライブラリを作ってみました． <br/>

[yuzumone/ExpandableTextView](https://github.com/yuzumone/ExpandableTextView) <br/>

動作は下の感じです． <br/>

{{< x user="yuzu_037" id="872263123054452736" >}}

もともとすでに実装はしていて，今も Twltrus の Reply 送信画面では上の動作のような感じにはなっていました． <br/>
Twltrus の実装では Fragment で boolean を持っていて TextView の ClickListener のところで Ellipsize する・しないを切り替えていました． <br/>
CustomView にしたほうが TextView の状態を Activity や Fragment で気にしなくてよくなるよなというところです． <br/>
あとは Android が正式に Kotlin をサポートということで Kotlin でライブラリを作ってみました． <br/>

