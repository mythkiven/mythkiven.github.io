---
layout: post
title:  "Swift_III_字符(串)"
categories: Swift
tags: Swift
author: 3行代码
---

* content
{:toc}


**Strings and Characters ：字符和字符串**

- Swift的String与OC中的NSString可以进行无缝桥接；
- Swift的字符串都是由独立编码的Unicode字符组成；
- Swift的字符串可以使用+进行连接；
- ...

### Unicode

Unicode是一套国际文本的编码标准。其中的每一个字符都可以被解释为一个或多个Unicode标量。字符的Unicode标量是一个唯一的21位数字。

例如：U+0061表示小写的拉丁字母A（"a"），或U+1F425表示小鸡表情（"🐥"）。

特殊字符：

- 转义特殊字符\0（空字符）， \\（反斜杠）， \t（水平制表）， \n（换行）， \r（回车），\"（双引号）和\'（单引号）
- 任意Unicode标量，都可写成\u{Ñ}，其中Ñ是用一个值等于一个有效的Unicode代码点的1-8位十六进制数

```swift
let wiseWords = "\"Imagination is more important than knowledge\" - Einstein"
// "Imagination is more important than knowledge" - Einstein
let dollarSign = "\u{0061}"        // a,  Unicode scalar U+0061
let blackHeart = "\u{2665}"      // ♥,  Unicode scalar U+2665
let sparklingHeart = "\u{1F496}" // 💖, Unicode scalar U+1F496
```

其他诸如：
```swift
let eAcute: Character = "\u{E9}"                         // é
let combinedEAcute: Character = "\u{65}\u{301}"          // e followed by ́
let precomposed: Character = "\u{D55C}"                  // 한
let decomposed: Character = "\u{1112}\u{1161}\u{11AB}"   // ᄒ, ᅡ, ᆫ
let regionalIndicatorForUS: Character = "\u{1F1FA}\u{1F1F8}" //   🇺🇸
```


### String Literals：字面字符串

```swift
// Swift会自动推断someString为String类型。
let someString = "Some string literal value"
```

### Initializing an Empty String:初始化空串

```swift
var emptyString = ""               // empty string literal
var anotherEmptyString = String()  // initializer syntax
// .isEmpty 属性
if emptyString.isEmpty {
    print("Nothing to see here")
}

```

### String Mutability：可变字符串
在OC中，是否可变是通过NSMutableString类来决定的，但是在此，仅仅是通过定义来决定: var or let
```swift
var variableString = "Horse"
variableString += " and carriage"
```

### Strings Are Value Types：String是值类型

1、Swift的String是值类型。Swift的String进行赋值或者在函数入参都会进行值拷贝,使用的是副本而不是传递指针。

2、Cocoa中的NSString作为参数传递给函数时，传递的其实是他的指针。除非手动copy，否则不会创建副本操作。

### Working with Characters:使用字符


```swift
for character in "Dog!🐶".characters {
    print(character)
}
// D
// o
// g
// !
// 🐶

let catCharacters: [Character] = ["C", "a", "t", "!", "🐱"]
let catString = String(catCharacters)
print(catString)
// Prints "Cat!🐱"
```

### Concatenating Strings and Characters：连接字符和字符串

```swift
var instruction = "look over"
instruction += string2

let exclamationMark: Character = "!"
welcome.append(exclamationMark)
```

### String Interpolation：字符串插值

```swift
let multiplier = 3
let message = "\(multiplier) times 2.5 is \(Double(multiplier) * 2.5)"
// message is "3 times 2.5 is 7.5"
```

注意：插值字符串中的括号中的表达式:不能包含在反斜杠\、回车或换行


### Counting Characters：字符计数

.characters.count 计数字符数量；countElements()弃用。

需要注意的是：Unicode字符的不同表达方式可能需要不同数量的内存空间来存储。所以Swift中字符在一个字符串中并不一定占用相同的内存空间。

