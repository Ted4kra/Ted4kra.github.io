
---
layout: post
title: "Flutter国际化-使用flutter_i18n的全流程"
description: "Flutter国际化方法：使用flutter_i18n插件"
tag: Flutter,国际化, 插件 
---


简介： flutter_i18n 有插件，比Intl简单。
## 1. 在Android Studio中安装flutter_i18n插件

搜索 **Flutter i18n** 安装，或者直接搜索Flutter

![17081109-4faac53ac41ae719](media/15644878757036/17081109-4faac53ac41ae719.png)


安装完成后， 需要重启。
安装成功后，再打开项目
 
 * 会多一个与lib同级的res文件夹，里面自动生成values文件夹，里面有string_en.arb文件。

![17081109-ab96196f5db616b9](media/15644878757036/17081109-ab96196f5db616b9.png)


 * 上面会多一个按钮
![17081109-8cf5df01d7b40207](media/15644878757036/17081109-8cf5df01d7b40207.png)


* 默认只添加了英文，可以添加自己需要的arb文件
![17081109-2feb7e67405885b1](media/15644878757036/17081109-2feb7e67405885b1.png)


* 每次更新arb文件后，会自动生成一个文件，在lib/ganerated/i18n.dart,如果自动生成不成功，点击按钮。
![17081109-8b01b08d6b0a57b4](media/15644878757036/17081109-8b01b08d6b0a57b4.png)



> 注意
> 1. strings.en.arb不能删除，否则会生成失败....原因不知道
> 2. 如果点击按钮，直接报错了，报错位置在右下角！！！会有一个红点。检查项目根目录中.idea里是否有misc.xml，如果没有就复制老项目的。然后再生成
![17081109-5e8cce3ebfba7e8f](media/15644878757036/17081109-5e8cce3ebfba7e8f.png)

## 2.配置main.dart 

1. 引入import 'generated/i18n.dart';
2. 在localizationsDelegates中增加S.delegate
3. supportedLocales： S.delegate.supportedLocales,
4. 如果只支持中文：localeResolutionCallback: S.delegate.resolution(fallback: Locale("zh","")) // Locale("zh","")与Locale("zh","CN")；CN是简体中文，如果是香港中文是Locale("zh","HK")


>  home: MyHomePage(title: "test.title"), 这里，这个title不能用S.of(context).apptitle。~~不知道为啥~~
> 经[墨尘心](https://www.jianshu.com/u/b450b7bd5957)指导：
>  > home: MyHomePage(title: "test.title"), 这里，这个title不能用S.of(context).apptitle。
因为这时候S还没被塞到context中，S.of(context)这时候还是null

```
import 'package:flutter_localizations/flutter_localizations.dart';
import 'generated/i18n.dart'; // 这个就是前面自动生成的

class MyApp extends StatelessWidget {
  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      localizationsDelegates: [
        GlobalMaterialLocalizations.delegate,
        GlobalWidgetsLocalizations.delegate,
        S.delegate,
      ],
      supportedLocales: S.delegate.supportedLocales,
//      localeResolutionCallback: S.delegate.resolution(fallback: Locale("zh","CN")),
      theme: ThemeData(
        primarySwatch: Colors.blue,
      ),
      home: MyHomePage(title: "test.title"),
//      onGenerateTitle: (context) => S.of(context).appName,
    );
  }
}
```

这里的***import 'package:flutter_localizations/flutter_localizations.dart';***，需要在pubspec.yaml文件中引入**flutter_localizations**；

```
dependencies:
  flutter:
    sdk: flutter
  flutter_localizations:
    sdk: flutter
```

## 3.使用的时候

S.of(context).hello("last word")
     
```
FlatButton(
  onPressed: (){
    seriesOauth.getData();
  },
  child: Text(S.of(context).hello("last word")),
),

```

>  ~~一个问题~~
```
  "selectedRowCountTitleZero": "没有选择条目",
  "selectedRowCountTitleMany": "选择了几个条目",
  "selectedRowCountTitleOne": "选择了一个条目",
  "selectedRowCountTitleOther": "选择了$selectedRowCount个条目"
```
在中文里面添加上面的条目，1.自动生成的i18n不会生成相应的代码，不知道为什么，可以自己改，但是每次更新的时候还是会还原 2.生成的代码错误...
 
```
  String selectedRowCountTitle(dynamic selectedRowCount个条目) {
    switch (selectedRowCount个条目.toString()) {
      case "0":
        return "没有选择条目";
      case "1":
        return "选择了一个条目";
      case "many":
        return "选择了几个条目";
      default:
        return "选择了$selectedRowCount个条目";
    }
  }
}
```
----
修改为：

```  
"selectedRowCountTitleOther": "选择了${selectedRowCount}个条目"
```
即可
