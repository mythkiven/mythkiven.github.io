---
layout: post
title:  "Swift3.0_VII_闭包"
categories: Swift3.0
tags: Swift3.0
author: 3行代码
---

* content
{:toc}

## 前言

**闭包：独立的代码块，可以传递及引用。**

- 闭包可以捕获和存储其所在上下文中任意常量和变量的引用；
- Swift会管理捕获过程中涉及到的所有内存管理；

闭包的形式：
- 全局函数都是闭包，特点是有函数名但没有捕获任何值。
- 嵌套函数都是闭包，特点是有函数名，并且可以在它封闭的函数中捕获值。
- 闭包表达式都是闭包，特点是没有函数名，可以使用轻量的语法在它所围绕的上下文中捕获值。

语法优化：
- 根据上下文推理参数及返回值类型
- 隐式返回单表达式闭包，即单表达式闭包可以省略 return 关键字
- 参数名称缩写
- 尾随闭包语法

### 闭包表达式

闭包表达式是一种编写内联闭包的方式，它简洁、紧凑。闭包表达式提供了数种语义优化，为的是以最简单的形式编程而不需要大量的声明或意图。以下以同一个sort函数进行几次改进，每次函数都更加简洁，以此说明闭包表达式的优化

##### 闭包表达式语法
闭合表达式语法具有以下一般构造形式：

```swift
{ (parameters) -> return type in
    statements
}
```
闭包表达式语法可以使用常量参数、变量参数和 inout 类型作为参数，但皆不可提供默认值。

##### sorted函数

Swift的标准函数库提供了一个名为sort的函数,他用于输出类型排序的闭包函数，给已知类型的数组数据的值排序:
```swift
public func sorted(by areInIncreasingOrder: (Element, Element) -> Bool)  ->  [Element]

使用:
 let students: Set = ["Kofi", "Abena", "Peter", "Kweku", "Akosua"]
 let descendingStudents = students.sorted(by: >)
 print(descendingStudents) //1 Prints "["Peter", "Kweku", "Kofi", "Akosua", "Abena"]"
 print(students.sorted()) //2 Prints "["Abena", "Akosua", "Kofi", "Kweku", "Peter"]"
 print(students.sorted(by: <)) //3  Prints "["Abena", "Akosua", "Kofi", "Kweku", "Peter"]"

```

1、先看一个排序的例子：

```swift
let names = ["Chris", "Alex", "Ewa", "Barry", "Daniella"]
func backward(_ s1: String, _ s2: String) -> Bool {
    return s1 > s2
}
var reversedNames = names.sorted(by: backward)
// reversedNames is equal to ["Ewa", "Daniella", "Chris", "Barry", "Alex"]

```


2、闭包表达式来改写上边的例子：
```swift
reversed = sorted(names, { (s1: String, s2: String) -> Bool in
return s1 > s2
})
```

3、可以用一行来表达：
```swift
reversedNames = names.sorted(by: { (s1: String, s2: String) -> Bool in return s1 > s2 } )
```

4、Swift可以根据上下文推断类型，因此可以省略String:
```swift
reversedNames = names.sorted(by: { s1, s2 in return s1 > s2 } )
```

5、单行表达式闭包可以通过省略 return 关键字来隐式返回单行表达式的结果
```swift
reversedNames = names.sorted(by: { s1, s2 in s1 > s2 } )
```

6、参数名简写:

-  Swift 自动为内联闭包提供了参数名称简写功能，
-  可以直接通过 $0,$1,$2等名字来引用闭包的参数值
-  关键字:in、return 可以省略,此时闭包表达式完全由闭包函数体构成
-  下边例子;$0和$1表示闭包中第一个和第二个String类型的参数。
```swift
reversedNames = names.sorted(by: { $0 > $1 } )
```

7、运算符函数：
Swift为String类型定义了大于算子（>），通过大于算子简写如下

```swift
reversedNames = names.sorted(by: >)
```

8、Trailing 闭包
```swift
reversedNames = names.sorted() { $0 > $1 }
```

