
# Swfit笔记

#基础部分

Swift 的类型是在 C 和 Objective-C 的基础上提出的，`Int`是整型；`Double`和`Float`是浮点型；`Bool`是布尔型；`String`是字符串。Swift 还有两个有用的集合类型，`Array`和`Dictionary`，请参考集合类型。

除了我们熟悉的类型，Swift 还增加了 Objective-C 中没有的类型比如元组（Tuple）。元组可以让你创建或者传递一组数据，比如作为函数的返回值时，你可以用一个元组可以返回多个值。

Swift 还增加了可选（Optional）类型，用于处理值缺失的情况。可选表示“那儿有一个值，并且它等于 x ”或者“那儿没有值”。可选有点像在 Objective-C 中使用nil，但是它可以用在任何类型上，不仅仅是类。可选类型比 Objective-C 中的nil指针更加安全也更具表现力，它是 Swift 许多强大特性的重要组成部分。

Swift 是一个类型安全的语言。

##常量和变量

常量的值一旦设定就不能改变，而变量的值可以随意更改

###声明常量和变量

常量和变量必须在使用前声明，用let来声明常量，用var来声明变量。
```swift
let maximumNumberOfLoginAttempts = 10 //常量
var currentLoginAttempt = 0 //变量
```

###类型标注

当你声明常量或者变量的时候可以加上类型标注(type annotation)，说明常量或者变量中要存储的值的类型。
```swift
var welcomeMessage: String
```
**注意：**一般来说你很少需要写类型标注。如果你在声明常量或者变量的时候赋了一个初始值，Swift可以推断出这个常量或者变量的类型，请参考类型安全和类型推断。在上面的例子中，没有给welcomeMessage赋初始值，所以变量welcomeMessage的类型是通过一个类型标注指定的，而不是通过初始值推断的。（注：和python类似）

##输出

可以使用println来输出，使用方法和c类似。

Swift 用字符串插值（string interpolation）的方式把常量名或者变量名当做占位符加入到长字符串中，Swift 会用当前常量或变量的值替换这些占位符。将常量或变量名放入圆括号中，并在开括号前使用反斜杠将其转义：

```swift
println("The current value of friendlyWelcome is \(friendlyWelcome)") //其中friendlyWelcome 是变量或者常量名，和c中的格式化字符功能类似
```
注意：字符串插值所有可用的选项，请参考字符串插值。

##注释

Swift中的注释与C中的注释非常相似。有以下几种类型：

```swift
//这是一个注释

/*这是一个
多行注释*/

/* 这是第一个多行注释的开头 
/* 这是第二个被嵌套的多行注释 */ 
这是第一个多行注释的结尾 */ 

```

##分号

Swift并不强制要求在结尾使用分号。有一种情况必须使用分好，同一行内写多条独立的语句
```
let cat = "????"; println(cat) 
// 输出 "????" 
```

##整数

Swift 提供了8，16，32和64位的有符号和无符号整数类型

###整数范围

你可以访问不同整数类型的min和max属性来获取对应类型的最大值和最小值：
```swift
let minValue = UInt8.min  // minValue 为 0，是 UInt8 类型的最小值 
let maxValue = UInt8.max  // maxValue 为 255，是 UInt8 类型的最大值  
```

###Int

- 在32位平台上，Int和Int32长度相同。
- 在64位平台上，Int和Int64长度相同

除非你需要特定长度的整数，一般来说使用Int就够了。这可以提高代码一致性和可复用性。即使是在32位平台上，Int可以存储的整数范围也可以达到-2147483648~2147483647，大多数时候这已经足够大了。

###UInt

特殊的**无符号类型**UInt，长度与当前平台的原生字长相同：
 
- 在32位平台上，UInt和UInt32长度相同。
- 在64位平台上，UInt和UInt64长度相同。

注意：尽量不要使用UInt，除非你真的需要存储一个和当前平台原生字长相同的无符号整数。除了这种情况，**最好使用Int，即使你要存储的值已知是非负的**


##浮点数

- Double表示64位浮点数。当你需要存储很大或者很高精度的浮点数时请使用此类型。至少有15位数字
- Float表示32位浮点数。精度要求不高的话可以使用此类型。Float最少只有6位数字

