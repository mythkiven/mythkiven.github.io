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

- Swift的String与OC中的NSString可以进行无缝桥接，可以引入Foundation框架的API展开工作；
- Swift的字符串都是由独立编码的Unicode字符组成；
- Swift的字符串可以使用+进行操控；

### String Literals：字面字符串

```swift
// Swift会自动推断someString为String类型。
let someString = "Some string literal value"
```

### Initializing an Empty String:初始化空串

```swift
var emptyString = ""               // empty string literal
var anotherEmptyString = String()  // initializer syntax
if emptyString.isEmpty {
    print("Nothing to see here")
}

```

### String Mutability：可变字符串
在OC中，是否可变是通过NSMutableString类来决定的，但是在此，仅仅是通过定义来决定 var or let
```swift
var variableString = "Horse"
variableString += " and carriage"
```

### Strings Are Value Types：String是值类型

1、Swift的String是值类型。new一个String之后，进行赋值或者在函数入参都会进行值拷贝。

换句话就是说，Swift的String操作都会对已有String拷贝副本，然后操作副本。而不是传递操作指针。

2、Cocoa中的NSString传递给函数时，传递的其实是他的指针。除非手动copy，否则不会创建副本操作。

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

### Unicode

Unicode是一套国际文本的编码标准。其中的每一个字符都可以被解释为一个或多个Unicode标量。字符的Unicode标量是一个唯一的21位数字。

例如：U+0061表示小写的拉丁字母A（"a"），或U+1F425表示小鸡表情（"🐥"）。

字符串中还有特殊字符：

```
- 转义特殊字符\0（空字符）， \\（反斜杠）， \t（水平制表）， \n（换行）， \r（回车）， \"（双引号）和\'（单引号）
- 任意Unicode标量，都可写成\u{Ñ}，其中Ñ是用一个值等于一个有效的Unicode代码点的1-8位十六进制数
  

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

### Counting Characters：字符计数

.count计数字符数量.
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

- 

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
```

注意：防止索引溢出String的长度

```swift
greeting[greeting.endIndex] // 运行时Error
greeting.index(after: greeting.endIndex) // 运行时Error
```

##### Inserting and Removing：插入取出

- 插入字符：insert(_:at:)
- 插入字符串：insert(contentsOf:at:) 
- 移除字符：remove(at:)
- 移除字符串：removeSubrange(_:)

```swift
var welcome = "hello"
welcome.insert("!", at: welcome.endIndex) // "hello!"
 
welcome.insert(contentsOf:" there".characters, at: welcome.index(before: welcome.endIndex)) // "hello there!"

welcome.remove(at: welcome.index(before: welcome.endIndex)) // "hello there"
 
let range = welcome.index(welcome.endIndex, offsetBy: -6)..<welcome.endIndex
welcome.removeSubrange(range) // "hello"
```




```swift

```


```swift

``````swift

```




```swift

```

```swift

```

```swift

```

```swift

```

```swift

```

```swift

```

```swift

```

```swift

```

```swift

```


```swift

```



```swift

```



```swift

``````swift

```




```swift

```


```swift

``````swift

```




```swift

```
```swift

```

```swift

```

```swift

```

```swift

```

```swift

```

```swift

```

```swift

```

```swift

```


```swift

```



```swift

```



```swift

``````swift

```




```swift

```


```swift

``````swift

```




```swift

```
```swift

```

```swift

```

```swift

```

```swift

```

```swift

```

```swift

```

```swift

```

```swift

```


```swift

```



```swift

```



```swift

``````swift

```




```swift

```


```swift

``````swift

```




```swift

```
```swift

```

```swift

```

```swift

```

```swift

```

```swift

```

```swift

```

```swift

```

```swift

```


```swift

```



```swift

```



```swift

``````swift

```




```swift

```


```swift

``````swift

```




```swift

```
```swift

```

```swift

```

```swift

```

```swift

```

```swift

```

```swift

```

```swift

```

```swift

```


```swift

```



```swift

```



```swift

``````swift

```




```swift

```


```swift

``````swift

```




```swift

```
```swift

```

```swift

```

```swift

```

```swift

```

```swift

```

```swift

```

```swift

```

```swift

```


```swift

```



```swift

```



```swift

``````swift

```




```swift

```


```swift

``````swift

```




```swift

```
```swift

```

```swift

```

```swift

```

```swift

```

```swift

```

```swift

```

```swift

```

```swift

```


```swift

```



```swift

```



```swift

``````swift

```




```swift

```


```swift

``````swift

```




```swift

```
```swift

```

```swift

```

```swift

```

```swift

```

```swift

```

```swift

```

```swift

```

```swift

```


```swift

```



```swift

```



```swift

``````swift

```




```swift

```


```swift

``````swift

```




```swift

```
```swift

```

```swift

```

```swift

```

```swift

```

```swift

```

```swift

```

```swift

```

```swift

```


```swift

```



```swift

```



```swift

``````swift

```




```swift

```


```swift

``````swift

```




```swift

```
