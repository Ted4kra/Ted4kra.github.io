---
layout: post
title: "为Flutter项目增加GitLab的CI"
description: "Gitlab自带CI，文章主要讲解如何在gitlab上使用Flutter的CI"
tag: flutter,CI, gitlab
---

本文主要实现了flutter项目的自动打包。每次上传代码到master分支，触发自动测试，自动打包Android的apk和iOS的ipa。其中apk文件放在Sources/build/outputs..常规目录下，ipa文件放在Sources/ios/build目录下。当然目录是可以更改的。

文章可能有点乱，谅解。

步骤同上篇文章iOS差不多。

.gitlab-ci.yml文件内容
```
stages:
  - build
  - archive

build_project:
  stage: build
  script:
    - cd Sources
    - flutter pub get
    - flutter clean
    - flutter doctor && flutter test

  tags:
    - ios_12-1
    - xcode_10-2-1
    - osx_10-14-3

build_project:
  stage: archive
  script:
    - cd Sources
    - flutter clean
    - flutter doctor --android-licenses
    - flutter doctor && flutter -v build apk
    - flutter doctor && flutter -v build ios
    - cd ios
    - xcodebuild clean archive -configuration Release -workspace Runner.xcworkspace -scheme Runner -archivePath 'build/Runner.xcarchive'
    - xcodebuild -exportArchive -configuration Release -archivePath 'build/Runner.xcarchive' -exportPath 'build/Runner.ipa' -allowProvisioningUpdates -exportOptionsPlist 'exportOptions.plist'

  tags:
    - ios_12-1
    - xcode_10-2-1
    - osx_10-14-3

```

步骤：
1. 在gitlab创建项目
2. 选择Runner，可以直接使用上一个文章里创建的Runner，记得在.gitlab-ci.yml中指定相对应的tag就行了；或者再创建一个。

  <img src="/images/flutterCI/1.png" alt="image">

1. 创建flutter项目
2. flutter doctor --android-licenses，创建Android的发布证书
3. 用Xcode打开iOS项目配置证书。选用自动配置
4. 提交代码


我遇到的问题：
* 第一次提交的时候提示没有.packages文件夹，在.gitignore文件里删除这个文件。再重新提交

```
$ flutter clean
.packages does not exist.
Did you run "flutter pub get" in this directory?
ERROR: Job failed: exit status 1
```

* iOS打包的时候需要使用-workspace来指定项目