##类型安全和类型推测

Swift 是一个**类型安全(type safe )**的语言,它会在编译你的代码时**进行类型检查(type checks)**

```swift

let meaningOfLife = 42 
// meaningOfLife 会被推测为 Int 类型

let pi = 3.14159 
// pi 会被推测为 Double 类型 

let anotherPi = 3 + 0.14159 
// anotherPi 会被推测为 Double 类型 

```

如果你没有给浮点字面量标明类型，Swift 会推测你想要的是Double

##数值型字面量

- 一个十进制数，没有前缀
- 一个二进制数，前缀是0b
- 一个八进制数，前缀是0o
- 一个十六进制数，前缀是0x

```swift

let decimalInteger = 17 
let binaryInteger = 0b10001       // 二进制的17 
let octalInteger = 0o21           // 八进制的17 
let hexadecimalInteger = 0x11     // 十六进制的17 

```

##元组

元组（tuples）把多个值组合成一个复合值。元组内的值可以使任意类型，并不要求是相同类型

可以把任意顺序的类型组合成一个元组，这个元组可以包含所有类型

```swift
let http404Error = (404, "Not Found") 
// http404Error 的类型是 (Int, String)，值是 (404, "Not Found") 

```
元组的内容分解（decompose）成单独的常量和变量

```swift
let (statusCode, statusMessage) = http404Error 
println("The status code is \(statusCode)") 
// 输出 "The status code is 404" 
println("The status message is \(statusMessage)") 
// 输出 "The status message is Not Found" 

```
如果你只需要一部分元组值，分解的时候可以把要忽略的部分用下划线（_）标记：
```swift
let (justTheStatusCode, _) = http404Error 
println("The status code is \(justTheStatusCode)") 
// 输出 "The status code is 404" 

```
你还可以通过下标来访问元组中的单个元素，下标从零开始：

```swift
println("The status code is \(http404Error.0)") 
// 输出 "The status code is 404" 
println("The status message is \(http404Error.1)") 
// 输出 "The status message is Not Found" 
```
你可以在定义元组的时候给单个元素命名：
```swift
let http200Status = (statusCode: 200, description: "OK") 

```
给元组中的元素命名后，你可以通过名字来获取这些元素的值：
```swift
println("The status code is \(http200Status.statusCode)") 
// 输出 "The status code is 200" 
println("The status message is \(http200Status.description)") 
// 输出 "The status message is OK" 
```
作为函数返回值时，元组非常有用

##可选
可选类型很相似于C#中的可空类型。



##类型别名

你可以使用typealias关键字来定义类型别名。

当你想要给现有类型起一个更有意义的名字时，类型别名非常有用。假设你正在处理特定长度的外部资源的数据：
```swift
typealias AudioSample = UInt16 
```

定义了一个类型别名之后，你可以在任何使用原始名的地方使用别名：

```swift
var maxAmplitudeFound = AudioSample.min 
// maxAmplitudeFound 现在是 0 
```

本例中，AudioSample被定义为UInt16的一个别名。因为它是别名，AudioSample.min实际上是UInt16.min，所以会给maxAmplitudeFound赋一个初值0。



#字符串和字符

##字符串字面量
字符串字面量可以包含以下特殊字符：

1. 转义特殊字符 \0 (空字符)、\\(反斜线)、\t (水平制表符)、\n (换行符)、\r (回车符)、\" (双引号)、\' (单引号)。
2. 单字节 Unicode 标量，写成 \xnn，其中 nn 为两位十六进制数。
3. 双字节 Unicode 标量，写成 \unnnn，其中 nnnn 为四位十六进制数。
4. 四字节 Unicode 标量，写成 \Unnnnnnnn，其中 nnnnnnnn 为八位十六进制数。


##初始化空字符串

```swift
var emptyString = ""               // empty string literal 
var anotherEmptyString = String()  // initializer syntax 
// 这两个字符串都为空，并且两者等价 
```

可以通过检查其 Boolean 类型的 isEmpty 属性来判断该字符串是否为空：
```swift
if emptyString.isEmpty { 
    println("Nothing to see here") 
} 
// 打印 "Nothing to see here" 
```

