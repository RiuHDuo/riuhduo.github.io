---
layout: post
title: SwiftUI学习(3)-布局管理(下) LazyVGrid LazyHGrid基础
date: 2021-04-20 18:01:20 +0800
description: SwiftUI LazyVGrid和LazyHGrid一个是垂直格子布局，一个是水平格子布局， iOS14新增的布局管理器。这2个布局管理器也是只会渲染需要显示的子元素。
img: 1.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [SwiftUI]
---

> **コニクマル**撰写
>
> 本文使用目前最新版本的xcode 12 + macOS Big sur + iOS14

## Grid布局

`LazyVGrid`和`LazyHGrid`一个是垂直格子布局，一个是水平格子布局， iOS14新增的布局管理器，和`LazyVStack`和`LazyHStack`一样都带有Lazy,表明这2个布局管理器也是只会渲染需要显示的子元素。

### LazyVGrid

LazyVGrid是通过指定列的数量然后来填充布局

```swift
  LazyVGrid(columns: [GridItem(.fixed(100)), GridItem(.fixed(100)), GridItem(.fixed(100))]){
      ForEach(0 ..< 30){ index in
          RoundedRectangle(cornerRadius: 5)
              .foregroundColor(Color(hue: 0.03 * Double(index) , saturation: 1, brightness: 1))
              .frame(height: 50)
              .overlay(Text("\(index)"))
      }
  }
  .padding()
```

上面代码通过指定3个宽度为100的列, 用ForEach来填充30个格子, ForEach以后内容在讨论，效果如下:

![image-20210420200818611](/assets/img/image-20210420200818611.png)

`LazyVGrid`填充的顺序是从上方一行一行填充的，填满一行在填满下一行， 从上图的显示的序号就就很容易看明白了。

通过设置LazyVGrid的`spacing`可以调整行间距。

```swift
LazyVGrid(columns: [GridItem(.fixed(100)), GridItem(.fixed(100)), GridItem(.fixed(100))], spacing: 30){
  ForEach(0 ..< 30){ index in
      RoundedRectangle(cornerRadius: 5)
          .foregroundColor(Color(hue: 0.03 * Double(index) , saturation: 1, brightness: 1))
          .frame(height: 50)
          .overlay(Text("\(index)"))
  }
}
.padding()
```

效果如下:

![image-20210420201950866](/assets/img/image-20210420201950866.png)

`LazyVGrid`通过设置PinnedViews类型, 可以让Header和Footer悬停在滚动时悬停在顶部和底部。通过`Section`来设置Section的header和footer

```swift
VStack {
  ScrollView {
      LazyVGrid(columns: [GridItem(.fixed(100)), GridItem(.fixed(100)), GridItem(.fixed(100))], pinnedViews: [.sectionHeaders, .sectionFooters]){
          ForEach(0 ..< 5){ index in
              Section(header: Text("Header \(index)")
                          .bold()
                          .font(.title)
                          .frame(maxWidth: .infinity, alignment: .leading)
                          .background(Color.white),
                      footer: Text("Footer \(index)")
                              .bold()
                              .font(.title)
                              .frame(maxWidth: .infinity, alignment: .leading)
                              .background(Color.white)
              ) {
                  ForEach(0 ..< 10){ idx in
                      RoundedRectangle(cornerRadius: 5)
                          .foregroundColor(Color(hue: 0.03 * Double(index * 10 + idx) , saturation: 1, brightness: 1))
                          .frame(height: 50)
                          .overlay(Text("\(index * 10 + idx)"))
                  }
              }
          }
      }
      .padding()
  }
}
.clipped()
```

点击预览模拟器上方的播放按钮进入动态预览可以对scrollview进行滚动,  效果和UITableView的header和footer悬停基本一致, 运行如下

<img src="/assets/img/2021-04-20 at 20.38.05.gif" alt="2021-04-20 at 20.38.05" style="zoom:50%;" />

`GridItem`通过设置`Size`来控制格子的宽度:

- `fixed`通过指定固定大小来确定格子宽度
- `flexible`弹性大小，会和其他flexible的格子分割剩余的空间，可以设置期望的最大和最小值。如果指定的最小值过大可能会超出屏幕
- `adaptive`自适应分布, 这个尺寸会提供一个或者多个格子。需要指定一个最小值, 也可以设置最大值，然后根据最小值在把自身占用的区域平分成若干个满足最小值最大值的的格子。可以理解`adaptive`为一个`flexible`大格子，获得空间后再把这个大格子平分成若干个满足设置定值的小格子。

