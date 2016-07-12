移除柯里化`func`声明语法
---

* 提议: [SE-0002](https://github.com/apple/swift-evolution/blob/master/proposals/0002-remove-currying.md)
* 作者: [Joe Groff](https://github.com/jckarter)
* 状态: **Acceppted**

## 引言

柯里化函数声明语法`func foo(x: Int)(y: Int)`作用有限，并且制造了大量语言和实现的复杂度。我们应该移除它。

## 动机

柯里化函数语法会造成连锁反应，导致语言的其他功能都很复杂：

- 柯里化造成了关键字规则和函数名声明的混淆。我们在柯里化参数是否作为函数的参数的延长部分(从一个新函数的参数列表开始)或者遵循不同的规则这个问题上，讨论了好几次。
- 在`var`和`inout`参数注解上有细微交互。在一个柯里化函数中，带有`inout`参数除了第一个分句不能部分应用[^1]外，其他没有任何语义限制，但是这限制了其可用性。对于`var`参数的问题是`var`被限制到什么程度；大量用户期望最外部分应用，而我们目前是最内部分应用。

现在标准库、Cocoa和大多数第三方的代码风格并没有真正使得ML[^2]风格参数的柯里化自由函数[^3]变得有用。在Cocoa和标准库中，大多数都是方法，这里仍可以通过`self.method`或者某天`.map{f($0)}`来一样获得有用的部分应用。柯里化函数的设计也早于关键字参数模型的设计。我们有计划的去掉 参数是一个元组(arguments-are-a-single-tuple)的模型(该模型被证明是错误的，如`autoclosure`和`inout`)，这也让我们越来越远离ML参数模型。

很多用户已经看到柯里化功能没啥用，要求选择使用Scala-style`f(_, 1)`自由部分应用。实际上即使是面向函数编程的用户也没在柯里化功能中看到太多的价值，这让我们感到没有柯里化可能更好。很明显它没有通过"would we add it if we didn't have it already"测试。

## 详细设计

我们移除了对多个参数模式声明的支持，函数签名的语法降低到只允许一个参数语句。考虑到代码迁移，已经使用柯里化声明语法的代码会被转成显示返回闭包：


```swift
  // Before:
  func curried(x: Int)(y: String) -> Float {
    return Float(x) + Float(y)!
  }

  // After:
  func curried(x: Int) -> (String) -> Float {
    return {(y: String) -> Float in
      return Float(x) + Float(y)!
    }
  }  
```

我不建议改变方法的语义，方法已经正式保持了函数类型`Self->Args->Return`。

## 对已有代码的影响

这是在移除一个语言功能，所以肯定会破坏已经使用了这个功能的代码。我们觉得柯里化很没有啥用，跟优秀的语言实践想矛盾。并且这里也可以有个自动合并的工具，所以为乐简化语言，这个影响是可以接受的。

## 其它选择

另外一个选择是保留当前的柯里化，但如上所述，这并不理想。虽然我没有提议采取任何马上的行动，但未来可选的设计是更符合语言习惯的方式来提供相似的功能：

- 类Scala的临时部分应用语法，如`foo(_, bar: 2)`是`{ x in foo(x, bar: 2)`的简写。可以说受益于面向关键字参数的API设计，这比传统的柯里化来的更容易读和灵活，也要求API设计者预先考虑参数顺序。
- 方法 and/or 操作符 切片语法。我们用`self.method`来部分绑定一个方法到他的`self`参数，并且可能潜在增加`.method(argument)`来部分绑定一个方法到他的非self参数。这对高阶方法（如`map`,`filter）`来说非常有用。类Haskell`(2+)`/`(+2)`语法对部分应用操作符可能也非常好。



---
译者备注：

1. [柯里化(currying)与部分应用(partial application)](http://www.jstips.co/zh_cn/curry-vs-partial-application/)
2. [partical application](https://en.wikipedia.org/wiki/Partial_application)
3. [函数加里化(Currying)和偏函数应用(Partial Application)的比较](http://www.vaikan.com/currying-partial-application/)
4. [函数式编程中局部应用（Partial Application）和局部套用（Currying）的区别](https://segmentfault.com/a/1190000000765247)

[^1]: 柯里化(currying)部分应用(partial application)的区别在于柯里化是f: X * Y -> R转变为f’: X -> (Y -> R)，而部分应用是f: X * Y -> R转变为f`: Y -> R。
[^2]: [ML语言](http://baike.baidu.com/view/1149599.htm)
[^3]: Free Functions, 相对于成员函数而言。[What is the meaning of the term “free function” in C++?](http://stackoverflow.com/questions/4861914/what-is-the-meaning-of-the-term-free-function-in-c)