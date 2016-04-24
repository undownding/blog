---
layout: post
title: 使用 GitCafe Pages 搭建 Maven Repository
date: 2015-6-10
---

随着 Android Studio 的普及，越来越多 Android 第三方开源项目使用 Gradle 作为构建工具。Gradle 其中的一个优势是优秀的包管理。我们只需要在脚本里定义要引用的库的名字，以及要引用的版本，构建的时候就会自动从 Maven 中央仓库中获取对应的 aar / jar。

而如果要上传自己的包到 Maven 中央仓库，并不是一件容易操作的事情。这里介绍利用 GitCafe 的 Pages 服务搭建 Maven Repository 的方法。

## 准备工作 

首先把 [aar-quickstart](https://gitcafe.com/UnDownDing/aar-quickstart) 项目 fork 一份到自己的账号上，并 clone 到本地。

把要构建的开源库项目作为 submodule 添加到本地的，例如下面这样：

`git submodule add https://gitcafe.com/fython/MaterialPreferenceCompat-AAR`

这样的话现在我们本地有一个 quickstart-aar 项目，且里面有目标第三方开源项目的 git 作为 submodule。

## 修改脚本

用文本编辑器打开 `build.gradle`,修改以下地方:

```
android {

    ...

    sourceSets {
        main {
            manifest.srcFile 'projectpath/library/src/main/AndroidManifest.xml'
            java.srcDirs = ['projectpath/library/src/main/java']
            res.srcDirs = ['projectpath/library/src/main/res']
        }
    }
}
```

这里把 `projectpath/library` 修改为开源项目对应的源码目录（相对目录）。

```
dependencies {
    compile 'com.android.support:appcompat-v7:22.1.0'
}

group = 'packagename'
version = '1.0.0'
def artifactId = 'library'
```

`dependencies` 部分自行按照开源项目的依赖修改。

下面 `group`，`version`，`artifactId` 都按照 Maven 的命名规则修改，下面是一个示例：

```
group = 'moe.feng.materialcompat'
version = '1.0.0'
def artifactId = 'library'
```

## 部署到 Pages

在 Shell 窗口中执行：

`./build-gitcafe-pages.sh`

即可自动构建出必要的文件，以及生成作为索引的 index.html，并且会自动部署到服务器上，如下图：

![](/content/images/articles/maven-repo.png)


## 在自己的项目中引用这个 library

修改项目的 `build.gradle` ，作如下修改：

```
repositories {
    maven { url "http://fython.gitcafe.io/MaterialPreferenceCompat-AAR" }
    ...
}

dependencies {
    compile 'moe.feng.materialcompat:library:1.0@aar'
    ...
}
```

其中 `url` 部分替换为该项目的 pages url，`dependencies` 部分按照上面修改过的那部分来操作。