总结，如下：
```swift
let names = ["Chris", "Alex", "Ewa", "Barry", "Daniella"]
func backward(_ s1: String, _ s2: String) -> Bool {
    return s1 > s2
}

var ss1 = names.sorted(by: backward)
var ss2 = names.sorted(by: {(s1:String,s2:String) -> Bool in return s1>s2 })
var ss3 = names.sorted(by: { s1, s2 in return s1 > s2 } )
var ss4 = names.sorted(by: { s1, s2 in s1 > s2 } )
var ss5 = names.sorted(by: { $0 > $1 } )
var ss6 = names.sorted(by: >)
var ss7 = names.sorted() { $0 > $1 }
以上 ss1到ss7结果相同，都是 ["Ewa", "Daniella", "Chris", "Barry", "Alex"]
```


### 尾随闭包(Trailing Closures)

如果需要将一个很长的闭包表达式作为最后一个参数传递给函数，可以使用trailing闭包来替代。Trailing 闭包是一个书写在函数括号之外(之后)的闭包表达式，函数支持将其作为最后一个参数调用。
```swift
func someFunctionThatTakesAClosure(closure: () -> Void) {
    // function body goes here
}
// Here's how you call this function without using a trailing closure:
someFunctionThatTakesAClosure(closure: {
    // closure's body goes here
})
// Here's how you call this function with a trailing closure instead:
someFunctionThatTakesAClosure() {
    // trailing closure's body goes here
}
```


上述的sort函数可以改写为：
```swift
reversedNames = names.sorted() { $0 > $1 }
```

如果一个封闭表达式是作为函数或方法唯一的参数,可以忽略()
```swift
reversedNames = names.sorted { $0 > $1 }
```


当闭包非常长以至于不能在一行中进行书写时，尾随闭包变得非常有用。

举例来说，Swift的Array类型有一个map方法，其获取一个闭包表达式作为其唯一参数。 数组中的每一个元素调用一次该闭包函数，并返回该元素所映射的值(也可以是不同类型的值)。 具体的映射方式和返回值类型由闭包来指定。

当提供给数组闭包函数后，map方法将返回一个新的数组，数组中包含了与原数组一一对应的映射后的值，
下例介绍了如何在map方法中使用尾随闭包将Int类型数组[16,58,510]转换为包含对应String类型的数组["OneSix", "FiveEight", "FiveOneZero"]:

```swift
let digitNames = [
    0: "Zero", 1: "One", 2: "Two", 3: "Three", 4: "Four",
    5: "Five", 6: "Six", 7: "Seven", 8: "Eight", 9: "Nine"
]
let numbers = [16, 58, 510]
```

通过调用numbers.map来调用映射后的String数组
使用如下：
```swift
let strings = numbers.map {
    (number) -> String in
    var number = number
    var output = ""
    repeat {
        output = digitNames[number % 10]! + output
        number /= 10
    } while number > 0
    return output
}
// strings is inferred to be of type [String]
// its value is ["OneSix", "FiveEight", "FiveOneZero"]
```
注意： 字典digitNames下标后跟着一个叹号 (!)，因为字典下标返回一个可选值 (optional value)，表明即使该 key不存在也不会查找失败。 在上例中，它保证了number % 10可以总是作为一个 digitNames字典的有效下标key。 因此叹号可以用于解析存储在可选下标项中的String类型值。



### 值捕获：Capturing Values

闭包可以在其定义的范围内捕捉（引用/得到）常量和变量，闭包可以引用和修改这些值，即使定义的常量和变量已经不复存在了依然可以修改和引用。

Swift最简单的闭包形式是嵌套函数，也就是定义在其他函数的函数体内的函数。 嵌套函数可以捕获其外部函数所有的参数以及定义的常量和变量。


下例解说：

makeIncrementor函数：
- 1、makeIncrementor返回类型为() -> Int。其返回的是一个闭包：incrementor；
- 2、其内嵌incrementor嵌套函数，参数为amount；

