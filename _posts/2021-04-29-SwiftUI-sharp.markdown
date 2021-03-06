---
layout: post
title: SwiftUI学习(5)-2D图形绘制
date: 2021-04-28 20:01:20 +0800
description: 介绍SwitfUI基本2D图形使用和绘制自定义图形。
img: swiftui_logo.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [SwiftUI]
---

> **コニクマル**撰写
>
> 本文使用目前最新版本的xcode 12 + macOS Big sur + iOS14

`Shape`是一个协议，可以通过这个协议绘制2D图形。`Shape`通过`foregroundColor`装饰器(modifier)填充(fill)颜色和通过`stroke`包装器来上色.

## 内置基本图形

SwiftUI 内置了一些继承`Shape`协议的图形如`Circle`(圆形) `Rectenge`（矩形） `RoundRectenge`（圆角矩形） `Eclipese`（椭圆）`Capsule`（胶囊）等，可以根据需求来使用比如做一个圆角按钮,通过将图形设置到`View`的`clipShape`装饰器(modifier)来裁剪成为一个`Capsule`（胶囊）形状的圆角的按钮:

```swift
  Button(action: {}, label: {
      Text("一个按钮")
          .foregroundColor(.white)
  })
  .frame(width: 300, height: 30, alignment: .center)
  .background(Color.orange)
  .clipShape(Capsule())
```

效果如下:

![image-20210428212940204](/assets/img/image-20210428212940204.png)

根据不同使用场景使用不同内置图形来快速完成UI。

## 绘制图形

通过`Path`来绘制不同的图形，`Path`通过一个传入一个` (inout Path) -> ()`参数为Path的闭包(closure)，通过参数Path绘制的图形的路径。

用法和UIKit中的UIBizerPath差不多只是参数略有区别,简单画个几个图形:

```swift
VStack {
		// 直线
    Path{ path in
        path.move(to: CGPoint(x: 0, y: 0))
        path.addLine(to: CGPoint(x: 150, y: 150))
    }
    .stroke(Color.orange, lineWidth: 5)
    .padding()
		// 矩形
    Path{ path in
        path.move(to: CGPoint(x: 0, y: 0))
        path.addRect(CGRect(x: 50, y: 0, width: 100, height: 100))
    }
    .stroke(Color.red, lineWidth: 5)
    .padding()
		// 圆弧
    Path{ path in
        path.addArc(center: CGPoint(x: 50, y: 50), radius: 150, startAngle: Angle(radians: 0), endAngle: Angle(degrees: 60), clockwise: false)
    }
    .stroke(Color.blue, lineWidth: 5)
    .padding()
}
```

<img src="/assets/img/image-20210428214656826.png" alt="image-20210428214656826" style="zoom:50%;" />

`Path`也是一个`Shape`,可以通过`Path`来裁剪出自定义图形，比如裁剪出一个五角星的图片:

