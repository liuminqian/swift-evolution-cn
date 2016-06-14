# 允许(大多数)关键字作为参数标签

* 提议: [SE-0001](https://github.com/apple/swift-evolution/blob/master/proposals/0001-keywords-as-argument-labels.md)
* 作者: [Doug Gregor](https://github.com/DougGregor)
* 状态: **Accepted** 并且[在Swift 2.2实现](https://github.com/apple/swift/commit/c8dd8d066132683aa32c2a5740b291d057937367) ([Bug](https://bugs.swift.org/browse/SR-344))

## 引言

参数标签作为Swift函数接口中重要的组成部分，不仅描述了在函数中对应参数的意义，而且也提高了可读性。对参数而言，有时候最自然的标签是一个语言的关键字，比如`in`, `repeat`, 或者`defer`。如果允许关键字作为参数标签，则可以更好地表达这样的接口。

## 动机

在某些函数中，对某个参数来说，最好的参数标签很可能就是语言的关键字。比如这样一个函数，找出一个collection中某个value的index。它的朴素命名应该是`indexOf(_:in:)`:

	indexOf(value, in: collection)

不过因为`in`是一个关键字，实际上不得不使用反引号来避免直接使用`in`，比如:

	indexOf(value, `in`: collection)

在Swift定义新API时，作者倾向选择非关键字(比如，这里的`within`)，甚至在这里是不理想的选择。不过，当在 忽略不必要字("omit needless words")的启发下引入Objective-C APIs时，这个问题也就出现了。比如：

	event.touchesMatching([.Began, .Moved], `in`: view)
	NSXPCInterface(`protocol`: SomeProtocolType.Protocol)

## 解决方案

除了`inout`, `var`, `let`之外所有的关键字都被允许作为参数标签。对语法有三个方面的影响：

* 调用表达式，如上面的例子。这里，我们没有语法歧义，因为在带括号的表达式中，"<keyword> \`:\`"不会出现在任何语法产生式中。这也是目前最重要的例子。

* 函数/下标/初始化器的声明：除了上面被排除的关键字之外，这里也没有任何歧义，因为关键字(`:`或者`_`)后面总是跟着一个标识符。如：

```swift
func touchedMatching(phase: NSTouchPhase, in view: NSView?) -> Set<NSTouch>
```

引入或者修改参数的关键字(当前只是"inout", "let", 和"var")，需要保留之前的含义。如果我们创作使用了这些关键字的API，他们仍是需要反引号的：

```swift
func addParameter(name: String, `inout`: Bool)
```

* 函数类型: 这个实际上比#2要简单很多，因为参数名后面总是跟着`:`：
```swift
(NSTouchPhase, in: NSView?) -> Set<NSTouch>
(String, inout: Bool) -> Void
```

## 对现有代码的影响

这个功能是严格可加(strictly additive)的，并且不会破坏现有代码：它只会让之前一些不规范的代码变规范，并不会改变任何规范代码的行为。

## 其他选择

最主要的其他选择就是啥也不做：Swift APIs将继续避免将关键字作为参数标签，即使其是标签最自然的词，并且现已引入的APIs继续使用反引号，或者需要重命名。这个选择将遗留近200个(现已引入的)APIs，这些APIs需要某种程度的重命名或者在调用点反引号。

另外一个选择是聚焦在`in`本身，这个在目前现已引入的APIs中最常见的。在现已引入的APIs的一次简单调研中发现`in`在关键字冲突中占据了90%的比例。此外，`in`是唯一一个在Swift语法中2个地方(循环和闭包)使用的关键字。因此它可以是上下文敏感的。不过，这个解决方案更为复杂了点(因为它需要更多上下文敏感的关键字解析)，也更缺乏通用性。