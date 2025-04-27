+++
title = "Using Kotlin's ReadWriteProperty for modern Android Preference handling"
author = ["yuzumone"]
date = 2018-04-27
slug = "using-kotlins-readwriteproprty-for-modern-android-preference-handling"
tags = ["Android"]
draft = false
toc = false
image = "https://images.unsplash.com/photo-1588690154757-badf4644190f?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8MXx8a290bGlufGVufDB8fDB8fHwy"
+++

{{< figure title="Photo by Louis Tsai on Unsplash" src="https://images.unsplash.com/photo-1588690154757-badf4644190f?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8MXx8a290bGlufGVufDB8fDB8fHwy" >}}

Kotlin の ReadWriteProperty を使って Delegate して Preference をいい感じにするやつを使ってみました． <br/>
ちなみに ReadWriteProperty は以下のように定義されています． <br/>

```kotlin
interface ReadWriteProperty<in R, T>
```

[ReadWriteProperty - Kotlin Programming Language](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.properties/-read-write-property/) <br/>


## ReadWriteProperty {#readwriteproperty}

例えば int を Preference に入れる ReadWriteProperty <br/>

```kotlin
class IntPreference(
        private val preferences: SharedPreferences,
        private val name: String,
        private val defaultValue: Int
) : ReadWriteProperty<Any, Int> {@WorkerThread
    override fun getValue(thisRef: Any, property: KProperty<*>): Int {
        return preferences.getInt(name, defaultValue)
    }override fun setValue(thisRef: Any, property: KProperty<*>, value: Int) {
        preferences.edit().putInt(name, value).apply()
    }
}
```


## 使う側 {#使う側}

あとは以下のような感じで Delegate <br/>

```kotlin
class SharedPreferenceStorage(context: Context) {
    private val prefs = context.applicationContext.getSharedPreferences(PREFS_NAME, MODE_PRIVATE)
    companion object {
        private const val PREFS_NAME = "example.prefs"
        private const val PREF_MESSAGE_COUNT = "pref_message_count"
    }
    var messageCount by IntPreference(prefs, PREF_MESSAGE_COUNT, 200)
}
```


## 何がいいか {#何がいいか}

例えば Preference に出し入れする値が多いと以下のように set()と get()を何回も書かなければなりません． <br/>

```kotlin
var hoge: String
    get() = preference.getString(HOGE, "")
    set(value) = preference.edit().putString(HOGE, value).apply()

var huga: String
    get() = preference.getString(HUGA, "")
    set(value) = preference.edit().putString(HUGA, value).apply()

var foo: String
    get() = preference.getString(FOO, "")
    set(value) = preference.edit().putString(FOO, value).apply()

var bar: String
    get() = preference.getString(BAR, "")
    set(value) = preference.edit().putString(BAR, value).apply()
```

上がこうなる． <br/>
きれい． <br/>

```kotlin
var hoge by StringPreference(prefs, HOGE, "hoge")
var huga by StringPreference(prefs, HUGA, "huga")
var foo by StringPreference(prefs, FOO, "foo")
var bar by StringPreference(prefs, BAR, "bar")
```


## まとめ {#まとめ}

Preference が多くなるだろうと思われるときは積極的に ReadWriteProperty を使っていこう． <br/>

