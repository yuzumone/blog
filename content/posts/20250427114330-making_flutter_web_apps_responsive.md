+++
title = "Making Flutter web apps responsive"
author = ["yuzumone"]
date = 2020-05-17
slug = "making-flutter-web-apps-responsive"
tags = ["Flutter"]
draft = false
toc = false
image = "https://images.unsplash.com/photo-1499951360447-b19be8fe80f5?q=80&w=2670&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
+++

{{< figure title="Photo by Domenico Loia on Unsplash" src="https://images.unsplash.com/photo-1499951360447-b19be8fe80f5?q=80&w=2670&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D" >}}

Flutter web でレスポンシブ対応したときのメモ． <br/>
responsive_builder を使うとめっちゃ簡単． <br/>

[responsive_builder | Flutter Package](https://pub.dev/packages/responsive_builder) <br/>

以下のようにデバイスに応じて Widget を返すようにするだけ． <br/>

```dart
ResponsiveBuilder(
  builder: (context, sizingInformation) {
    if (sizingInformation.deviceScreenType == DeviceScreenType.desktop) {
      return DesktopWidget();
    }
    if (sizingInformation.deviceScreenType == DeviceScreenType.tablet) {
      return TabletWidget();
    }
    if (sizingInformation.deviceScreenType == DeviceScreenType.mobile) {
      return MobileWidget();
    }
    return Container(color:Colors.purple);
  },
);
```

今回は横のマージンだけ変えたかったのでマージンだけ渡せるようにした． <br/>

```dart
class MainPage extends StatelessWidget {
  final double sideMergin;
  const MainPage({Key key, this.sideMergin = 160.0}) : super(key: key);
  ...
}
```

