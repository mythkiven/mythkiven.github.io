---
layout: post
title:  "Swift3.0_I_入门体验"
categories: Swift
tags: Swift
author: 3行代码
---

* content
{:toc}

## 前言

- 必备武器：playground+官方文档；
- 系列章节我会以中英结合的形式展现，不要问为什么；
- 本笔记针对有一定swift或OC基础的；
- 后续相关的文章都基于[The Swift Programming Language (Swift 3.0.1)](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/TheBasics.html) ，基本上是围绕文档翻译一遍。

先热热身：
- Swift包含C和OC的所有基础数据类型，再次基础上还引入了OC没有的高级数据类型；
- constants ：Swift中的常量比C中的更加强大，使用更安全；
- tuples ：Swift亦引入了OC没有的高级数据类型，元组；
- optional types：处理未知类型，比nil更加安全；
- type safety：Swift是类型安全的，代码里值的类型非常明确：需要String类型，你就不能错误地传递Int；


### I.Constants and Variables:变常量

##### 1.Declaring Constants and Variables:声明 
let常量不用多说，var变量的需要注意： type是如何定义的。

``` swift
// type声明的方式1：初始值推断。
let maximumNumberOfLoginAttempts = 10
var currentLoginAttempt = 0
var x = 0.0, y = 0.0, z = 0.0

// 方式2：Type Annotations 类型标注
var welcomeMessage: String // 定义一个String类型的变量:welcomeMessage
welcomeMessage = "Hello" 
var red, green, blue: Double // 定义多个同类型变量
```

##### 2.Naming Constants and Variables：命名

常量和变量名几乎可以包含任何字符，包括Unicode字符：
```swift
// 1、
let π = 3.14159
let 你好 = "你好世界"
let 🐶🐮 = "dogcow"

// 2、
/* 
注意：常量和变量的名字不能包含空格、数学符号、箭头、私用的（非法的）Unicode代码点、连线与制表符。也不能以数字开头。
*/

// 3、
/*
如果使用与Swift保留关键字相同的名称作为常量或者变量名，可以使用反引号（`）将关键字包围的方式将其作为名字使用。例子如下：  尽量还是避免使用关键字
*/
let `let` = 10
print(`let`)

// 4、常量的值一旦设定后就不能更改.如果尝试去更改，编译器会报错
```

##### 3.Printing Constants and Variables：打印

``` swift
// 默认情况下，print会加一个换行符来结束当前行
print(friendlyWelcome)                 // Prints "Bonjour!\n" 
// 可以给terminator赋一个空字符串，防止换行
print(friendlyWelcome, terminator: "") // Prints "Bonjour!"

/*
print(_:separator:terminator)函数是一个打印一个或多个值到相应输出区域的全局的函数。separator 和 terminator 参数有默认值，一般可以忽略之
*/

// Swift 用字符串插值（string interpolation）的方式把常量名或者变量名当做占位符加入到长字符串中，并且去提示Swift 用当前常量或变量的值替换这些占位符。将常量或变量名放入圆括号中，并在开括号前使用反斜杠将其转义： 
print("The current value of friendlyWelcome is \(friendlyWelcome)")
// Prints "The current value of friendlyWelcome is Bonjour!"

```

### Comments：注释

``` swift

// This is a comment.

/* This is also a comment
 but is written over multiple lines. */

/* 嵌套注释: This is the start of the first multiline comment.
 /* This is the second, nested multiline comment. */
 This is the end of the first multiline comment. */

```

### Semicolons：分号

```swift
// 单行代码可以不用注释，
let cat = "🐱"
let dog = "🐶"
// 多行代码合并到一行，需要分号
let cat = "🐱"; print(cat)
// Prints "🐱"


