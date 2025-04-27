+++
title = "Revisiting Android Spinners"
author = ["yuzumone"]
date = 2018-08-27
slug = "revisiting-android-spinners"
tags = ["Android"]
draft = false
toc = false
image = "https://images.unsplash.com/photo-1484417894907-623942c8ee29?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8NDN8fGNvZGluZ3xlbnwwfHwwfHx8Mg%3D%3D"
+++

{{< figure title="Photo by Emile Perron on Unsplash" src="https://images.unsplash.com/photo-1484417894907-623942c8ee29?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8NDN8fGNvZGluZ3xlbnwwfHwwfHx8Mg%3D%3D" >}}


## arrays.xml {#arrays-dot-xml}

arrays は string-array で設定します． <br/>
integer-array でやる場合は自分で spinner の xml を書かないといけなくなると思います． <br/>

```xml
<resources>
    <string-array name="message_count">
        <item>100</item>
        <item>200</item>
        <item>500</item>
        <item>1000</item>
        <item>2000</item>
    </string-array>
</resources>
```


## 使う側 {#使う側}

string-array でやれば android.R.layout.simple_spinner_item と android.R.layout.simple_spinner_dropdown_item が使えるので楽です． <br/>
逆に android~ を使わない場合は自分で ArrayAdapter を作成して Spinner#setAdapter() すれば良さそうです． <br/>

```kotlin
val spinnerAdapter = ArrayAdapter.createFromResource(this,
                R.array.message_count, android.R.layout.simple_spinner_item)
spinnerAdapter.setDropDownViewResource(android.R.layout.simple_spinner_dropdown_item)
spinnerCount.adapter = spinnerAdapter
spinnerCount.onItemSelectedListener = object : AdapterView.OnItemSelectedListener {
    override fun onNothingSelected(parent: AdapterView<*>?) {}override fun onItemSelected(parent: AdapterView<*>?, view: View?, position: Int, id: Long) {
        val count = parent!!.getItemAtPosition(position).toString().toInt()
        // なんか
    }
}
```


## まとめ {#まとめ}

Spinner でググっても記事の日付が古いものしか出てこなかったのでメモで書いてみました． <br/>

