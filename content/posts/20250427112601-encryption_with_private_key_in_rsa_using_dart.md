+++
title = "Encryption with private key in RSA using Dart"
author = ["yuzumone"]
date = 2020-07-13
slug = "encryption-with-private-key-in-rsa-using-dart"
tags = ["Flutter"]
draft = false
toc = false
image = "https://images.unsplash.com/photo-1522794338816-ee3a17a00ae8?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8MTR8fGtleXxlbnwwfHwwfHx8Mg%3D%3D"
+++

{{< figure title="Photo by Debby Hudson on Unsplash" src="https://images.unsplash.com/photo-1522794338816-ee3a17a00ae8?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8MTR8fGtleXxlbnwwfHwwfHx8Mg%3D%3D" >}}

どうしても RSA Private key を使用して署名をしなきゃいけない案件があったのでやりました． <br/>

使用したライブラリは下記です． <br/>
死ぬほど助かりました． <br/>

[encrypt | Dart Package](https://pub.dev/packages/encrypt) <br/>

このライブラリを選択したのは parseKeyFromFile が用意されていたからです． <br/>
ほかにも似たライブラリはあるんですが，KeyPair をジェネレートまたは JWT から生成しかなさそうでした． <br/>
今回は環境変数に base64 エンコードした key をセットして再署名していきます． <br/>

```dart
Future<T> parseKeyFromFile<T extends RSAAsymmetricKey>(String filename) async {
   final file = File(filename);
   final key = await file.readAsString();
   final parser = RSAKeyParser();
   return parser.parse(key) as T;
}
```

というわけでコード． <br/>

```dart
import 'package:encrypt/encrypt.dart';var key = bool.hasEnvironment('PRIVATE_KEY')
    ? String.Environment('PRIVATE_KEY')
    : '';def sign() {
  var parser = RSAKeyParser();
  var privateKey = parser.parse(utf8.decode(base64.decode(key)));
  var signer =
      Signer(RSASigner(RSASignDigest.SHA256, privateKey: privateKey));
  var content = 'This is content.'
  return signer.sign(content).base64;
}
```

Flutter web で問題なく動いたので Android ・ iOS も動きそうな気がします． <br/>