##字符串可变性

您可以通过将一个特定字符串分配给一个变量（对其进行修改）或者常量（保证其不会被修改）来指定该字符串是否可以被修改：

```swift
var variableString = "Horse" 
variableString += " and carriage" 
// variableString 现在为 "Horse and carriage" 
let constantString = "Highlander" 
constantString += " and another Highlander" 
// 这会报告一个编译错误(compile-time error) - 常量不可以被修改。 
```

##字符串是值类型


##使用字符(Characters)

Swift 的 String 类型表示特定序列的字符值的集合。每一个字符值代表一个 Unicode 字符。您可利用 for-in 循环来遍历字符串中的每一个字符：
```swift
for character in "Dog!????" { 
    println(character) 
} 
// D 
// o 
// g 
// ! 
// ???? 
```

```swift
let yenSign: Character = "¥" 
```

##计算字符数量

通过调用全局`countElements` 函数，并将字符串作为参数进行传递可以获取该字符串的字符数量。

```swift
let unusualMenagerie = "Koala ????, Snail ????, Penguin ????, Dromedary ????" 
println("unusualMenagerie has \(countElements(unusualMenagerie)) characters") 
// prints "unusualMenagerie has 40 characters" 
```

**注意：**
 
1. 不同的 Unicode 字符以及相同 Unicode 字符的不同表示方式可能需要不同数量的内存空间来存储，所以Swift 中的字符在一个字符串中表示并不一定占用相同的内存空间。因此，字符串的长度不得不通过迭代字符串中每一个字符的长度来进行计算。如果您正在处理一个长字符串，需要注意 countElements 函数必须遍历字符串中的字符，以精准计算字符串的长度。

2. 另外需要注意的是通过 countElements 返回的字符数量并不总是与包含相同字符的 NSString 的 length 属性相同。NSString 的 length 属性是基于利用 UTF-16 表示的十六位code units数目，而不是基于 Unicode 字符。为了解决这个问题，NSString 的 length 属性在被 Swift的 String值访问时会被称为utf16count。


##字符串插值

