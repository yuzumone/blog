+++
title = "Prometheus time unittest"
author = ["yuzumone"]
date = 2023-01-28
slug = "prometheus-time-unittest"
tags = ["Prometheus"]
draft = false
toc = false
image = "https://images.unsplash.com/photo-1563861826100-9cb868fdbe1c?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8Mnx8Y2xvY2t8ZW58MHx8MHx8fDI%3D"
+++

{{< figure title="" src="https://images.unsplash.com/photo-1563861826100-9cb868fdbe1c?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8Mnx8Y2xvY2t8ZW58MHx8MHx8fDI%3D" >}}

現在の unixtime を返してくれる time() を使ったときの Prometheus Unittest の書き方です． <br/>

よくあるのは SSL 証明書が切れそうなときアラートを発破したいという時かと思います． <br/>
このアラートのテストをしたいと思ったとき，time() をどう mock するかみたいなことを考えるかと思います． <br/>

```yaml
rules:
  - alert: SSLCertExpiringSoon
    expr: probe_ssl_earliest_cert_expiry - time() < 86400 * 3
    for: 1m
```

答えはだいたい下記の issue に書いてあります．eval_time が 0s の時 time() は 0 を，例えば 1m の時は 60 を返すようになっているようです． <br/>
promtool unittests fail with rate() &amp; time() · Issue #4817 · prometheus/prometheus <br/>
You can't perform that action at this time. You signed in with another tab or window. You signed out in another tab or… <br/>

github.com <br/>

ということでこんな感じに書けると思います． <br/>

```yaml
- interval: 15s
  input_series:
  - series: probe_ssl_earliest_cert_expiry
    values: 0+0x100
  alert_rule_test:
  - alertname: SSLCertExpiringSoon
    eval_time: 0s
    exp_alerts: []
  - alertname: SSLCertExpiringSoon
    eval_time: 90s
    exp_alerts:
      - exp_labels:
          severity: warning
```

