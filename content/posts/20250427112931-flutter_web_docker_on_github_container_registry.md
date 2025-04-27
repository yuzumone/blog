+++
title = "Flutter web docker on GitHub Container Registry"
author = ["yuzumone"]
date = 2020-07-09
slug = "flutter-web-docker-on-github-container-registry"
tags = ["Flutter"]
draft = false
toc = false
image = "https://images.unsplash.com/photo-1504383633899-a17806f7e9ad?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwcm9maWxlLXBhZ2V8NDB8fHxlbnwwfHx8fHw%3D"
+++

{{< figure title="Photo by Victoire Joncheray on Unsplash" src="https://images.unsplash.com/photo-1504383633899-a17806f7e9ad?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwcm9maWxlLXBhZ2V8NDB8fHxlbnwwfHx8fHw%3D" >}}

GitHub Container Registry が GA したみたいなので使ってみます． <br/>
ついでなので Flutter web で． <br/>
普通に docker push するのは普通過ぎるので actions でやります． <br/>
サンプルレポジトリは下記です． <br/>

[yuzumone/flutter-web-docker-sample](https://github.com/yuzumone/flutter-web-docker-sample) <br/>


## Dockerfile {#dockerfile}

サンプルなので flutter create でやってます． <br/>
実際の場合は git clone するか COPY かどっちかですかね． <br/>

```Dockerfile
FROM cirrusci/flutter:beta AS buildRUN flutter config --enable-webRUN flutter create sample
WORKDIR /home/cirrus/sample
RUN flutter build webFROM nginx
COPY --from=build /home/cirrus/sample/build/web /usr/share/nginx/html
```


## workflow.yml {#workflow-dot-yml}

run してるだけだし shell と変わらないけど． <br/>
tag どうするか悩んで github.sha 使ったんですけどベストプラクティスがあれば知りたい． <br/>

```yaml
jobs:
  build:
    name: Build image
    runs-on: ubuntu-18.04
    steps:
      - uses: actions/checkout@master
      - name: Login
        run: docker login docker.pkg.github.com -u ${{ github.actor }} -p ${{ secrets.GITHUB_TOKEN }}
      - name: Build
        run: docker build -t docker.pkg.github.com/${{ github.repository }}/${IMAGE_NAME}:${{ github.sha }} --file Dockerfile .
      - name: Push
        run: docker push docker.pkg.github.com/${{ github.repository }}/${IMAGE_NAME}:${{ github.sha }}
```

あと IMAGE_NAME も今回は env で指定したけど今回みたいに 1 個しか作らないなら下の感じでやるのが良さそう． <br/>

```bash
echo ${{ github.repository }} | awk -F '/' '{print $2}'
```