```swift
var body: some View {
    VStack {
        Image("Image")
            .resizable()
            .aspectRatio(contentMode: .fill)
            .frame(width: 200, height: 200, alignment: .center)
            .clipShape(Path(path))

    }
}


func path( path: inout Path){
    path.move(to: CGPoint(x: 68.23, y: 133.35))
    path.addLine(to: CGPoint(x: 46.56, y: 144.74))
    path.addLine(to: CGPoint(x: 46.63, y: 144.7))
    path.addCurve(to: CGPoint(x: 24.43, y: 137.91), control1: CGPoint(x: 38.62, y: 148.96), control2: CGPoint(x: 28.68, y: 145.92))
    path.addCurve(to: CGPoint(x: 22.71, y: 127.64), control1: CGPoint(x: 22.75, y: 134.77), control2: CGPoint(x: 22.15, y: 131.16))
    path.addLine(to: CGPoint(x: 26.88, y: 103.3))
    path.addLine(to: CGPoint(x: 26.87, y: 103.38))
    path.addCurve(to: CGPoint(x: 22.23, y: 88.67), control1: CGPoint(x: 27.81, y: 98.01), control2: CGPoint(x: 26.08, y: 92.52))
    path.addLine(to: CGPoint(x: 4.55, y: 71.43))
    path.addLine(to: CGPoint(x: 4.61, y: 71.49))
    path.addCurve(to: CGPoint(x: 4.2, y: 48.28), control1: CGPoint(x: -1.92, y: 65.19), control2: CGPoint(x: -2.1, y: 54.8))
    path.addCurve(to: CGPoint(x: 13.44, y: 43.47), control1: CGPoint(x: 6.68, y: 45.71), control2: CGPoint(x: 9.92, y: 44.02))
    path.addLine(to: CGPoint(x: 37.87, y: 39.91))
    path.addLine(to: CGPoint(x: 37.8, y: 39.92))
    path.addCurve(to: CGPoint(x: 50.35, y: 30.97), control1: CGPoint(x: 43.19, y: 39.17), control2: CGPoint(x: 47.88, y: 35.82))
    path.addLine(to: CGPoint(x: 61.28, y: 8.83))
    path.addLine(to: CGPoint(x: 61.24, y: 8.9))
    path.addCurve(to: CGPoint(x: 83.2, y: 1.34), control1: CGPoint(x: 65.22, y: 0.75), control2: CGPoint(x: 75.05, y: -2.64))
    path.addCurve(to: CGPoint(x: 90.63, y: 8.64), control1: CGPoint(x: 86.4, y: 2.9), control2: CGPoint(x: 89.01, y: 5.46))
    path.addLine(to: CGPoint(x: 101.56, y: 30.78))
    path.addCurve(to: CGPoint(x: 114.13, y: 39.91), control1: CGPoint(x: 103.99, y: 35.71), control2: CGPoint(x: 108.69, y: 39.13))
    path.addLine(to: CGPoint(x: 138.36, y: 43.43))
    path.addLine(to: CGPoint(x: 138.28, y: 43.42))
    path.addCurve(to: CGPoint(x: 152.25, y: 61.97), control1: CGPoint(x: 147.25, y: 44.69), control2: CGPoint(x: 153.51, y: 52.99))
    path.addCurve(to: CGPoint(x: 147.6, y: 71.29), control1: CGPoint(x: 151.75, y: 65.5), control2: CGPoint(x: 150.12, y: 68.77))
    path.addLine(to: CGPoint(x: 129.92, y: 88.52))
    path.addLine(to: CGPoint(x: 129.98, y: 88.47))
    path.addCurve(to: CGPoint(x: 125.09, y: 103.09), control1: CGPoint(x: 126.06, y: 92.25), control2: CGPoint(x: 124.23, y: 97.71))
    path.addLine(to: CGPoint(x: 129.26, y: 127.43))
    path.addLine(to: CGPoint(x: 129.25, y: 127.36))
    path.addCurve(to: CGPoint(x: 115.93, y: 146.38), control1: CGPoint(x: 130.82, y: 136.29), control2: CGPoint(x: 124.86, y: 144.8))
    path.addCurve(to: CGPoint(x: 105.63, y: 144.84), control1: CGPoint(x: 112.42, y: 147), control2: CGPoint(x: 108.8, y: 146.46))
    path.addLine(to: CGPoint(x: 83.77, y: 133.35))
    path.addLine(to: CGPoint(x: 83.84, y: 133.39))
    path.addCurve(to: CGPoint(x: 68.42, y: 133.25), control1: CGPoint(x: 79.03, y: 130.83), control2: CGPoint(x: 73.27, y: 130.78))
    path.addLine(to: CGPoint(x: 68.23, y: 133.35))
}
```

效果如下:

![image-20210428215403344](/assets/img/image-20210428215403344.png)

## 自定义图形

可以通过继承`Shape`来实现自定义形状。

`Shape`协议需要实现path方法:

```swift
func path(in rect: CGRect) -> Path
```

path通过返回一个`Path`来绘制图形和上面说到`Path`的用法类似。

可以将之前的五角星变成自定义`Shape`

