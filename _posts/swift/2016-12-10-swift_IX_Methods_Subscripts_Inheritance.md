---
layout: post
title:  "Swift3.0_IX_方法、下标和继承 "
categories: Swift
tags: Swift
author: 3行代码
---

* content
{:toc}

# 方法 Methods

方法是与某些特定类型相关联的函数。

- 类、结构体、枚举都可以定义实例方法；
- 实例方法为给定类型的实例封装了具体的任务与功能；

- 类、结构体、枚举也可以定义类型方法；
- 类型方法与类型本身相关联，与 OC中的类方法类似；

- 结构体和枚举能够定义方法是 Swift 与 C/OC的主要区别之一；
- 在OC中，类是唯一能定义方法的类型；；
- 但在Swift中，不仅能选择是否要定义一个类/结构体/枚举，还能灵活地在创建的类型（类/结构体/枚举）上定义方法。

1、实例方法

- 实例方法是属于某个特定类、结构体或者枚举类型实例的方法；
- 实例方法提供访问和修改实例属性的方法或提供与实例目的相关的功能，并以此来支撑实例的功能。实例方法的语法与函数完全一致
- 实例方法要写在它所属的类型的前后大括号之间
- 实例方法能够隐式访问它所属类型的所有的其他实例方法和属性
- 实例方法只能被它所属的类的某个特定实例调用
- 实例方法不能脱离于现存的实例而被调用
案例：

```swift
// Counter类，它可用于计算次数的动作时发生的数量：

class Counter {
    // 本Counter类还声明一个变量属性count，以跟踪当前计数器值。
    var count = 0
    // 实例方法:increment()递增计数器1
    func increment() {
        count += 1
    }
    // 实例方法:increment(by: Int) 将计数器增加指定的整数值
    func increment(by amount:Int) {
        count += amount
    }
    // reset() 将计数器重置为零
    func reset() {
        count = 0
    }
}

let counter = Counter()
// the initial counter value is 0
counter.increment()
// the counter's value is now 1
counter.increment(by: 5)
// the counter's value is now 6
counter.reset()
// the counter's value is now 0
```

函数参数可以同时有一个局部名称（在函数体内部使用）和一个外部名称（在调用函数时使用）(通过参数标签，参数名称是实现)。方法参数也一样，因为方法就是函数，只是这个函数与某个类型相关联了。


2、 self属性

- 类型的每一个实例都有一个隐含属性叫做self，self完全等同于该实例本身；
- 你可以在一个实例的实例方法中使用这个隐含的self属性来引用当前实例；
- 一般不用在类型实例里面写self,Swift默认指当前实例的属性或者方法；
- 一般使用self用于严谨的区分参数和属性。
```swift
// 上面例子中的increment方法还可以这样写：
func increment() {
    self.count += 1
}

// 区分参数和属性
struct Point {
    // x,y是属性
    var x = 0.0, y = 0.0
    // x是参数
    func isToTheRightOfX(x: Double) -> Bool {
        // 要进行区分:否则Swift会认为X都是参数。
        return self.x > x
    }
}
let somePoint = Point(x: 4.0, y: 5.0)
if somePoint.isToTheRightOfX(1.0) {
    print("This point is to the right of the line where x == 1.0")
}
// 打印 "This point is to the right of the line where x == 1.0"
```

3、在实例方法中修改值类型

- 结构体和枚举是值类型。默认情况下，值类型的属性不能在它的实例方法中被修改；
- 但是，如果你确实需要在某个特定的方法中修改结构体或者枚举的属性，你可以为这个方法选择可变(mutating)行为，然后就可以从其方法内部改变它的属性；
- 并且这个方法做的任何改变都会在方法执行结束时写回到原始结构中；
- 方法还可以给它隐含的self属性赋予一个全新的实例，这个新实例在方法结束时会替换现存实例；
- 要使用可变方法，将关键字mutating 放到方法的func关键字之前就可以了：

