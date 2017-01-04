---
layout: post
title:  "Swift3.0_VIII_枚举、类、结构体、属性"
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


# 类

与其他编程语言所不同的是，Swift 并不要求为自定义类和结构去创建独立的接口和实现文件。只需要在单一文件中定义一个类或者结构体，系统将会自动生成面向其它代码的外部接口。

### 类和结构体的对比

1、共同点：

- 定义要存储值的属性
- 定义方法以提供功能
- 定义下标，以使用下标语法访问其值
- 定义初始化程序以设置其初始状态
- 扩展其功能超出默认实现
- 符合协议以提供某种类型的标准功能

2、类独有功能：

- 继承使一个类能够继承另一个类的特性。
- 类型转换使您能够在运行时检查和解释类实例的类型。
- Deinitializers使一个类的实例释放它分配的任何资源。
- 引用计数允许对类实例的多个引用。

注意：结构在code中传递并不会使用引用计数：Structures are always copied when they are passed around in your code, and do not use reference counting.

不同之处：

- 与结构不同，类实例不接收默认的成员初始化

3、定义：

```swift
class SomeClass {
    // class 类定义
}
struct SomeStructure {
    // structure  结构体定义
}
```
例如：
```swift
struct Resolution {
    var width = 0
    var height = 0
}
class VideoMode {
    var resolution = Resolution()
    var interlaced = false
    var frameRate = 0.0
    var name: String?
}
```

4、实例：

```swift
let someResolution = Resolution() // 结构体实例
let someVideoMode = VideoMode()  // 类的实例
```

5、属性：

```swift
// 点语法访问:
print("The width of someResolution is \(someResolution.width)")
// Prints "The width of someResolution is 0"

// 访问子属性：
print("The width of someVideoMode is \(someVideoMode.resolution.width)")
// Prints "The width of someVideoMode is 0"

// 点语法赋值
// 与OC不同，Swift允许直接设置结构属性的子属性
someVideoMode.resolution.width = 1280
print("The width of someVideoMode is now \(someVideoMode.resolution.width)")
// Prints "The width of someVideoMode is now 1280"
```

6、结构体成员构造器：
```swift
// 与结构不同，类实例不接收默认的成员初始化
let vga = Resolution(width: 640, height: 480)
```

### 结构和枚举是值类型

- 值类型被赋予给一个变量、常量或者被传递给一个函数的时候，其值会被拷贝；
- 在 Swift 中，所有的结构体和枚举类型都是值类型。这意味着它们的实例，以及实例中所包含的任何值类型属性，在代码中传递的时候都会被复制；
- 在 Swift 中，所有的基本类型：整数（Integer）、浮点数（floating-point）、布尔值（Boolean）、字符串（string)、数组（array）和字典（dictionary），都是值类型，并且在底层都是以结构体的形式所实现；

```swift
let hd = Resolution(width: 1920, height: 1080)
// 将hd赋予ciname时，实际上是将hd储存的值copy给ciname，存在ciname实例中。结果就是两个完全独立的实例恰巧包含相同的数值。
var cinema = hd
cinema.width = 2048
```
枚举也遵守相同的规则
```swift
enum CompassPoint {
    case north, south, east, west
}
var currentDirection = CompassPoint.west
let rememberedDirection = currentDirection
currentDirection = .east
if rememberedDirection == .west {
    print("The remembered direction is still .west")
}
// Prints "The remembered direction is still .west"
```

### 类是引用类型

与值类型不同，引用类型在被赋予到一个变量、常量或者被传递到一个函数时，其值不会被拷贝。因此，引用的是已存在的实例本身而不是其拷贝

```swift
let tenEighty = VideoMode()
tenEighty.resolution = hd
tenEighty.interlaced = true
tenEighty.name = "1080i"
tenEighty.frameRate = 25.0

let alsoTenEighty = tenEighty
alsoTenEighty.frameRate = 30.0

print("The frameRate property of tenEighty is now \(tenEighty.frameRate)")
// Prints "The frameRate property of tenEighty is now 30.0"

// 因为类是引用类型，所以tenEight和alsoTenEight实际上引用的是相同的VideoMode实例。换句话说，它们是同一个实例的两种叫法。
```

注意：tenEighty和alsoTenEighty被声明为常量而不是变量。他们存的是指针，只要指针不改变就ok，指针指向的实例的属性是可以改变的。

### 恒等运算符
因为类是引用类型，有可能有多个常量和变量在幕后同时引用同一个类实例。（对于结构体和枚举来说，这并不成立。因为它们作为值类型，在被赋予到常量、变量或者传递到函数时，其值总是会被拷贝。）

