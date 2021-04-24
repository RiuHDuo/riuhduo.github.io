---
layout: post
title: SwiftUI学习(2)-布局管理(上) VStack/HStack/ZStack学习
date: 2021-04-18 22:01:20 +0800
description: SwiftUI VStack/HStack/ZStack学习
img: 1.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [SwiftUI]
---

# SwiftUI学习(2)-布局管理上

> By **コニクマル**
>
> 本文使用目前最新版本的xcode 12 + macOS Big sur + iOS14

### VStack 垂直布局

VStack是将其子元素垂直排列的布局。

```swift
VStack {
        RoundedRectangle(cornerRadius: 35)
            .foregroundColor(.blue)
            .frame(width: 150, height: 100)

        RoundedRectangle(cornerRadius: 35)
            .foregroundColor(.green)
            .frame(width: 200, height: 100)

        RoundedRectangle(cornerRadius: 35)
            .foregroundColor(.red)
            .frame(width: 100, height: 100)
    }
```

效果如下：

<img src="/assets/img/image-20210418221124470.png" alt="image-20210418221124470" style="zoom:50%;" />

通过设置VStack的参数`spacing`来设置元素的间距, 通过设置`alignment`来设置元素的对齐方式, **这个对齐方式是只元素之间的对齐方式, 比如3个矩形左边对齐，中心对齐，右边对齐。并不是在VStack内的位置。**

```swift
VStack(alignment:.trailing, spacing: 50) {
          RoundedRectangle(cornerRadius: 35)
              .foregroundColor(.blue)
              .frame(width: 150, height: 100)

          RoundedRectangle(cornerRadius: 35)
              .foregroundColor(.green)
              .frame(width: 200, height: 100)

          RoundedRectangle(cornerRadius: 35)
              .foregroundColor(.red)
              .frame(width: 100, height: 100)
      }
```

效果如下：

<img src="/assets/img/image-20210418221640585.png" alt="image-20210418221640585" style="zoom:50%;" />

如果没有设置VStack的`frame`,  VStack会根据元素的大小来计算自身的大小。

给上面例子中VStack设置个背景。

```swift
VStack(alignment:.trailing, spacing: 50) {
		...
}
.background(Color.orange)
```

可以看到VStack宽度等于最宽的矩形宽度, 高度等于3个矩形高度+`spacing`的大小。

<img src="/assets/img/image-20210418222153835.png" alt="image-20210418222153835" style="zoom:50%;" />

通过设置`frame`的maxWidth和maxHeight为infinity可以让VStack大小等于父节点的大小。

```swift
VStack(alignment:.trailing, spacing: 50) {
		...
}
.frame(maxWidth:.infinity, maxHeight: .infinity)
.background(Color.orange)
```

效果如下：

<img src="/assets/img/image-20210418222730463.png" alt="image-20210418222730463" style="zoom:50%;">

设置`frame`同时设置`alignment`属性来控制子元素在VStack中的位置。

```swift
VStack(alignment:.trailing, spacing: 50) {
		...
}
.frame(maxWidth:.infinity, maxHeight: .infinity, alignment: .bottomTrailing)
.background(Color.orange)
```

将3个矩形设置到右下角,效果如下：

<img src="/assets/img/image-20210418223526195.png" alt="image-20210418223526195" style="zoom:50%;" />

通过设置`edgesIgnoringSafeArea`可以是布局延伸到safe area区域。

```swift
VStack(alignment:.trailing, spacing: 50) {
		...
}
.frame(maxWidth:.infinity, maxHeight: .infinity, alignment: .bottomTrailing)
.background(Color.orange)
.edgesIgnoringSafeArea(.all)
```

这样就使得VStack填满屏幕了。

<img src="/assets/img/image-20210418224013719.png" alt="image-20210418224013719" style="zoom:50%;">

## HStack和ZStack

HStack和ZStack和VStack只是方向不同,HStack是水平排列, ZStack是在Z轴上堆叠排列, 其他用法和VStack基本一致。

## LazyVStack/LazyHStack

`LazyVStack`和`LazyHStack`是iOS14新引入的布局管理器, 通过名字可以看出是个Lazy类型的布局管理, 和`VStack`/`HStack`区别在于`VStack`/`HStack`会一次性渲染所有的子元素而`LazyVStack`和`LazyHStack`会在元素需要显示的时候在渲染元素。`LazyVStack`和`LazyHStack`渲染较多超出屏幕元素时候比`VStack`/`HStack`有着更好的性能,特别是配合scrollview做滚动列表的时候。