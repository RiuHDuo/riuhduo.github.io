---
layout: post
title: SwiftUI学习(1)-简单入门
date: 2021-04-18 14:40:20 +0800
description: View是SwiftUI中的一个基础协议(protocol)。该协议用来渲染页面和提供页面的一些修饰器(modifier)。通过继承View协议来创建自定义View。ContentView实现了View协议中的body计算属性(computed property ). body需要返回自定义页面的内容。
img: 1.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [SwiftUI]
---

> **コニクマル**撰写
>
> 本文使用目前最新版本的xcode 12 + macOS Big sur + iOS14

## Hello world

所有的程序入门都是从显示hello world开始的。

- 打开Xcode 新建工程, 并选择Interface为SwifUI, Life Cycle默认选项SwiftUI,这样就可用通过SwiftUI构建完整应用，也可以使用UIKit AppDelegate，xcode会自动创建传统的AppDelegate等类通过UIHostingController来加载SwiftUI组件。

 <img src="/assets/img/image-20210418145302335.png" alt="image-20210418152506454" style="zoom:50%;" />

- Xcode 会自动生成`ContentView`文件, 打开该文件就会自动生成Hello world!的代码。

```swift
struct ContentView: View {
    var body: some View {
        Text("Hello, world!")
            .padding()
    }
}
```



- 点击预览页面上方的`resume`按钮或者按option + command+ p就会出现预览界面。

  <img src="/assets/img/image-20210418152506454.png" alt="image-20210418152506454" style="zoom:50%;" />

- 这样就完成了hello world的显示。

## 常见的一些View

SwiftUI提供许多基础控件，目前来看SwiftUI只处于早期阶段，随着时间推移应该会有越来越丰富的控件提供。

- 展示控件
  - `Text` 用于显示文本的控件
  - `Image`用于显示图片的控件
  - `Color`用于显示颜色
  - `Toggle`bool选择器通过设置不同的style可以实现开关控件， checkbox等
  - `Rectangle`就是个矩形
  - `RoundedRectangle` 圆角矩形
  - `Circle`圆形
- 布局管理
  - `VStack`可以创建垂直布局
  - `HStack`可以创建水平布局
  - `ZStack`可以创建Z轴方向上堆叠布局

## View协议

`View`是SwiftUI中的一个基础协议(protocol)。该协议用来渲染页面和提供页面的一些修饰器(modifier)。 通过继承`View`协议来创建自定义View。

`ContentView`实现了`View`协议中的body计算属性(computed property ). `body`需要返回自定义页面的内容。

`body`的类型是`some View`, `some View`表示无论返回什么类型都一定是满足`View`协议的类型。

`View`协议提供丰富的修饰器(modifier)来配置view的样式比如：

- `frame`用于控制View的大小
- `padding`提供View的内边距
- `background`提供View的背景
- `foregroundColor`提供View前景颜色

每调用`View`的修饰器(modifier)返回一个新`View`, **最终显示的效果跟调用修饰器顺序是有关系的**。

比如

```swift
  Text("Text 1")
      .frame(width: 100, height: 30)
      .background(Color.red)
      .padding()
```

和

```swift
  Text("Text 2")
      .frame(width: 100, height: 30)
      .padding()
      .background(Color.red)
```

运行效果如下:

![image-20210418214540628](/assets/img/image-20210418214540628.png)