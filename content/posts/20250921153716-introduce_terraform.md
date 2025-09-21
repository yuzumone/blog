+++
title = "Introduce Terraform"
author = ["yuzumone"]
date = 2025-09-21
slug = "introduce-terraform"
tags = ["Terraform"]
draft = false
toc = true
image = "https://images.unsplash.com/photo-1607234591335-3ebee22bd2b4?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.1.0&ixid=M3wxMjA3fDB8MHxzZWFyY2h8MTF8fHRlcnJhZm9ybXxlbnwwfHwwfHx8Mg%3D%3D"
+++

最近転職活動を少しやっていまして，その中でお断りされる理由が以下の 2 つです． <br/>

1.  パブリッククラウドの経験 <br/>
    これはプライベートクラウドをばりばりやっている弊社ではどうしようもない． <br/>
    その代わり自社 DC 運用経験というのがあるわけですが，オンプレの「オ」の字も考えていない企業からしたらムダな経験ってワケですね． <br/>
2.  Terraform の経験 <br/>
    パブリッククラウドの IaC を Terraform でやるなんてもう当たり前です． <br/>
    上述のとおりプライベートクラウドかつ自社向け PF が提供されているので，これまで利用する必要がありませんでした． <br/>

正直 Terraform というか構成管理については Python Fabric から始まり，Chef を利用しそこから Ansible に移行するまで実施していたので，やればできるだろぐらいに考えていました． <br/>
とはいえ Terraform の実務経験がないことに渋られるのもなんかムカついてきたので，チーム内で Terraform を利用することにしました． <br/>
利用するだけだと職務経歴書に書いても響かないと思うので，Provider の作成も実施しました． <br/>
今回は Terraform Provider を自作して導入してみた，ということで記事を書かせていただきます． <br/>


## 教材 {#教材}

以下のとおり進めるだけで簡単に Provider が作成できました． <br/>
資料が優秀． <br/>

[Implement a provider with the Terraform Plugin Framework](https://developer.hashicorp.com/terraform/tutorials/providers-plugin-framework/providers-plugin-framework-provider) <br/>

テンプレートも GitHub にあるのでこれを clone して上の通り進めるだけです． <br/>

[terraform-provider-scaffolding-framework](https://github.com/hashicorp/terraform-provider-scaffolding-framework) <br/>


## data から作成する {#data-から作成する}

チュートリアルでも Implement data source があるのでそのまま進めば data から作成することになります． <br/>
data いらない resource から作るぜというのもできるのですが，data の場合は Read だけとりあえず実装すれば動くので雰囲気がつかみやすいです． <br/>
resource の場合は Read/Create/Update/Delete の 4 つ実装する必要があります． <br/>


## Provider の配布 {#provider-の配布}

実務で Provider を作成するとコードは GitHub Enterprise に置くことになると思います． <br/>
Provider の配布ですが Registry を用意する必要があり，GitHub にとりあえずバイナリ置けば OK という感じではありません． <br/>
普通チームで利用することになると思うので，作成前にどのように Provider を共有するかは考えておいた方が良いです． <br/>


## チーム内展開 {#チーム内展開}

もちろんチームメンバーは Terraform を利用したことがないのでここが一番つらかったです． <br/>
インフラエンジニアなので BGP/OSPF などのルーティングプロトコルを理解した上でアプライアンス機器の運用・管理するのが主務なのであり，Terraform テンプレートの作成を含むコードを書くというのに慣れていないメンバーがほとんどです． <br/>
Ansible はその点 yaml を利用しており，モジュールや何を記載するかなどの問題はありつつも，フォーマットで問題になることがありませんでした． <br/>
Terraform テンプレートというか hcl をみんなで書いていくというのは，かなり難易度が高いです． <br/>
初期導入ということで大きなインパクトがないところに利用しているため，ドキュメントを共有することでとりあえずは運用できている状態です． <br/>
このへんはどのように浸透させていくかまだまだ考えていたいと思っています． <br/>
(実はプライベートクラウドの IaC のために，Terraform を利用していこうという噂があったのでチーム内への導入はわりとすんなりできました．) <br/>


## まとめ {#まとめ}

職務経歴書には早速 Terraform って書こうと思います． <br/>