```swift
struct Point {
    var x = 0.0, y = 0.0
    mutating func moveBy(x deltaX: Double, y deltaY: Double) {
        x += deltaX
        y += deltaY
    }
}
var somePoint = Point(x: 1.0, y: 1.0)
somePoint.moveBy(x: 2.0, y: 3.0)
print("The point is now at (\(somePoint.x), \(somePoint.y))")
// Prints "The point is now at (3.0, 4.0)"
```

注意：不能在结构体类型的常量类型上调用可变方法，因为其属性不能被改变，即使属性是变量属性。

```swift
let fixedPoint = Point(x: 3.0, y: 3.0)
fixedPoint.moveByX(2.0, y: 3.0)
// 这里将会报告一个错误
```

4、在可变方法中给self赋值

可变方法能够赋给隐含属性self一个全新的实例。上面Point的例子可以用下面的方式改写：

```swift
struct Point {
    var x = 0.0, y = 0.0
    mutating func moveBy(x deltaX: Double, y deltaY: Double) {
        self = Point(x: x + deltaX, y: y + deltaY)
    }
}
// 新版的可变方法moveBy(x:y:)创建了一个新的结构体实例，它的 x 和 y 的值都被设定为目标值。调用这个版本的方法和调用上个版本的最终结果是一样的。
```

枚举的可变方法可以把self设置为同一枚举类型中不同的成员：
```swift
enum TriStateSwitch {
    case Off, Low, High
    mutating func next() {
        switch self {
        case .Off:
            self = .Low
        case .Low:
            self = .High
        case .High:
            self = .Off
        }
    }
}
var ovenLight = TriStateSwitch.Low
ovenLight.next()
// ovenLight 现在等于 .High
ovenLight.next()
// ovenLight 现在等于 .Off
```
上面的例子中定义了一个三态开关的枚举。每次调用next()方法时，开关在不同的电源状态（Off，Low，High）之间循环切换。

### 类型方法

- 实例方法是被某个类型的实例调用的方法；
- 也可以定义在类型本身上调用的方法，这种方法就叫做类型方法；
- 在方法的func关键字之前加上关键字static，来指定类型方法；
- 类还可以用关键字class来允许子类重写父类的方法实现；
- 类型方法和实例方法一样用点语法调用；

注意：
在OC中，只能为OC的类类型（classes）定义类型方法。在Swift中，你可以为所有的类、结构体和枚举定义类型方法。每一个类型方法都被它所支持的类型显式包含。


```swift
class SomeClass {
    class func someTypeMethod() {
        // 在这里实现类型方法
    }
}
SomeClass.someTypeMethod()
```

- 在类型方法的方法体（body）中，self指向这个类型本身，而不是类型的某个实例；
- 这意味着可以用self来消除类型属性和类型方法参数之间的歧义；
- 一个类型方法可以直接通过类型方法的名称调用本类中的其它类型方法，而无需在方法名称前面加上类型名称；
- 类似地，在结构体和枚举中，也能够直接通过类型属性的名称访问本类中的类型属性，而不需要前面加上类型名称；