如下：
```swift
var word = "cafe"
print("\(word.characters.count)") // Prints "4"

word += "\u{301}"
print("\(word.characters.count)") // Prints "4"

word += "\u{1F496}"
print("\(word.characters.count)") // Prints "5"

```

### Accessing and Modifying a String：访问修改字符串

##### 索引访问

String值具有相关联的索引类型,String.Index 索引对应每一个位置的字符。
方法如下：
```swift

    public typealias Index = String.CharacterView.Index
    public var startIndex: String.Index { get }
    public var endIndex: String.Index { get }
    public func index(after i: String.Index) -> String.Index
    public func index(before i: String.Index) -> String.Index
// 从起始位置：String.Index算起，第String.IndexDistance个位置
    public func index(_ i: String.Index, offsetBy n: String.IndexDistance) -> String.Index

    let s = "Swift"
    s.startIndex // 0
    s.endIndex // 5
    s.index(after: s.startIndex) // 1
    s.index(after: s.endIndex) // Error
    s.index(before: s.startIndex) // Error
    s.index(before: s.endIndex) // 4
    s.index(s.endIndex, offsetBy: -2) //从最后向前 -2的位置 3

    let i = s.index(s.startIndex, offsetBy: 4)
    print(s[i])// Prints "t"
```

```swift
let greeting = "Guten Tag!"
greeting[greeting.startIndex]
// G
greeting[greeting.index(before: greeting.endIndex)]
// !
greeting[greeting.index(after: greeting.startIndex)]
// u
let index = greeting.index(greeting.startIndex, offsetBy: 2)
greeting[index]
// t
for index in greeting.characters.indices {
    print("\(greeting[index]) ", terminator: "")
}
// Prints "G u t e n   T a g ! "

// 注意：防止索引溢出String的长度
greeting[greeting.endIndex] // 运行时Error
greeting.index(after: greeting.endIndex) // 运行时Error
```

##### Inserting and Removing：插入取出

- 插入字符：insert(_:at:)
- 插入字符串：insert(contentsOf:at:) 
- 移除字符：remove(at:)
- 移除字符串：removeSubrange(_:)
- 拼接字符串：append(_ c: Character)、append(other: String)

```swift
var welcome = "hello"
welcome.insert("!", at: welcome.endIndex) // "hello!"
welcome.insert(contentsOf:" there".characters, at: welcome.index(before: welcome.endIndex)) // "hello there!"
welcome.remove(at: welcome.index(before: welcome.endIndex)) // "hello there"
let range = welcome.index(welcome.endIndex, offsetBy: -6)..<welcome.endIndex
welcome.removeSubrange(range) // "hello"

var jj = "1"
jj.append("_+🐂") // "1_+🐂"
jj.append("s") // "1_+🐂s"
```

### Comparing Strings：比较字符串

- Swift 提供了三种方式来比较字符串的值：字符串相等、前缀相等和后缀相等
- Swift 中字符串的比较不区分语言环境

字符串相等：
```swift
let quotation     = "We're a lot alike, you and I."
let sameQuotation = "We're a lot alike, you and I."
if quotation == sameQuotation {
    print("These two strings are considered equal") // 会打印
}

// 2、相等会涉及到 Unicode标量的问题：

// 比如：
let eAcuteQuestion = "Voulez-vous un caf\u{E9}?"
let combinedEAcuteQuestion = "Voulez-vous un caf\u{65}\u{301}?"
if eAcuteQuestion == combinedEAcuteQuestion {
    print("These two strings are considered equal") // 会打印
}
// 又例如：
let latinCapitalLetterA: Character = "\u{41}"
let cyrillicCapitalLetterA: Character = "\u{0410}"
if latinCapitalLetterA != cyrillicCapitalLetterA {
    print("These two characters are not equivalent.") // 会打印
}
```

前后缀相等：

通过调用字符串的hasPrefix/hasSuffix方法来检查字符串是否拥有特定前缀/后缀。 两个方法均需要以字符串作为参数传入并传出Boolean值。 两个方法均执行基本字符串和前缀/后缀字符串之间逐个字符的比较操作

