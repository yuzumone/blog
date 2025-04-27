+++
title = "Microsoft Graph API Calendar"
author = ["yuzumone"]
date = 2022-10-23
slug = "microsoft-graph-api-calendar"
tags = ["MicrosoftGraphAPI"]
draft = false
toc = false
image = "https://images.unsplash.com/photo-1435527173128-983b87201f4d?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8MXx8Y2FsZW5kYXJ8ZW58MHx8MHx8fDI%3D"
+++

{{< figure title="Photo by Eric Rothermel on Unsplash" src="https://images.unsplash.com/photo-1435527173128-983b87201f4d?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8MXx8Y2FsZW5kYXJ8ZW58MHx8MHx8fDI%3D" >}}

最近歯医者に行きはじめました．そこで困るようになったのが予約です． <br/>
完全リモートワークなのでどの日時でも問題ないのですが，個人の予定表と仕事の予定表をどちらも確認しつつ決めないといけません． <br/>
個人予定表と仕事予定表を一括で見られるようにするために，Microsoft Graph API を利用してかんたんな同期をやってみます． <br/>

ちなみに API が簡単に使えるカレンダーサービスとして TimeTree がありそちらも試してみたのですが，やはり広告が出てしまうのと API 経由で登録したイベントの反映がちょっと遅いような気がしたのでやめました． <br/>


## 手順 {#手順}

流れは下記のとおりです． <br/>

-   Azure でアプリを作成する <br/>
-   Token 取得 <br/>
-   イベント作成 API 叩く on Power Automate <br/>

まずは Azure portal でアプリを作成します． <br/>

<https://portal.azure.com/> <br/>

アプリ作成後設定するのは証明書とシークレットと API のアクセス制御です． <br/>

証明書とシークレットはよくある client_secret です． <br/>
作成後値とクライアントシークレット ID というのが表示されますが，控えておくのは値の方です． <br/>
作成後時間が経つと値は表示されなくなるので気をつけてください． <br/>

API のアクセス制御はどのエンドポイントが叩けるかの設定です． <br/>
雑に設定してしまいましたが，アプリケーションの許可で Calendars.Read と Calendars.ReadWrite があれば問題ないと思います． <br/>

Token の取得はこんな感じです． <br/>

```bash
curl -d "client_id={client_id}" \
     -d "client_secret={client_secret}" \
     -d "scope=https%3A%2F%2Fgraph.microsoft.com%2F.default" \
     -d "grant_type=client_credentials" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -X POST https://login.microsoftonline.com/{tenant_id}/oauth2/v2.0/token
```

詳細はこちら． <br/>

[Get access without a user - Microsoft Graph](https://learn.microsoft.com/ja-jp/graph/auth-v2-servicehttps://learn.microsoft.com/ja-jp/graph/auth-v2-service) <br/>

Token が取得できたらあとはイベント作成 API を叩くだけです． <br/>
今回は仕事用の予定表を作成して，個人のものとは分ける感じにします． <br/>

```bash
curl -X POST -H "Authorization: Bearer {token}" \
      -H "Content-type: application/json" \
      -H 'Prefer: outlook.timezone="TokyoStandard Time"' \
      -d '{"subject": "Let\'s go for lunch", "start": {"dateTime": "2022-10-23T12:00:00", "timeZone": "Tokyo Standard Time"}, "end": {"dateTime": "2022-10-23T13:00:00", "timeZone": "Tokyo Standard Time"}}'
'https://graph.microsoft.com/v1.0/users/{user_id}/calendars/{calendar_id}/events'
```

[Create Event - Microsoft Graph v1.0](https://learn.microsoft.com/ja-jp/graph/api/user-post-events) <br/>

タイムゾーンについての情報． <br/>
日本は「Tokyo Standard Time」 <br/>

[dateTimeTimeZone resource type - Microsoft Graph v1.0](https://learn.microsoft.com/ja-jp/graph/api/resources/datetimetimezone) <br/>

calendar_id は CalendarGroup を辿ると取得できます． <br/>

[Get calendarGroup - Microsoft Graph v1.0](https://learn.microsoft.com/ja-jp/graph/api/calendargroup-get) <br/>

curl で雑に確認できたらあとは Power Automate で設定するだけです． <br/>
変更に追随するのは大変なので，Office イベント作成をトリガーに上記の一連作業が行われるようにします． <br/>
また情報の持ち出しに問われたらめんどくさいので，トリガーされた Office イベントの情報は時間情報だけ利用するようにしてタイトル「予定あり」とだけ追加するようにしました． <br/>

これで歯医者さんで予定決めるからといってスマホを 2 台眺める必要がなくなりました． <br/>

