---
layout: post

title: SwiftUI学习(7)-Combine入门Part III-Subject&AnyPublisher&Cancellable

date: 2021-10-25 10:01:20 +0800

description: Combine使用基础之Subject&AnyPublisher&Cancellable。

img: swiftui_logo.png # Add image post (optional)

fig-caption: # Add figcaption (optional)

tags: [SwiftUI]
---

> **コニクマル**撰写
>
> 本文使用目前最新版本的 xcode 13 + macOS Big sur + iOS15

## Subject(主题)

> 应该可以这样翻译吧?

`Subject`是一个集成`Publisher`的协议,比起`Publisher`, `Subject`提供了一些方法来发布事件。

`Subject`提供了几个 send 方法：

- send(\_ :Output): 用于发送一个事件给`Subscriber`
- send(subscription: Subscription): 发布订阅内容给发布者

Swift 提供了`PassthroughSubject`和`CurrentValueSubject`两个`Subject`使用。

- `PassthroughSubject`: 直接发布事件给`Subscriber`

  ```swift
  let passThrough = PassthroughSubject<Int, Never>()
  passThrough.sink(receiveCompletion: {print("PassThrough Completion ", $0)}, receiveValue: {print("PassThrough Broadcast ", $0)})
  passThrough.send(1)
  passThrough.send(2)
  passThrough.send(completion: .finished)
  ```

  输出如下:

  ```swift
  PassThrough Broadcast  1
  PassThrough Broadcast  2
  PassThrough Completion  finished
  ```

- `CurrentValueSubject`: 初始化一个`Output`类型的值并发布给`Subscriber`，每当发布一个事件给`Subscriber`，这个值就会随之更新。或者更新该值也会发布事件给`Subscriber`。

  ```swift
  let currentValue = CurrentValueSubject<Int, Never>(0)
  currentValue.sink(receiveCompletion: {print("PassThrough Completion ", $0)}, receiveValue: {print("PassThrough Broadcast ", $0)})
  currentValue.value = 1
  currentValue.send(2)
  print(currentValue.value)
  ```

  输出如下:

  ```swift
  PassThrough Broadcast  0
  PassThrough Broadcast  1
  PassThrough Broadcast  2
  2
  ```

## AnyPublisher

有些时候并不希望然后`Subscriber`了解发布者者的具体内容, 比如使用`PassthroughSubject`或者自定义的`Publisher`时候，并不想然后调用的地方看这个`Publisher`具体实现。这个时候可以使用 eraseToAnyPublisher 将 Publisher 转换为`AnyPublisher`来隐藏`Publihser`细节。

例如:

```swift
let passThrough = PassthroughSubject<Int, Never>()
let publisher = passThrough.eraseToAnyPublisher()
```

按 Optional 键并点击 publisher，可以看到 publisher 是一个 AnyPublisher<Int, Never>类型的对象。

## Cancellable

`Cancellable` 是一个协议而订阅内容(Subscription)继承了这个协议，在`Part II-Subscriber`文字里，当订阅成功后会调用`Subscriber`的`func receive(subscription: Subscription)`，这个 subscription 继承了`Cancellable`协议，调用`subscription.cancel()`就能订阅。

例如：

```swift
class TestSubscriber: Subscriber{
    private var subscription: Subscription? = nil

    func receive(subscription: Subscription) {
        print("Received Subscription")
        subscription.request(.max(1))
        self.subscription = subscription
    }

    func receive(_ input: Int) -> Subscribers.Demand {
        print("Received Value", input)
        return .unlimited
    }

    func receive(completion: Subscribers.Completion<Never>) {
        print("Received completion")
    }

    func cancel(){
        self.subscription?.cancel()
    }
}
let passThrough = PassthroughSubject<Int, Never>()
let subscriber = TestSubscriber()
passThrough.subscribe(subscriber)
passThrough.send(1)
passThrough.send(2)
subscriber.cancel()
passThrough.send(3)
passThrough.send(4)
```

输出:

```swift
Received Subscription
Received Value 1
Received Value 2
```

当 cancel 后`Subscriber`就无法收到数据了。

### AnyCancellable

`AnyCancellable`是一个 final class, 当 class 被释放了会调用改类的`cancel()`取消操作。

给上面自定义的 Subscriber 添加一个 AnyCancellable 的只读对象:

```swift
...
var cancellable: AnyCancellable{
  return AnyCancellable {
      print("Cancel")
      self.cancel()
  }
}
...
```

编写测试代码：

```swift
let passThrough = PassthroughSubject<Int, Never>()
func doIt(){
    let subscriber = TestSubscriber()
    passThrough.subscribe(subscriber)
    passThrough.send(1)
    passThrough.send(2)
    // 获取AnyCancellable
    let c = subscriber.cancellable
}
doIt()
passThrough.send(3)
passThrough.send(4)
```

运行结果:

```swift
Received Subscription
Received Value 1
Received Value 2
Cancel
```

在 doIt()函数运行完后, `AnyCancellable`类型的 c 变量自动释放了。
