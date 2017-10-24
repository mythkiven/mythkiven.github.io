---
layout: post
title:  "Swift3.0_V_控制流"
categories: Swift
tags: Swift
author: 3行代码
---

* content
{:toc}

- Swift提供了所有C语言中相似的控制流结构。包括for和while循环；if和switch条件语句；break和continue跳转语句等;
- 相对于C语言，Swift中switch语句的case语句后，不会自动跳转到下一个语句(这样就避免了C语言中因为忘记break而造成的错误);
- 另外case语句可以匹配多种类型，包括数据范围，元组，或者特定的类型等;
- switch语句中已匹配的数值也可以被用在后续的case语句体中，where关键词还能被加入任意的case语句中，来增加匹配的方式。

### for

- for-in:对于数据范围，序列，集合等中的每一个元素，都执行一次


```swift
for index in 1...5 {
    print("\(index) times 5 is \(index * 5)")
}
// 1 times 5 is 5
// 2 times 5 is 10
// 3 times 5 is 15
// 4 times 5 is 20
// 5 times 5 is 25
```

忽略变量模式：

```swift
let base = 3
let power = 10
var answer = 1
for _ in 1...power {
    answer *= base
}
print("\(base) to the power of \(power) is \(answer)")
// Prints "3 to the power of 10 is 59049"
```

### While

while循环方式：

- while循环，在每次循环开始前,判断循环条件是否成立
- repeat-while循环，在每次循环之后,判断循环条件是否成立
- repeat-while 类似 do-while

```swift
while condition {
    statements
}
```

```swift
Repeat {
    statements
} while condition
```



### Conditional Statements:条件语句

Swift提供了两种条件分支语句的方式，if语句和switch语句。一般if语句比较常用，但是只能检测少量的条件情况。switch语句用于大量的条件可能发生时的条件语句。

##### if

```swift
temperatureInFahrenheit = 90
if temperatureInFahrenheit <= 32 {
    print("It's very cold. Consider wearing a scarf.")
} else if temperatureInFahrenheit >= 86 {
    print("It's really warm. Don't forget to wear sunscreen.")
} else {
    print("It's not that cold. Wear a t-shirt.")
}
// Prints "It's really warm. Don't forget to wear sunscreen."
```

##### switch

- fallthrough 关键字，可让每一个case都有机会执行；
- 不用break,匹配OK自动终止；
- 每一种情况，case之后要跟上可执行语句：

```swift
switch anotherCharacter {
case "a": // Invalid！！！！！！
case "A":
    print("The letter A")
```

一般结构：

```swift
switch some value to consider {
    case value 1:
        respond to value 1
    case value 2, value 3:
         respond to value 2 or 3
    default:
         otherwise, do something else
}
```

案例：匹配字符

```swift
let someCharacter: Character = "z"
switch someCharacter {
case "a":
    print("The first letter of the alphabet")
case "z":
    print("The last letter of the alphabet")
default:
    print("Some other character")
}
// Prints "The last letter of the alphabet"
```

**1、匹配复合数值**

```swift
let someCharacter: Character = "e"
switch someCharacter {
case "a", "e", "i", "o", "u":
    print("\(someCharacter) is a vowel")
case "b", "c", "d", "f", "g", "h", "j", "k", "l", "m",
     "n", "p", "q", "r", "s", "t", "v", "w", "x", "y", "z":
    print("\(someCharacter) is a consonant")
default:
    print("\(someCharacter) is not a vowel or a consonant")
}
// Prints "e is a vowel"
```

**2、范围匹配**

```swift
let approximateCount = 62
let countedThings = "moons orbiting Saturn"
var naturalCount: String
switch approximateCount {
case 0:
    naturalCount = "no"
case 1..<12:
    naturalCount = "several"
case 12..<1000:
    naturalCount = "hundreds of"
default:
    naturalCount = "many"
}
print("There are \(naturalCount) \(countedThings).")
// Prints "There are dozens of moons orbiting Saturn."
```

**3、元组匹配**

// 匹配坐标：
```swift
let somePoint = (1, 1)
switch somePoint {
case (0, 0):
    print("(0, 0) is at the origin")
case (_, 0):
    print("(\(somePoint.0), 0) is on the x-axis")
case (0, _):
    print("(0, \(somePoint.1)) is on the y-axis")
case (-2...2, -2...2):
    print("(\(somePoint.0), \(somePoint.1)) is inside the box")
default:
    print("(\(somePoint.0), \(somePoint.1)) is outside of the box")
}
// Prints "(1, 1) is inside the box"
```

