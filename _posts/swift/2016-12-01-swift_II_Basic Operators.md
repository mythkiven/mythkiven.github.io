---
layout: post
title:  "Swift_II_基本运算符"
categories: Swift
tags: Swift
author: 3行代码
---

* content
{:toc}

Swift 支持大多数的标准C语言的运算符，并且改进了不少特性来减少常见的编码错误。例如：
- 赋值符 = 不返回值。防止与==弄混；
- 算术运算符：+-*/%会检测防止值溢出；
- 以及C语言没有的表达:（ a..< b和a...b）

### Terminology：术语

运算符有一元、二元、三元之分；
- 一元：-、！、++ 如：-1,!b,i++
- 二元：算术运算符，如：2+3
- 三元：Swift中就一个，三目运算符：(a?b:c)

### Assignment Operator：赋值运算符

```swift
let b = 10
var a = 5
a = b
// a is now equal to 10

// 用元组中：
let (x, y) = (1, 2)
// x is equal to 1, and y is equal to 2

// 无返回值：
if x = y {
    // This is not valid, because x = y does not return a value.
}
```

### Arithmetic Operators：算术运算符
是不允许值溢出的，如果值溢出，应该使用溢出运算符 a &- b;

- \+ 求和，也用于string的拼接，"hello, " + "world"  // equals "hello, world";
- \- 求差;
- \* 求积;
- / 求商、倍数 9/5=1;
- % 求余、模运算 9%5=4;
- \- 元减运算符 -3(负数3);
- +=等 符合运算符 var a = 1;a += 2;
- == != > < >= <= 比较运算符，返回BOOL值；
- === 恒等于、 !===不恒等于，用于判断是否引用同一对象；
- 三目运算符：条件?答案1:答案2  条件成立则返回答案1，否则答案2
- 空合运算符：(a ?? b):是对可选类型a的判空处理，如果a包含数值就解析返回值，否则返回b。
等价于 `a != nil ? a! : b` :可选类型a是空就返回b，不是空就强制解析返回a!空合就是这段代码的简短表达
```swift
let defaultColorName = "red"
var userDefinedColorName: String?   // defaults to nil
var colorNameToUse = userDefinedColorName ?? defaultColorName
```


### Range Operators:区间运算符

- (a...b) 定义一个a、b间的闭区间。可用于for循环；
``` swift
for index in 1...5 {
    print("\(index) times 5 is \(index * 5)") // 范围是>=1 <=5
}
```
- （a..<b) 定义一个a、b间的半闭区间。可用于数组区间 // >
```swift
let names = ["Anna", "Alex", "Brian", "Jack"]
let count = names.count
for i in 0..<count {
    print("Person \(i + 1) is called \(names[i])")
}


```

~是不是很好用~

### Logical Operators：逻辑运算符

- ! 逻辑非，
- && 逻辑与
- || 逻辑或


OK,over~~