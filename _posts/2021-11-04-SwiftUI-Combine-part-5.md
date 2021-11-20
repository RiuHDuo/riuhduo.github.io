---
layout: post


title: SwiftUI学习(11)-Combine入门Part V-Reducing Elements&Applying Mathematical&Applying Macthing Criteria Operators

date: 2021-11-04 10:01:20 +0800

description: Combine使用基础之-元素减少&数学运算&规则匹配。

img: swiftui_logo.png # Add image post (optional)

fig-caption: # Add figcaption (optional)

tags: [SwiftUI]
---
> **コニクマル**撰写
>
> 本文使用目前最新版本的 xcode 13 + macOS Monterey + iOS15

## 元素减少(Reducing Elements)

#### collect

`collect`操作是将`Publisher`发布的事件元素收集起来成为一个数组再发布出去。

`collect`有三个重载版本:

##### 1.collect()

将`Publisher`发布的元素收集起来，直到收到`Publisher`发出finish，再发布出一个数组出去。流程图如下:

![image-20211104201157660](/assets/img/image-20211104201157660.png)

举个例子：

```swift

var store = Set<AnyCancellable>()

let input = PassthroughSubject<Int, Never>()
let output = PassthroughSubject<[Int], Never>()

input.collect().subscribe(output)

output.sink(receiveCompletion: { print("Completed", $0)}, receiveValue: { print("Received Value", $0)})
    .store(in: &store)

input.send(1)
input.send(2)
input.send(3)
input.send(completion: .finished)

```

- 将input通过collect()后给output
- input 分别发布 1， 2， 3和finish

输出

```swift
Received Value [1, 2, 3]
Completed finished
```

如果移除`input.send(completion: .finished)`，则没有输出。

自定义一个Error

```swift
enum TestError: Error{
    case error
}
```

将`input`和`output`的错误类型改成自定义`TextError`并把`input.send(completion: .finished)`改成`input.send(completion: .failure(TestError.error))`。

输出:

```swift
Completed  failure(__lldb_expr_4.TestError.error)
```

> `collect()`只会在收到finish的时候才发送事件元素。

##### 2.collect(Int)

第一个参数是Int类型，当收到元素数量等于传入参数或者`Publisher`发出finish, 将这些元素组成一个数组发布出去。。流程图如下:

![image-20211104202927831](/assets/img/image-20211104202927831.png)

举个例子:

```swift
let input = PassthroughSubject<Int, Never>()
let output = PassthroughSubject<[Int], Never>()

input.collect(3).subscribe(output)


output.sink(receiveCompletion: { print("Completed ", $0)}, receiveValue: { print("Received Value ", $0)})
    .store(in: &store)

input.send(1)
input.send(2)
input.send(3)
input.send(4)
input.send(5)
input.send(6)
input.send(7)
input.send(completion: .finished)	
```

输出:

```swift
Received Value  [1, 2, 3]
Received Value  [4, 5, 6]
Received Value  [7]
Completed  finished
```

如果把`input`和`output`错误类型改成上文TestError并且将`input.send(completion: .finished)	`改成`input.send(completion: .failure(TestError.error))`。

输出:

```swift
Received Value  [1, 2, 3]
Received Value  [4, 5, 6]
Completed  failure(__lldb_expr_12.TestError.error)
```

> collect收到finish后会将没有达到数量的元素组成数组发布出去，收到failure则不会。

##### 3.collect(_:options:)

第一个参数是个枚举类型:

