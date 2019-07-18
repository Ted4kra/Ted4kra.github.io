---
layout: post
title: "Flutter构建一个透明背景的Widget"
description: "Flutter构建一个透明背景的Widget"
tag: flutter，半透明背景 
---

想做一个如下图这样透明背景的widget，但是正常push出来的，在最根部的Container设置成Colors.transparent,页面也不是透明的。
本文记录的是造出来这个页面的历程；看代码的直接[点这里](https://github.com/Ted4kra/popupView)
<img src="/images/popupview/want.png" alt="img">
### 1. 先展示出来
经过不懈努力，终于试出来一个方法(真的是试出来的😢😢)：

```Dart
 showCupertinoModalPopup(
    context: context, 
    builder: (_) => FeedBackPage()
 );
```
使用这个方法push出来的页面，默认就是透明背景的。当然，这个Cupertino对应的Material方法是：
```Dart
showDialog(
    context: context, 
    builder: (_) => FeedBackPage()
);
```
但是showDialog比showCupertinoModalPopup出来的页面要低一点,而且showCupertinoModalPopup默认的动画更明显一点(本人太菜，还不会手写动画🌚🌚)：

```Dart
onPressed: () {
   showCupertinoModalPopup(context: context, builder: (_) => FeedBackPage());
   showDialog(context: context, builder: (_) => FeedBackPage());
},
```
<img src="/images/popupview/different.png" alt="img">
### 2.在show出来的widget上添加东西

```Dart
class _FeedBackPageState extends State<FeedBackPage> {
  @override
  Widget build(BuildContext context) {
    return Center(
      child: Container(
        color: Colors.white,
        width: 300,
        height: 300,
        child: Center(
          child: Text("这里是文字啊"),
        ),
      ),
    );
  }
}
```
首先把代码写成这个样子，先只是展示一个Text，出来的是这个样子。
💢💢这什么玩意，我没有对Text进行任何设置，怎么就有样式了。我观望flutter半年的，一般情况下，加上一个Scaffold就能解决问题；结果呢，加上Scaffold之后就不会是透明的了，不信你试试。

<img src="/images/popupview/default.png" alt="img">

先不管这个，直接再添加一个TextField上去：

```
class _FeedBackPageState extends State<FeedBackPage> {
  @override
  Widget build(BuildContext context) {
    return Center(
      child: Container(
        color: Colors.white,
        width: 300,
        height: 300,
        child: Column(
          children: <Widget>[
            Text("这里是文字啊"),
            TextField()
          ],
        ),
      ),
    );
  }
}
```

然后，em...噩梦👿👿

<img src="/images/popupview/error.png" alt="img">


按照我开发flutter两周的经验，这个错误还是可以通过加一个Scaffold解决。💢💢
这时的控制台：
```
flutter: The following assertion was thrown building TextField(dirty, state: _TextFieldState#c1f8e):
flutter: No Material widget found.
flutter: TextField widgets require a Material widget ancestor.
flutter: In material design, most widgets are conceptually "printed" on a sheet of material. In Flutter's
flutter: material library, that material is represented by the Material widget. It is the Material widget
flutter: that renders ink splashes, for instance. Because of this, many material library widgets require that
flutter: there be a Material widget in the tree above them.
flutter: To introduce a Material widget, you can either directly include one, or use a widget that contains
flutter: Material itself, such as a Card, Dialog, Drawer, or Scaffold.
flutter: The specific widget that could not find a Material ancestor was:
flutter:   TextField
```
看了控制台，心情舒爽多了；
> To introduce a Material widget, you can either directly include one, or use a widget that contains Material itself, such as a Card, Dialog, Drawer, or Scaffold.

这里的such as，除了Scaffold，还是有Card，Dialog，Drawer；很显然，Card很适合我们😄😄(虽然Card可能不是这么用的，但是，反正能这么实现)；于是改代码,顺便调整一个嵌套结构,加了一个close按钮：
```Dart
class _FeedBackPageState extends State<FeedBackPage> {
  @override
  Widget build(BuildContext context) {
    return Center(
      child: Container(
        color: Colors.white,
        width: 300,
        height: 300,
        child: Card(
          child: Column(
            children: <Widget>[
              Text("这里是文字啊"),
              TextField(),
              IconButton(
                icon: Icon(
                  Icons.close,
                  size: 35,
                ),
                onPressed: () => Navigator.pop(context),
              )
            ],
          ),
        ),
      ),
    );
  }
}
```
这时候界面是这样的：

<img src="/images/popupview/normal.png" alt="img">


嗯，界面忽然就可爱多了🐶。

### 3.下一步，以为可以处理细节了：
TextField设置多行，此时键盘上默认的是换行，所以加一个失去焦点的方法,同时在最外层套一个GestureDetector
```Dart
class _FeedBackPageState extends State<FeedBackPage> {
  TextEditingController contentController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: ()=> endEditing(),
      child: Center(...),
    );
  }

  endEditing() {
    FocusScope.of(context).requestFocus(FocusNode());
  }
}
```
这时，无意间点了透明的地方，这个玩意自己pop了😭😭。无奈，在最外层又添加了一个全屏的透明的Container：

```Dart
class _FeedBackPageState extends State<FeedBackPage> {
  TextEditingController contentController = TextEditingController();
  @override
  Widget build(BuildContext context) {
    return GestureDetector(
      onTap: ()=> endEditing(),
      child: Container(
        color: Colors.transparent,
        child: body(),
      ),
    );
  }
```
这样，点起来看似舒服多了；
> 在公司写的时候，无意间发现，点击屏幕边缘的时候，还是会自动的pop，但是重新写一遍就没了？？？？[黑人问号脸.jpg]
如果你遇到了，可以在最外层添加一层Material，并设置type为MaterialType.transparency,如果没遇见，当我没说..溜了溜了。

### 4.下一个内容
接下来就是键盘遮挡TextField的问题了，理想状态应该是，键盘弹出来之后，白色的widget是可以上下滑动的，但是没有实现出来🌚🌚；只是加了一个bottom的Margin，让Card在中心靠上的合适位置显示了；等实现了再更新吧。
最后的效果和[代码](https://github.com/Ted4kra/popupView)：
<img src="/images/popupview/done.gif" alt="gif">

