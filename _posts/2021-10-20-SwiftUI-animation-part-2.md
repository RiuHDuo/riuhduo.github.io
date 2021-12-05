---
layout: post
title: SwiftUI学习(6)-SwiftUI动画入门Part2
date: 2021-10-20 20:01:20 +0800
description: 介绍SwitfUI基本动画的使用。
img: swiftui_logo.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [SwiftUI]
---

>  **コニクマル**撰写
>
>  本文使用目前最新版本的xcode 13 + macOS Big sur + iOS15
>
>  在做过渡动效果最好使用模拟器或者真机进行测试, Xcode 的Previews 渲染的效果经常出错

## 过渡(Transition)

### 基本使用

过渡效果用于View出现或者消失的动画效果, 举个简单例子点击一个按钮出现一个圆, 代码如下

```swift
@State var isShow = false
var body: some View {
  VStack(alignment:.center, spacing: 50) {
      Button("Clicked ME") {
          self.isShow.toggle()
      }
      if isShow {
          Circle()
              .frame(width: 100, height: 100)
              .foregroundColor(Color.blue)
      }
  }
  .frame(maxWidth:.infinity, maxHeight: .infinity, alignment: .top)
}
```

运行下:

<img src="/assets/img/2021-10-20-21.54.32.gif" alt="2021-10-20-21.54.32" style="zoom:25%;" />

上面没有添加过渡效果, 给圆添加一个淡入淡出的效果, 修改代码：

```swift
...
Circle()
  .frame(width: 100, height: 100)
  .foregroundColor(Color.blue)
  .transition(.opacity)
...
```

并且给`isShow`变化的地方加上`withAnimation`，如果不加上`withAnimation`也是没有效果的, 修改如下

```swift
...
withAnimation(.easeIn(duration: 1)) {
    self.isShow.toggle()
}
...
```

运行效果:

<img src="/assets/img/2021-10-20-21.54.33.gif" alt="2021-10-20-21.54.32" style="zoom:50%;" />

### 非对称过渡(Asymmetric)

可以通过`asymmetric`来分别设置出现和消失的过渡效果, 修改下代码

```swift
...
Circle()
  .frame(width: 100, height: 100)
  .foregroundColor(Color.blue)
  .transition(.asymmetric(insertion: .slide, removal: .scale))
...
```

这样圆形出现的时候是一个滑动`slide`的效果，消失的时候又是一个缩放`scale`的效果:

<img src="/assets/img/2021-10-20-21.54.34.gif" alt="2021-10-20-21.54.32" style="zoom:50%;" />

### 合并过渡(Combines)

可以通过`combined(with:)`将多个过渡效果合并起来同时显示，比如做一个同时滑如和淡入淡出的过渡动画, 修改代码：

```swift
...
Circle()
  .frame(width: 100, height: 100)
  .foregroundColor(Color.blue)
  .transition(.slide.combined(with: .opacity))
...
```

运行效果:

<img src="/assets/img/2021-10-20-21.54.35.gif" alt="2021-10-20-21.54.35" style="zoom:50%;" />

## **MatchedGeometryEffect**

`MatchedGeometryEffect`类似Hero动画,是两个元素切换的时候形成过渡。

先做一个列表界面如下

```swift
struct FirstView: View{
    @Binding var isShowSecond: Bool
    @Binding var shardId: String
    var shared: Namespace.ID
    var body: some View{
        ScrollView{
            LazyVStack(alignment:.leading, spacing: 0) {
                ForEach(0 ..< 10){ i in
                    let id =  "shard\(i)"
                    HStack{
                        Image("Image")
                            .resizable()
                            .matchedGeometryEffect(id: id, in: shared)
                            .aspectRatio(contentMode: .fit)
                            .clipShape(Circle())
                            .frame(width: 50, height: 50)
                            .foregroundColor(Color.red)
                            .animation(.easeOut)
                        Text("星のカービィ")
                        Spacer()
                    }
                    .onTapGesture {
                        shardId = id
                        isShowSecond.toggle()
                    }
                    .padding()
                    Divider()
                }
            }
        }
    }
}
```

在做一个详情页面:

```swift
struct SecondView: View{
    var shared: Namespace.ID
    var sharedId: String
    @State var isShow = false
    var body: some View{
        VStack(alignment:.center, spacing: 50) {
            Image("Image")
                .resizable()
                .aspectRatio(contentMode: .fit)
                .clipShape(Circle())
                .matchedGeometryEffect(id: sharedId, in: shared)
                .frame(width: 200, height: 200)
                .foregroundColor(Color.green)
                .animation(.easeOut)
                .onAppear {
                    withAnimation(.easeOut.delay(0.5)){
                        isShow.toggle()
                    }
                }
            
            if isShow {
            Text("桜井政博が生みの親で、彼が開発に関わった作品を「桜井カービィ」と呼ぶこともある。第1作はゲームボーイ対応ソフトとして日本で1992年4月27日に発売し世界売上で500万本以上を記録。シリーズ累計販売本数は全世界で2016年時点で3800万本以上にも及ぶ[1]。漫画やアニメ、小説といったメディアミックス作品も多数製作されている。また、星のカービィのテーマカフェである「KIRBY CAFÉ」が2016年8月8日より順次、大阪、名古屋、東京、博多で期間限定店舗としてオープンし、2019年12月12日には東京ソラマチ4Fで常設店舗がオープンした。さらに、2020年12月3日より､阪急三番街のB1F　キデイランド大阪梅田店において､カービィのグッズ常設売り場となるKIRBY’S PUPUPU MARKET[2]がオープンした。")
                .animation(.easeOut)
                .padding()
            }
        }
        .frame(maxWidth:.infinity, maxHeight: .infinity, alignment: .top)
    }
}
```

两个界面的`Image`设置`matchedGeometryEffect`当切换时候`id`和Namespace.ID相同时就会触发类似Hero的过渡效果。

添加`ContentView`代码:

```swift
struct ContentView: View {
    @Namespace var shared
    @State var isShowSecond = false
    @State var sharedID = ""
    var body: some View {
        ZStack {
            if isShowSecond {
                SecondView(shared: shared, sharedId: sharedID)
            }else{
                FirstView(isShowSecond: $isShowSecond, shardId: $sharedID, shared: shared)
            }
        }
    }
}
```



运行效果如下:

<img src="/assets/img/2021-10-20-21.54.36.gif" alt="2021-10-20-21.54.35" style="zoom:50%;" />