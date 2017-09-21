---
title: iOS工具使用（一）：使用Buck编译你的app
date: 2017-09-20 16:46:16
tags:
---

[Buck](https://buckbuild.com/setup/getting_started.html)是一套通用的构建系统，由 Facebook开源。最大的特色是智能的增量编译可以极大地提高构建速度。最早的时候，Buck只能用在安卓上，现在已经适配了 iOS。

## 关于Buck

Buck能增快构建速度的主要原因是**缓存了编译结果**，通过持续监视项目目录的文件变化，每次编译时只编译有改动的文件。另外功能是 HTTP Cache Server，即通过一台缓存文件服务器来保存大家的编译结果，这样只要团队里其中一人编译过的文件，其他人就不用再编译了，直接下载就行。

## Buck安装与编译尝试
### 1. 安装

step1. 首先确保自己有command line tool,并且安装了Xcode哦；

step2. 运行buck还需要python环境，所以你还需要安装python,这里就不详细说明安装方法，具体参考这里:[在mac上搭建python环境](http://blog.justbilt.com/2014/07/02/setup_python_on_mac/)

step3. 安装java JDK
```bash
brew tap caskroom/cask

brew cask install java
```

step4. 安装buck
```bash
brew tap facebook/fb

brew install buck

```
现在你可以cd到buck目录下，确认是否安装成功
```bash
cd buck

buck -h
```
如果安装成功，你会看到以下画面:
![image](/images/buckhelp.png)

### 2.配置BUCK文件

使用Buck编译，需要2个关键的buck配置文件：
    
1..bugconfigz文件: [.bugconfig](https://buckbuild.com/concept/buckconfig.html)主要用于配置需要使用的语言、平台、项目等信息。

2.BUCK文件: BUCK用于定义编译规则，[iOS编译规则](https://buckbuild.com/rule/apple_asset_catalog.html) 戳这里。

你需要创建.bugconfig和BUCK并放在自己 **项目的根目录** 下，以下是我使用的.bugconfig和BUCK文件实例（'ZhiHuNewsSwift'是一个swift项目名称）

.bugconfig文件
```
[cxx]
  default_platform = iphonesimulator-x86_64
  cflags = -g -fmodules -fobjc-arc -D BUCK -w
  combined_preprocess_and_compile = true

[swift]
  version = 3.1
  compiler_flags = -DBUCK -whole-module-optimization -enable-testing -suppress-warnings
  
[apple]
  iphonesimulator_target_sdk_version = 10.3
  iphoneos_target_sdk_version = 10.3
  
[project]
  ide_prompt = false
```

BUCK文件
```
apple_resource(
    name = 'AppResources',
    dirs = [],
    files = glob(['ZhiHuNewsSwift/*.png','ZhiHuNewsSwift/*.xib','ZhiHuNewsSwift/*.storyboard']),
)

apple_asset_catalog(
  name = 'AppAsset',
  dirs = ['ZhiHuNewsSwift/Assets.xcassets'],
  app_icon = 'AppIcon',
)

apple_library(
    name = 'AppPods',
    preprocessor_flags = ['-fobjc-arc'],
    srcs = glob([
      'Pods/**/*.m',
      'Pods/**/*.mm',
      'Pods/**/*.swift',
    ]),
    exported_headers = glob([
      'Pods/**/*.h',
    ]),
    frameworks = [
    '$SDKROOT/System/Library/Frameworks/UIKit.framework',
    '$SDKROOT/System/Library/Frameworks/Foundation.framework',
    '$SDKROOT/System/Library/Frameworks/CFNetwork.framework',
)
 
apple_binary(
    name = 'AppBinary',
    srcs = glob([
        'ZhiHuNewsSwift/*.swift',
      ]),
    deps = [':AppResources', ':AppAsset',':AppPods'],
    frameworks = [
        '$SDKROOT/System/Library/Frameworks/Foundation.framework',
        '$SDKROOT/System/Library/Frameworks/UIKit.framework',
    ],
)

apple_bundle(
    name = 'App',
    binary = ':AppBinary',
    tests = [':AppTest'],
    extension = 'app',
    info_plist = 'ZhiHuNewsSwift/Info.plist',
    info_plist_substitutions = {
        'PRODUCT_BUNDLE_IDENTIFIER': 'com.arjunalabs.ZhiHuNewsSwift',
        'CURRENT_PROJECT_VERSION': '1',
    },
)

apple_package(
  name = 'ZhiHuNewsSwiftAppPackage',
  bundle = ':App',
)

apple_test(
    name = 'AppTest',
    test_host_app = ':App',
    srcs = glob(['ZhiHuNewsSwiftTests/*.swift']),
    info_plist = 'ZhiHuNewsSwiftTests/Info.plist',
    frameworks = [
        '$SDKROOT/System/Library/Frameworks/Foundation.framework',
        '$PLATFORM_DIR/Developer/Library/Frameworks/XCTest.framework',
        '$SDKROOT/System/Library/Frameworks/UIKit.framework',
    ],
)
```
### 3. Build/Run app using Buck

在开始编译之前，需要设置xcode, cd到buck根目录，执行：
```bash
buck project --ide xcode
```

接下来，就可以开始编译你的项目啦，cd到项目的根目录，执行：
```bash
buck build :App
```
如果想在模拟器上运行你的app,则在项目的根目录，执行：
```bash
buck install --run :App
```

## 参考资料
 [Buck sample from airbnb](https://github.com/airbnb/BuckSample)
 
 [Fast scaling iOS at Uber](https://atscaleconference.com/videos/blazing-fast-scaling-ios-at-uber/)
 
 [Uber monorepo using Buck](https://eng.uber.com/ios-monorepo/)

[Buck as an alternative build too from Realm news](https://academy.realm.io/posts/altconf-uri-baghin-buck-an-alternative-build-tool/)

## Having Fun !
