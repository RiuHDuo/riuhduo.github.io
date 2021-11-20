---
layout: post

title: SwiftUI学习(12)-Combine入门Part VI-Applying Sequence Operations

date: 2021-11-04 10:01:20 +0800

description: Combine使用基础之-Applying Sequence Operations

img: swiftui_logo.png # Add image post (optional)

fig-caption: # Add figcaption (optional)

tags: [SwiftUI]


---

> **コニクマル**撰写
>
> 本文使用目前最新版本的 xcode 13 + macOS Monterey + iOS15

### 序列操作(Applying Sequence Operations)

#### drop(untilOutputFrom:)

会一直舍弃`Input`发出的事件，直到参数传入的将另外一个`Publisher`发出事件后才会把`Input`发布的事件再发布出去。

流程图如下:

![image-20211120142425666](/assets/img/image-20211120141751768.png)

举个例子:

```swift
var store = Set<AnyCancellable>()
let input = PassthroughSubject<Int, Never>()
let publisher = PassthroughSubject<String, Never>()
let output = PassthroughSubject<Int, Never>()

input.drop(untilOutputFrom: publisher).subscribe(output).store(in: &store)

output.sink(receiveCompletion: { print("Completed ", $0)}, receiveValue: { print("Received Value ", $0)})
    .store(in: &store)


input.send(1)
input.send(2)
publisher.send("any")
input.send(3)
input.send(4)
```

输出:

```swift
Received Value  3
Received Value  4
```

#### dropFirst(_:)

传入一个Int参数n，表示舍弃`Input`发布的前n个事件，直到第n个事件后再发布出去。

流程图如下:

![image-20211120143737948](/assets/img/image-20211120143737948.png)

#### drop(while:)/tryDrop(while:)

传入一个闭包， 当闭包返回false的之前会把所有的`Input`的事件全部丢弃，闭包返回false后，就继续发布出去。**当返回false之后，这个闭包也不再回别调用了**。`tryDrop`可以在闭包里throw异常。

流程图如下:

![image-20211120144511242](/assets/img/image-20211120144511242.png)

#### append(_:)

append有三个重载版本:

- 1.传入一个动态参数列表比如`append(100, 200,300)`

- 2.传入一个数组比如`append([100, 200,300])`

这2个版本差不多只是参数形式不同, 当`Input`发布完成事件后将，将`append`内容插入到`finish`之前发布出去。

流程图如下:

![image-20211120152111952](/assets/img/image-20211120151538587.png)

- 3.传入一个`Publisher`

  当`Input`发布完成事件后将，将`Publisher`发布内容继续发布出去直到``Publisher` 发布 `finish`之后再发布`finish`事件。在`Input`发布的`finish`之前的`Publisher`发布的事件会被丢弃：

  流程图如下:

  ![image-20211120154733853](/assets/img/image-20211120152848440.png)

  举个例子:

  ```swift
  var store = Set<AnyCancellable>()
  let input = PassthroughSubject<Int, Never>()
  let output = PassthroughSubject<Int, Never>()
  let publisher = PassthroughSubject<Int, Never>()
  
  input.append(publisher).subscribe(output).store(in: &store)
  
  output.sink(receiveCompletion: { print("Completed ", $0)}, receiveValue: { print("Received Value ", $0)})
      .store(in: &store)
  
  
  input.send(1)
  input.send(2)
  input.send(3)
  input.send(4)
  publisher.send(100)
  input.send(completion: .finished)
  publisher.send(200)
  publisher.send(300)
  publisher.send(completion: .finished)
  ```

  输出:

  ```swift
  Received Value  1
  Received Value  2
  Received Value  3
  Received Value  4
  Received Value  200
  Received Value  300
  Completed  finished
  ```

  

#### prepend(_:)

和`append(_:)`正好相反，`Input`之前插入事件.

prepend有三个重载版本:

- 1.传入一个动态参数列表比如`prepend(100, 200,300)`

- 2.传入一个数组比如`prepend([100, 200,300])`

这2个版本差不多只是参数形式不同, 马上将`prepend`内容插入到第一个事件之前发布出去，无论`Input`有没有发布事件。

流程图:

![image-20211120154651912](/assets/img/image-20211120154244578.png)

- 3.传入一个`Publisher`, 先发布`Publisher`的事件，当`Publisher`发布`finish`后，才会发布`Input`的事件，**在此之前`Input`发布的事件会被舍弃**。

  流程图如下:

  ![image-20211120154534763](/assets/img/image-20211120154534763.png)

#### prefix(_:)

这个和`dropFirst`相反，传入一个Int参数n, 只会重新发布`Input`的前n个事件。之后的全部舍弃。当发布完第n元素后发布`finish`

流程图如下：

![image-20211120155245840](/assets/img/image-20211120155245840.png)

#### prefix(while:)/tryPrefix(while:)

和`drop(while:)/tryDrop(while:)`相反，传入一个闭包， 当闭包返回false的之前会把所有的`Input`的事件重新发布出去，闭包返回false发布`finish`。`tryPrefix`可以在闭包里throw异常。

流程图:

![image-20211120155942306](/assets/img/image-20211120155942306.png)

#### prefix(untilOutputFrom:)

和`drop(untilOutputFrom:)`相反, 传入一个参数`Publisher`，重新发布`Input`发布的事件，直到`Publisher`发布第一个事件后，发布`finish`。

流程如下:

![image-20211120160659545](/assets/img/image-20211120160659545.png)