---
layout: post
title: SwiftUI学习(7)-Combine入门Part I-Publisher
date: 2021-10-20 20:01:20 +0800
description: Combine使用基础之Publisher。
img: swiftui_logo.png # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [SwiftUI]
---

>  **コニクマル**撰写
>
>  本文使用目前最新版本的xcode 13 + macOS Big sur + iOS15

# 简介

用[苹果自己话](https://developer.apple.com/documentation/combine/receiving-and-handling-events-with-combine)说:*`Combine`提供了APP处理事件的响应式声明框架。相较于之前的继承`Delegate`回调或者使用`完成闭包`参数,你可以创建一个从事件源头开的是处理链。该处理链每一个环节都是一个`Combine运算`，每个运算都会对上一个运算传递出来的内容进行特定处理。*

听起来很抽象, 举个不严谨的例子， 使用URLSession请求网络解析JSON数据

- 之前的做法:

  ```swift
  let task = URLSession.shared.dataTask(with: URL(string:"https://www.url.com")!) { data, resp, error in
      guard error == nil else{
          // 处理错误
          return
      }
      guard data != nil else{
          // 处理错误
          return
      }
      
      let decoder = JSONDecoder()
      if let json = try? decoder.decode(JsonTarget.self, from: data!){
      	self.response = json
      }else{
        self.response = Json(code: -1)
      }
  }
  task.resume()
  ```

- `Combine`的写法:

  ```swift
  let publisher =  URLSession.shared.dataTaskPublisher(for:URL(string:"https://www.url.com")!)
  let cancellable =
      .map({return $0.data})
      .decode(type: JsonTarget.self, decoder: JSONDecoder())
      .replaceError(with: JsonTarget(code: -1))
      .assign(to: \.resposne, on: self)
  ```
  
  - `URLSession.shared.dataTaskPublisher`返回的是发布者(Publisher)
  - `map` `decode`和`replaceError`是操作(Operator)
  - `assign`是订阅者(Subscriber)
  - `cancellable`是一个`Cancellable`可以用其`cancel()`方法来取消相关操作的。
  
  `Combine`的流程如下:
  
  ![image-20211022152214994](/Users/riuhduo/Documents/GitHub/riuhduo.github.io/assets/img/image-20211022152214994.png)
  
  - 首先订阅者请求订阅(Subscriber)
  - 然后发布者返回订阅内容(Subscription)
  - 再次订阅者请求数据
  - 发布者不断发布数据给订阅者
  - 发布者发送完成事件，结束整个流程

## 发布者(Publisher)

`Publisher`是`Combine`框架的一个很重要的的协议。`Publisher`是发布事件给一个或者多个订阅者。好比使用`NotificationCenter`编程时调用`NotificationCenter.default.post`来发布事件，这个发布事件的对象就相当于一个`Publisher`, 只不过不是使用`Combine`框架而已。

一个`Publisher`可以传递事件给一个或者多个订阅者(Subscriber)实例。 `Publisher`包含了`Output`类型和`Failure`错误类型。可以通过操作(Operator)来转换这个`Output`类型和`Failure`类型，订阅者(Subscriber)的`Intput`和`Failure`类型要符合`Publisher`的`Output`类型和`Failure`错误类型或者经过操作(Operator)转换后的`Output`类型和`Failure`错误类型才可以订阅。

### Publisher协议

先看看`Publisher`协议：

```swift
public protocol Publisher {
    associatedtype Output

    associatedtype Failure : Error
    
    func receive<S>(subscriber: S) where S : Subscriber, Self.Failure == S.Failure, Self.Output == S.Input
}

```

协议包含`Output`类型和错误`Failure`类型，以及一个接受订阅者的方法。简单实现一个自定义的`Publisher`,如下:

```swift
class TestPublisher: Publisher{
    func receive<S>(subscriber: S) where S : Subscriber, Never == S.Failure, Int == S.Input {
        subscriber.receive(1)
        DispatchQueue.global().asyncAfter(deadline: DispatchTime.now() + 2){
            subscriber.receive(2)
            subscriber.receive(completion: .finished)
        }
    }
    
    typealias Output = Int
    
    typealias Failure = Never
}


let  testPublisher = TestPublisher()
testPublisher.sink(receiveCompletion: { print("completed", $0)}, receiveValue: {print("received value", $0)})

```

输出如下:

```sh
received value 1
received value 2
completed finished
```

在`subscriber.receive(completion: .finished)`之后加一行`subscriber.receive(3)`,修改后:

```swift
...
DispatchQueue.global().asyncAfter(deadline: DispatchTime.now() + 2){
    subscriber.receive(2)
    subscriber.receive(completion: .finished)
    subscriber.receive(3)
}
...
```

运行下:

```sh
received value 1
received value 2
completed finished
```

可以看到当`subscriber.receive(completion:)`调用完了后,流程已经结束了，之后在发送事件，订阅者无法收到的事件的。

### 便捷发布者(Convenience Publishers)

> Swift内置了一些便捷的Publisher, 方便快速使用。

#### Future

从名字看就是未来某个时刻会发出一个事件然后就结束。

举个栗子，在10秒后发出一个1：

```swift
let future = Future<Int, Never>{ promise in
    DispatchQueue.global().asyncAfter(deadline: DispatchTime.now() + 10) {
        promise(.success(1))
    }
}
future.sink(receiveCompletion: { print("Received value", $0)}, receiveValue: { print("Receive value:", $0)}).store(in: &store)
```

运行10s后看到输出:

```sh
Receive value: 1
Received value finished
```

`Future<Int, Never>`: Int表示`Publisher`的Output类型, `Never`是`Failure`类型，`Never`表示不会发生错误。构造函数里的promise参数用来成功或者失败发送事件。

#### Just

这是一个马上发出事件的`Publisher`。

```swift
var store = Set<AnyCancellable>()
Just(1).sink(receiveCompletion: { print("Received value", $0)}, receiveValue: { print("Receive value:", $0)}).store(in: &store)
```

运行后马上看到输出:

```sh
Receive value: 1
Received value finished
```

#### Deferred

这个是延时创建`Publisher`,当被订阅的时候才会创建`Publisher`。

举个例子,  5s后发布一个:

```swift
let date = Date()
let publisher = Deferred { () -> Just<Int> in
    print("Create Pulisher", 0 - date.timeIntervalSinceNow)
    return Just(1)
}

DispatchQueue.global().asyncAfter(deadline: DispatchTime.now() + 5){
    print("Subscribe", 0 - date.timeIntervalSinceNow)
    publisher.sink(receiveCompletion: { print("completed", $0, 0 - date.timeIntervalSinceNow)}, receiveValue: {print("received value", $0, 0 - date.timeIntervalSinceNow)})
}

print("Run", 0 - date.timeIntervalSinceNow)
```

输出如下:

```swift
Run 0.0022840499877929688
Subscribe 5.497779011726379
Create Pulisher 5.497919082641602
received value 1 5.4987300634384155
completed finished 5.49881899356842
```

可以看到当调用`sink`后才创建`Just Publisher`。

#### Empty

`Empty`不会发送任何内容，当订阅后立刻发送complete事件。

#### Fail

`Fail`不会发送任何内容，当订阅后立刻发送指定Error。

#### Record

`Record`可以先用`Record.Recording`来记录一系列事件，稍后在通过`Record`发送给订阅者。

举个例子

```swift
var recording = Record<Int, Never>.Recording()
DispatchQueue.global().asyncAfter(deadline: DispatchTime.now() + 1) {
    let publisher = Record<Int, Never>(recording: recording)
    publisher.sink(receiveCompletion: { print("Received completed", $0)}, receiveValue: { print("Received Value", $0)})
}
DispatchQueue.global().asyncAfter(deadline: DispatchTime.now() + 2) {
    let publisher2 = Record<Int, Never>(recording: recording)
    publisher2.sink(receiveCompletion: { print("Received 2 completed", $0)}, receiveValue: { print("Received 2 Value", $0)})
}
DispatchQueue.global().asyncAfter(deadline: DispatchTime.now() + 3) {
    let publisher3 = Record<Int, Never>(recording: recording)
    publisher3.sink(receiveCompletion: { print("Received 3 completed", $0)}, receiveValue: { print("Received 3 Value", $0)})
}


recording.receive(1)
recording.receive(2)
recording.receive(completion: .finished)
```

共用一个Recording记录了发送1，2和finish事件。创建3个Record来发布Recording的内容。
