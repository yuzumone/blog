+++
title = "Release tabler for displaying ASCII table format in Dart"
author = ["yuzumone"]
date = 2021-06-06
slug = "release-tabler-for-displaying-ascii-table-format-in-dart"
tags = ["Dart"]
draft = false
toc = false
image = "https://images.unsplash.com/photo-1530018607912-eff2daa1bac4?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwcm9maWxlLXBhZ2V8Mnx8fGVufDB8fHx8fA%3D%3D"
+++

{{< figure title="Photo by Abel Y Costa on Unsplash" src="https://images.unsplash.com/photo-1530018607912-eff2daa1bac4?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwcm9maWxlLXBhZ2V8Mnx8fGVufDB8fHx8fA%3D%3D" >}}

go-lang の tablewriter 的な ASCII テーブルで表示するやつが欲しかったの作りました． <br/>

[tabler | Dart Package](https://pub.dev/packages/tabler) <br/>

example のとおりですが，こんな感じで使えます． <br/>

```dart
void main() {
  var t = Tabler();
  t.add(['a', 'b', 'c']);
  t.add([1, 22, 333]);
  print(t);
  // +---+----+-----+
  // | a | b  | c   |
  // | 1 | 22 | 333 |
  // +---+----+-----+
}
```

Dart は dart2native あるし，null safety だし Flutter が無かったとしても結構好きなんですが，こんな基本的と言えるライブラリが無いとは． <br/>
それだけ Flutter でしか使われてないんだろうなぁ． <br/>

