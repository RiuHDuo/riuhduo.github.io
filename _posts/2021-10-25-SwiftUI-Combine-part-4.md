---
layout: post

title: SwiftUI学习(7)-Combine入门Part IV-Mapping&Filter Operators

date: 2021-10-25 10:01:20 +0800

description: Combine使用基础之-Mapping&Filter Operators。

img: swiftui_logo.png # Add image post (optional)

fig-caption: # Add figcaption (optional)

tags: [SwiftUI]
---

> **コニクマル**撰写
>
> 本文使用目前最新版本的 xcode 13 + macOS Monterey + iOS15

## Operator概述

Combine框架里， `Publisher`包含非常多的Operator。`Publisher`通过一些方法转换对其发出的事件元素进行操作这个就是`Operator`。`Publisher`发布数据，然后`Opeartor`加工数据并返回一个`新的Publisher`。下面会介绍下这些操作，已助于理解和使用。

## 元素映射(Mapping Elements)

#### map/tryMap

和Swift标准库里的`map` 操作类似，通过一个闭包函数将来把从pulisher传下来的元素进行转换。`tryMap`和`map`操作相同只不过可以在闭包里进行`throw`异常。

`Map`操作如下图所示, 将一个每一个元素转换成另外一个:

![image-20211030171142668](/Users/riuhduo/Documents/GitHub/riuhduo.github.io/assets/img/image-20211030171142668.png)

举个例子:

```swift
let input = PassthroughSubject<Int, Never>()
let output = PassthroughSubject<Int, Never>()
let sub = input.map { value in
    return value * 2
}.subscribe(output)
output.sink(receiveValue: { print("Output: ", $0)})

input.send(1)
input.send(2)
input.send(3)
```

- 创建一个PassthroughSubject作为Input
- Input进行map操作，将每个元素乘以2
- 将Input通过map转换后的publisher附加(attach)到output subject上
- 订阅通过`sink`来input和output
- input 发送数据

输出如下:

```swift
Output: 2
Output: 4
Output: 6
```

#### mapError

`mapError`和`map`类似，只不过`map`转换的是`publisher`发布事件元素,而`mapError`转换的是`publisher`是抛出的异常.

举个例子, 当收到数据3后抛出error1,转换error1成error2

```swift
enum TestError: Error{
    case error1
    case error2
}

let input = PassthroughSubject<Int, Never>()
let output = PassthroughSubject<Int, TestError>()
let sub = input.tryMap { value in
    if value == 3 {
        throw TestError.error1
    }
    return value * 2
}
.mapError({ _ in
    return TestError.error2
})
.subscribe(output)

output.sink(receiveCompletion: {print("Output Completed:", $0)}, receiveValue: {print("Output Value:", $0)})

input.send(1)
input.send(2)
input.send(3)
```

- 自定义`error1`和`error2`
- input进行tryMap操作，当value == 3抛出`error1`
- 通过`mapError`将error1转换成`error2`, 并附加到output上
- input 发送数据

输出:

```swift
Output Value: 2
Output Value: 4
Output Completed: failure(__lldb_expr_111.TestError.error2)
```

当input发送3的时候，output收到Test.error2错误.

#### replaceNil

替换`Publisher`发送下来的事件元素的nil数据。

`replaceNil`流程如下图:

![image-20211030165323997](/Users/riuhduo/Documents/GitHub/riuhduo.github.io/assets/img/image-20211030165323997.png)

举个例子:

```swift
let input = PassthroughSubject<Int?, Never>()
let output = PassthroughSubject<Int, Never>()

let v = input.replaceNil(with: 5).subscribe(output)

output.sink(receiveCompletion: {print("Output Completed:", $0)}, receiveValue: {print("Output Value:", $0)})

input.send(1)
input.send(nil)
input.send(3)

```

- input进行`replaceNil`操作将nil转换成5，并附加到Output
- input分别发1, nil, 3

输出:

```swift
Output Value: 1
Output Value: 5
Output Value: 3
```

output收到1， 5， 3

#### scan

`scan`有点类似Swift标准库里的`reduce`操作， 第一个参数是初始值， 第二参数是一个闭包.将`Publisher`发送下来的事件元素和上一次闭包中返回的值一起传入闭包中并返回新值。

`scan`的事件流程如下图:

![image-20211030164637049](/Users/riuhduo/Documents/GitHub/riuhduo.github.io/assets/img/image-20211030164637049.png)

举个例子:

```Swift

let input = PassthroughSubject<Int, Never>()
let output = PassthroughSubject<Int, Never>()

let sub = input.scan(0, { $0 + $1}).subscribe(output)

output.sink(receiveValue: {print("Output vaule:", $0)})

input.send(1)
input.send(2)
input.send(3)
```

- input进行scan操作
- input 分别发送 1， 2， 3

