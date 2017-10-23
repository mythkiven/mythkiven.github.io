---
layout: post
title:  "Swift_II_基本运算符"
categories: Swift3.0
tags: Swift3.0
author: 3行代码
---

* content
{:toc}

### 概述
Swift 支持大多数的标准C语言的运算符，并且改进了不少特性来减少常见的编码错误。例如：
- 赋值符 = 不返回值。防止与==弄混；
- 算术运算符：+-*/%会检测防止值溢出；
- 以及C语言没有的表达:（ a..< b和a...b）

**Swift3更新：移除了自增自减(++、--)等运算。详见下文：**

### Terminology：术语

运算符有一元、二元和三元运算符：

- 一元运算符对单一操作对象操作（如-a）。前置运算符需紧排操作对象之前（如!b），后置运算符需紧跟操作对象之后（如i++）
- 二元运算符操作两个操作对象（如2 + 3）
- 三元运算符操作三个操作对象，和C语言一样，Swift只有一个，就是三目运算符（a ? b : c）
- 注意：受运算符影响的值叫操作数，在表达式1 + 2中，加号+是二元运算符，它的两个操作数是值1和2。

### Assignment Operator：赋值运算符

```swift
// 赋值运算（a = b），表示用b的值来初始化或更新a的值：
let b = 10
var a = 5
a = b

// 用元组中：
let (x, y) = (1, 2)
// x is equal to 1, and y is equal to 2

// 无返回值：
if x = y {
    // 这个特性使你无法把（==）错写成（=），所以：if x = y是错误代码
}
```

### Arithmetic Operators：算术运算符

**注意：**
- Swift3 移除了++、--自增自减运算；
- Swift3 不支持浮点的求余运算；
- 与C语言和OC不同的是，Swift 默认情况下不允许在数值运算中出现溢出情况。但是你可以使用 Swift 的溢出运算符来实现溢出运算（如a &+ b）


- 求和：\+ ，也用于string的拼接：`"hello, " + "world"  // equals "hello, world";`
- 求差：\- ;
- 求积：\* ;
- 求商：/ ，9/5=1;
- 求余：% ，9%5=4;【非模运算！】
- 元减运算符:\- ，-3(负数3);
- 复合运算符：+= -= *= /= %=  ， var a = 1;a += 2;
- 比较运算符：== != > < >= <= ，返回BOOL值；
- 恒等: === 恒等于、 !===不恒等于，用于判断是否引用同一对象；
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

``` swift
let names = ["Anna", "Alex", "Brian", "Jack"]
let count = names.count
for i in 0..<count {
    print("Person \(i + 1) is called \(names[i])")
}
```

~是不是很好用~

### Logical Operators：逻辑运算符

- !逻辑非
- &&逻辑与
- &#124;&#124;逻辑或

可以通过括号明确优先级。

<br> 
OK,over~~