字符串插值是一种全新的构建字符串的方式，可以在其中包含常量、变量、字面量和表达式。您插入的字符串字面量的每一项都被包裹在以反斜线为前缀的圆括号中：
```swift
let multiplier = 3 
let message = "\(multiplier) times 2.5 is \(Double(multiplier) * 2.5)" 
// message is "3 times 2.5 is 7.5" 
```
**注意：**您插值字符串中写在括号中的表达式不能包含非转义双引号 (") 和反斜杠 (\)，并且不能包含回车或换行符。


##比较字符串

###字符串相等
如果两个字符串以同一顺序包含完全相同的字符，则认为两者字符串相等：

```swift
let quotation = "We're a lot alike, you and I." 
let sameQuotation = "We're a lot alike, you and I." 
if quotation == sameQuotation { 
    println("These two strings are considered equal") 
} 
// prints "These two strings are considered equal" 
```


###前缀/后缀相等
通过调用字符串的 `hasPrefix/hasSuffix` 方法来检查字符串是否拥有特定前缀/后缀。两个方法均需要以字符串作为参数传入并返回 Boolean 值。两个方法均执行基本字符串和前缀/后缀字符串之间逐个字符的比较操作。

```swift
let romeoAndJuliet = [ 
    "Act 1 Scene 1: Verona, A public place", 
    "Act 1 Scene 2: Capulet's mansion", 
    "Act 1 Scene 3: A room in Capulet's mansion", 
    "Act 1 Scene 4: A street outside Capulet's mansion", 
    "Act 1 Scene 5: The Great Hall in Capulet's mansion", 
    "Act 2 Scene 1: Outside Capulet's mansion", 
    "Act 2 Scene 2: Capulet's orchard", 
    "Act 2 Scene 3: Outside Friar Lawrence's cell", 
    "Act 2 Scene 4: A street in Verona", 
    "Act 2 Scene 5: Capulet's mansion", 
    "Act 2 Scene 6: Friar Lawrence's cell" 
] 
//您可以利用 hasPrefix 方法使用romeoAndJuliet数组来计算话剧中第一幕的场景数：

var act1SceneCount = 0 
for scene in romeoAndJuliet { 
    if scene.hasPrefix("Act 1 ") { 
        ++act1SceneCount 
    } 
} 
println("There are \(act1SceneCount) scenes in Act 1") 
// prints "There are 5 scenes in Act 1" 

//可使用hasSuffix方法来计算发生在Capulet公馆和Lawrence牢房内以及周围的场景数。
var mansionCount = 0 
var cellCount = 0 
for scene in romeoAndJuliet { 
    if scene.hasSuffix("Capulet's mansion") { 
        ++mansionCount 
    } else if scene.hasSuffix("Friar Lawrence's cell") { 
        ++cellCount 
    } 
} 
println("\(mansionCount) mansion scenes; \(cellCount) cell scenes") 
// prints "6 mansion scenes; 2 cell scenes" 

```


##大写和小写字符串
您可以通过字符串的 `uppercaseString` 和 `lowercaseString` 属性来访问一个字符串的大写/小写版本。
```swift
let normal = "Could you help me, please?" 
let shouty = normal.uppercaseString 
// shouty 值为 "COULD YOU HELP ME, PLEASE?" 
let whispered = normal.lowercaseString 
// whispered 值为 "could you help me, please?" 
```


#集合类型

Swift语言提供经典的`数组`和`字典`两种集合类型来存储集合数据。数组用来按顺序存储相同类型的数据。字典虽然无序存储相同类型数据值但是需要由独有的标识符引用和寻址（就是键值对）。

Swift语言里的数组和字典中存储的**数据值类型必须明确**

##数组
Swift 中的数组是类型安全的，并且它们中包含的类型必须明确


###数组构造语句

```swift
var shoppingList: String[] = ["Eggs", "Milk"] 
```
或者 

```swift
var shoppingList = ["Eggs", "Milk"] 
```

###访问和修改数组
可以通过数组的方法和属性来访问和修改数组，或者下标语法。 还可以使用数组的只读属性`count`来获取数组中的数据项数量。
```swift
println("The shopping list contains \(shoppingList.count) items.") 
```

使用布尔项`isEmpty`来作为检查count属性的值是否为0的捷径。
```swift
if shoppingList.isEmpty { 
    println("The shopping list is empty.") 
} else { 
    println("The shopping list is not empty.") 
} 
// 打印 "The shopping list is not empty."（shoppinglist不是空的） 
```

使用append方法在数组后面添加新的数据项
```swift
shoppingList.append("Flour") 

```
使用加法赋值运算符（+=）也可以直接在数组后面添加数据项
```swift
shoppingList += "Baking Powder" 
shoppingList += ["Chocolate Spread", "Cheese", "Butter"] 
```

直接使用下标语法来获取数组中的数据项

```swift
var firstItem = shoppingList[0] 

```
Swift 中的数组索引总是从**零**开始


可以利用下标来一次改变一系列数据值,即使新数据和原有数据的数量是不一样的。

```swift
shoppingList[4...6] = ["Bananas", "Apples"] 
```

**注意：** 我们**不能**使用下标语法在数组尾部添加新项。如果我们试着用这种方法对索引越界的数据进行检索或者设置新值的操作，我们会引发一个**运行期错误**。我们可以使用索引值和数组的count属性进行比较来在使用某个索引之前先检验是否有效。除了当count等于0时（说明这是个空数组），最大索引值一直是count - 1，因为数组都是零起索引。


###数组的遍历
使用for-in
```swift
for item in shoppingList { 
    println(item) 
} 
```

如果我们同时需要每个数据项的值和索引值，可以使用全局`enumerate`函数来进行数组遍历。`enumerate`返回一个由每一个数据项索引值和数据值组成的键值对组

```swift
for (index, value) in enumerate(shoppingList) { 
    println("Item \(index + 1): \(value)") 
} 
// Item 1: Six eggs 
// Item 2: Milk 
// Item 3: Flour 
// Item 4: Baking Powder 
// Item 5: Bananas 
```

###创建并且构造一个数组

```swift
var someInts = Int[]() 
println("someInts is of type Int[] with \(someInts。count) items。") 
// 打印 "someInts is of type Int[] with 0 items。"（someInts是0数据项的Int[]数组） 
```

Swift 中的Array类型还提供一个可以创建特定大小并且所有数据都被默认的构造方法。我们可以把准备加入新数组的数据项数量（count）和适当类型的初始值（repeatedValue）传入数组构造函数：
```swift
var threeDoubles = Double[](count: 3, repeatedValue:0.0) 
// threeDoubles 是一种 Double[]数组, 等于 [0.0, 0.0, 0.0]

var anotherThreeDoubles = Array(count: 3, repeatedValue: 2.5) 
// anotherThreeDoubles is inferred as Double[], and equals [2.5, 2.5, 2.5]
```

我们可以使用加法操作符（+）来组合两种已存在的相同类型数组。
```swift
var sixDoubles = threeDoubles + anotherThreeDoubles 
// sixDoubles 被推断为 Double[], 等于 [0.0, 0.0, 0.0, 2.5, 2.5, 2.5] 
```


##字典

字典是一种存储**相同类型**多重数据的存储器。每个值（value）都关联独特的键（key），键作为字典中的这个值数据的标识符。和数组中的数据项不同，字典中的数据项并**没有具体顺序**。

Swift 的字典使用`Dictionary`定义,其中KeyType是字典中键的数据类型，ValueType是字典中对应于这些键所存储值的数据类型。
 
KeyType的唯一限制就是**可哈希的**，这样可以保证它是独一无二的，所有的 Swift 基本类型（例如String，Int， Double和Bool）都是默认可哈希的，并且所有这些类型都可以在字典中当做键使用。未关联值的枚举成员（参见枚举）也是默认可哈希的。


###字典字面语句

一个键值对是一个key和一个value的结合体。在字典字面语句中，每一个键值对的键和值都由冒号分割

```swift
var airports: Dictionary = ["TYO": "Tokyo", "DUB": "Dublin"] 
//或者
var airports = ["TYO": "Tokyo", "DUB": "Dublin"] 

```

**注意：** airports字典被声明为变量（用var关键字）而不是常量（let关键字）因为后来更多的机场信息会被添加到这个示例字典中。

###读取和修改字典
count来获取某个字典的数据项数量

```swift
println("The dictionary of airports contains \(airports.count) items.") 
// 打印 "The dictionary of airports contains 2 items."（这个字典有两个数据项） 
```
在字典中使用下标语法来添加新的数据

```swift
airports["LHR"] = "London" 
// airports 字典现在有三个数据项 
```

###移除键值对

可以使用下标语法来通过给某个键的对应值赋值为nil来从字典里移除一个键值对

```swift
airports["APL"] = "Apple Internation" 
// "Apple Internation"不是真的 APL机场, 删除它 
airports["APL"] = nil 
// APL现在被移除了 
```

`removeValueForKey`方法也可以用来在字典中移除键值对

```swift
if let removedValue = airports.removeValueForKey("DUB") { 
    println("The removed airport's name is \(removedValue).") 
} else { 
    println("The airports dictionary does not contain a value for DUB.") 
} 
// 打印 "The removed airport's name is Dublin International."（被移除的机场名字是都柏林国际） 
```

#控制流
##For 循环
Swift 提供两种 for 循环形式：
 
for-in 用来遍历一个范围(range)，队列(sequence)，集合(collection)，系列(progression)里面所有的元素执行一系列语句。
 
for 条件递增语句(for-condition-increment)，用来重复执行一系列语句直到特定条件达成，一般通过在每次循环完成后增加计数器的值来实现。

###For-In

例子1：
```swift
for index in 1...5 { 
    println("\(index) times 5 is \(index * 5)") 
} 
// 1 times 5 is 5 
// 2 times 5 is 10 
// 3 times 5 is 15 
// 4 times 5 is 20 
// 5 times 5 is 25 
```

面的例子中，index 是一个每次循环遍历开始时被自动赋值的常量。这种情况下，index 在使用前不需要声明，**只需要将它包含在循环的声明中，就可以对其进行隐式声明，而无需使用 let 关键字声明**。

**注意**：index 常量只存在于循环的生命周期里。如果你想在循环完成后访问 index 的值，又或者想让 index 成为一个变量而不是常量，你必须在循环之前自己进行声明。

如果你不需要知道范围内每一项的值，你可以使用下划线（_）替代变量名来忽略对值的访问

```swift
let base = 3 
let power = 10 
var answer = 1 
for _ in 1...power { 
    answer *= base 
} 
println("\(base) to the power of \(power) is \(answer)") 
// prints "3 to the power of 10 is 59049 
```

使用 for-in 遍历一个数组所有元素：

```swift
let names = ["Anna", "Alex", "Brian", "Jack"] 
for name in names { 
    println("Hello, \(name)!") 
} 
// Hello, Anna! 
// Hello, Alex! 
// Hello, Brian! 
// Hello, Jack! 
```

使用for-in遍历一个字典:

```swift
let numberOfLegs = ["spider": 8, "ant": 6, "cat": 4] 
for (animalName, legCount) in numberOfLegs { 
    println("\(animalName)s have \(legCount) legs") 
} 
// spiders have 8 legs 
// ants have 6 legs 
// cats have 4 legs 
```

使用 for-in 循环来遍历字符串中的字符：

```swift
for character in "Hello" { 
    println(character) 
} 
// H 
// e 
// l 
// l 
// o 
```

###For条件递增
用法基本和c语言一样:

```swift
for var index = 0; index < 3; ++index { 
    println("index is \(index)") 
} 
// index is 0 
// index is 1 
// index is 2 

```

循环执行流程如下：
 
1、循环首次启动时，初始化表达式（initialization expression）被调用一次，用来初始化循环所需的所有常量和变量。
2、条件表达式（condition expression）被调用，如果表达式调用结果为 false，循环结束，继续执行 for 循环关闭大括号(})之后的代码。如果表达式调用结果为 true，则会执行大括号内部的代码（statements）。
3、执行所有语句（statements）之后，执行递增表达式（increment expression）。通常会增加或减少计数器的值，或者根据语句（statements）输出来修改某一个初始化的变量。当递增表达式运行完成后，重复执行第2步，条件表达式会再次执行。


