---
layout: post
title:  "Swift3.0_VIII_枚举、类和结构"
categories: Swift3.0
tags: Swift3.0
author: 3行代码
---

* content
{:toc}

枚举为一组相关的值定义了一个共同的类型，使你可以在你的代码中以类型安全的方式来使用这些值。

在 Swift 中，枚举类型是一等（first-class）类型。它们采用了很多在传统上只被类（class）所支持的特性，例如计算属性（computed properties），用于提供枚举值的附加信息，实例方法（instance methods），用于提供和枚举值相关联的功能。枚举也可以定义构造函数（initializers）来提供一个初始值；可以在原始实现的基础上扩展它们的功能；还可以遵循协议（protocols）来提供标准的功能。

# 枚举

### 枚举语法：

```swift
enum SomeEnumeration {
    // enumeration definition goes here
}
```

指南针案例：
```swift
enum CompassPoint {
    case north
    case south
    case east
    case west
}
```

注意
与 C 和 Objective-C 不同，Swift 的枚举成员在被创建时不会被赋予一个默认的整型值。在上面的CompassPoint例子中，north，south，east和west不会被隐式地赋值为0，1，2和3。相反，这些枚举成员本身就是完备的值，这些值的类型是已经明确定义好的CompassPoint类型。

多个成员值可以出现在同一行上，用逗号隔开：
```swift
enum Planet {
    case mercury, venus, earth, mars, jupiter, saturn, uranus, neptune
}
```

可以这样使用：
```swift
var directionToHead = CompassPoint.west
```

当directionToHead的类型已知时，再次为其赋值可以使用更简短的点语法：
```swift
directionToHead = .east
```

使用 Switch 语句匹配枚举值
```swift
directionToHead = .south
switch directionToHead {
case .north:
    print("Lots of planets have a north")
case .south:
    print("Watch out for penguins")
case .east:
    print("Where the sun rises")
case .west:
    print("Where the skies are blue")
default:
    print("Not a safe place for humans")
}
// Prints "Watch out for penguins"
```


当不需要匹配每个枚举成员的时候，你可以提供一个default分支来涵盖所有未明确处理的枚举成员：
```swift
let somePlanet = Planet.earth
switch somePlanet {
case .earth:
    print("Mostly harmless")
default:
    print("Not a safe place for humans")
}
// 打印 "Mostly harmless”
```


在 Swift 中，使用如下方式定义表示两种商品条形码(条形码、QR码)的枚举：
```swift
enum Barcode {
    case upc(Int, Int, Int, Int)
    case qrCode(String)
}
// “定义一个名为Barcode的枚举类型，它的一个成员值是具有(Int，Int，Int，Int)类型关联值的upc，另一个成员值是具有String类型关联值的qrCode。”
```

然后可以使用任意一种条形码类型创建新的条形码，例如：
```swift
var productBarcode = Barcode.upc(8, 85909, 51226, 3)
// 创建了一个名为productBarcode的变量，并将Barcode.upc赋值给它，关联的元组值为(8, 85909, 51226, 3)。

// 同一个商品可以被分配一个不同类型的条形码，例如：
productBarcode = .qrCode("ABCDEFGHIJKLMNOP")

switch productBarcode {
case .upc(let numberSystem, let manufacturer, let product, let check):
    print("UPC: \(numberSystem), \(manufacturer), \(product), \(check).")
case .qrCode(let productCode):
    print("QR code: \(productCode).")
}
// Prints "QR code: ABCDEFGHIJKLMNOP."

// 如果一个枚举成员的所有关联值都被提取为常量，或者都被提取为变量，为了简洁，你可以只在成员名称前标注一个let或者var：
switch productBarcode {
case let .upc(numberSystem, manufacturer, product, check):
    print("UPC: \(numberSystem), \(manufacturer), \(product), \(check).")
case let .qrCode(productCode):
    print("QR code: \(productCode).")
}
// 输出 "QR code: ABCDEFGHIJKLMNOP."
```

### 原始值

枚举成员可以被默认值（称为原始值）预填充，这些原始值的类型必须相同。

```swift
enum ASCIIControlCharacter: Character {
    case tab = "\t"
    case lineFeed = "\n"
    case carriageReturn = "\r"
}
// 枚举类型ASCIIControlCharacter的原始值类型被定义为Character，并设置了一些比较常见的 ASCII 控制字符
```


原始值的隐式赋值
```swift
enum Planet: Int {
    case mercury = 1, venus, earth, mars, jupiter, saturn, uranus, neptune
}
//  venus == 2  earth ==3...
```

使用枚举成员的rawValue属性可以访问该枚举成员的原始值：
```swift
let earthsOrder = Planet.earth.rawValue
// earthsOrder is 3
 
let sunsetDirection = CompassPoint.west.rawValue
// sunsetDirection is "west"
```


使用原始值初始化枚举实例

```swift
let possiblePlanet = Planet(rawValue: 7)
// possiblePlanet is of type Planet? and equals Planet.uranus
// possiblePlanet是Planet?类型，或者说“可选的Planet”。
```

原始值构造器是一个可失败构造器，因为并不是每一个原始值都有与之对应的枚举成员
```swift
// 如果你试图寻找一个位置为11的行星，通过原始值构造器返回的可选Planet值将是nil：

let positionToFind = 11
if let somePlanet = Planet(rawValue: positionToFind) {
    switch somePlanet {
    case .earth:
        print("Mostly harmless")
    default:
        print("Not a safe place for humans")
    }
} else {
    print("There isn't a planet at position \(positionToFind)")
}
// Prints "There isn't a planet at position 11"
```

### 递归枚举

递归枚举是一种枚举类型，它有一个或多个枚举成员使用该枚举类型的实例作为关联值。使用递归枚举时，编译器会插入一个间接层。你可以在枚举成员前加上indirect来表示该成员可递归。

```swift
// 在枚举成员前加上indirect来表示该成员可递归。
enum ArithmeticExpression {
    case number(Int)
    indirect case addition(ArithmeticExpression, ArithmeticExpression)
    indirect case multiplication(ArithmeticExpression, ArithmeticExpression)
}

// 在枚举类型开头加上indirect关键字来表明它的所有成员都是可递归的：
indirect enum ArithmeticExpression {
    case number(Int)
    case addition(ArithmeticExpression, ArithmeticExpression)
    case multiplication(ArithmeticExpression, ArithmeticExpression)
}

// 应用：
let five = ArithmeticExpression.number(5)
let four = ArithmeticExpression.number(4)
let sum = ArithmeticExpression.addition(five, four)
let product = ArithmeticExpression.multiplication(sum, ArithmeticExpression.number(2))

// 要操作具有递归性质的数据结构，使用递归函数是一种直截了当的方式
func evaluate(_ expression: ArithmeticExpression) -> Int {
    switch expression {
    case let .number(value):
        return value
    case let .addition(left, right):
        return evaluate(left) + evaluate(right)
    case let .multiplication(left, right):
        return evaluate(left) * evaluate(right)
    }
}

print(evaluate(product))
// 打印 "18"
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
