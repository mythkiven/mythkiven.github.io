---
layout: post
title:  "Swift3.0_IV_集合"
categories: Swift3.0
tags: Swift3.0
author: 3行代码
---

* content
{:toc}

Swift 提供了三种集合类：
- 数组 arrays 值有序的集合；
- 集合 sets 值唯一而无序的集合;
- 字典 dictionaries 键值对且无序的集合；
- 这三种集合是泛型集合，集合中储存的数值必须是明确的

如下示意：

![](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Art/CollectionTypes_intro_2x.png)


### 1、Mutability of Collections：可变集合

- 如果你创建了一个数组,set或者字典，并且申明他为变量，那么该集合就是可变的。
- 如果你申明了数组，set或者字典为常量，那么他就没办法修改。他的内容和大小都没法修改。
- 如果集合不需要修改，尽量申明成常量。这样编译器会优化集合性能。

### 2、Arrays：数组

- 数组使用有序列表存储同一类型的多个值。相同的值可以多次出现在一个数组的不同位置中；
- swift中的Array 是桥接自Foundation的NSArray；

##### 2.1 Array Type Shorthand Syntax：数组简写

- Swift数组应遵循Array<SomeType>这样的形式，其中SomeType是这个数组中唯一允许存在的数据类型；
- 简写方式：[SomeType]这样的简单语法；
- 两种形式在功能上是一样的，一般都用简写形式

##### 2.2 Creating an Empty Array：创建空数组

```swift
// Int类型的空数组：
var someInts = [Int]()
print("someInts is of type [Int] with \(someInts.count) items.")
// 追加元素：
someInts.append(3)
// someInts now contains 1 value of type Int
// 置空：
someInts = []
// someInts is now an empty array, but is still of type [Int]
```

##### 2.3 Creating an Array with a Default Value：创建带默认值的数组
数组的类型从默认值推断，例如下方的 Double数组
```swift
var threeDoubles = Array(repeating: 0.0, count: 3)
// threeDoubles is of type [Double], and equals [0.0, 0.0, 0.0]
```
##### 2.4 Creating an Array by Adding Two Arrays Together：叠加数组
只有类型相同的数组，才能叠加
```swift
var anotherThreeDoubles = Array(repeating: 2.5, count: 3)
// anotherThreeDoubles is of type [Double], and equals [2.5, 2.5, 2.5]
// 都是double类型
var sixDoubles = threeDoubles + anotherThreeDoubles
// sixDoubles is inferred as [Double], and equals [0.0, 0.0, 0.0, 2.5, 2.5, 2.5]
```
##### 2.5 Creating an Array with an Array Literal：数组文本创建数组
形如：[ 值1，值2，值3 ]
```swift
// 如下创建[String]类型数组：
// 1、
var shoppingList: [String] = ["Eggs", "Milk"]
// 2、
var shoppingList2 = ["Eggs", "Milk"]
```
##### 2.6 Accessing and Modifying an Array：访问修改数组

**通过下标访问\修改:**

- count属性:确定元素个数；
- isEmpty属性：是否为空；
- append(_ newElement: Element)方法：添加新元素；
- 数组加法：`shoppingList += ["Baking Powder"]`；
- 索引：`var firstItem = shoppingList[0]` 索引的第一项，是0；
- 索引单个修改：`shoppingList[0] = "Six eggs"`；
- 索引范围修改：`shoppingList[4...6] = ["Bananas", "Apples"]`；
- 索引插入：`shoppingList.insert("Maple Syrup", at: 0)`；
- 索引删除：`let mapleSyrup = shoppingList.remove(at: 0)`；
- 删除最后一个元素：`let apples = shoppingList.removeLast()`
- 注意：越界会造成运行时错误；
```swift
var shoppingList: [String] = ["Eggs", "Milk"] // 
shoppingList.append("Flour") // 
shoppingList += ["Baking Powder"] // 
shoppingList += ["Chocolate Spread", "Cheese", "Butter"] // 

var firstItem = shoppingList[0] // 
shoppingList[0] = "Six eggs" // 
shoppingList[4...6] = ["Bananas", "Apples"] // 
shoppingList.insert("Maple Syrup", at: 0) // 
let mapleSyrup = shoppingList.remove(at: 0) // 
firstItem = shoppingList[0] // 
firstItem = shoppingList[0] // 
```

##### 2.7 Iterating Over an Array：数组遍历

1、for -in：
```swift
for item in shoppingList {
    print(item)
}
// Six eggs
// Milk
// Flour
// Baking Powder
// Bananas
```

2、使用全局enumerate函数来进行数组遍历,返回一个由每一个数据项索引值和数据值组成的元组:
```swift
for (index, value) in shoppingList.enumerated() {
    print("Item \(index + 1): \(value)")
}
// Item 1: Six eggs
// Item 2: Milk
// Item 3: Flour
// Item 4: Baking Powder
// Item 5: Bananas
```

### 3、Sets:集合

- 一个集合存储了一串无序的相同类型的数据；
- swift set类型桥接自Foundation中的NSSet类；
- 存进一个集合的类型必须是可以哈希化的。

**哈希值**

- 哈希值是一个int值，所有相同的对象的哈希值是一样的。例如：a == b，就可以推导出：a.hashValue == b.hashValue；
- Swift所有的基本类型（String,Int,Double,Bool）默认是可以哈希化的。可以作为Sets的值或者字典的key；
- 没有关联值得枚举值也是默认可以哈希化的；

