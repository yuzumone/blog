+++
title = "Migrating to null safety on Dart"
author = ["yuzumone"]
date = 2021-03-21
slug = "migrating-to-null-safety-on-dart"
tags = ["Dart"]
draft = false
toc = true
image = "https://images.unsplash.com/photo-1612953117963-5fd4f0736aa8?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwcm9maWxlLXBhZ2V8MTZ8fHxlbnwwfHx8fHw%3D"
+++

{{< figure title="Photo by Jamie Davies on Unsplash" src="https://images.unsplash.com/photo-1612953117963-5fd4f0736aa8?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwcm9maWxlLXBhZ2V8MTZ8fHxlbnwwfHx8fHw%3D" >}}

いよいよ Dart の null safety が stable となりました． <br/>
これまで何回か紹介しているライブラリ ipaddr を null safety にしていく過程をメモしておきます． <br/>
参考までに pullrq は下記です． <br/>

[Sound null safety](https://github.com/yuzumone/ipaddr/pull/27) <br/>

ドキュメントどおりに進めていきます． <br/>

[Migrating to null safety](https://dart.dev/null-safety/migration-guide) <br/>


## Check dependency status {#check-dependency-status}

まずは使用しているライブラリで null safety に対応してないものがないかを確認します． <br/>
ここでつまずくと何もできないです． <br/>
ipaddr ではライブラリをほぼ使っていないので下記の通りでした． <br/>

```bash
$ dart pub outdated --mode=null-safety
Showing dependencies that are currently not opted in to null-safety.
[✗] indicates versions without null safety support.
[✓] indicates versions opting in to null safety.Package Name  Current  Upgradable  Resolvable  Latestdev_dependencies:
pedantic      ✗1.9.2   ✓1.11.0     ✓1.11.0     ✓1.11.0
test          ✗1.15.7  ✓1.16.5     ✓1.16.5     ✓1.16.52 upgradable dependencies are locked (in pubspec.lock) to older versions.
To update these dependencies, use `dart pub upgrade`.
```


## Update dependencies {#update-dependencies}

依存関係に問題が無かったので pub upgrade します． <br/>
null-safety オプションを付けて実行すると pubspec.yaml をいい感じに書き換えてくれます． <br/>

```bash
$ dart pub upgrade --null-safety
$ git diff -U0
diff --git a/pubspec.yaml b/pubspec.yaml
index e20a246..24695a1 100644
--- a/pubspec.yaml
+++ b/pubspec.yaml
@@ -10,2 +10,2 @@ dev_dependencies:
-  pedantic: ^1.8.0
-  test: ^1.6.0
+  pedantic: ^1.11.0
+  test: ^1.16.5
$ dart pub get
```


## Migrate {#migrate}

いよいよ migrating です． <br/>
サポートツールがあるので下記の通り実行します． <br/>
Web GUI を見ろと言われるので見ます． <br/>

```bash
$ dart migrate
```

あとは _**!**_ maker を付けて non-nullable であることを指定するなどを行っていきます． <br/>
終わったら APPLY MIGRATION ボタンクリックで適応されます． <br/>
めっちゃ楽ですね． <br/>

IPv4Address コンストラクタでは null を許容しないようになるので null チェックが常に false になるなども表示されます． <br/>
ignore: unnecessary_null_comparison を指定することで引き続き null check を記載することも可能です． <br/>


## Analyze &amp; Test {#analyze-and-test}

migrating が終わったらコード解析とテストも忘れずにやっておきます． <br/>

```bash

$ dart analyze
$ dart test
```


## Finish {#finish}

はい，これで終了です． <br/>
サポートツールが便利なのであっけなく終わりました． <br/>
package version はメジャーバージョンをあげることを推奨されているので，特に強いこだわりがなければそれで良さそうです． <br/>