```

### Integers
Swift提供的有符号和无符号整数由8,16,32和64位组成。8位无符号整数类型是UInt8,32位有符号整数的类型是Int32。

##### 1.Integer Bounds：边界

```swift
let minValue = UInt8.min  // minValue is equal to 0, and is of type UInt8
let maxValue = UInt8.max  // maxValue is equal to 255, and is of type UInt8
```

##### 2.Int
一般来说，你不需要给整数指定一个特定的长度,Swift会自动处理：
- 在32位平台上，Int和Int32长度相同。
- 在64位平台上，Int和Int64长度相同。
- 32位平台上， Int可以存储的整数范围也可以达到-2,147,483,648~2,147,483,647。所以一般情况不用担心大小的问题。

##### 3.UInt
UInt长度与当前平台的原生字长相同：
- 在32位平台上，UInt和UInt32长度相同。
- 在64位平台上，UInt和UInt64长度相同。

注意：尽量不要使用UInt，除非你真的需要存储一个和当前平台原生字长相同的无符号整数。统一使用Int可以提高代码的可复用性，避免不同类型数字之间的转换。

### Floating-Point Numbers

浮点类型比整数类型表示的范围更大，可以存储比Int类型更大或者更小的数字。Swift 提供了两种有符号浮点数类型：
- Double表示64位浮点数。存储很大或者很高精度的浮点数;
- Float表示32位浮点数。精度要求不高可使用此类型;

注意： Double精确度很高，可以精确到15位小数位数，而Float只能精确到6位小数位数。Swift在推断浮点型数字时，优先使用Double类型。

### Type Safety and Type Inference：类型安全

Swift是一个类型安全的语言。并且可以进行类型推断。

- 1、编译代码的时候它执行类型检查，并会将任何类型不匹配类型标注为错误；
- 2、如果没有给值指定一个类型，Swift会通过类型推断去推断出相对应的类型；
- 3、由于有类型推断，Swift相对于C语言和Objective-C来说，只需要更少的类型声明。
代码如下：
```swift
let meaningOfLife = 42       // meaningOfLife is inferred to be of type Int
let pi = 3.14159             // pi is inferred to be of type Double
let anotherPi = 3 + 0.14159  // anotherPi is also inferred to be of type Double。Swift优先推断Double
```

### Numeric Literals

1、整数常量可以写为：
- 一个十进制数，无前缀 
- 一个二进制数字，用0b前缀
- 一个八进制数，有0o前缀
- 一个十六进制数字，用0x前缀

```swift
let decimalInteger = 17
let binaryInteger = 0b10001       // 17 in binary notation
let octalInteger = 0o21           // 17 in octal notation
let hexadecimalInteger = 0x11     // 17 in hexadecimal notation
```

2、Numeric Literals还有一个可选的指数(exponent),十进制用e/E指定，十六进制用p/P指定:

- 1.25e2 表示1.25 × 10^2，等于125.0 
- 1.25e-2 表示1.25 × 10^-2，等于0.0125  

- 0xFp2 表示15 × 2^2，等于60.0
- 0xFp-2 表示15 × 2^-2，等于3.75  

如下都表示十进制12.1875:
```swift
let decimalDouble = 12.1875
let exponentDouble = 1.21875e1
let hexadecimalDouble = 0xC.3p0
```

3、Numeric Literals可以包含额外的格式，使它们更容易阅读。这两个整数和浮点数可以用额外的零填充并且可以包含下划线，以帮助可读性。无论何种类型的格式不会影响其基本数值：

```swift
let paddedDouble = 000123.456
let oneMillion = 1_000_000
let justOverOneMillion = 1_000_000.000_000_1
```

### Numeric Type Conversion 数字类型转换

一句话：一般情况下，整数型常变量都建议使用Int类型。

##### Integer Conversion:整型转换

- SomeType(ofInitialValue)类型转换
- 基本运算符两侧的数字需要是同类型的，
```swift
let twoThousand: UInt16 = 2_000
let one: UInt8 = 1
let twoThousandAndOne = twoThousand + UInt16(one)// 不能直接相加，需要类型转换
```

##### Integer and Floating-Point Conversion：整型浮点型转换
必须显式指定类型，否则报错
```swift
let three = 3 
let pointOneFourOneFiveNine = 0.14159
let papi =  three+pointOneFourOneFiveNine // 报错
let pi = Double(three) + pointOneFourOneFiveNine // OK,is inferred to be of type Double
let integerPi = Int(pi)
// integerPi equals 3, and is inferred to be of type Int

