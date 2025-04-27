+++
title = "Using Firebase Cloud Messaging and Analytics without google-services.json"
author = ["yuzumone"]
date = 2018-08-04
slug = "using-firebase-cloud-messaging-and-analytics-without-google-services-json"
tags = ["Android"]
draft = false
toc = false
image = "https://images.unsplash.com/photo-1581051357242-e92ba4df7790?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8MTV8fGZpcmViYXNlfGVufDB8fDB8fHwy"
+++

{{< figure title="Photo by Daniel Sturgess on Unsplash" src="https://images.unsplash.com/photo-1581051357242-e92ba4df7790?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8MTV8fGZpcmViYXNlfGVufDB8fDB8fHwy" >}}


## イケてない {#イケてない}

-   GitHub に上げれないので CI/CD しようと思ったときめんどくさい <br/>
-   apply plugin: ‘com.google.gms.google-services’ を app/build.gradle の 1 番下に書く必要がある <br/>
-   FCM と com.google.gms.google-services の仲が悪い（らしい） <br/>

ということで環境変数に APP_ID などを設定して FCM と Analytics を使う方法について書いていきます． <br/>


## FCM {#fcm}

root の build.gradle（これは公式のやつと変わらない） <br/>

```gradle
buildscript {
    ext.kotlin_version = '1.2.51'
    repositories {
        google()
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.1.3'
        classpath 'com.google.gms:google-services:3.2.0'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}allprojects {
    repositories {
        google()
        jcenter()
    }
}
```

app/build.gradle <br/>

```gradle
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'android {
    compileSdkVersion 27
    defaultConfig {
        applicationId "net.yuzumone.test"
        minSdkVersion 21
        targetSdkVersion 27
        versionCode 1
        versionName "1.0"
        buildConfigField("String", "APP_ID", "\"${System.env.FCM_APP_ID}\"")
        buildConfigField("String", "API_KEY", "\"${System.env.FCM_API_KEY}\"")
    }
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
} dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation"org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
    implementation 'com.android.support:appcompat-v7:27.1.1'
    implementation 'com.google.firebase:firebase-messaging:17.1.0'
}
```

Application <br/>

```kotlin
class TestApplication : Application() {
    override fun onCreate() {
        super.onCreate()
        FirebaseApp.initializeApp(this, FirebaseOptions.Builder()
                .setApplicationId(BuildConfig.APP_ID)
                .setApiKey(BuildConfig.API_KEY)
                .build(), getString(R.string.app_name))
    }
}
```

ポイント <br/>

-   buildConfigField を使用して環境変数から BuildConfig へ APP_ID を読み込む <br/>
-   Application クラスで FirebaseApp#initializeApp を呼び，APP_ID と API_KEY を入れる <br/>

これだけでとりあえず FCM は使えるようになる． <br/>
環境変数に値を設定するのを忘れずに． <br/>


## Analytics {#analytics}

FCM にものに追加で設定する必要がある． <br/>

app/build.gradle <br/>

```gradle
android {
    defaultConfig {
        // 追加
        resValue("string", "google_app_id", "\"${System.env.FCM_APP_ID}\"")
    }
} dependencies {
    // 追加
    implementation 'com.google.firebase:firebase-analytics:16.0.1'
}
```

上に示している通り，Analytics を使うには string.xml に google_app_id という name で APP_ID を追加で設定する必要がある． <br/>
string.xml に直で書くと意味がないので，resValue を使用して string.xml に値をセットするようにしてみた． <br/>
とりあえずこれでエラーが出なくなったので OK っぽい． <br/>

