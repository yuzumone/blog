+++
title = "I tried Dart frontend using shelf"
author = ["yuzumone"]
date = 2021-05-15
slug = "i-tried-dart-frontend-using-shelf"
tags = ["Dart"]
draft = false
toc = false
image = "https://images.unsplash.com/photo-1592178036823-ccb0e0e67562?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8Nnx8c2hlbGZ8ZW58MHx8MHx8fDI%3D"
+++

{{< figure title="" src="https://images.unsplash.com/photo-1592178036823-ccb0e0e67562?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8Nnx8c2hlbGZ8ZW58MHx8MHx8fDI%3D" >}}

backend はこちら． <br/>

[I tried Dart backend]({{< relref "20250427110213-i_tried_dart_backend.md" >}}) <br/>

Flutter Web とか AngularDart などあるんですけどちょっと雑にやりたいという場合です． <br/>
コードは下記の通り． <br/>

```dart
const index = '''<!doctype html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.0.1/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-+0n0xVW2eSR5OomGNYDnhzAbDsOXxcvSN1TPprVMTNDbiYZCxYbOOl7+AMvyTG2x" crossorigin="anonymous">
    <title>Hello, world!</title>
  </head>
  <body>
    <main role="role">
      <div class="container">
        <h1 class="mt-5">{{ title }}</h1>
      </div>
    </main>
  </body>
</html>''';void main(List<String> args) async {
  var handler = const shelf.Pipeline()
      .addMiddleware(shelf.logRequests())
      .addHandler(_echoRequest);
  var server = await io.serve(handler, _hostname, port);
  print('Serving at http://${server.address.host}:${server.port}');
}Future<shelf.Response> _echoRequest(shelf.Request request) async {
  var template = Template(index);
  var output = template.renderString({'title': 'hoge'});
  var headers = {'content-type': 'text/html'}
  return shelf.Response.ok(output, headers: headers);
}
```

index.html を静的ファイルで用意するのが普通だと思うんですが，dart compile して動かしてみたかったので dart ファイルに収めました． <br/>
go-bindata などのように executable に静的ファイルを include するような仕組みが Dart にもあるといいですね． <br/>
気が向いたら作りたい． <br/>

html の中身は mustache template を使ってみました． <br/>
というか Dart におけるテンプレートエンジンでこれだ！みたいなのは今のところないです． <br/>
まぁこれより複雑な GUI が欲しいならそれこそ Flutter web を使うべきだと思います． <br/>