```

**注意：如下代码，如果数字没有被明确指定类型，那么他们的类型会在编译需要时被推测。**
**但是如果已经显式之处了类型，必须先进行类型转换，才能够运算**
```swift
let twoThousand: UInt16 = 2_000
let one: UInt8 = 10
let e1 = twoThousand+10 // ok  类型会在编译需要时被推测
let e2 = twoThousand+one // 报错

```

### Type Aliases：类型别名

自定义符合自己风格或者更有意义的名字：AudioSample代表UInt16

```swift
typealias AudioSample = UInt16
var maxAmplitudeFound = AudioSample.min
```

### Booleans

```swift
let orangesAreOrange = true
let turnipsAreDelicious = false
if turnipsAreDelicious {
    print("Mmm, tasty turnips!")
} else {
    print("Eww, turnips are horrible.")
}
// Prints "Eww, turnips are horrible."

```

```swift
let i = 1
if i {
    // 报错而不会编译，因为在该使用BOOL的地方使用了非BOOL值
}

if i == 1 {
    // compile successfully
}
```

### Tuples：元组

把多个值符合组成一个复合值，其中类型没要求，是任意类型。

```swift
// 1、http404Error的元组
let http404Error = (404, "Not Found")
// http404Error is of type (Int, String), and equals (404, "Not Found")

// 2、元组的内容可以取出作为单独的常量和变量：
let (statusCode, statusMessage) = http404Error
print("The status code is \(statusCode)")
// Prints "The status code is 404"
print("The status message is \(statusMessage)")
// Prints "The status message is Not Found"

// 3、可以忽略不需要的元组值，通过使用下滑线_
let (justTheStatusCode, _) = http404Error
print("The status code is \(justTheStatusCode)")
// Prints "The status code is 404"

// 4、使用索引访问元组
print("The status code is \(http404Error.0)")
// Prints "The status code is 404"
print("The status message is \(http404Error.1)")
// Prints "The status message is Not Found"

// 5、定义元组时可以分别命名各个元素
let http200Status = (statusCode: 200, description: "OK")
print("The status code is \(http200Status.statusCode)")
// Prints "The status code is 200"
print("The status message is \(http200Status.description)")
// Prints "The status message is OK"

```
注意：
- 元组在临时数据中非常有用，但不适合用于创建复杂的数据结构(使用类或者结构体等)。
- 将元组用作函数返回值是不错的选择。

### Optionals：可选类型

在C或者OC中没有可选类型的概念，譬如下面的代码，可以看出其用处

```swift
// 将String转Int：
let possibleNumber = "123"
let possibleNumber2 = "www.3code.info"
let convertedNumber = Int(possibleNumber) //被推测为类型 "Int?", or "optional Int"
let convertedNumber2 = Int(possibleNumber2) //不会报错的，是可选类型
```
那么convertedNumber的类型是：Int?,而不是Int，意即是不明确的。？表示可能是Int，也可能是其他类型宁，或者是不存的。

##### nil
Swift中的nil和OC的不太一样，这里并不是指针，他是指某种类型的缺省值。任意类型都可以被设置为nil，而不仅仅是对象类型。

```swift
var serverResponseCode: Int? = 404
var serverResponseCode2: Int = 404
 
serverResponseCode = nil // OK
serverResponseCode2 = nil // 报错

var surveyAnswer: String?
// 缺省的定义，value默认是nil

```
注意：nil是不能在不可选类型中使用的。

##### If Statements and Forced Unwrapping:if和强制解析

强制解析:可以在可选类型后添加!来取出其中的数值，前提是存在数值。这就是强制解析。！表示我知道这个可选类型一定有数值，请使用这个数值。
```swift
let possibleNumber = "123"
let convertedNumber = Int(possibleNumber) // 可选类型

