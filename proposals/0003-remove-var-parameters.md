从函数参数中移除`var`
---

* 提议: [SE-0003](https://github.com/apple/swift-evolution/blob/master/proposals/0003-remove-var-parameters-patterns.md)
* 作者: [David Farler](https://github.com/bitjammer)
* 状态: **Acceppted**
* 审查: [Joe Pamer](https://github.com/jopamer)

## 注意

这个提议从开始到现在已经做了几次重大改变。可查看文档的结尾来了解其历史，以及为啥提议这样的改变。

## 引言

函数参数被标记成`inout`和`var`在语义上有点不一样。同样给定一个可变局部拷贝的值，被标记为`inout`的参数会自动回写。

函数参数磨人是不可变的：

```swift
func foo(i: Int) {
  i += 1 // 不合法
}

func foo(var i: Int) {
  i += 1 // 可以，但是调用者不能观察到这个变化。
}
```

这里局部拷贝`x`可变但不能回写到原来的值，所以调用者不能直接观察到这个变化。对于那些值类型，你得用`inout`来标记参数：

```swift
func doSomethingWithVar(var i: Int) {
  i = 2 // 这对调用者Int不会产生影响，但能局部修改。
}

func doSomethingWithInout(inout i: Int) {
  i = 2 // 这对调用者的Int产生影响。
}

var x = 1
print(x) // 1

doSomethingWithVar(x)
print(x) // 1

doSomethingWithInout(&x)
print(x) // 2
```

## 动机

用`var`标注函数参数，其作用有限。为了优化一行代码，而导致`inout`(满足了大部分人的期望)语义混淆。为了强调这些值是唯一拷贝的事实，并没有`inout`回写语义，那么我们不应该允许`var`。

出于下述问题，才有这次变化：

- 在函数参数中，`var`经常和`inout`混淆。
- `var`导致会认为值类型也有引用语义。
- 函数参数是恒等模式(not refuctable patterns)，如在*if-*, *while-*, *guard-*, *for-in-*, 和 *case* 语句中。

## 设计

对解析器来说，这是一个微不足道的变化。在Swift 2.2中，会有deprecation的警告，而在Swift 3中，会变成错误。

## 对已有代码的影响

为了移除这些`var`，使用了一个纯机械的迁移：引入一个临时变量，不可变的浅拷贝了那些值。比如：

```swift
func foo(i: Int) {
  var i = i
}
```

不过，浅拷贝不是最理想的修复方式，可能会引起反面模式(anti-pattern)。希望Swift用户能重新考虑下某些已经用了这些代码的地方，但是针对本次改动，不严格要求去做。

## 其它选择

[Original SE-0003 Proposal](https://github.com/apple/swift-evolution/blob/8cd734260bc60d6d49dbfb48de5632e63bf200cc/proposals/0003-remove-var-parameters-patterns.md) 包含了对所有非恒等模式（同样的在函数参数中），移除`var`绑定。

重新考虑了从非恒等模式中移除`var`，是因为在Swift2中将他放到可用的可变模式已经产生了负担。你可在swift-evolution邮件列表[Initial Discussion of Reconsideration](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160118/007326.html)中查看本次讨论。

最后的结论也发送到了swift-evolution列表，[Note on Revision of the Proposal](https://lists.swift.org/pipermail/swift-evolution/Week-of-Mon-20160125/008145.html)。