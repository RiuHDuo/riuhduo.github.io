---
layout: post
title: SwiftUI学习(6)-SwiftUI动画入门Part3
date: 2021-12-04 20:01:20 +0800
description: Shap动画。
img: swiftui_logo.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [SwiftUI]
---

# SwiftUI动画入门Part 3-自定义Shape动画

![Kapture 2021-12-05 at 16.46.05](/assets/img/Kapture 2021-12-05 at 16.46.05.gif)

本文将介绍如何绘制上图中的正弦波动画。

> 关于自定义Shape，可以参考之前文章:
>
> [SwiftUI学习(5)-2D图形绘制](https://meatball.tech/SwiftUI-sharp/)

## 自定义正弦波Shape

正弦图形函数：y = sin(x * 波长)* 振幅, 定义波长等于 1/4的宽度， 振幅为1/3高度，原点(x: -波长，y: 1/2 高度)，`WaveShape`代码如下：

```swift
struct WaveShape: Shape{
    func path(in rect: CGRect) -> Path {
                var points = [CGPoint]()
                let h = rect.height / 2
                // 振幅
                let amplitude = rect.height / 3
                // 波长
                let wavelength = rect.width / 4
        
                for i in 0 ..< Int(rect.width) {
                    let x = CGFloat(i) - wavelength
                    let y = sin(2 * CGFloat.pi / wavelength * x) * amplitude + h
                    points.append(CGPoint(x: CGFloat(x), y: y))
                }
                return Path{ path in
                    path.move(to: points.first!)
                    points.forEach { p in
                        path.addLine(to: p)
                    }
                    path.addEllipse(in: CGRect(origin: points.last!.applying(CGAffineTransform(translationX: -5, y: -5)), size: CGSize(width: 10, height: 10)))
                }
    }
}
```

在view 中显示`WaveShape`并上颜色:

```swift
    var body: some View {
        WaveShape()
            .stroke(LinearGradient(colors: [Color(hue: 13 / 360, saturation: 0.79, brightness: 0.97), Color(hue: 40 / 360, saturation: 0.66, brightness: 0.93)], startPoint: UnitPoint(x: 0, y: 0.5), endPoint: UnitPoint(x: 1, y: 0.5)), lineWidth: 7)
      }
```

得到一个静态正弦波页面:

![image-20211205170714819](/assets/img/image-20211205170649846.png)

## 给正弦波添加动画

SwiftUI动画的背后是一个`Animatable`的协议, `Animatable`大概张这样:

```swift
protocol Animatable{
		associatedtype AnimatableData : VectorArithmetic
		var animatableData: AnimatableData
}
```

当使用View进行动画的时候, SwiftUI会多出生成改View，并给`animatableData`不同值。

`Shape`协议继承了`Animatable`, 通过实现`animatableData`就可以实现动画了。

在`WaveShape`添加一个`time`属性,并实现`Animatable`协议,将time属性作为动画参数:

```swift
...
var time: CGFloat

var animatableData: CGFloat{
    get{
        return time
    }
    set{
        time = newValue
    }
}
...
```

修改`path`函数,将绘制和`time`关联起来:

```swift
...
// 每个点y根据时间偏移
 let y = sin(2 * CGFloat.pi / wavelength * x +  wavelength * time) * a + h
...
```

在显示正弦波的View修改如下:

```swift
   @State var animation: Bool = false
    var body: some View {
        WaveShape(time: self.animation ? 1: 0)
            .stroke(LinearGradient(colors: [Color(hue: 13 / 360, saturation: 0.79, brightness: 0.97), Color(hue: 40 / 360, saturation: 0.66, brightness: 0.93)], startPoint: UnitPoint(x: 0, y: 0.5), endPoint: UnitPoint(x: 1, y: 0.5)), lineWidth: 7)
            .onAppear {
                withAnimation(.linear(duration: 60).repeatForever(autoreverses: false)) {
                    self.animation.toggle()
                }
            }
        
    }
```

运行效果就如一开始那样:

![Kapture 2021-12-05 at 16.46.05](assets/img/Kapture 2021-12-05 at 16.46.05.gif)

