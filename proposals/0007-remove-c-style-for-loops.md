# 用conditions 和 incrementers 替换C风格 for-loop

* 提议: [SE-0007](https://github.com/apple/swift-evolution/blob/master/proposals/0007-remove-c-style-for-loops.md)
* 作者: [Erica Sadun](https://github.com/erica)
* 状态: **Accepted** for Swift 3.0 ([Rationale](http://thread.gmane.org/gmane.comp.lang.swift.evolution/11148/), [Swift 2.2 bug](https://bugs.swift.org/browse/SR-226), [Swift 3.0 bug](https://bugs.swift.org/browse/SR-227))
* 审查: [Doug Gregor](https://github.com/DougGregor)

## 引言

C风格`for-loop`的出现是机械地从C迁移过来的，而不是真正Swift构造的。很少用到，并且很不像Swift。

更多Swift类型的构造器已经在`for-in`声明和`stride`中可用。移除循环可以简化语言，并且减少`--`和`++`(这已经从语言中移除了)的大部分使用场景。

这个构造器的价值有限，并且我相信应该认真考虑移除他。

提议在 Swift Evolution 列表 [C-style For Loops](http://news.gmane.org/find-root.php?message_id=40642775%2dF58D%2d49B0%2d9BF3%2d38913FD6924C%40ericasadun.com) 帖子讨论，并在 [\[Review\] Remove C-style for-loops with conditions	and incrementers](http://thread.gmane.org/gmane.comp.lang.swift.evolution/9744/focus=10621) 帖子审查。

## For Loops的优势

Swift的设计支持使用熟悉的变量和控制结构来完成浅学习曲线。`for-loop`模仿了C，并降低了精通这个控制流所需要的努力。

## For Loops的劣势

1. `for-in` 和 `stride` 提供了和使用Swift方法（并且没有历史术语的包袱）一样的行为。
2. `for-loops`相比`for-in`在简洁性上有很大的表达劣势。
3. `for-loop` 的实现没有用到集合或者其它Swift的核心类型。
4. `for-loop` 鼓励使用一元的`--`和`++`，这个在语言中被移除掉了。
5. 分号来界定声明提高了那些从非C系语言过来的用户的学习难度。
6. 如果不存在`for-loop`, 我敢打赌会在Swift 3里考虑。

## 解决方案

我建议在Swift 2.x deprecate for-loop，然后在Swift 3完全移除。并且从Swift编程语言中移除以来匹配当前2.2的更新。

## 另外的选择

不移除`for-loop`，失去提高语言效率以及放弃不需要的控制流的机会。

## 对已有代码的影响


对Apple Swift代码库进行搜索，发现这个功能是很少使用的。Swift-Evolution邮件列表的社区成员也确认在很多pro级别的apps很少使用，并可以抛弃`for-loop`来解决问题。如：

```swift
char *blk_xor(char *dst, const char *src, size_t len)
{
 const char *sp = src;
 for (char *dp = dst; sp - src < len; sp++, dp++)
   *dp ^= *sp;
 return dst;
}
```

相比


```swift
func blk_xor(dst: UnsafeMutablePointer<CChar>, src:
UnsafePointer<CChar>, len: Int) -> UnsafeMutablePointer<CChar> {
   for i in 0..<len {
       dst[i] ^= src[i]
   }
   return dst
}
```

搜索github上的Swift gists也说明这个方法大多是语言新手在使用，精通者往往会抛弃这种写法。如：

```swift
for var i = 0 ; i < 10 ; i++ {
    print(i)
}
```
和

```swift
var array = [10,20,30,40,50]
for(var i=0 ; i < array.count ;i++){
    println("array[i] \(array[i])")
}
```

## 社区反馈

* "I am certainly open to considering dropping the C-style for loop.  IMO, it is a rarely used feature of Swift that doesn’t carry its weight.  Many of the reasons to remove them align with the rationale for removing -- and ++. " -- Chris Lattner, clattner@apple.com
* "My intuition *completely* agrees that Swift no longer needs C-style for loops. We have richer, better-structured looping and functional algorithms. That said, one bit of data I’d like to see is how often C-style for loops are actually used in Swift. It’s something a quick crawl through Swift sources on GitHub could establish. If the feature feels anachronistic and is rarely used, it’s a good candidate for removal." -- Douglas Gregnor, dgregor@apple.com
* "Every time I’ve used a C-style for loop in Swift it was because I forgot that .indices existed. If it’s removed, a fixme pointing that direction might be useful." -- David Smith, david_smith@apple.com
* "For what it's worth we don't have a single C style for loop in the Lyft codebase." -- Keith Smiley, keithbsmiley@gmail.com
* "Just checked; ditto Khan Academy." -- Andy Matsuchak, andy@andymatuschak.org
* "We’ve developed a number of Swift apps for various clients over the past year and have not needed C style for loops either." -- Eric Chamberlain, eric.chamberlain@arctouch.com
* "Every time I've tried to use a C-style for loop, I've ended up switching to a while loop because my iteration variable ended up having the wrong type (e.g. having an optional type when the value must be non-optional for the body to execute). The Postmates codebase contains no instances of C-style for loops in Swift." -- Kevin Ballard, kevin@sb.org
* "I found a couple of cases of them in my codebase, but they were trivially transformed into “proper” Swift-style for loops that look better anyway. If it were a vote, I’d vote for eliminating C-style." -- Sean Heber, sean@fifthace.com