```swift
let xz = "qqqq-2222-++++"
if xz.hasPrefix("qqq") {
    print("qqq") // qqq
}
if xz.hasSuffix("-+") {
    print("YES")
}else{
    print("NO") // NO
}
```

### 大小写


使用方法：
- .uppercased() 弃用.uppercaseString
- .lowercased() 弃用.lowercaseString

```swift
let normal = "Could you help me, please?"
let shouty = normal.uppercased()
// shouty 值为 "COULD YOU HELP ME, PLEASE?"
let whispered = normal.lowercased()
// whispered 值为 "could you help me, please?"
```

### Unicode Representations of Strings：字符串的Unicode交涉

- 一个Unicode字符串被写入文本或进行储存时，会使用Unicode标量进行编码；
- 每种编码形式都有小的编码单元组成；
- 编码形式包括UTF-8编码的形式、UTF-16编码形式和UTF-32的编码形式；
- 编码单元是各个编码形式中的单个单元：UTF-8编码串为8位码单位、UTF-16编码串为16位代码单位、UTF-32编码串为32位代码单位。

```swift
// 它是由D，o，g，‼，以及🐶字符（DOG FACE或Unicode标U+1F436）组成
let dogString = "Dog‼🐶"
```

##### UTF-8

可以通过遍历字符串的utf8属性来访问它的UTF-8表示。 其为UTF8View类型的属性，UTF8View是无符号8位 (UInt8) 值的集合，每一个UInt8值都是一个字符的 UTF-8 表示：
![](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Art/UTF8_2x.png)

```swift
let dogString = "Dog‼🐶"
for codeUnit in dogString.utf8 {
    print("\(codeUnit) ", terminator: "")
}
print("")
// 68 111 103 226 128 188 240 159 144 182
```
上面的例子中，前四个10进制代码单元值 (68, 111, 103, 33) 代表了字符D o g和!，它们的 UTF-8 表示与 ASCII 表示相同。 后四个代码单元值 (240, 159, 144, 182) 是DOG FACE的4字节 UTF-8 表示。

##### UTF-16

可以通过遍历字符串的utf16属性来访问它的UTF-16表示。 其为UTF16View类型的属性，UTF16View是无符号16位(UInt16) 值的集合，每一个UInt16都是一个字符的 UTF-16 表示：
![](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Art/UTF16_2x.png)
```swift
    for codeUnit in dogString.utf16 {
        print("\(codeUnit) ")
    }
    print("\n")
    // 68 111 103 33 55357 56374
```
同样，前四个代码单元值(68, 111, 103, 33)代表了字符D o g和!，它们的UTF-16代码单元和 UTF-8 完全相同。
第五和第六个代码单元值(55357 和 56374)是DOG FACE字符的UTF-16表示。 第一个值为U+D83D(十进制值为 55357)，第二个值为U+DC36(十进制值为 56374)。

##### Unicode Scalar Representation：Unicode标量表示

可以通过遍历字符串的unicodeScalars属性来访问它的Unicode标量表示。其为UnicodeScalarView类型的属性，UnicodeScalarView是UnicodeScalar的集合。 

每一个UnicodeScalar拥有一个值属性，可以返回对应的21位数值，用UInt32来表示。
![](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Art/UnicodeScalar_2x.png)
```swift
for scalar in dogString.unicodeScalars {
    print("\(scalar.value) ", terminator: "")
}
print("")
// Prints "68 111 103 8252 128054 "
```
同样，前四个代码单元值 (68, 111, 103, 33) 代表了字符D o g和!。 第五位数值，128054，是一个十六进制1F436的十进制表示。 其等同于DOG FACE的Unicode 标量 U+1F436。

作为查询字符值属性的一种替代方法，每个UnicodeScalar值也可以用来构建一个新的字符串值，比如在字符串插值中使用：


```swift
for scalar in dogString.unicodeScalars {
    println("\(scalar) ")
}
// D
// o
// g
// !
// 🐶
```