如果能够判定两个常量或者变量是否引用同一个类实例将会很有帮助。为了达到这个目的，Swift 内建了两个恒等运算符：
 
- 等价于（===）  “等价于”表示两个类类型（class type）的常量或者变量引用同一个类实例。
- 不等价于（!==）
- 以上和=，!=是不同的，注意区分

```swift
if tenEighty === alsoTenEighty {
    print("tenEighty and alsoTenEighty refer to the same VideoMode instance.")
}
// Prints "tenEighty and alsoTenEighty refer to the same VideoMode instance."
```
### 指针

- Swift弱化了指针的概念，其实这里的引用类型，就是指针的体现。
- 注意Swift中的指针，与C中的指针类似，但并不直接指向某个内存地址，不要求用*来表明创建的是一个引用

### 选择类还是结构体

- 结构体实例总是通过值传递，类实例总是通过引用传递。这意味两者适用不同的任务

构建结构体的条件：
- 该数据结构的主要目的是用来封装少量相关简单数据值。
- 有理由预计该数据结构的实例在被赋值或传递时，封装的数据将会被拷贝而不是被引用。
- 该数据结构中储存的值类型属性，也应该被拷贝，而不是被引用。
- 该数据结构不需要去继承另一个既有类型的属性或者行为。
或者：
- 几何形状的大小，封装一个width属性和height属性，两者均为Double类型。
- 一定范围内的路径，封装一个start属性和length属性，两者均为Int类型。
- 三维坐标系内一点，封装x，y和z属性，三者均为Double类型。

在所有其它案例中，定义一个类，生成一个它的实例，并通过引用来管理和传递。实际中，这意味着绝大部分的自定义数据构造都应该是类，而非结构体。

### 字符串、数组、和字典类型的赋值与复制

- Swift 中，许多基本类型，诸如String，Array和Dictionary类型均以结构体的形式实现。这意味着被赋值给新的常量或变量，或者被传入函数或方法中时，它们的值会被拷贝。

- Objective-C 中NSString，NSArray和NSDictionary类型均以类的形式实现，而并非结构体。它们在被赋值或者被传入函数或方法时，不会发生值拷贝，而是传递现有实例的引用。

注意：
以上是对字符串、数组、字典的“拷贝”行为的描述。在你的代码中，拷贝行为看起来似乎总会发生。然而，Swift 在幕后只在绝对必要时才执行实际的拷贝。Swift 管理所有的值拷贝以确保性能最优化，所以你没必要去回避赋值来保证性能最优化。

# 属性：Properties

- 属性：将值跟特定的类、结构或枚举关联；
- 存储属性：存储常量或变量作为实例的一部分。存储属性只能用于类和结构体；
- 计算属性：计算一个值。计算属性可以用于类、结构体和枚举；
- 类型属性：属性也可以直接作用于类型本身，这种属性称为类型属性；
- 属性观察器：监控属性值的变化，以此来触发一个自定义的操作；
- 属性观察器可以添加到自己定义的存储属性上，也可以添加到从父类继承的属性上。
- 常量属性在构造过程完成之前必须要有初始值，变量属性则无。

### 储存属性 Stored Properties

简单来说，一个存储属性就是存储在特定类或结构体实例里的一个常量或变量。存储属性可以是变量存储属性（用关键字var定义），也可以是常量存储属性（用关键字let定义）。

1、在定义属性的时候指定默认值:默认构造器

```swift
class ShoppingListItem {
    var name: String?
    var quantity = 1
    var purchased = false
}
var item = ShoppingListItem()
```

2、在构造过程中设置或者修改储存属性的值，甚至是常量的储存属性的值：

```swift
class SurveyQuestion {
    let text: String
    var response: String?
    init(text: String) {
        self.text = text
    }
    func ask() {
        print(text)
    }
}
// 修改常量属性的值：
let beetsQuestion = SurveyQuestion(text: "How about beets?")
beetsQuestion.ask()
// 打印 "How about beets?"
beetsQuestion.response = "I also like beets. (But not with cheese.)"
```

3、常量结构体的储存属性：

