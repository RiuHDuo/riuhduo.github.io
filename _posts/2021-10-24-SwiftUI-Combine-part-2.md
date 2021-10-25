---
layout: post

title: SwiftUI学习(7)-Combine入门Part II-Subscriber

date: 2021-10-24 10:01:20 +0800

description: Combine使用基础之Subscriber。

img: swiftui_logo.png # Add image post (optional)

fig-caption: # Add figcaption (optional)

tags: [SwiftUI]
---

>  **コニクマル**撰写
>
>  本文使用目前最新版本的xcode 13 + macOS Big sur + iOS15

## 订阅者-Subscriber 

`Subscriber`也是一个协议，协议包含了Input的数据类型，这个类型对应着`Publisher`的Output类型以及请求数据的数量。

### 实现Subscriber协议

先看看Subscriber协议

```swift
public protocol Subscriber : CustomCombineIdentifierConvertible {
    associatedtype Input
    associatedtype Failure : Error
  
    func receive(subscription: Subscription)

    func receive(_ input: Self.Input) -> Subscribers.Demand
    func receive(completion: Subscribers.Completion<Self.Failure>)
}
```

- CustomCombineIdentifierConvertible 这个协议用来实现对`Publisher`流的唯一ID，系统用已经扩展实现了。
- `Input`是输入类型，对应`Publisher`的`Output`类型
- `Failure`是错误类型，对应`Publisher`的`Failure`类型
- `  func receive(subscription: Subscription)`
  - 当订阅成功后调用,传递了订阅的订阅内容(Subscription)
  - 当`Subscriber`可以准备好开始接受数据了，可以调用`subscription`的`request(_:)`方法请求数据,`request(_:)`传递`Subscribers.Demand`作为参数。
    - `Subscribers.Demand.max` 表示请求最多几个
    - `Subscribers.Demand.none` 表示请求0个元素
    - `Subscribers.Demand.unlimited`表示请求无限个
- `func receive(_ input: Self.Input) -> Subscribers.Demand`
  - 当订阅者接收消息的时候调用
  - `input`就是接受到`Input`类型数据
  - 需要返回一个`Subscribers.Demand`表示接下来还要接收几个参数
- `func receive(completion: Subscribers.Completion<Self.Failure>)`
  - 当事件结束了调用

接下来自定义一个Subscriber：

```swift
class TestSubscriber: Subscriber{
    func receive(subscription: Subscription) {
        print("Received Subscription")
        subscription.request(.max(1))
    }
    
    func receive(_ input: Int) -> Subscribers.Demand {
        print("Received Value", input)
        return .unlimited
    }
    
    func receive(completion: Subscribers.Completion<Never>) {
        print("Received completion")
    }
}
```

简单运行下:

```swift
let sub = TestSubscriber()
(0 ..< 5).publisher.receive(subscriber: sub)
```

运行结果:

```swift
Received Subscription
Received Value 0
Received Value 1
Received Value 2
Received Value 3
Received Value 4
Received completion
```

然后把`    func receive(subscription: Subscription)`也就第一个函数修改下:

```swift
 subscription.request(.none)
```

再次运行:

```swift
Received Subscription
```

收不到数据也收不到完成。

在修改下代码:

```swift
...
 func receive(subscription: Subscription) {
        print("Received Subscription")
        subscription.request(.max(2))
    }
    
    func receive(_ input: Int) -> Subscribers.Demand {
        print("Received Value", input)
        return .none
    }
...
```

再次运行:

```swift
Received Subscription
Received Value 0
Received Value 1
```

再次修改代码

```swift
....
    func receive(_ input: Int) -> Subscribers.Demand {
        print("Received Value", input)
         return  input == 0 ? .max(2) : .none
    }
....
```

再次运行:

```swift
Received Subscription
Received Value 0
Received Value 1
Received Value 2
Received Value 3
```

通过两次修改可以发现接受的事件数量等于` subscription.request`的次数加上`func receive(_ input: Int) -> Subscribers.Demand`返回的次数和。

### sink

在`Part I-Publisher`中的很多例子都是用到`sink(receiveCompletion:receiveValue:)`,这个方法是通过`receiveCompletion`和`receiveValue`闭包创建订阅者来完成订阅。

- `receiveCompletion`当Publisher返回完成后调用
- `receiveValue`当接受到事件后调用.

`sink(receiveCompletion:receiveValue:)`一订阅成功就马上请求的无限个数据, **返回的Cancellable对象必须持有，否则Cancellable被释放，数据流就会被取消。**

如果不需要处理完成或者错误可以使用`sink(receiveValue: )`这个方法。

### assign

`assgin`会将接受到的数据分发(assign)给标记成发布者的属性(Property)进行重新发布。

`assgin`可以通过keypath或者inout把接受到的数据分配给属性(Property)。举个例子:

```swift
class Test{
    @Published var i: Int = 0
    @Published var ii: Int = 0
}

```

变量`i`和`ii`需要使用`@Published`属性修饰, `@Published`就标记这些属性是Published, 通过$来获取属性对应publisher。`@Published`修饰的属性在SwiftUI开发中比较常用, 是连接View和ViewModel的很好方式。

测试下上面的代码:

```swift

class Test{
    @Published var i: Int = 0
    @Published var ii: Int = 0
}

let test = Test()

Just(1).assign(to: \.i, on: test)
Just(3).assign(to: &test.$ii)
test.$i.sink(receiveValue: { print("i", $0)})
test.$ii.sink(receiveValue: { print("ii", $0)})

```

输出如下:

```swift
test.$i.sink(receiveValue: { print("i", $0)})
test.$ii.sink(receiveValue: { print("ii", $0)})	
```