- `byTime(Scheduler, Scheduler.SchedulerTimeType.Stride)`

  表示每间隔一定时间, 将此期间收到的事件元素组成一个数组发布出去。

  - 第一个参数：执行的线程,`Scheduler`是一个协议表示执行的线程。
  - 第二个参数: 这个就是间隔。

  流程图:

  ![image-20211104210449202](/assets/img/image-20211104210449202.png)

  使用DispatchQueue.main.asyncAfter来延时发送数据, 如下:

  ```swift
  
  var store = Set<AnyCancellable>()
  let input = PassthroughSubject<Int, Never>()
  let output = PassthroughSubject<[Int], Never>()
  
  input.collect(.byTime(RunLoop.main, .seconds(1))).subscribe(output).store(in: &store)
  
  
  DispatchQueue.main.asyncAfter(deadline: DispatchTime.now()) {
      input.send(1)
  }
  
  DispatchQueue.main.asyncAfter(deadline: DispatchTime.now() + 0.5) {
      input.send(2)
  }
  
  DispatchQueue.main.asyncAfter(deadline: DispatchTime.now() + 0.8) {
      input.send(3)
  }
  
  DispatchQueue.main.asyncAfter(deadline: DispatchTime.now() + 1.1) {
      input.send(4)
  }
  
  DispatchQueue.main.asyncAfter(deadline: DispatchTime.now() + 1.5) {
      input.send(5)
  }
  
  output.sink(receiveCompletion: { print("Completed ", $0)}, receiveValue: { print("Received Value ", $0)})
      .store(in: &store)
  ```

  输出:

  ```swift
  Received Value  [1, 2, 3]
  Received Value  [4, 5]
  ```

- `byTimeOrCount(Scheduler, Scheduler.SchedulerTimeType.Stride, Int)`

  表示每间隔一定时间或者收到的事件元素数量达到一定数量后, 将此期间收到的事件元素或者达到数量的所有元素组成一个数组发布出去。

  - 第一个参数:执行的线程,`Scheduler`是一个协议表示执行的线程。
  - 第二个参数: 这个就是间隔。
  - 第三个参数: 达到的元素数量。 

   流程如下图:

  ![image-20211104211223460](/assets/img/image-20211104211223460.png)

使用DispatchQueue.main.asyncAfter来延时发送数据, 如下:

```swift
var store = Set<AnyCancellable>()
let input = PassthroughSubject<Int, Never>()
let output = PassthroughSubject<[Int], Never>()

input.collect(.byTimeOrCount(RunLoop.main, .seconds(1), 3)).subscribe(output).store(in: &store)


DispatchQueue.main.asyncAfter(deadline: DispatchTime.now()) {
    input.send(1)
}

DispatchQueue.main.asyncAfter(deadline: DispatchTime.now() + 0.5) {
    input.send(2)
    input.send(3)
    input.send(4)
}

DispatchQueue.main.asyncAfter(deadline: DispatchTime.now() + 0.8) {
    input.send(5)
}

DispatchQueue.main.asyncAfter(deadline: DispatchTime.now() + 1.1) {
    input.send(6)
}

DispatchQueue.main.asyncAfter(deadline: DispatchTime.now() + 1.5) {
    input.send(7)
}

output.sink(receiveCompletion: { print("Completed ", $0)}, receiveValue: { print("Received Value ", $0)})
    .store(in: &store)

```

输出:

```swift
Received Value  [1, 2, 3]
Received Value  [4, 5]
Received Value  [6, 7]
```

当时间到或者数量到的时候抛出数组。

第二参数是个Options, 根据传入的Scheduler来定义的参数, 有些Scheduler事不存在options的比如Runloop就没有。

#### IgnoreOutput

这个就是忽略所有收到的数据,当收到finish或者failure后发出事件。

##### reduce

将`Publisher`发送下来的事件元素和上一次闭包中返回的值一起传入闭包中，直到收到finish事件后将计算结果发布出去。和`scan`的操作差不多, 区别是`scan`每次收到事件元素后通过闭包处理的完的新值会发布出去,而且`reduce`只会在收到finish后把最后计算完成的的值发布出去。

流程如下:

![image-20211104212724693](/assets/img/image-20211104212724693.png)

举个例子:

```swift

var store = Set<AnyCancellable>()
let input = PassthroughSubject<Int, TestError>()
let output = PassthroughSubject<Int, TestError>()

input.reduce(0, {$0 + $1}).subscribe(output).store(in: &store)

output.sink(receiveCompletion: { print("Completed ", $0)}, receiveValue: { print("Received Value ", $0)})
    .store(in: &store)

input.send(1)
input.send(2)
input.send(3)
input.send(completion: .finished)
```