通过下面代码，显示各个格子大小, `GeometryReader`是一个特殊View能获取一些尺寸坐标信息，稍后讨论：

```swift
LazyVGrid(columns: [GridItem(.flexible()), GridItem(.adaptive(minimum: 50), spacing: 0)]){
    ForEach(0 ..< 30){ idx in
        GeometryReader{r in
            RoundedRectangle(cornerRadius: 5)
                .foregroundColor(Color(hue: 0.01 * Double(idx) , saturation: 1, brightness: 1))
                .overlay(Text("\(r.size.width)"))
        }
        .frame(height: 50)

    }
}
.padding()
```

效果如下:

![image-20210420205538547](/assets/img/image-20210420205538547.png)

很容易看出左边flexble的格子等右边3个小格子的宽度之和。

`GridItem`通过设置`spacing`的值来控制格子之间的距离

```swift
LazyVGrid(columns: [GridItem(.flexible(), spacing: 10), GridItem(.flexible(), spacing: 100), GridItem(.flexible(), spacing: 50)]){
    ForEach(0 ..< 30){ idx in
        RoundedRectangle(cornerRadius: 5)
            .foregroundColor(Color(hue: 0.0333 * Double(idx) , saturation: 1, brightness: 1))
        .frame(height: 50)
    }
}
.padding()
```

运行效果如下:

![image-20210422195905397](/assets/img/image-20210422195905397.png)

从运行效果可以看出只会在格子的右侧添加设置的空间。

`LazyVGrid`每行的高度取决于最高的格子大小, `GridItem`的`Alignment`来控制格子中元素位于格子内的位置, 类似于`frame`的`Alignment`的用法。

```swift
LazyVGrid(columns: [GridItem(.flexible(), alignment: .topLeading), GridItem(.flexible(), alignment: .bottomLeading), GridItem(.flexible(),  alignment: .trailing)]){
    ForEach(0 ..< 30){ idx in
        RoundedRectangle(cornerRadius: 5)
            .foregroundColor(Color(hue: 0.0333 * Double(idx) , saturation: 1, brightness: 1))
            .frame(width: CGFloat((idx % 3) + 1) * 20, height: CGFloat((idx % 3) + 1) * 20)
    }
}
.padding()
```

运行效果如下：

![image-20210422200902008](/assets/img/image-20210422200902008.png)

### LazyHGrid

`LazyHGrid`和`LazyVGrid`用法相似`LazyHGrid`指定的是行数, 从上到下填充到指定行数后再填充第二列。

### 奇淫巧技

`LazyHGrid`和`LazyVGrid`都是固定格式大小, 如何实现不规则布局, 如下图

![image-20210422201643799](/assets/img/image-20210422201643799.png)

先看一段简单代码:

```swift
LazyVGrid(columns: [GridItem(.flexible()), GridItem(.flexible()), GridItem(.flexible()]){
    RoundedRectangle(cornerRadius: 5)
        .foregroundColor(Color(hue: 0.3 * Double(1) , saturation: 1, brightness: 1))
        .frame(width: 450, height: 100)
    RoundedRectangle(cornerRadius: 5)
        .foregroundColor(Color(hue: 0.3 * Double(2) , saturation: 1, brightness: 1))
        .frame(width: 200,height: 100)
    RoundedRectangle(cornerRadius: 5)
        .foregroundColor(Color(hue: 0.3 * Double(3) , saturation: 1, brightness: 1))
        .frame(width: 50,height: 100)
}
.padding()
```

定义一个3列的格子，放了3个矩形, 运行看下效果:

![image-20210422202326372](/assets/img/image-20210422202326372.png)

`LazyVGrid`并不会裁剪超出格子的View。

那么就可以定义一个2倍宽度的格子使其占满2个。再把下面一个格子用Color.clear来替代。

附上代码:

```swift
LazyVGrid(columns: [GridItem(.flexible()), GridItem(.flexible()), GridItem(.flexible())]){
      ForEach(0 ..< 11){ idx in
          GeometryReader { r in
              RoundedRectangle(cornerRadius: 5)
                  .foregroundColor(Color(hue: 0.1 * Double(idx) , saturation: 1, brightness: 1))
                  .frame(width: idx == 6 ? 2 * r.size.width  + 10 : r.size.width )
          }
          .frame(height: 100)
          if idx == 6 {
              Color.clear
          }
      }
  }
  .padding()
```