// 可以通过if来判断上诉可选类型是否有值
if convertedNumber != nil {
    // 通过!来取出可选类型的值。
    print("convertedNumber has an integer value of \(convertedNumber!).")
    // Prints "convertedNumber has an integer value of 123."
}
```
注意：只有在确定可选类型有值之后，才可以强制解析值，否则运行时会报错

##### Optional Binding：可选绑定

可选绑定用来判断可选类型是否包含值，如果包含就把值赋给临时常变量，可以用在if while 中，如下
标准格式;
```js
if let constantName = someOptional {
    statements
}
```
如下代码表示：如果可选类型possibleNumber(Int？)包含一个数值，那么就将其值赋给新建的常量actualNumber，

```swift
let possibleNumber = "123"
if let actualNumber = Int(possibleNumber) {
    print("\"\(possibleNumber)\" has an integer value of \(actualNumber)")
} else {
    print("\"\(possibleNumber)\" could not be converted to an integer")
}
// Prints ""123" has an integer value of 123"
```

如果一个可选绑定的值是nil或者BOOL计算结果是false。那么整个if的条件都是false的。如下：

```swift

if let firstNumber = Int("4"), let secondNumber = Int("42"), firstNumber < secondNumber && secondNumber < 100 {
    print("\(firstNumber) < \(secondNumber) < 100")
}

//【上边代码等价于下边代码】 Prints "4 < 42 < 100"

if let firstNumber = Int("4") {
    if let secondNumber = Int("42") {
        if firstNumber < secondNumber && secondNumber < 100 {
            print("\(firstNumber) < \(secondNumber) < 100")
        }
    }
}

```

##### Implicitly Unwrapped Optionals：隐式可选类型
上边介绍的可选类型，可以通过if判断是否有值，然后强制解析取值，或者可选绑定取值。

但也有一种情形:隐式可选类型，定义之后就一直是值的，需要将？改为！。其实隐式可选类型是普通的可选类型，但是可以被当做非可选类型来使用，并不需要每次都是用解析来取值。一般用于类的构造中。

如下可以看出差异:隐式可选类型可以自动解析类型

```swift
// 可选类型
let possibleString: String? = "An optional string."
let forcedString: String = possibleString! 


// 隐式可选类型
let assumedString: String! = "An implicitly unwrapped optional string."
let implicitString: String = assumedString  
// 当做普通的可选类型进行判断：
if assumedString != nil {
    print(assumedString)
}
// 可选绑定：
if let definiteString = assumedString {
    print(definiteString)
}

```

注意：
- 在隐式可选类型没有数据的时候尝试取值，会触发运行时错误。
- 如果一个变量可能变成nil的话，请不要使用隐式可选类型，要使用可选类型。

### Error Handling：错误处理

可以在函数声明中添加关键字：throws。那么当函数执行遇到错误时，会抛出一个错误。
```swift
func canThrowAnError() throws {
    // this function may or may not throw an error
}
// 如果出现错误，在下边会抛出
do {
    try canThrowAnError()
    // no error was thrown
} catch {
    // an error was thrown
}
```

如下细分了错误的情况，以及相应的措施：
```swift
func makeASandwich() throws {
    // ...
}
 
do {
    try makeASandwich()
    eatASandwich()
} catch SandwichError.outOfCleanDishes {
    washDishes()
} catch SandwichError.missingIngredients(let ingredients) {
    buyGroceries(ingredients)
}
```

### Assertions：断言

可选类型可以判断值是否存在，处理缺省情况。然而当值缺省时，代码可能就没法继续了。这是需要在代码中植入断言，代码触发断言来结束运行。

格式：assert(_:_:file:line:) 这是全局函数。

##### Debugging with Assertions：使用断言调试
在调试环境中的合适位置加入断言，将有助于代码调试。
```swift
let age = -3
assert(age >= 0, "A person's age cannot be less than zero")
// 或者如下：
assert(age >= 0)
```
注意：当代码编译时，断言将会被禁用。

##### When to Use Assertions：何时使用断言

当条件可能是假，但是最终一定要保证为真，那么就得使用断言。如下情形：
- 断言是否"数组越界";
- 断言给函数传递的值，是否合法；
- 可选类型是否为nil；

注意：断言会终止程序运行。


OK.以上是基础部分。花了我整整3.5h，由于速度比较快，不能保证准确度。后续将会加快进度，略去基础的细节，呈现实用的部分。