incrementor嵌套函数：
- 1、没有参数，从上下文中捕获了两个值：runningTotal和amount；
- 2、runningTotal在闭包种被引用指针，amount在闭包中引用副本；
- 3、作为闭包被当做返回值返回

makeIncrementer(forIncrement: 10)
- makeIncrementer(forIncrement:)函数每次调用都会增加10；

incrementByTen：
- 使用makeIncrementer(forIncrement:)定义的整型常量；
- 该常量指向一个每次调用会加10的incrementor函数；
- 每次调用incrementByTen，即：调用incrementor时，runningTotal都会自动增加amount，并返回。


```swift
func makeIncrementer(forIncrement amount: Int) -> () -> Int {
    var runningTotal = 0
    func incrementer() -> Int {
        runningTotal += amount
        print("amount:\(amount) runningTotal:\(runningTotal)");
        return runningTotal
    }
    print("runningTotal:\(runningTotal)");
    return incrementer
}

var incrementByTen = makeIncrementer(forIncrement: 10)

incrementByTen() // 10
incrementByTen() // 20

// runningTotal:0
// amount:10 runningTotal:10
// amount:10 runningTotal:20
```

注意:如果将闭包赋值给一个类实例的属性，并且该闭包通过访问该实例或其成员而捕获了该实例，则将在闭包和该实例间创建一个循环强引用。Swift使用捕获列表来打破这种循环强引用。

### 闭包是引用类型

- 在上述例子中，incrementByTen是常量，但常量指向的闭包仍然可以增加其捕获的变量runningTotal值，因为函数闭包都是引用类型。

- 无论将函数/闭包赋值给一个常量还是变量，实际上都是将常量/变量的值设置为对应函数/闭包的引用。 上面的例子中，incrementByTen指向闭包的引用是一个常量，而并非闭包内容本身。

- 这也意味着如果将闭包赋值给了两个不同的常量/变量，两个值都会指向同一个闭包：
alsoIncrementByTen指向makeIncrementer(forIncrement: 10)。

```swift
let alsoIncrementByTen = incrementByTen
alsoIncrementByTen()
// returns a value of 50

```

### 闭包逃逸：Escaping Closures

当一个闭包作为参数传到一个函数中，但是这个闭包在函数返回之后才被执行，即：该闭包从函数中逃逸。

- 定义逃逸闭包： 当定义接受闭包作为参数的函数时，在参数名之前标注@escaping，用来指明这个闭包是允许“逃逸”出这个函数的。
- 一种能使闭包“逃逸”出函数的方法是，将这个闭包保存在一个函数外部定义的变量中。
 举个例子，很多启动异步操作的函数接受一个闭包参数作为 completion handler。这类函数会在异步操作开始之后立刻返回，但是闭包直到异步操作结束后才会被调用。在这种情况下，闭包需要“逃逸”出函数，因为闭包需要在函数返回之后被调用。例如：

```swift
var completionHandlers: [() -> Void] = []
func someFunctionWithEscapingClosure(completionHandler: @escaping () -> Void) {
    completionHandlers.append(completionHandler)
}
/* someFunctionWithEscapingClosure(_:) 函数接受一个闭包作为参数，该闭包被添加到一个函数外定义的数组中。如果你不将这个参数标记为 @escaping，就会得到一个编译错误 */
```

- 将一个闭包标记为 @escaping 意味着必须在闭包中显式地引用self。比如说，在下面的代码中，传递到 someFunctionWithEscapingClosure(_:) 中的闭包是一个逃逸闭包，这意味着它需要显式地引用 self。相对的，传递到 someFunctionWithNonescapingClosure(_:) 中的闭包是一个非逃逸闭包，这意味着它可以隐式引用 self。
```swift
func someFunctionWithNonescapingClosure(closure: () -> Void) {
    closure()
}
 
class SomeClass {
    var x = 10
    func doSomething() {
        someFunctionWithEscapingClosure { self.x = 100 }
        someFunctionWithNonescapingClosure { x = 200 }
    }
}
 
let instance = SomeClass()
instance.doSomething()
print(instance.x)
// Prints "200"
 
completionHandlers.first?()
print(instance.x)
// Prints "100"

```

