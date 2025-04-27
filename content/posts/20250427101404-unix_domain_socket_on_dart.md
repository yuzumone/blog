+++
title = "Unix domain socket on Dart"
author = ["yuzumone"]
date = 2022-07-30
slug = "unix-domain-socket-on-dart-"
tags = ["Dart"]
draft = false
toc = false
image = "https://images.unsplash.com/photo-1534224039826-c7a0eda0e6b3?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwcm9maWxlLXBhZ2V8NjF8fHxlbnwwfHx8fHw%3D"
+++

{{< figure title="Photo by israel palacio on Unsplash" src="https://images.unsplash.com/photo-1534224039826-c7a0eda0e6b3?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwcm9maWxlLXBhZ2V8NjF8fHxlbnwwfHx8fHw%3D" >}}

Dart で Unix domain socket やっていきます． <br/>

[Support making HTTP requests through unix sockets](https://github.com/dart-lang/sdk/issues/42716) <br/>


## 環境 {#環境}

2.17.0–94.0.dev 以上であれば動くはず． <br/>

```bash
$ dart --version
Dart SDK version: 2.17.6 (stable) (Tue Jul 12 12:54:37 2022 +0200) on "linux_x64"
```


## Get {#get}

```dart
import 'dart:convert';
import 'dart:io';void main(List<String> arguments) async {
  HttpClient client = HttpClient()
    ..connectionFactory = (Uri uri, String? proxyHost, int? proxyPort) {
      assert(proxyHost == null);
      assert(proxyPort == null);
      var address = InternetAddress("/var/run/docker.sock",
          type: InternetAddressType.unix);
      return Socket.startConnect(address, 0);
    }
    ..findProxy = (Uri uri) => 'DIRECT';final response =
      await client.getUrl(Uri.parse('http://localhost/_ping')).then((request) {
    return request.close();
  });
  print(response.statusCode);
  final responseText = await response
      .transform(utf8.decoder)
      .fold('', (String x, String y) => x + y);
  print(responseText);
  client.close();
}
```

実行結果 <br/>

```bash
$ dart run bin/socket.dart
200
OK
```

curl だとこれ相当です <br/>

```bash
$ curl --unix-socket /var/run/docker.sock http://localhost/_ping
OK
```


## Post {#post}

```dart
import 'dart:convert';
import 'dart:io';void main(List<String> arguments) async {
  HttpClient client = HttpClient()
    ..connectionFactory = (Uri uri, String? proxyHost, int? proxyPort) {
      assert(proxyHost == null);
      assert(proxyPort == null);
      var address = InternetAddress("/var/run/docker.sock",
          type: InternetAddressType.unix);
      return Socket.startConnect(address, 0);
    }
    ..findProxy = (Uri uri) => 'DIRECT';final data = {
    'Image': 'hello-world',
  };
  final response = await client
      .postUrl(Uri.parse('http://localhost/containers/create'))
      .then((request) {
    request.headers.contentType = ContentType('application', 'json');
    request.write(json.encode(data));
    return request.close();
  });
  print(response.statusCode);
  final res = await postResponse
      .transform(utf8.decoder)
      .fold('', (String x, String y) => x + y);
  print(res);
  client.close();
}
```

実行結果 <br/>

```bash
$ dart run bin/socket.dart
201
{"Id":"636416b53763d106d54555d4709992000bc7709826daaa1f350295bb12d51d1b","Warnings":[]}
```

curl だと <br/>

```bash
$ curl --unix-socket /var/run/docker.sock http://localhost/containers/create
 -X POST -H "Content-Type: application/json" -d '{"Image": "hello-world"}'
{"Id":"ceb6473c162fbb5238703e07253ba5985938295b511453ca59e6b33a8cbbd224","Warnings":[]}
```

