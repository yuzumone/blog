+++
title = "Getting values from other Views in Android Data Binding"
author = ["yuzumone"]
date = 2018-07-07
slug = "getting-values-from-other-views-in-android-data-binding"
tags = ["Android"]
draft = false
toc = false
image = "https://images.unsplash.com/photo-1517852058149-07c7a2e65cc6?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwcm9maWxlLXBhZ2V8MXx8fGVufDB8fHx8fA%3D%3D"
+++

{{< figure title="Photo by Henri L. on Unsplash" src="https://images.unsplash.com/photo-1517852058149-07c7a2e65cc6?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwcm9maWxlLXBhZ2V8MXx8fGVufDB8fHx8fA%3D%3D" >}}

```xml
<layout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    >
  <data>
    <variable
        name="viewModel"
        type="example.MyViewModel"/>
  </data>

  <android.support.constraint.ConstraintLayout
      android:layout_width="match_parent"
      android:layout_height="match_parent"
      android:padding="16dp"
      >
    <android.support.design.widget.TextInputEditText
        android:id="@+id/edit_name"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        />

    <Button
        android:id="@+id/button_ok"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:onClick="@{(v) -> viewModel.action(editName.getText().toString())}"
        app:layout_constraintTop_toBottomOf="@+id/edit_name"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        />
  </android.support.constraint.ConstraintLayout>
</layout>
```

ポイント <br/>

-   Java or Kotlin 側で使うように CamelCase で view の ID を書く <br/>

