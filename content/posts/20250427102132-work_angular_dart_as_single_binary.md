+++
title = "Work Angular Dart as single binary"
author = ["yuzumone"]
date = 2022-03-27
slug = "work-angular-dart-as-single-binary"
tags = ["Dart"]
draft = false
toc = false
image = "https://images.unsplash.com/photo-1489875347897-49f64b51c1f8?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwcm9maWxlLXBhZ2V8MzZ8fHxlbnwwfHx8fHw%3D"
+++

{{< figure title="Photo by Caspar Camille Rubin on Unsplash" src="https://images.unsplash.com/photo-1489875347897-49f64b51c1f8?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwcm9maWxlLXBhZ2V8MzZ8fHxlbnwwfHx8fHw%3D" >}}

テンプレートをさくっと作成． <br/>

```bash
$ dart create -t server-shelf foo
$ cd foo
$ ngdart create web
```

とりあえず shelf を動かしてみるとこんな感じ． <br/>

```bash
$ dart bin/server.dart
Server listening on port 8080
2022-03-27T09:27:03.905499  0:00:00.014259 GET     [200] /
2022-03-27T09:27:09.375476  0:00:00.005037 GET     [200] /echo/foo---
$ curl http://localhost:8080/
Hello, World!
$ curl http://localhost:8080/echo/foo
foo
```

次に Angular Dart の方を build して動かしてみます． <br/>

```bash
$ cd web
$ webdev build
$ cd build
$ python -m http.server 8080
Serving HTTP on 0.0.0.0 port 8080 (http://0.0.0.0:8080/) ...
127.0.0.1 - - [27/Mar/2022 09:34:48] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [27/Mar/2022 09:34:49] "GET /styles.css HTTP/1.1" 200 -
127.0.0.1 - - [27/Mar/2022 09:34:49] "GET /main.dart.js HTTP/1.1" 200 -
127.0.0.1 - - [27/Mar/2022 09:34:49] "GET /favicon.png HTTP/1.1" 200 -
```

とりあえず下記の 4 ファイルをバイナリに含めるようにして，shelf でいい感じに返せば動きそうですね． <br/>

-   index.html <br/>
-   favicon.png <br/>
-   main.dart.js <br/>
-   styles.css <br/>

良さげなやつがあるのでこれを利用します． <br/>

[aspen | Dart Package](https://pub.dev/packages/aspen) <br/>

ただし更新がされておらず色々修正する必要がありそうだったので git clone して path で追加します． <br/>

修正点は下記で dependencies の upgrade や AssetId.resolve の引数が String から Uri に変更されている部分などです． <br/>

```bash
$ git diff
diff --git a/aspen_builder/lib/src/bundle_generator.dart b/aspen_builder/lib/src/bundle_generator.dart
index 0bbcf48..ab55884 100644
--- a/aspen_builder/lib/src/bundle_generator.dart
+++ b/aspen_builder/lib/src/bundle_generator.dart
@@ -101,8 +101,9 @@ class BundleGenerator extends GeneratorForAnnotation<Asset> {
       assetPathReader = annotation.read('path');
     }-    var assetId =
-        AssetId.resolve(assetPathReader.stringValue, from: buildStep.inputId);
+    var assetId = AssetId.resolve(Uri.parse(assetPathReader.stringValue),
+        from: buildStep.inputId);
+
     if (!await buildStep.canRead(assetId)) {
       error(element, 'Asset ${assetId} cannot be found');
       return Future.value();
diff --git a/aspen_builder/pubspec.yaml b/aspen_builder/pubspec.yaml
index 8c90058..70c1732 100644
--- a/aspen_builder/pubspec.yaml
+++ b/aspen_builder/pubspec.yaml
@@ -7,11 +7,11 @@ environment:
   sdk: '>=2.8.0 <3.0.0'dependencies:
-  analyzer: '>=0.37.0 <0.41.0'
+  analyzer: ^3.4.1
   aspen: ^0.3.0
-  build: ^1.2.0
-  csslib: ^0.16.1
-  source_gen: ^0.9.4+3
+  build: ^2.2.1
+  csslib: ^0.17.1
+  source_gen: ^1.2.1
   z85: ^0.1.0
```

また build_runner する際そのままだと Angular Dart の方も見に行ってしまうので build.yaml で lib/assets.dart だけ runner が動くようにします． <br/>

```yaml
targets:
  $default:
    builders:
      aspen_builder|bundle_builder:
        enabled: true
        generate_for:
          include:
            - lib/assets.dart
```

そして肝心の assets.dart の中身はこんな感じ．BinaryAsset はエンコード（z85）周りですんなり動かなさそうだったので諦めました． <br/>

```dart
import 'package:aspen/aspen.dart';
import 'package:aspen_assets/aspen_assets.dart';part 'assets.g.dart';@Asset('asset:foo/web/build/index.html')
const index = TextAsset(text: _index$content);@Asset('asset:foo/web/build/main.dart.js')
const mainDart = TextAsset(text: _mainDart$content);@Asset('asset:foo/web/build/styles.css')
const stylesCss = TextAsset(text: _stylesCss$content);// @Asset('asset:foo/web/build/favicon.png')
// const favicon = BinaryAsset(encoded: _favicon$content);
```

server.dart の router 周りを下記のように Assets 化したそれぞれを読み取ってそれぞれに合わせた content-type で返すようにします． <br/>

```dart
final _router = Router()
  ..get('/', _rootHandler)
  ..get('/main.dart.js', _jsHander)
  ..get('/styles.css', _cssHander);Response _rootHandler(Request req) {
  final index = assets.index;
  final headers = {'content-type': 'text/html'};
  return Response.ok(index.text, headers: headers);
}Response _jsHander(Request req) {
  final js = assets.mainDart;
  final headers = {'content-type': 'text/javascript'};
  return Response.ok(js.text, headers: headers);
}Response _cssHander(Request req) {
  final css = assets.stylesCss;
  final headers = {'content-type': 'text/css'};
  return Response.ok(css.text, headers: headers);
}
```

残念ながら no-sound-null-safety じゃないと動きません． <br/>

```bash
$ dart --no-sound-null-safety  run bin/server.dart
```

コンパイルも no-sound-null-safety で． <br/>

```bash
$ dart compile exe bin/server.dart --no-sound-null-safety -o server
Info: Compiling without sound null safety
Generated: /home/yuzumone/foo/server
```

これで Angular Dart の成果物を（とりあえず） single binary で動かせます． <br/>

```bash
$ ./server
Server listening on port 8080
2022-03-27T11:37:30.338261  0:00:00.000278 GET     [200] /
2022-03-27T11:37:30.484749  0:00:00.000083 GET     [200] /styles.css
2022-03-27T11:37:30.487764  0:00:00.006299 GET     [200] /main.dart.js
2022-03-27T11:37:30.594303  0:00:00.000041 GET     [404] /main.dart.js.map
2022-03-27T11:37:30.730615  0:00:00.000022 GET     [404] /favicon.png
```

更新されていないのがしんどすぎるのでこれをパクって sound-null-safety で動くようにしたい．あとは shelf_static あたりをパクって String の内容を渡しつつ，content-type などなどを自力で指定しなくても動いてくれると完璧． <br/>