``` 
可用使用自定义类型来设置值类型或字典类型，自定义类型需要从swift标准库遵守哈希化协议。遵守哈希化协议的类型必须提供一个叫做hashValue的方法（返回类型为Int）。hashValue的值在不同的程序或相同的程序中返回不要求相同。因为哈希化协议是可变的。遵守协议的类型必须实现运算符（== or ‘is equal’）。Equatable协议要求必须实现“==”成一个等价关系。见下面的例子：

a == a （自反性）
a == b 相当于 b == a（对称性）
a == b && b == c 相当于 a == c (传递性)
```


##### 3.1 Set Type Syntax:集合类型语法

集合类型的语法是Set<Element>，Element是允许存入集合中的类型。
和数组不同的是，集合没有简写方法。

##### 3.2 Creating and Initializing an Empty Set：创建空的集合

```swift
// letters变量被推断为Set<Character>
var letters = Set<Character>()
print("letters is of type Set<Character> with \(letters.count) items.")
// Prints "letters is of type Set<Character> with 0 items."

letters.insert("a")
// letters now contains 1 value of type Character
letters = []
// letters is now an empty set, but is still of type Set<Character>

```

##### 3.3 Creating a Set with an Array Literal:创建数组文本集合
注意的是，创建集合时，必须带上关键字Set。
```swift
// favoriteGenres是字符串集合，即:Set<String> 类型，仅仅储存String，而且是可变的。其余类似
var favoriteGenres: Set<String> = ["Rock", "Classical", "Hip hop"]
// 简写：
var favoriteGenres: Set = ["Rock", "Classical", "Hip hop"]
```

##### 3.4 Accessing and Modifying a Set：访问修改集合

-  count属性：`favoriteGenres.count`；
-  .isEmpty属性: `if favoriteGenres.isEmpty {}`
-  insert(_:)方法：`favoriteGenres.insert("www.3code.info")`
-  remove(_:)方法：`let removedGenre = favoriteGenres.remove("Rock")`
-  contains(_:)方法：是否包含item，`if favoriteGenres.contains("Funk") {`

##### 3.5 Iterating Over a Set:遍历集合
- for-in;
- sorted():返回一个value的集合

```swift
for genre in favoriteGenres {
    print("\(genre)")
}
// Jazz
// Hip hop
// Classical


for genre in favoriteGenres.sorted() {
    print("\(genre)")
}
// Classical
// Hip hop
// Jazz
```

##### 3.6 Performing Set Operations：集合运算

![](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Art/setVennDiagram_2x.png)

- intersection(_:) 交集
- symmetricDifference(_:) 对称差
- union(_:) 并集
- subtracting(_:) 差集

```swift
let oddDigits: Set = [1, 3, 5, 7, 9]
let evenDigits: Set = [1, 2, 4, 6, 8]
let singleDigitPrimeNumbers: Set = [2, 3, 5, 7]

oddDigits.union(evenDigits).sorted()
// [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
oddDigits.intersection(evenDigits).sorted()
// [1]
oddDigits.subtracting(singleDigitPrimeNumbers).sorted()
// [1, 9]
oddDigits.symmetricDifference(singleDigitPrimeNumbers).sorted()
// [1, 2, 9]
```

Set Membership and Equality

a是b的一个超集，因为a包含了所有元素b。相反地，b是a子集，因为在所有的元素b也被包含a。bc彼此没有相同的元素。
![](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Art/setEulerDiagram_2x.png)
运算方法：

- "=="(is equal)方法:判断两个集合是否拥有相同的元素,相同返回true，反之false
- isSubsetOf(_:)方法：判断目标集合是是否为参数集合的子集，是则返回true，反之false
- isSupersetOf(_:)方法：判断目标集合是否为参数集合的超集，是则返回true，反之false
- isStrictSubsetOf(_:)和isStrictSupersetOf(_:)方法：判断目标是否是子集和超集，但是必须是两个集合不相等，满足以上条件返回true，反之false
- isDisjointWith(_:)方法：判断两个集合有无相同元素，有相同元素则返回false，反之true。

```swift
let houseAnimals: Set = ["🐶", "🐱"]
let farmAnimals: Set = ["🐮", "🐔", "🐑", "🐶", "🐱"]
let cityAnimals: Set = ["🐦", "🐭"]
 
houseAnimals.isSubset(of: farmAnimals)
// true
farmAnimals.isSuperset(of: houseAnimals)
// true
farmAnimals.isDisjoint(with: cityAnimals)
// true
```

### 4、Dictionaries：字典

- Swift的Dictionary类桥接自Foundation的NSDictionary类
- 字典Key类型必须符合Hashable协议
- 写法：Dictionary<Key, Value>
- 简写：[Key: Value]

##### 4.1 创建

1、创建[Int: String]类型的空字典：
```swift
var namesOfIntegers = [Int: String]()
```

2、操作
```swift
// 插入元素：[16:"sixteen"]
namesOfIntegers[16] = "sixteen" 

// 置空，类型不变：[Int: String]
namesOfIntegers = [:]
```

3、实例创建字典：[ 键1：值1，关键2：值2，重点3：值3 ]
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
