+++
title = "Flutter Android Release Build on GitHub Actions"
author = ["yuzumone"]
date = 2020-08-30
slug = "flutter-android-release-build-on-github-actions"
tags = ["Flutter"]
draft = false
toc = false
image = "https://images.unsplash.com/photo-1590595906931-81f04f0ccebb?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwcm9maWxlLXBhZ2V8MTZ8fHxlbnwwfHx8fHw%3D"
+++

{{< figure title="Photo by Richy Great on Unsplash" src="https://images.unsplash.com/photo-1590595906931-81f04f0ccebb?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwcm9maWxlLXBhZ2V8MTZ8fHxlbnwwfHx8fHw%3D" >}}


## workflow.yml {#workflow-dot-yml}

ほんと単純で keystore を用意して，いわゆる local.properties を作成して build しているだけですね． <br/>
flavor も考えるとつらいですが自分のケースの場合は — dart-define=&lt;foo=bar&gt; で済むケースがほとんどです． <br/>

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-java@v1
        with:
          java-version: '12.x'
      - uses: subosito/flutter-action@v1
        with:
          channel: 'stable'
      - run: echo "${{secrets.ANDROID_RELEASE_BASE64_KEY}}" | base64 -d > ./keystore.jks
      - run: echo "keyAlias=keystore" >> android/key.properties
      - run: echo "keyPassword=${{secrets.ANDROID_RELEASE_KEY_PASSWORD}}" >> android/key.properties
      - run: echo "storeFile=`pwd`/keystore.jks" >> android/key.properties
      - run: echo "storePassword=${{secrets.ANDROID_RELEASE_STORE_PASSWORD}}" >> android/key.properties
      - name: build apk
        run: flutter build apk --build-number ${GITHUB_RUN_NUMBER}
      - name: deploygate
        run: |
          curl \
          -H "Authorization: token ${{secrets.DEPLOYGATE_API_KEY}}" \
          -F "file=@build/app/outputs/flutter-apk/app-release.apk" \
          "https://deploygate.com/api/users/{username}/apps"
```


## android/app/build.gradle {#android-app-build-dot-gradle}

上で作成した key.properties を読むようにします． <br/>

```gradle
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
  keystoreProperties.load(newFileInputStream(keystorePropertiesFile))
}signingConfigs {
  release {
    keyAlias keystoreProperties['keyAlias']
    keyPassword keystoreProperties['keyPassword']
    storeFile file(keystoreProperties['storeFile'])
    storePassword keystoreProperties['storePassword']
  }
}
```