案例：
下面的例子定义了一个名为LevelTracker结构体。它监测玩家的游戏发展情况（游戏的不同层次或阶段）。这是一个单人游戏，但也可以存储多个玩家在同一设备上的游戏信息。游戏初始时，所有的游戏等级（除了等级 1）都被锁定。每次有玩家完成一个等级，这个等级就对这个设备上的所有玩家解锁。
```swift
// LevelTracker结构体用类型属性和方法监测游戏的哪个等级已经被解锁。它还监测每个玩家的当前等级。

struct LevelTracker {
    // 类型属性：highestUnlockedLevel监测玩家已解锁的最高等级
    static var highestUnlockedLevel = 1
    // 实例属性:currentLevel来监测每个玩家当前的等级
    var currentLevel = 1

    // 类型方法：一旦新等级被解锁，它会更新highestUnlockedLevel的值
    static func unlock(_ level: Int) {
        if level > highestUnlockedLevel { highestUnlockedLevel = level }
    }
    // 如果某个给定的等级已经被解锁，它将返回true
    static func isUnlocked(_ level: Int) -> Bool {
        // 尽管没有使用类似LevelTracker.highestUnlockedLevel的写法，这个类型方法还是能够访问类型属性highestUnlockedLevel
        return level <= highestUnlockedLevel
    }
    
    // 实例方法，管理currentLevel属性
    // 这个方法会在更新currentLevel之前检查所请求的新等级是否已经解锁
    @discardableResult
    mutating func advance(to level: Int) -> Bool {
        if LevelTracker.isUnlocked(level) {
            currentLevel = level
            return true
        } else {
            return false
        }
    }
    // 在调用advance(to:)时候忽略返回值，不会产生编译警告，所以函数被标注为@ discardableResult属性
    //@discardableResult 将此属性应用于函数或方法声明以在不使用其结果的情况下调用返回值的函数或方法时抑制编译器警告。
}
```

下面，Player类使用LevelTracker来监测和更新每个玩家的发展进度：
```swift
class Player {
    var tracker = LevelTracker()
    let playerName: String
    func complete(level: Int) {
        LevelTracker.unlock(level + 1)
        tracker.advance(to: level + 1)
    }
    init(name: String) {
        playerName = name
    }
}
```
Player类创建一个新的LevelTracker实例来监测这个用户的进度。它提供了complete(level:)方法，一旦玩家完成某个指定等级就调用它。这个方法为所有玩家解锁下一等级，并且将当前玩家的进度更新为下一等级。

还可以为一个新的玩家创建一个Player的实例，然后看这个玩家完成等级一时发生了什么：
```swift
var player = Player(name: "Argyrios")
player.complete(level: 1)
print("highest unlocked level is now \(LevelTracker.highestUnlockedLevel)")
// 打印 "highest unlocked level is now 2"
```


# 下标 Subscripts

- 下标可以定义在类、结构体和枚举中，是访问集合，列表或序列中元素的快捷方式；
- 可以使用下标的索引，设置和获取值，而不需要再调用对应的存取方法；
- 举例：用下标访问一个Array实例中的元素可以写作someArray[index]；
- 举例：访问Dictionary实例中的元素可以写作someDictionary[key]；
- 一个类型可以定义多个下标，通过不同索引类型进行重载。下标不限于一维，你可以定义具有多个入参的下标满足自定义类型的需求。

1、下标语法

下标允许你通过在实例名称后面的方括号中传入一个或者多个索引值来对实例进行存取。语法类似于实例方法语法和计算型属性语法的混合。与定义实例方法类似，定义下标使用subscript关键字，指定一个或多个输入参数和返回类型；与实例方法不同的是，下标可以设定为读写或只读。这种行为由 getter 和 setter 实现，有点类似计算型属性：

```swift
subscript(index: Int) -> Int {
    get {
      // 返回一个适当的 Int 类型的值
    }

    set(newValue) {
      // 执行适当的赋值操作
    }
}
// 如同计算型属性，可以不指定 setter 的参数（newValue）。如果不指定参数，setter 会提供一个名为newValue的默认参数
```

只读下标：
```swift
// 如同只读计算型属性，可以省略只读下标的get关键字：
subscript(index: Int) -> Int {
    // 返回一个适当的 Int 类型的值
}
```

只读下标的实现：
```swift
// 这里定义了一个TimesTable结构体，用来表示传入整数的乘法表：
struct TimesTable {
    let multiplier: Int
    subscript(index: Int) -> Int {
        return multiplier * index
    }
}
let threeTimesTable = TimesTable(multiplier: 3)
print("six times three is \(threeTimesTable[6])")
// 打印 "six times three is 18"

// 在上例中，创建了一个TimesTable实例，用来表示整数3的乘法表。数值3被传递给结构体的构造函数，作为实例成员multiplier的值。
```