```swift
// 属性
struct FixedLengthRange {
    // 变量属性
    var firstValue: Int
    // 常量属性
    let length: Int
}
// 使用默认构造器：
var rangeOfThreeItems = FixedLengthRange(firstValue: 0, length: 3)
// 修改变量属性
rangeOfThreeItems.firstValue = 6
// 不能修改常量属性，error
rangeOfThreeItems.length = 10

// 常量结构体:变量属性不可以修改
// 如果创建了一个结构体的实例并将其赋值给一个常量，则无法修改该实例的任何属性，即使有属性被声明为变量也不行：
let hd = rangeOfThreeItems
hd.firstValue = 2048 
// this will report an error, even though firstValue is a variable property


```

总结：
- 结构体是值类型；当器声明为常量的时候，其所有属性，皆是常量，皆不可修改；
- 类则不同，他是引用类型，即便是定义为常量，也不影响修改属性；
- 变量结构体中的常量属性，不能被修改，变量属性可以修改；
- 常量结构体中的常变量属性皆不可修改；

4、延时储存属性

延迟存储属性是指当第一次被调用的时候才会计算其初始值的属性。

- 在属性声明前使用lazy来标示一个延迟存储属性；
- 必须将延迟存储属性声明成变量（var），因为属性的初始值可能在实例构造完成之后才会得到。而常量属性在构造过程完成之前必须要有初始值，因此无法声明成延迟属性。

案例：
```swift
// DataImporter 是一个负责将外部文件中的数据导入的类。这个类的初始化会消耗不少时间。
class DataImporter {
    var fileName = "data.txt"
    // 这里会提供数据导入功能
}
class DataManager {
    lazy var importer = DataImporter()
    // data属性初始值是一个空的字符串（String）数组
    var data = [String]()
    // 这里会提供数据管理功能
}
let manager = DataManager()
manager.data.append("Some data")
manager.data.append("Some more data")
// the DataImporter instance for the importer property has not yet been created

// 只有使用的时候才会创建
print(manager.importer.fileName)
// Prints "data.txt"
```

注意：如果一个被标记为lazy的属性在没有初始化时就同时被多个线程访问，则无法保证该属性只会被初始化一次。

5、存储属性和实例变量

OC为类实例存储值和引用提供两种方法，除了属性之外，还可以使用实例变量作为属性值的后端存储。

Swift编程语言中把这些理论统一用属性来实现。Swift中的属性没有对应的实例变量，属性的后端存储也无法直接访问。这就避免了不同场景下访问方式的困扰，同时也将属性的定义简化成一个语句。属性的全部信息——包括命名、类型和内存管理特征——都在唯一一个地方（类型定义中）定义。

### 计算属性 Computed Properties

除存储属性外，类、结构体和枚举可以定义计算属性。计算属性不直接存储值，而是提供一个getter 和一个可选的setter，来间接获取和设置其他属性或变量的值。

```swift
// Point 封装了一个 (x, y) 的坐标
struct Point {
    var x = 0.0, y = 0.0
}

// Size 封装了一个 width 和一个 height
struct Size {
    var width = 0.0, height = 0.0
}

// Rect 表示一个有原点和尺寸的矩形
struct Rect {
    var origin = Point()
    var size = Size()
// 一个矩形的中心点可以从原点（origin）和大小（size）算出，所以不需要将它以显式声明的 Point来保存
    var center: Point {
        get {
            let centerX = origin.x + (size.width / 2)
            let centerY = origin.y + (size.height / 2)
            return Point(x: centerX, y: centerY)
        }
        set(newCenter) {
            origin.x = newCenter.x - (size.width / 2)
            origin.y = newCenter.y - (size.height / 2)
        }
    }
}
var square = Rect(origin: Point(x: 0.0, y: 0.0),
                  size: Size(width: 10.0, height: 10.0))
let initialSquareCenter = square.center
square.center = Point(x: 15.0, y: 15.0)
print("square.origin is now at (\(square.origin.x), \(square.origin.y))")
// Prints "square.origin is now at (10.0, 10.0)"
```

![](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Art/computedProperties_2x.png)


2、简化setter声明
如果计算属性的 setter 没有定义表示新值的参数名，则可以使用默认名称 newValue。下面是使用了简化 setter 声明的 Rect 结构体代码：

```swift
struct AlternativeRect {
    var origin = Point()
    var size = Size()
    var center: Point {
        get {
            let centerX = origin.x + (size.width / 2)
            let centerY = origin.y + (size.height / 2)
            return Point(x: centerX, y: centerY)
        }
        set {
            origin.x = newValue.x - (size.width / 2)
            origin.y = newValue.y - (size.height / 2)
        }
    }
}
```

3、只读属性

只有getter没有setter的计算属性就是只读计算属性。