输出:

```swift
Output vaule: 1
Output vaule: 3
Output vaule: 6
```

ouptut收到: 1 , 3, 6

### 元素过滤(Filtering Elements)

#### filter/tryFilter:

`filter`和swift标准库里的`filter`差不多，传递一个闭包返回true表示接受元素，false表示过滤掉元素. `tryFilter`可以在闭包里抛出异常。

`filter`流程如下:

![image-20211030171202605](/Users/riuhduo/Documents/GitHub/riuhduo.github.io/assets/img/image-20211030171202605.png)

举个例子:

```swift
let input = PassthroughSubject<Int, Never>()
let output = PassthroughSubject<Int, Never>()


let sub = input.filter{$0 > 1}.subscribe(output)
output.sink(receiveValue: {print("Output vaule:", $0)})

input.send(1)
input.send(2)
input.send(3)
```

- 过滤掉小于等于1事件元素

输出:

```swift
Output vaule: 2
Output vaule: 3
```

output 只有2和3

#### compactMap/tryCompactMap

`compactMap`将从`Publisher`接受到的事件元素进行转换，如果返会nil的值会被过滤掉。`tryCompactMap`可以在闭包中抛出异常。

`compactMap`流程如下:

![image-20211030172605779](/Users/riuhduo/Library/Application Support/typora-user-images/image-20211030172605779.png)

举个例子:

```swift
let input = PassthroughSubject<String, Never>()
let output = PassthroughSubject<Int, Never>()


let sub = input.compactMap{Int($0)}.subscribe(output)

output.sink(receiveValue: {print("Output vaule:", $0)})

input.send("1")
input.send("a")
input.send("2")

```

- input输入字符串通过`compactMap`转换成为Int

输出:

```swift
Output vaule: 1
Output vaule: 2
```

output 输出1和2, Int("a")返会nil被`compactMap`过滤掉了

#### removeDuplicates/tryRemoveDuplicates

`removeDuplicates`会过滤掉和**上一次**接受事件元素相同的事件元素, 如果需要自定义比较方法可以传入闭包来进行比较。`tryRemoveDuplicates`可以在传入的比较闭包里抛出异常.

流程如下:

![image-20211030173630754](/Users/riuhduo/Documents/GitHub/riuhduo.github.io/assets/img/image-20211030173630754.png)

举个例子:

```swift
let input = PassthroughSubject<Int, Never>()
let output = PassthroughSubject<Int, Never>()


let sub = input.removeDuplicates().subscribe(output)

output.sink(receiveValue: {print("Output vaule:", $0)})

input.send(1)
input.send(2)
input.send(2)
input.send(3)
input.send(3)
input.send(1)
```

- input连续输入1，2， 2， 3，3，1
- `removeDuplicates`操作

输出:

```swift
Output vaule: 1
Output vaule: 2
Output vaule: 3
Output vaule: 1
```

output过滤掉连续相同的输入，第一个和最后一个1并不是连续输入的所以都没有被过滤.

#### relpaceEmpty

如果`Publisher`没有发布任何事件就`send(completion: .finished)`，将返回`relpaceEmpty`的参数内容。

比如:

```swift
let input = PassthroughSubject<Int, Never>()
let output = PassthroughSubject<Int, Never>()

let sub = input.replaceEmpty(with: 42).subscribe(output)
output.sink(receiveValue: {print("Output vaule:", $0)})

input.send(completion: .finished)

```

输出:

```swift
Output vaule: 42
```

`Publisehr`直接结束则，ouput收到的是`relpaceEmpty`替换后的值.

修改代码在finished之前添加一行发送个数据:

```swift
...
input.send(1)
input.send(completion: .finished)
```

输出:

```swift
Output vaule: 1
```

当`Publisher`有事件发出， `replaceEmpty`就不会进行替换了。

#### replaceError

`replaceError`可以将上游抛出的异常替换成一个传入的事件元素。`replaceError`是用正常的内容替换错误, `mapError`使用异常替换异常.

举个例子:

```swift
enum TestError: Error{
    case error
}

let input = PassthroughSubject<Int, TestError>()
let output = PassthroughSubject<Int, Never>()


let sub = input.tryMap({ value -> Int in
    guard value < 2 else{
        throw TestError.error
    }
    return value
})
.replaceError(with: 42)
.subscribe(output)


output.sink(receiveValue: {print("Output vaule:", $0)})

input.send(1)
input.send(2)
input.send(3)

```

- tryMap里判断接受到的事件元素大于等于2的时候抛出异常
- 用`replaceError`将异常替换成42

输出:

```swift
Output vaule: 1
Output vaule: 42
```

- 当input发出2的时候抛出异常，ouput收到替换后的值42
- 因为发出2的时候抛出了异常, 所以收不到上。