### 自动闭包

自动闭包是一种自动创建的闭包，用于包装传递给函数作为参数的表达式。这种闭包不接受任何参数，当它被调用的时候，会返回被包装在其中的表达式的值。这种便利语法让你能够省略闭包的花括号，用一个普通的表达式来代替显式的闭包。

我们经常会调用采用自动闭包的函数，但是很少去实现这样的函数。举个例子来说，assert(condition:message:file:line:) 函数接受自动闭包作为它的 condition 参数和 message 参数；它的 condition 参数仅会在 debug 模式下被求值，它的 message 参数仅当 condition 参数为 false 时被计算求值。

自动闭包让你能够延迟求值，因为直到你调用这个闭包，代码段才会被执行。延迟求值对于那些有副作用（Side Effect）和高计算成本的代码来说是很有益处的，因为它使得你能控制代码的执行时机。下面的代码展示了闭包如何延时求值。

```swift
var customersInLine = ["Chris", "Alex", "Ewa", "Barry", "Daniella"]
print(customersInLine.count)
// Prints "5"
 
let customerProvider = { customersInLine.remove(at: 0) }
print(customersInLine.count)
// Prints "5"
 
print("Now serving \(customerProvider())!")
// Prints "Now serving Chris!"
print(customersInLine.count)
// Prints "4"
/*
尽管在闭包的代码中，customersInLine 的第一个元素被移除了，不过在闭包被调用之前，这个元素是不会被移除的。如果这个闭包永远不被调用，那么在闭包里面的表达式将永远不会执行，那意味着列表中的元素永远不会被移除。请注意，customerProvider 的类型不是 String，而是 () -> String，一个没有参数且返回值为 String 的函数。
*/
```

将闭包作为参数传递给函数时，能获得同样的延时求值行为。
```swift
// customersInLine is ["Alex", "Ewa", "Barry", "Daniella"]
func serve(customer customerProvider: () -> String) {
    print("Now serving \(customerProvider())!")
}
serve(customer: { customersInLine.remove(at: 0) } )
// Prints "Now serving Alex!"
```

下面这个版本的 serve(customer:) 完成了相同的操作，不过它并没有接受一个显式的闭包，而是通过将参数标记为 @autoclosure 来接收一个自动闭包。现在你可以将该函数当作接受 String 类型参数（而非闭包）的函数来调用。customerProvider 参数将自动转化为一个闭包，因为该参数被标记了 @autoclosure 特性。
```swift
// customersInLine is ["Ewa", "Barry", "Daniella"]
func serve(customer customerProvider: @autoclosure () -> String) {
    print("Now serving \(customerProvider())!")
}
serve(customer: customersInLine.remove(at: 0))
// Prints "Now serving Ewa!"
```

注意 过度使用 autoclosures 会让你的代码变得难以理解。上下文和函数名应该能够清晰地表明求值是被延迟执行的。

如果你想让一个自动闭包可以“逃逸”，则应该同时使用 @autoclosure 和 @escaping 属性。
```swift
// customersInLine is ["Barry", "Daniella"]
var customerProviders: [() -> String] = []
func collectCustomerProviders(_ customerProvider: @autoclosure @escaping () -> String) {
    customerProviders.append(customerProvider)
}
collectCustomerProviders(customersInLine.remove(at: 0))
collectCustomerProviders(customersInLine.remove(at: 0))
 
print("Collected \(customerProviders.count) closures.")
// Prints "Collected 2 closures."
for customerProvider in customerProviders {
    print("Now serving \(customerProvider())!")
}
// Prints "Now serving Barry!"
// Prints "Now serving Daniella!"
```
在上面的代码中，collectCustomerProviders(_:) 函数并没有调用传入的 customerProvider 闭包，而是将闭包追加到了 customerProviders 数组中。这个数组定义在函数作用域范围外，这意味着数组内的闭包能够在函数返回之后被调用。因此，customerProvider 参数必须允许“逃逸”出函数作用域。