2、下标用法

下标的确切含义取决于使用场景。下标通常作为访问集合，列表或序列中元素的快捷方式。你可以针对自己特定的类或结构体的功能来自由地以最恰当的方式实现下标。

案例：
```swift
// Swift 的Dictionary类型实现下标用于对其实例中储存的值进行存取操作。为字典设值时，在下标中使用和字典的键类型相同的键，并把一个和字典的值类型相同的值赋给这个下标：

var numberOfLegs = ["spider": 8, "ant": 6, "cat": 4]
numberOfLegs["bird"] = 2
```
注意:Swift 的Dictionary类型的下标接受并返回可选类型的值。上例中的numberOfLegs字典通过下标返回的是一个Int?或者说“可选的int”。Dictionary类型之所以如此实现下标，是因为不是每个键都有个对应的值，同时这也提供了一种通过键删除对应值的方式，只需将键对应的值赋值为nil即可。

3、下标选项

下标可以接受任意数量的入参，并且这些入参可以是任意类型。下标的返回值也可以是任意类型。下标可以使用变量参数和可变参数，但不能使用输入输出参数，也不能给参数设置默认值。

下标的重载：一个类或结构体可以根据自身需要提供多个下标实现，使用下标时将通过入参的数量和类型进行区分，自动匹配合适的下标，这就是下标的重载。

虽然接受单一入参的下标是最常见的，但也可以根据情况定义接受多个入参的下标。例如下例定义了一个Matrix结构体，用于表示一个Double类型的二维矩阵。Matrix结构体的下标接受两个整型参数：
```swift
struct Matrix {
    let rows: Int, columns: Int
    var grid: [Double]
    init(rows: Int, columns: Int) {
        self.rows = rows
        self.columns = columns
        grid = Array(count: rows * columns, repeatedValue: 0.0)
    }
    func indexIsValidForRow(row: Int, column: Int) -> Bool {
        return row >= 0 && row < rows && column >= 0 && column < columns
    }
    subscript(row: Int, column: Int) -> Double {
        get {
            assert(indexIsValidForRow(row, column: column), "Index out of range")
            return grid[(row * columns) + column]
        }
        set {
            assert(indexIsValidForRow(row, column: column), "Index out of range")
            grid[(row * columns) + column] = newValue
        }
    }
}
```

Matrix提供了一个接受两个入参的构造方法，入参分别是rows和columns，创建了一个足够容纳rows * columns个Double类型的值的数组。通过传入数组长度和初始值0.0到数组的构造器，将矩阵中每个位置的值初始化为0.0

你可以通过传入合适的row和column的数量来构造一个新的Matrix实例：
```swift
var matrix = Matrix(rows: 2, columns: 2)
```
上例中创建了一个Matrix实例来表示两行两列的矩阵。该Matrix实例的grid数组按照从左上到右下的阅读顺序将矩阵扁平化存储：
![](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Art/subscriptMatrix01_2x.png)

将row和column的值传入下标来为矩阵设值，下标的入参使用逗号分隔：
```swift
matrix[0, 1] = 1.5
matrix[1, 0] = 3.2
```
上面两条语句分别调用下标的 setter 将矩阵右上角位置（即row为0、column为1的位置）的值设置为1.5，将矩阵左下角位置（即row为1、column为0的位置）的值设置为3.2：

![](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Art/subscriptMatrix02_2x.png)

Matrix下标的 getter 和 setter 中都含有断言，用来检查下标入参row和column的值是否有效。为了方便进行断言，Matrix包含了一个名为indexIsValidForRow(_:column:)的便利方法，用来检查入参row和column的值是否在矩阵范围内：
```swift
func indexIsValidForRow(row: Int, column: Int) -> Bool {
    return row >= 0 && row < rows && column >= 0 && column < columns
}
```
断言在下标越界时触发：
```swift
let someValue = matrix[2, 2]
// 断言将会触发，因为 [2, 2] 已经超过了 matrix 的范围
```