##While 循环
有两种形式
- while 循环，每次在循环开始时计算条件是否符合；
- do-while 循环，每次在循环结束时计算条件是否符合。

###while

```swift
var i=0
while i<5{
    println(i++)
}
```

###do-while

```swift
var j=0
do{
    println(j++)
}while j<5

```
##条件语句

Swift 提供两种类型的条件语句：if语句和switch语句

###if

```swift
temperatureInFahrenheit = 90 
if temperatureInFahrenheit <= 32 { 
    println("It's very cold. Consider wearing a scarf.") 
} else if temperatureInFahrenheit >= 86 { 
    println("It's really warm. Don't forget to wear sunscreen.") 
} else { 
    println("It's not that cold. Wear a t-shirt.") 
} 
// prints "It's really warm. Don't forget to wear sunscreen." 
```

###Switch

```swift
switch `some value to consider` { 
case `value 1`: 
    `respond to value 1` 
case `value 2`, 
`value 3`: 
    `respond to value 2 or 3` 
default: 
    `otherwise, do something else` 
} 
```

**不存在隐式的贯穿(Fallthrough)**
与C语言和Objective-C中的switch语句不同，在 Swift 中，当匹配的case块中的代码执行完毕后，程序会终止switch语句，而不会继续执行下一个case块。这也就是说，**不需要在case块中显式地使用break语句**。这使得switch语句更安全、更易用，也避免了因忘记写break语句而产生的错误。