输出:

```swift
Received Value  6
Completed  finished
```

## 数学运算(Applying Mathematical)

### count

统计`Publisher`发布的事件元素数量，当收到finish事件后发布收到的事件元素数量。

比如统计收到的元素数量：

```swift
var store = Set<AnyCancellable>()

let input = PassthroughSubject<Int, TestError>()
let output = PassthroughSubject<Int, TestError>()

input.count().subscribe(output).store(in: &store)
output.sink(receiveCompletion: {print("Complete", $0)}, receiveValue: {print("Received Value", $0)}).store(in: &store)

input.send(1)
input.send(2)
input.send(3)
input.send(4)
input.send(5)
input.send(completion: .finished)
```

输出:

```Swift
Received Value 5
Complete finished
```

### max/min

统计`Publisher`发布的事件元素中的最大/小值，当收到finish事件后发布收到的事件的最大/小值。

> max/min无参数版本只能支持继承 `Comparable`协议的`Output`类型

如果`Output`数据类型没有继承`Comparable`可以使用带有闭包的版本，通过闭包比较元素的大小。

比如统计`Input`发送元素的最小值:

```swift

var store = Set<AnyCancellable>()

let input = PassthroughSubject<Int, TestError>()
let output = PassthroughSubject<Int, TestError>()

input.min().subscribe(output).store(in: &store)
output.sink(receiveCompletion: {print("Complete", $0)}, receiveValue: {print("Received Value", $0)}).store(in: &store)

input.send(1)
input.send(2)
input.send(3)
input.send(4)
input.send(5)
input.send(completion: .finished)
```

输出:

```swift
Received Value 1
Complete finished
```

### tryMax/Min(by: (Output, Output) throws ->  Bool)

和`max/min(by: (Output, Output)->Bool)`一样，只不过闭包里可以跑出错误。

## 规则匹配(Applying Matching Criteria)

### contains/tryContains

判断`Publisher`发布的事件元素中是否包含指定的元素或者规则，当收到finish事件后发布true/false表示是否包含。

> 无参数版本只能支持继承 ``Equatable`协议的`Output`类型

如果`Output`数据类型没有继承`Equatable`可以使用带有闭包的版本，通过闭包来判断元素是否符合规则。tryContains可以在传入的闭包里添加throw异常。

比如判断是否有大于3的元素:

```swift
var store = Set<AnyCancellable>()

let input = PassthroughSubject<Int, TestError>()
let output = PassthroughSubject<Bool, TestError>()

input.contains(where: {$0 > 3}).subscribe(output).store(in: &store)
output.sink(receiveCompletion: {print("Complete", $0)}, receiveValue: {print("Received Value", $0)}).store(in: &store)

input.send(1)
input.send(2)
input.send(3)
input.send(4)
input.send(5)
input.send(completion: .finished)
```

输出:

```Swift
Received Value true
Complete finished
```

### allSatisfy/tryAllSatisfy

判读`Publisher`发出的元素是不是全部满足传入闭包的里的规则，如果当收到finish后,之前闭包返回的值全是true则发送出true。`tryAllSatisfy`可以在闭包里跑出异常。

比如判读发送的数据是不是都是2的倍数：

```swift
var store = Set<AnyCancellable>()

let input = PassthroughSubject<Int, TestError>()

let output = PassthroughSubject<Bool, TestError>()

input.allSatisfy({$0 % 2  == 0}).subscribe(output).store(in: &store)
output.sink(receiveCompletion: {print("Complete", $0)}, receiveValue: {print("Received Value", $0)}).store(in: &store)

input.send(2)
input.send(4)
input.send(8)
input.send(6)
input.send(10)
input.send(completion: .finished)
```

输出:

```swift
Received Value true
Complete finished
```