# 继承 Inheritance

一个类可以继承另一个类的方法，属性和其它特性。当一个类继承其它类时，继承类叫子类，被继承类叫超类（或父类）。在 Swift 中，继承是区分「类」与其它类型的一个基本特征。

在 Swift 中，类可以调用和访问超类的方法，属性和下标，并且可以重写这些方法，属性和下标来优化或修改它们的行为。Swift 会检查你的重写定义在超类中是否有匹配的定义，以此确保你的重写行为是正确的。

可以为类中继承来的属性添加属性观察器，这样一来，当属性值改变时，类就会被通知到。可以为任何属性添加属性观察器，无论它原本被定义为存储型属性还是计算型属性。

### 定义基类

不继承于其它类的类，称之为基类。

注意：Swift中的类并不是从一个通用的基类继承而来。如果你不为你定义的类指定一个超类的话，这个类就自动成为基类。

```swift
// Vehicle基类也定义了一个名为makeNoise的方法。这个方法实际上不为Vehicle实例做任何事，但之后将会被Vehicle的子类定制：

class Vehicle {
    var currentSpeed = 0.0
    var description: String {
        return "traveling at \(currentSpeed) miles per hour"
    }
    func makeNoise() {
        // 什么也不做-因为车辆不一定会有噪音
    }
}
```

可以用初始化语法创建一个Vehicle的新实例，即类名后面跟一个空括号
```swift
let someVehicle = Vehicle()
```

## 子类生成

子类生成指的是在一个已有类的基础上创建一个新的类。子类继承超类的特性，并且可以进一步完善。你还可以为子类添加新的特性。

```swift
class SomeClass: SomeSuperclass {
    // 这里是子类的定义
}
```
案例：
```swift
class Bicycle: Vehicle {
    var hasBasket = false
}

let bicycle = Bicycle()
bicycle.hasBasket = true

// 可以修改Bicycle实例所继承的currentSpeed属性，和查询实例所继承的description属性：
bicycle.currentSpeed = 15.0
print("Bicycle: \(bicycle.description)")
// 打印 "Bicycle: traveling at 15.0 miles per hour"

// 子类还可以继续被其它类继承，下面的示例为Bicycle创建了一个名为Tandem（双人自行车）的子类：
class Tandem: Bicycle {
    var currentNumberOfPassengers = 0
}

let tandem = Tandem()
tandem.hasBasket = true
tandem.currentNumberOfPassengers = 2
tandem.currentSpeed = 22.0
print("Tandem: \(tandem.description)")
// 打印："Tandem: traveling at 22.0 miles per hour"
```

### 重写 

子类可以为继承来的实例方法，类方法，实例属性，或下标提供自己定制的实现。这种行为叫重写。

如果要重写某个特性，需要在重写定义的前面加上override关键字。这么做，就表明了你是想提供一个重写版本，而非错误地提供了一个相同的定义。意外的重写行为可能会导致不可预知的错误，任何缺少override关键字的重写都会在编译时被诊断为错误。

override关键字会提醒 Swift 编译器去检查该类的超类（或其中一个父类）是否有匹配重写版本的声明。这个检查可以确保你的重写定义是正确的。

1、 访问超类的方法，属性及下标

当你在子类中重写超类的方法，属性或下标时，有时在你的重写版本中使用已经存在的超类实现会大有裨益。比如，你可以完善已有实现的行为，或在一个继承来的变量中存储一个修改过的值。

在合适的地方，你可以通过使用super前缀来访问超类版本的方法，属性或下标：
- 在方法someMethod()的重写实现中，可以通过super.someMethod()来调用超类版本的someMethod()方法。
- 在属性someProperty的 getter 或 setter 的重写实现中，可以通过super.someProperty来访问超类版本的someProperty属性。
- 在下标的重写实现中，可以通过super[someIndex]来访问超类版本中的相同下标。