**每一个case块都必须包含至少一条语句**

*错误例子：*

```swift
let anotherCharacter: Character = "a" 
switch anotherCharacter { 
case "a": //此处错误第一个case块是空的
case "A": 
    println("The letter A") 
default: 
    println("Not the letter A") 
} 
// this will report a compile-time error 
```

**范围匹配**

case块的模式也可以是一个值的范围

```swift
let count = 3_000_000_000_000 
let countedThings = "stars in the Milky Way" 
var naturalCount: String 
switch count { 
case 0: 
    naturalCount = "no" 
case 1...3: 
    naturalCount = "a few" 
case 4...9: 
    naturalCount = "several" 
case 10...99: 
    naturalCount = "tens of" 
case 100...999: 
    naturalCount = "hundreds of" 
case 1000...999_999: 
    naturalCount = "thousands of" 
default: 
    naturalCount = "millions and millions of" 
} 
println("There are \(naturalCount) \(countedThings).") 
// prints "There are millions and millions of stars in the Milky Way." 

```

**元组 (Tuple)** 

你可以使用元组在同一个switch语句中测试多个值。元组中的元素可以是值，也可以是范围。另外，使用下划线(_)来匹配所有可能的值。

下面的例子展示了如何使用一个(Int, Int)类型的元组来分类下图中的点(x, y)：

