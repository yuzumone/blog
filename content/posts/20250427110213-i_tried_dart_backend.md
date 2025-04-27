+++
title = "I tried Dart backend"
author = ["yuzumone"]
date = 2021-02-27
slug = "i-tried-dart-backend"
tags = ["Dart"]
draft = false
toc = false
image = "https://images.unsplash.com/photo-1606300015207-d4e499ffca80?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwcm9maWxlLXBhZ2V8N3x8fGVufDB8fHx8fA%3D%3D"
+++

{{< figure title="Photo by Goran Ivos on Unsplash" src="https://images.unsplash.com/photo-1606300015207-d4e499ffca80?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwcm9maWxlLXBhZ2V8N3x8fGVufDB8fHx8fA%3D%3D" >}}

Flutter には盛り上がりがみられる Dart でバックエンドもやってみました． <br/>
この時気になるのがフレームワークどうするか問題ですね． <br/>
メジャーどころは下記の 2 つです． <br/>

[Aqueduct](https://github.com/stablekernel/aqueduct) <br/>

最終 commit 2020/09/01 <br/>

[Angel](https://github.com/angel-dart/angel) <br/>

最終 commit 2020/05/05 <br/>

とこんな感じで開発はどちらも止まっており，もはや選択肢に入らない状態です． <br/>
というわけで shelf と shelf_router で実装するのがよさそうな雰囲気です． <br/>

[shelf](https://pub.dev/packages/shelf) <br/>

[shelf_router](https://pub.dev/packages/shelf_router) <br/>

ひな形が stagehand で作れるのでそちらで． <br/>
ただし，shelf_router は初期だと含まれていないので pubspec.yaml に追加してください． <br/>

```bash
$ stagehand server-shelf
Creating server-shelf application `server_test`:
  /home/yuzumone/server_test/.gitignore
  /home/yuzumone/server_test/CHANGELOG.md
  /home/yuzumone/server_test/README.md
  /home/yuzumone/server_test/analysis_options.yaml
  /home/yuzumone/server_test/bin/server.dart
  /home/yuzumone/server_test/pubspec.yaml
  6 files written.
--> to provision required packages, run 'pub get'
--> run your app via `dart bin/server.dart`.
```

初期 server.dart は下記の通り． <br/>

```dart
import 'dart:io';import 'package:args/args.dart';
import 'package:shelf/shelf.dart' as shelf;
import 'package:shelf/shelf_io.dart' as io;// For Google Cloud Run, set _hostname to '0.0.0.0'.
const _hostname = 'localhost';void main(List<String> args) async {
  var parser = ArgParser()..addOption('port', abbr: 'p');
  var result = parser.parse(args);// For Google Cloud Run, we respect the PORT environment variable
  var portStr = result['port'] ?? Platform.environment['PORT'] ?? '8080';
  var port = int.tryParse(portStr);if (port == null) {
    stdout.writeln('Could not parse port value "$portStr" into a number.');
    // 64: command line usage error
    exitCode = 64;
    return;
  }var handler = const shelf.Pipeline()
      .addMiddleware(shelf.logRequests())
      .addHandler(_echoRequest);var server = await io.serve(handler, _hostname, port);
  print('Serving at http://${server.address.host}:${server.port}');
}shelf.Response _echoRequest(shelf.Request request) =>
    shelf.Response.ok('Request for "${request.url}"');
```

これを shelf_router を使っていい感じに GET と POST を生やしてみます． <br/>
下記の感じで handler のところを書き換えます． <br/>
shelf_router を使うと対応 method と endpoint が分かりやすくなります． <br/>
ちなみに shelf_router の README では app.handler を渡す感じで記載されていますが，deprecated になっており，下記のように app 自体を渡すのが正解です． <br/>

```dart
var app = Router();
app.get('/status', (shelf.Request request) {
  return shelf.Response.ok('ok');
});
app.post('/post', (shelf.Request request) async {
  final content = await request.readAsString();
  final data = json.decode(content);
  return shelf.Response.ok('Hello, ${data["text"]}');
});
var server = await io.serve(app, _hostname, port);
```

せっかく生やしたので叩いてみます． <br/>

```bash
$ curl http://localhost:8080/status
ok
$ curl -X POST -d '{"text": "world"}' http://localhost:8080/post
Hello, world
```

ちなみに shelf_router_generator という package もありこちらは Flask のようにデコレータで method や endpoint を指定できます． <br/>
こっちも良さそうです． <br/>

[shelf_router_generator | Dart Package](https://pub.dev/packages/shelf_router_generator) <br/>

あとこれは完全に備考ですが，shelf はもちろん dart2native に対応しています． <br/>
Aqueduct は対応していなかったので一応． <br/>

```bash
$ dart2native bin/server.dart -o test
Generated: /home/yuzumone/server_test/test
```


## Backlinks {#backlinks}

-   [I tried Dart frontend using shelf]({{< relref "20250427105056-i_tried_dart_frontend_using_shelf" >}}) <br/>

