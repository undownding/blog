---
layout: post
title: 在 Android 中使用 data-binder 绑定布局 xml 与数据
date: 2015-5-31
---


在前几天的 Google IO 2015 中，Google 在 support-v7 中新增了 data-binder，使用 data-binder 可以直接在布局的 xml 中绑定布局与数据，从而简化代码。

因为 data-binder 是包含在 support-v7 包里面的，所以可以向下兼容到最低 Android 2.1 (API level 7+).

---

## 构建环境

需要在 Android SDK Manager 中更新 Support repository 到最新版本，并使用 Android Studio 1.3.0-beta1 或更高的版本。

为了使用 data-binder，我们必须在 build.gradle 中声明对它的依赖：

```grovvy
dependencies {
       classpath "com.android.tools.build:gradle:1.2.3"
       classpath "com.android.databinding:dataBinder:1.0-rc0"
   }
}
```

同样的我们要保证所有的子项目中能找到 jcenter 的中央仓库：

```grovvy
allprojects {
   repositories {
       jcenter()
   }
}
```

在我们需要引用到 data-binder 的子项目中，我们需要引入 data binding 的插件支持：

```grovvy
apply plugin: 'com.android.application'
apply plugin: 'com.android.databinding'
```

data binding 插件必须你项目的 provided 和 configuration dependencies 列表中。

## Data Binding 布局文件
---

### 第一个数据绑定表达式


数据绑定的布局文件并不像以往的布局文件那样以一个 View 作为根元素，而是在根元素在有一个 data 元素以及一个 view 根元素。如果只有 view 作为根元素的话，这个布局文件并不能绑定数据。以下是一个正确的示例：

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"/>
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.lastName}"/>
   </LinearLayout>
</layout>
```

data 描述的 user 变量是这样用到布局里的：

```xml
<variable name="user" type="com.example.User"/>
```

Expressions within the layout are written in the attribute properties using the “@{}” syntax. Here, the TextView’s text is set to the firstName property of user:

在布局引用这个 data 属性的变量应该使用 "@{}" 语句来描述。这里引用的是把 user 的 fristname 属性引用到 TextView 中：

```xml
<TextView android:layout_width="wrap_content"
          android:layout_height="wrap_content"
          android:text="@{user.firstName}"/>
```

---

### 数据对象

（这块我认为没啥好翻译的了）

```java
public class User {
   private final String firstName;
   private final String lastName;
   public User(String firstName, String lastName) {
       this.firstName = firstName;
       this.lastName = lastName;
   }
   public String getFirstName() {
       return this.firstName;
   }
   public String getLastName() {
       return this.lastName;
   }
}
```

建议使用 [jackdaw](https://github.com/vbauer/jackdaw) 自动生成 Setter 与 Getter.

---

### 绑定数据到 Activity

不啰唆，直接上代码：

```java
@Override
protected void onCreate(Bundle savedInstanceState) {
   super.onCreate(savedInstanceState);
   ActivityMainBinding binding = DataBindingUtil.setContentView(this, R.layout.main_activity);
   User user = new User("Test", "User");
   binding.setUser(user);
}
```

完事了。概括来说就是用 `DataBindingUtil.setContentView()` 来取代以往的 `setContentView()`。

如果只是要生成 View 对象而不是显示到 Activity 上，那么应该用以下的代码：

```java
MainActivityBinding binding = MainActivityBinding.inflate(getLayoutInflater());
```

如果绑定的对象是 RecyclerView 或 ListView 的 Item，那么应该这样做：

```java
ListItemBinding binding = ListItemBinding.inflate(layoutInflater, viewGroup,
false);
//or
ListItemBinding binding = DataBindingUtil.inflate(layoutInflater, R.layout.list_item, viewGroup, false);
```

---
进阶：如果需要绑定可观察对象，以及想了解绑定的表达式，请参考原文：
[https://developer.android.com/tools/data-binding/guide.html](https://developer.android.com/tools/data-binding/guide.html)
（完）
