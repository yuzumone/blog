+++
title = "That’s how I create flutter launcher icon"
author = ["yuzumone"]
date = 2020-12-28
slug = "thats-how-i-create-flutter-launcher-icon"
tags = ["Flutter"]
draft = false
toc = false
image = "https://images.unsplash.com/photo-1637607699029-8568cd757a5a?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8M3x8bWFjJTIwZG9ja3xlbnwwfHwwfHx8Mg%3D%3D"
+++

{{< figure title="Photo by TheRegisti on Unsplash" src="https://images.unsplash.com/photo-1637607699029-8568cd757a5a?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8M3x8bWFjJTIwZG9ja3xlbnwwfHwwfHx8Mg%3D%3D" >}}

ランチャーアイコンを作ってくれる flutter_launcher_icons はとても便利です． <br/>
Android の Adaptive Icon を SVG でやりたい場合は不十分です（雑に見てみた感じ png しか指定できないようでした）． <br/>

ということでこんな感じでやると良さそうというのをメモ． <br/>

[flutter_launcher_icons](https://pub.dev/packages/flutter_launcher_icons) <br/>

Adaptive Icon なので普通に ic_launcher_foreground.svg 的なものを用意します． <br/>
用意したら AndroidStuio の Image Asset Studio を起動します． <br/>
詳細は下記を． <br/>

[Create app icons with Image Asset Studio | Android Developers](https://developer.android.com/studio/write/image-asset-studio) <br/>

使用する svg の指定と Scaling あたりを指定したら脳死で Next を押して Finish します． <br/>
これで Android 側はアイコンが良い感じに設定されます． <br/>

これで Android 側は OK ですが iOS 側が残っています． <br/>
iOS 側は flutter_launcher_icons で設定します． <br/>
Image Asset Studio ではアイコンのほかにも Play Store 用に大きいサイズの画像を吐き出してくれているので，それを有効活用します． <br/>

```yaml
flutter_icons:
  android: false
  ios: true
  image_path: "android/app/src/main/ic_launcher-web.png"
```

あとは flutter_launcher_icons を実施します． <br/>

```bash
flutter pub get
flutter pub run flutter_launcher_icons:main
```

iOS 側の画像サイズの知見が全くないのであれですが，シミュレータで見た感じぼやけているような感じは無かったので大丈夫そう． <br/>