```swift
let somePoint = (1, 1) 
switch somePoint { 
case (0, 0): 
    println("(0, 0) is at the origin") 
case (_, 0): 
    println("(\(somePoint.0), 0) is on the x-axis") 
case (0, _): 
    println("(0, \(somePoint.1)) is on the y-axis") 
case (-2...2, -2...2): 
    println("(\(somePoint.0), \(somePoint.1)) is inside the box") 
default: 
    println("(\(somePoint.0), \(somePoint.1)) is outside of the box") 
} 
// prints "(1, 1) is inside the box" 
```

![](http://www.cocoachina.com/cms/uploads/allimg/140611/8370_140611131555_1.png)


**值绑定 (Value Bindings)**

case块的模式允许将匹配的值绑定到一个临时的常量或变量，这些常量或变量在该case块里就可以被引用了——这种行为被称为值绑定。

```swift
let anotherPoint = (2, 0) 
switch anotherPoint { 
case (let x, 0): 
    println("on the x-axis with an x value of \(x)") 
case (0, let y): 
    println("on the y-axis with a y value of \(y)") 
case let (x, y): 
    println("somewhere else at (\(x), \(y))") 
} 
// prints "on the x-axis with an x value of 2" 
```


#函数

##函数的声明与调用
函数声明，以`func`关键字为前缀，返回类型用->符号标明

```swift
func sayHello(personName: String) -> String {
    let greeting = "Hello, " + personName + "!"
    return greeting
}
```

函数调用

```swift
println(sayHello("Anna"))
// prints "Hello, Anna!"
println(sayHello("Brian"))
// prints "Hello, Brian!"
```

##函数的参数和返回值

```swfit
func halfOpenRangeLength(start: Int, end: Int) -> Int {
    return end - start
}
println(halfOpenRangeLength(1, 10))
// prints "9"
```

##无参函数

```swift
func sayHelloWorld() -> String {
    return "hello, world"
}
println(sayHelloWorld())
// prints "hello, world"
```

##没有返回值的函数

```swift
func sayGoodbye(personName: String) {
    println("Goodbye, \(personName)!")
}
sayGoodbye("Dave")
// prints "Goodbye, Dave!"
```

**注意：**严格地说，sayGoodbye功能确实还返回一个值，即使没有返回值定义。函数没有定义返回类型但返 回了一个void返回类型的特殊值。它是一个简直是空的元组，实际上零个元素的元组，可以写为（）。

##多返回值函数

你可以使用一个元组类型作为函数的返回类型返回一个有多个值组成的一个复合作为返回值。

```swift
func count(string: String) -> (vowels: Int, consonants: Int, others: Int) {
    var vowels = 0, consonants = 0, others = 0
    for character in string {
        switch String(character).lowercaseString {
        case "a", "e", "i", "o", "u":
            ++vowels
        case "b", "c", "d", "f", "g", "h", "j", "k", "l", "m",
        "n", "p", "q", "r", "s", "t", "v", "w", "x", "y", "z":
        ++consonants
        default:
        ++others
        }
    }
    return (vowels, consonants, others)
}

```
调用

```swift

let total = count("some arbitrary string!")
println("\(total.vowels) vowels and \(total.consonants) consonants")
// prints "6 vowels and 13 consonants"

```

##函数参数名

###外部参数名






