+++
title = "Flutter web oidc authentication"
author = ["yuzumone"]
date = 2020-11-23
slug = "flutter-web-oidc-authentication"
tags = ["Flutter"]
draft = false
toc = false
image = "https://images.unsplash.com/photo-1562770584-eaf50b017307?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8N3x8a2V5fGVufDB8fDB8fHwy"
+++

{{< figure title="Photo by Kelly Sikkema on Unsplash" src="https://images.unsplash.com/photo-1562770584-eaf50b017307?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8N3x8a2V5fGVufDB8fDB8fHwy" >}}

Flutter web で OpenID Connect やってたメモ． <br/>
以下を参考にしました． <br/>

[openid_client | Dart Package](https://pub.dev/packages/openid_client) <br/>

まずは認証ページを出すところまで． <br/>
Flutter web は SPA なので Client Secret は渡せるようになっていますが渡しません． <br/>

```dart
var clientId = 'foobar';
var uri = Uri.parse('https://hogehoge.com');
var issuer = await Issuer.discover(uri);
var client = Client(issuer, clientId);
var authenticator = Authenticator(client, scopes: scopes);
authenticator.authorize();
```

authorize の中身． <br/>
state を LocalStorage に入れて，認証ページを href に入れているだけですね． <br/>

```dart
void authorize() {
  _forgetCredentials();
  window.localStorage['openid_client:state'] = flow.state;
  window.location.href = flow.authenticationUri.toString();
}
```

さあここからサンプルと違うところです． <br/>
サンプルの場合はそもそも Flutter web ではないので 1 枚の View で構成されています． <br/>
しかし Flutter の場合は違います． <br/>
多くの場合以下のような MainApp を用意して home なり routes で子 View がいてその子 View に認証ボタンのようなものを用意するのが多いのではないでしょうか． <br/>

```dart
class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
    ...
    );
  }
}
```

この場合，認証が完了して query parameter で渡された値は当たり前ですが子 View に伝搬されません． <br/>
ということで MyApp の build で openid_client でやっている実装をやってみました． <br/>

```dart
@override
Widget build(BuildContext context) {
  checkExp();
  if (window.location.href != null) {
    var uri = Uri(query: Uri.parse(window.location.href).fragment);
    var q = uri.queryParameters;
    if (q.containsKey('access_token') ||
        q.containsKey('code') ||
        q.containsKey('id_token')) {
      if (window.localStorage['openid_client:state'] == q['state']) {
        window.localStorage['openid_client:auth'] = json.encode(q);
      }
    }
  }
  ...
}
```

checkExp() は期限の確認をしています． <br/>
中身はこんな． <br/>

```dart
void checkExp() {
  if (['', null].contains(window.localStorage['openid_client:auth'])) {
    return;
  }
  var auth = json.decode(window.localStorage['openid_client:auth']);
  var token = auth['id_token'];
  var jwt = json.decode(utf8.decode(base64Url.decode(token.split('.')[1])));
  var exp = jwt['exp'];
  var now = DateTime.now().millisecondsSinceEpoch ~/ 1000;
  if (exp < now) {
    window.localStorage['openid_client:auth'] = '';
  }
}
```

あとは以下のような条件でログイン後の View を出し分けする感じです． <br/>

```dart
['', null].contains(window.localStorage['ipenid_client:auth'])
```

多分 runApp() のところらへんで Authenticator を持って～～ってやればサンプル通りにかけるんだけどこれで動くからいいかなと． <br/>

あと openid は関係ないんだけど，localStrage 参照しているのもあってテストは platform 指定しないと動かなかったのでメモ． <br/>

```dash
$ flutter test --platform chrome -v
```

