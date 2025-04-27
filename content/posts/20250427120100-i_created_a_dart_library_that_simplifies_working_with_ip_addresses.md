+++
title = "I created a Dart library that simplifies working with IP addresses"
author = ["yuzumone"]
date = 2020-03-14
slug = "i-created-a-dart-library-that-simplifies-working-with-ip-addresses"
tags = ["Dart"]
draft = false
toc = false
image = "https://images.unsplash.com/photo-1520085601670-ee14aa5fa3e8?q=80&w=2670&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
+++

{{< figure title="Photo by Alexandru Acea on Unsplash" src="https://images.unsplash.com/photo-1520085601670-ee14aa5fa3e8?q=80&w=2670&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D" >}}

Dart で ipaddress をいい感じにするライブラリを作りました． <br/>

[yuzumone/ipaddr](https://github.com/yuzumone/ipaddr) <br/>

[ipaddr | Dart Package](https://pub.dev/packages/ipaddr) <br/>


## モチベーション {#モチベーション}

生活していると「あーこのアドレスがこのセグメント内かどうか調べたいわー」となることが多いと思います． <br/>
これまでは Python でやることが多かったのですが，Dart でやりたい欲がふつふつと湧いてきました． <br/>
軽く見た感じ Python の ipaddress 相当のものがなさそうだったので作りました． <br/>


## 使い方 {#使い方}

Readme の通りですが． <br/>

```dart
import 'package:ipaddr/ipaddr.dart' as ipaddr;

main() {
  var address = ipaddr.IPv4Address('192.168.10.10');
  var network = ipaddr.IPv4Network('192.168.10.0/24');
  if (network.addresses.contains(address)) {
    print('${address} is included ${network}');
    // 192.168.10.10 is included 192.168.10.0/24
  }
}
```


## ライブラリを公開してみて {#ライブラリを公開してみて}

Maven とかと比べるとめっちゃ楽です． <br/>
やることは以下のコマンドだけ． <br/>

```bash
pub publish --dry-run
pub publish
```

注意点としては dry-run はパッケージ名の重複は見てくれないというところです． <br/>
成功したからいける～と思ったらできなくてひどい目にあいました． <br/>
一応 pub.dev で検索してからいけそうだったのでそのライブラリ名にしたんですが，管理されていないのは名前が完全一致していてもトップに表示されないみたいです． <br/>
ぴえん． <br/>

