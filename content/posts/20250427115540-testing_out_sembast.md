+++
title = "Testing out sembast"
author = ["yuzumone"]
date = 2020-04-25
slug = "testing-out-sembast"
tags = ["Dart", "Flutter"]
draft = false
toc = false
image = "https://images.unsplash.com/photo-1523961131990-5ea7c61b2107?q=80&w=2048&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D"
+++

{{< figure title="Photo by fabio on Unsplash" src="https://images.unsplash.com/photo-1523961131990-5ea7c61b2107?q=80&w=2048&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D" >}}

Flutter でモバイルだけでなくデスクトップや Web などマルチプラットフォームに対応している NoSQL データベースライブラリ sembast を使ってみたのでメモ． <br/>

[sembast | Dart Package](https://pub.dev/packages/sembast) <br/>


## JSON から DB をつくる {#json-から-db-をつくる}

DB から JSON への export，import に対応しているんだけどそれなりにしないと import できない． <br/>

[tekartik/sembast.dart](https://github.com/tekartik/sembast.dart/blob/master/sembast/doc/storage_format.md) <br/>

```dart
import 'package:sembase/utils/semast_import_export.dart';var stores = {
  'name': 'message'
  'keys': keys,
  'values': values,
};
var data = {
  'sembase_export': 1,
  'version': 1,
  stores: [stores],
};
var db = await importDatabase(data, databaseFactory, 'mydata.db');
```


## クエリ {#クエリ}

[tekartik/sembast.dart](https://github.com/tekartik/sembast.dart/blob/master/sembast/doc/queries.md) <br/>

```dart
var store = stringMapStoreFactory.store('message');
var finder = Finder(filter: Filter,matches('message', query);
var records = store.find(_db, finder: finder);
records.then((value) =>
  setState(() =>
    _records.addAll(value.map((e)=>e.value).toList())));
```


## マルチプラットフォーム対応 {#マルチプラットフォーム対応}

DB を作成するときにモバイルと Web とかで databaseFactory が違うんだけど作者がええかんじにする utils を公開しているのでこれを使う． <br/>

[tekartik/app_flutter_utils.dart](https://github.com/tekartik/app_flutter_utils.dart/tree/master/app_sembast) <br/>

```dart
import 'package:tekartik_app_flutter_idb/app_sembast.dart';var packageName = 'net.hugahuga.hogehoge';
var databaseFactory = getDatabaseFactory(packageName: packageName);
var db = await importDatabase(data, databaseFactory, 'mydata.db');
```


## おわり {#おわり}

これまで sqflite 使っていたけど Web に対応してないし，これを機に sembast にしてもいいかも． <br/>
Realm が Dart 対応を進めてるらしいのでそれの動向も気になる． <br/>