- 只读计算属性总是返回一个值，可以通过点运算符访问，但不能设置新的值。
- 必须使用var关键字定义计算属性，包括只读计算属性，因为它们的值不是固定的。
- let关键字只用来声明常量属性，表示初始化后再也无法修改的值。

只读计算属性的声明可以去掉 get 关键字和花括号：
```swift
struct Cuboid {
    var width = 0.0, height = 0.0, depth = 0.0
    var volume: Double {
        return width * height * depth
    }
}
let fourByFiveByTwo = Cuboid(width: 4.0, height: 5.0, depth: 2.0)
print("the volume of fourByFiveByTwo is \(fourByFiveByTwo.volume)")
// Prints "the volume of fourByFiveByTwo is 40.0"
/*
结构体还有一个名为 volume 的只读计算属性用来返回立方体的体积。为 volume 提供 setter 毫无意义，因为无法确定如何修改 width、height 和 depth 三者的值来匹配新的 volume。然而，Cuboid 提供一个只读计算属性来让外部用户直接获取体积是很有用的。*/
```


### 属性观察器 Property Observers

属性观察器监控和响应属性值的变化，每次属性被设置值的时候都会调用属性观察器，即使新值和当前值相同的时候也不例外。

- 可以为除了延迟存储属性之外的其他存储属性添加属性观察器;
- 也可以通过重写属性的方式为继承的属性（包括存储属性和计算属性）添加属性观察器
- 你不必为非重写的计算属性添加属性观察器，因为可以通过它的 setter 直接监控和响应值的变化

可以为属性添加如下的一个或全部观察器：
- willSet 在新的值被设置之前调用:willSet 观察器会将新的属性值作为常量参数传入，在 willSet 的实现代码中可以为这个参数指定一个名称，如果不指定则参数仍然可用，这时使用默认名称 newValue 表示。

- didSet 在新的值被设置之后立即调用:didSet 观察器会将旧的属性值作为参数传入，可以为该参数命名或者使用默认参数名 oldValue。如果在 didSet 方法中再次对该属性赋值，那么新值会覆盖旧的值。

- 父类的属性在子类的构造器中被赋值时，它在父类中的 willSet 和 didSet 观察器会被调用，随后才会调用子类的观察器
- 在父类初始化方法调用之前，子类给属性赋值时，观察器不会被调用
- 默认值 oldValue 表示旧值的参数名。

案例：
```swift
class StepCounter {
// 定义了一个 Int 类型的属性 totalSteps，它是一个存储属性，包含 willSet 和 didSet 观察器。
    var totalSteps: Int = 0 {
        willSet(newTotalSteps) {
            print("About to set totalSteps to \(newTotalSteps)")
        }
        didSet {
            if totalSteps > oldValue  {
                print("Added \(totalSteps - oldValue) steps")
            }
        }
    }
}
let stepCounter = StepCounter()
// 当 totalSteps 被设置新值的时候，它的 willSet 和 didSet 观察器都会被调用，即使新值和当前值完全相同时也会被调用。

stepCounter.totalSteps = 200
// About to set totalSteps to 200
// Added 200 steps
stepCounter.totalSteps = 360
// About to set totalSteps to 360
// Added 160 steps
stepCounter.totalSteps = 896
// About to set totalSteps to 896
// Added 536 steps
```

注意：如果将属性通过 in-out 方式传入函数，willSet 和 didSet 也会调用。这是因为 in-out 参数采用了拷入拷出模式：即在函数内部使用的是参数的 copy，函数结束后，又对参数重新赋值。


### 全局变量和局部变量 Global and Local Variables

计算属性和属性观察器所描述的功能也可以用于全局变量和局部变量。全局变量是在函数、方法、闭包或任何类型之外定义的变量。局部变量是在函数、方法或闭包内部定义的变量。

- 全局或局部变量都属于存储型变量，跟存储属性类似，它为特定类型的值提供存储空间，并允许读取和写入。
- 在全局或局部范围都可以定义计算型变量和为存储型变量定义观察器。计算型变量跟计算属性一样，返回一个计算结果而不是存储值，声明格式也完全一样。
- 全局的常量或变量都是延迟计算的，跟延迟存储属性相似，
- 全局的常量或变量不需要标记lazy修饰符。
- 局部范围的常量或变量从不延迟计算。

### 类型属性 Type Properties

类型属性用于定义某个类型所有实例共享的数据，比如所有实例都能用的一个常量（就像 C 语言中的静态常量），或者所有实例都能访问的一个变量（就像 C 语言中的静态变量）。