![](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Art/coordinateGraphSimple_2x.png)

**4、值绑定**

在case匹配的同时，可以将switch语句中的值绑定给一个临时的常量或者变量，以便在case的语句中使用。

```swift
let anotherPoint = (2, 6)
switch anotherPoint {
case (let x, 0): // 此后 x == 2
    print("on the x-axis with an x value of \(x)")
case (0, let y): // 此后 y == 6
    print("on the y-axis with a y value of \(y)")
case let (x, y): // x == 2 y == 6.因此匹配
    print("somewhere else at (\(x), \(y))")
}
// print "somewhere else at (2, 6)"
```

**5、Where关键词**

可以在case中增加where来增强判断条件：

```swift
let yetAnotherPoint = (1, -1)
switch yetAnotherPoint {
    case let (x, y) where x == y: // 此后 x == 1 y == -1
        print("(\(x), \(y)) is on the line x == y")
    case let (x, y) where x == -y: // 满足条件。
        print("(\(x), \(y)) is on the line x == -y")
    case let (x, y):
        print("(\(x), \(y)) is just some arbitrary point")
}
// prints "(1, -1) is on the line x == -y"
```

**6、复合情况**

```swift
let stillAnotherPoint = (9, 0)
switch stillAnotherPoint {
case (let distance, 0), (0, let distance):
    print("On an axis, \(distance) from the origin")
default:
    print("Not on an axis")
}
// Prints "On an axis, 9 from the origin"

```

### Control Transfer Statements:控制跳转语句

- continue:结束当前循环，开始下一次循环；
- break:终止整个循环过程，用于for中：结束for循环执行for后续代码;switch中：break该case分支匹配结束，如果没有匹配成功，就继续后续的case的匹配。其实和fallthrough在switch中的作用一样
- fallthrough:switch中，让每一个case,在fallthrough之前的语句都会执行，不论是否匹配成功。（switch默认在匹配成功之后，后边的case不会被执行）；
- return 函数中返回数值,或者提前退出;
- throw

### Labeled Statements:标签语句

switch和循环可以互相嵌套，循环之间也可以互相嵌套，因此在使用break或者continue的时候，需要知道到底是对哪个语句起作用。这就需要用到标签语句。标签语句的一般形式如下：


```swift
label name: while condition {
    statements
}
```

案例：

```swift
let finalSquare = 25
var board = [Int](repeating: 0, count: finalSquare + 1)
board[03] = +08; board[06] = +11; board[09] = +09; board[10] = +02
board[14] = -10; board[19] = -11; board[22] = -02; board[24] = -08
var square = 0
var diceRoll = 0


gameLoop: while square != finalSquare {
    diceRoll += 1
    if diceRoll == 7 { diceRoll = 1 }
    switch square + diceRoll {
    case finalSquare:
        // 注意写法： 结束gameLoop游戏
        break gameLoop
    case let newSquare where newSquare > finalSquare:
        // 注意写法： 结束当前的gameloop游戏，开始下一次的游戏
        continue gameLoop
    default:
        // this is a valid move, so find out its effect
        square += diceRoll
        square += board[square]
    }
}
print("Game over!")
```

### Early Exit：提前退出

使用guard申明，和if一样只判断布尔值。如果条件是true那么guard以后的代码将被调用，如果是false那么就会调用else里面的内容。和if不同的是，guard是一直有else的。

```swift
func greet(person: [String: String]) {
    guard let name = person["name"] else {
        return
    }
    
    print("Hello \(name)!")
    
    guard let location = person["location"] else {
        print("I hope the weather is nice near you.")
        return
    }
    
    print("I hope the weather is nice in \(location).")
}
 
greet(person: ["name": "John"])
// Prints "Hello John!"
// Prints "I hope the weather is nice near you."
greet(person: ["name": "Jane", "location": "Cupertino"])
// Prints "Hello Jane!"
// Prints "I hope the weather is nice in Cupertino."
```

### Checking API Availability：检查API可用性

swift已经內建了对于测试api可用性的支持，从而确保不会调用一个SDK不支持的API。

```swift
if #available(platform name version, ..., *) {
    statements to execute if the APIs are available
} else {
    fallback statements to execute if the APIs are unavailable
}
```

案例：

```swift
if #available(iOS 10, macOS 10.12, *) {
    print("YES")
    // Use iOS 10 APIs on iOS, and use macOS 10.12 APIs on macOS
} else {
    print("NO")
    // Fall back to earlier iOS and macOS APIs
}
```