```swift
struct Star: Shape {
    func path(in rect: CGRect) -> Path {
        var path = Path()
        path.move(to: CGPoint(x: 68.23, y: 133.35))
        path.addLine(to: CGPoint(x: 46.56, y: 144.74))
        path.addLine(to: CGPoint(x: 46.63, y: 144.7))
        path.addCurve(to: CGPoint(x: 24.43, y: 137.91), control1: CGPoint(x: 38.62, y: 148.96), control2: CGPoint(x: 28.68, y: 145.92))
        path.addCurve(to: CGPoint(x: 22.71, y: 127.64), control1: CGPoint(x: 22.75, y: 134.77), control2: CGPoint(x: 22.15, y: 131.16))
        path.addLine(to: CGPoint(x: 26.88, y: 103.3))
        path.addLine(to: CGPoint(x: 26.87, y: 103.38))
        path.addCurve(to: CGPoint(x: 22.23, y: 88.67), control1: CGPoint(x: 27.81, y: 98.01), control2: CGPoint(x: 26.08, y: 92.52))
        path.addLine(to: CGPoint(x: 4.55, y: 71.43))
        path.addLine(to: CGPoint(x: 4.61, y: 71.49))
        path.addCurve(to: CGPoint(x: 4.2, y: 48.28), control1: CGPoint(x: -1.92, y: 65.19), control2: CGPoint(x: -2.1, y: 54.8))
        path.addCurve(to: CGPoint(x: 13.44, y: 43.47), control1: CGPoint(x: 6.68, y: 45.71), control2: CGPoint(x: 9.92, y: 44.02))
        path.addLine(to: CGPoint(x: 37.87, y: 39.91))
        path.addLine(to: CGPoint(x: 37.8, y: 39.92))
        path.addCurve(to: CGPoint(x: 50.35, y: 30.97), control1: CGPoint(x: 43.19, y: 39.17), control2: CGPoint(x: 47.88, y: 35.82))
        path.addLine(to: CGPoint(x: 61.28, y: 8.83))
        path.addLine(to: CGPoint(x: 61.24, y: 8.9))
        path.addCurve(to: CGPoint(x: 83.2, y: 1.34), control1: CGPoint(x: 65.22, y: 0.75), control2: CGPoint(x: 75.05, y: -2.64))
        path.addCurve(to: CGPoint(x: 90.63, y: 8.64), control1: CGPoint(x: 86.4, y: 2.9), control2: CGPoint(x: 89.01, y: 5.46))
        path.addLine(to: CGPoint(x: 101.56, y: 30.78))
        path.addCurve(to: CGPoint(x: 114.13, y: 39.91), control1: CGPoint(x: 103.99, y: 35.71), control2: CGPoint(x: 108.69, y: 39.13))
        path.addLine(to: CGPoint(x: 138.36, y: 43.43))
        path.addLine(to: CGPoint(x: 138.28, y: 43.42))
        path.addCurve(to: CGPoint(x: 152.25, y: 61.97), control1: CGPoint(x: 147.25, y: 44.69), control2: CGPoint(x: 153.51, y: 52.99))
        path.addCurve(to: CGPoint(x: 147.6, y: 71.29), control1: CGPoint(x: 151.75, y: 65.5), control2: CGPoint(x: 150.12, y: 68.77))
        path.addLine(to: CGPoint(x: 129.92, y: 88.52))
        path.addLine(to: CGPoint(x: 129.98, y: 88.47))
        path.addCurve(to: CGPoint(x: 125.09, y: 103.09), control1: CGPoint(x: 126.06, y: 92.25), control2: CGPoint(x: 124.23, y: 97.71))
        path.addLine(to: CGPoint(x: 129.26, y: 127.43))
        path.addLine(to: CGPoint(x: 129.25, y: 127.36))
        path.addCurve(to: CGPoint(x: 115.93, y: 146.38), control1: CGPoint(x: 130.82, y: 136.29), control2: CGPoint(x: 124.86, y: 144.8))
        path.addCurve(to: CGPoint(x: 105.63, y: 144.84), control1: CGPoint(x: 112.42, y: 147), control2: CGPoint(x: 108.8, y: 146.46))
        path.addLine(to: CGPoint(x: 83.77, y: 133.35))
        path.addLine(to: CGPoint(x: 83.84, y: 133.39))
        path.addCurve(to: CGPoint(x: 68.42, y: 133.25), control1: CGPoint(x: 79.03, y: 130.83), control2: CGPoint(x: 73.27, y: 130.78))
        path.addLine(to: CGPoint(x: 68.23, y: 133.35))
        
        
        return path
    }
}
```

然后将修改代码

```swift
  Image("Image")
                .resizable()
                .aspectRatio(contentMode: .fill)
                .frame(width: 200, height: 200, alignment: .center)
                .clipShape(Star())
```