- 实例属性属于一个特定类型的实例，每创建一个实例，实例都拥有属于自己的一套属性值，实例之间的属性相互独立。
- 也可以为类型本身定义属性，无论创建了多少个该类型的实例，这些属性都只有唯一一份。这种属性就是类型属性。
- 存储型类型属性可以是变量或常量，计算型类型属性跟实例的计算型属性一样只能定义成变量属性。
- 跟实例的存储型属性不同，必须给存储型类型属性指定默认值，因为类型本身没有构造器，也就无法在初始化过程中使用构造器给类型属性赋值。
- 存储型类型属性是延迟初始化的，它们只有在第一次被访问的时候才会被初始化。即使它们被多个线程同时访问，系统也保证只会对其进行一次初始化，并且不需要对其使用 lazy 修饰符。

1、类型属性语法

在C或OC中，与某个类型关联的静态常量和静态变量，是作为全局（global）静态变量定义的。但是在 Swift 中，类型属性是作为类型定义的一部分写在类型最外层的花括号内，因此它的作用范围也就在类型支持的范围内。

使用关键字 static 来定义类型属性。在为类定义计算型类型属性时，可以改用关键字 class 来支持子类对父类的实现进行重写。

```swift
// 下面的例子演示了存储型和计算型类型属性的语法：
struct SomeStructure {
    static var storedTypeProperty = "Some value."
    static var computedTypeProperty: Int {
        return 1
    }
}
enum SomeEnumeration {
    static var storedTypeProperty = "Some value."
    static var computedTypeProperty: Int {
        return 6
    }
}
class SomeClass {
    static var storedTypeProperty = "Some value."
    static var computedTypeProperty: Int {
        return 27
    }
    class var overrideableComputedTypeProperty: Int {
        return 107
    }
}
```
注意：
例子中的计算型类型属性是只读的，但也可以定义可读可写的计算型类型属性，跟计算型实例属性的语法相同。

2、获取和设置类型属性的值

跟实例属性一样，类型属性也是通过点运算符来访问。但是，类型属性是通过类型本身来访问，而不是通过实例。比如：
```swift
print(SomeStructure.storedTypeProperty)
// Prints "Some value."
SomeStructure.storedTypeProperty = "Another value."
print(SomeStructure.storedTypeProperty)
// Prints "Another value."
print(SomeEnumeration.computedTypeProperty)
// Prints "6"
print(SomeClass.computedTypeProperty)
// Prints "27"
```

下面的例子定义了一个结构体，使用两个存储型类型属性来表示两个声道的音量，每个声道具有 0 到 10 之间的整数音量。下图展示了如何把两个声道结合来模拟立体声的音量。当声道的音量是 0，没有一个灯会亮；当声道的音量是 10，所有灯点亮。本图中，左声道的音量是 9，右声道的音量是 7
![](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Art/staticPropertiesVUMeter_2x.png)

上面所描述的声道模型使用 AudioChannel 结构体的实例来表示：
```swift
struct AudioChannel {
    // 存储型类型属性：音量的最大上限阈值
    static let thresholdLevel = 10
    // 存储型类型属性：表示所有 AudioChannel 实例的最大音量，初始值是0
    static var maxInputLevelForAllChannels = 0
    // 存储型实例属,包含 didSet 属性观察器来检查每次设置后的属性值。
    var currentLevel: Int = 0 {
        didSet {
            if currentLevel > AudioChannel.thresholdLevel {
                // 将当前音量限制在阀值之内
                currentLevel = AudioChannel.thresholdLevel
            }
            if currentLevel > AudioChannel.maxInputLevelForAllChannels {
                // 存储当前音量作为新的最大输入音量
                AudioChannel.maxInputLevelForAllChannels = currentLevel
            }
        }
    }
}

// 可以使用结构体 AudioChannel 创建两个声道 leftChannel 和rightChannel，用以表示立体声系统的音量：
var leftChannel = AudioChannel()
var rightChannel = AudioChannel()

// 如果将左声道的 currentLevel 设置成 7，类型属性 maxInputLevelForAllChannels 也会更新成 7：
leftChannel.currentLevel = 7
print(leftChannel.currentLevel)
// Prints "7"
print(AudioChannel.maxInputLevelForAllChannels)
// Prints "7"

// 如果试图将右声道的 currentLevel 设置成 11，它会被修正到最大值 10，同时 maxInputLevelForAllChannels 的值也会更新到 10：
rightChannel.currentLevel = 11
print(rightChannel.currentLevel)
// Prints "10"
print(AudioChannel.maxInputLevelForAllChannels)
// Prints "10"
```

