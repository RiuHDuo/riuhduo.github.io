---
layout: post
title: SwiftUI学习(6)-SwiftUI动画入门上
date: 2021-04-28 20:01:20 +0800
description: 介绍SwitfUI基本动画的使用。
img: swiftui_logo.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [SwiftUI]
---

>  **コニクマル**撰写
>
> 本文使用目前最新版本的xcode 12 + macOS Big sur + iOS14

SwiftUI和UIKit一样提供了显式(Explicit)和隐式(Implicit )动画。

## 显式动画

在UIKit中显式动画通过`UIView.animate`来触发, 如下

```swift
UIView.animate(withDuration: 0.5) {
    someview.alpha = 0
}
```

在SwiftUI中则使用`withAnimation`来完成显式动画,举个例子:

```swift
@State var alphaValue: Double = 1
  var body: some View {
      VStack {
          Text("Hello, World!")
              .opacity(self.alphaValue)
              .padding()

          Button("启动") {
              withAnimation(.easeIn(duration: 0.5)) {
                  self.alphaValue = 0
              }
          }
      }
  }
```

![2021-05-09 -2003.36](/assets/img/2021-05-09 -2003.36.gif)

## 隐式动画

通过设置`View`的`animation`装饰器(modifier), 当`View`相关的属性值变化后,提供会自动添加过渡的动画。比如

```swift
@State var isClicked: Bool = false
var body: some View {
    VStack {
        RoundedRectangle(cornerRadius: 25)
            .foregroundColor(self.isClicked ? .red : .green)
            .frame(width: self.isClicked ? CGFloat(300.0) : CGFloat(200.0), height: 200)
            .padding()
            .animation(.default)
        Button("启动") {
            self.isClicked.toggle()
        }
    }
}
```

运行效果如果下:

<img src="/assets/img/2021-05-09-20-16-50.gif" alt="2021-05-09-20-16-50" style="zoom:50%;" />

## Animation结构体

系统的`Animation`结构体为我们提供一些常用的动画比如`easeOut` ` easeIn` `easeInOut` `spring`等常用的动画。无论显式还是隐式动画都需要使用到`Animation`可以通过直接传入`Animation`静态成员,

```swift
RoundedRectangle(cornerRadius: 25)
		....
    .animation(Animation.easeOut)
```

也可以调用`Animation`的静态方法设置一些参数比如设置动画时间1秒

```swift
RoundedRectangle(cornerRadius: 25)
		....
    .animation(Animation.easeOut(duration: 1))
```

### 重复动画

`Animation`具有`repeatForever` `repeatCount`的装饰器可以用来完成一些循环动画比如:

```swift
@State var isClicked: Bool = false
var body: some View {
    VStack {
        Image(systemName: "suit.heart.fill")
            .resizable()
            .scaledToFit()
            .foregroundColor(.red)
            .frame(width: 200, height: 200, alignment: .center)
            .scaleEffect(self.isClicked ? 0.5: 1)
            .animation(Animation.linear(duration: 0.5).repeatForever())
            .padding()

        Button("启动") {
            self.isClicked.toggle()
        }
        .padding()
    }
}
```

效果如下

![2021-05-09-20-45-00](/assets/img/2021-05-09-20-45-00.gif)

### 延迟动画

`Animation`的`delay`装饰器可以延迟动画启动时间,对于多个`View`的联合的动画相当有用, 比如实现各加载的indicator:

```swift
@State var isClicked: Bool = false
var body: some View {
  VStack {
      HStack{
          let scale: CGFloat = isClicked ? 0 : 1
          ForEach(0 ..< 5){ index in
              Circle()
                  .frame(width: 50, height: 50, alignment: .center)
                  .scaleEffect(scale)
                  .foregroundColor(.red)
                  .animation(Animation.linear(duration: 0.8).repeatForever().delay(0.2 * Double(index)))
          }
      }

      Button("启动") {
          self.isClicked.toggle()
      }
      .padding()
  }
}
```

效果如下:

![2021-05-09-20-54-35](/assets/img/2021-05-09-20-54-35.gif)