2、重写方法

在子类中，你可以重写继承来的实例方法或类方法，提供一个定制或替代的方法实现

```swift
// 定义了Vehicle的一个新的子类，叫Train，它重写了从Vehicle类继承来的makeNoise()方法
class Train: Vehicle {
    override func makeNoise() {
        print("Choo Choo")
    }
}
```

3、重写属性

你可以重写继承来的实例属性或类型属性，提供自己定制的 getter 和 setter，或添加属性观察器使重写的属性可以观察属性值什么时候发生改变。

- 可以将一个继承来的只读属性重写为一个读写属性，只需要在重写版本的属性里提供 getter 和 setter 即可；
- 但是，不可以将一个继承来的读写属性重写为一个只读属性；
- 提供定制的 getter（或 setter）来重写任意继承来的属性，无论继承来的属性是存储型的还是计算型的属性；
- 子类并不知道继承来的属性是存储型的还是计算型的，它只知道继承来的属性会有一个名字和类型；
- 在重写一个属性时，必需将它的名字和类型都写出来。这样才能使编译器去检查你重写的属性是与超类中同名同类型的属性相匹配的；

注意
如果在重写属性中提供了setter，那么也一定要提供getter。如果不想在重写版本中的getter里修改继承来的属性值，可以直接通过super.someProperty来返回继承来的值，其中someProperty是要重写的属性的名字。

```swift
class Car: Vehicle {
    var gear = 1
    override var description: String {
        return super.description + " in gear \(gear)"
    }
}
// 重写的description属性首先要调用super.description返回Vehicle类的description属性。之后，Car类版本的description在末尾增加了一些额外的文本来提供关于当前档位的信息。
```

4、重写属性观察器

可以通过重写属性为一个继承来的属性添加属性观察器。当继承来的属性值发生改变时，就会被通知到，无论那个属性原本是如何实现的。

注意：不可以为继承来的常量存储型属性或继承来的只读计算型属性添加属性观察器。这些属性的值是不可以被设置的，所以，为它们提供willSet或didSet实现是不恰当。
此外还要注意，不可以同时提供重写的 setter 和重写的属性观察器。如果想观察属性值的变化，并且已经为那个属性提供了定制的 setter，那么在 setter 中就可以观察到任何值变化了。

```swift
// 下面的例子定义了一个新类叫AutomaticCar，它是Car的子类。AutomaticCar表示自动挡汽车，它可以根据当前的速度自动选择合适的挡位:
class AutomaticCar: Car {
    override var currentSpeed: Double {
        didSet {
            gear = Int(currentSpeed / 10.0) + 1
        }
    }
}
/*
当你设置AutomaticCar的currentSpeed属性，属性的didSet观察器就会自动地设置gear属性，为新的速度选择一个合适的挡位。具体来说就是，属性观察器将新的速度值除以10，然后向下取得最接近的整数值，最后加1来得到档位gear的值。例如，速度为35.0时，挡位为4：
*/

let automatic = AutomaticCar()
automatic.currentSpeed = 35.0
print("AutomaticCar: \(automatic.description)")
// 打印 "AutomaticCar: traveling at 35.0 miles per hour in gear 4"
```

### 防止重写

可以通过把方法，属性或下标标记为final来防止它们被重写，只需要在声明关键字前加上final修饰符即可（例如：final var，final func，final class func，以及final subscript）。

如果重写了带有final标记的方法，属性或下标，在编译时会报错。在类扩展中的方法，属性或下标也可以在扩展的定义里标记为 final 的。

可以通过在关键字class前添加final修饰符（final class）来将整个类标记为 final 的。这样的类是不可被继承的，试图继承这样的类会导致编译报错。

