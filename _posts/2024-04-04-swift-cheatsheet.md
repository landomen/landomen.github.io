---
title: Swift Cheatsheet for Android/Kotlin Developers
description: We cover common Swift patterns you might see when looking at iOS code, try to understand them, and compare their implementation in Kotlin.
date: 2024-04-04 11:00:00 +0100
categories: [Swift, Kotlin, Android, iOS]
tags: [swift, kotlin, android, ios]
image: /assets/img/posts/swift/cheatsheet/cover.webp
---


Android developers primarily work with Kotlin. However, knowing how to read and understand Swift can be useful when referencing how some feature is implemented on iOS. It’s also helpful if you are exploring Kotlin Multiplatform.

This also applies to iOS developers looking at the Android code. Knowing how to read and understand Kotlin will make it easier.

We’ll cover some common Swift patterns you might see when looking at iOS code, try to understand them and compare their implementation in Kotlin.



> Note: this is not a full comparison between Swift and Kotlin. We cover basics and some common patterns. Let me know in the comments if you are interested in more advanced patterns and differences between the two languages, and would like to see a follow-up article.

# 1. Basics

## Variables

Variables and constants in Swift are defined using the  `var`  and  `let`  keyword, respectfully. Type annotations can be provided with a semi-colon, but are not required.


```swift
let animDurationMillis: Int = 500
var clickCount = 0
```

```kotlin
val animDurationMillis: Int = 500
var clickCount = 0
```


The only difference between the languages at this point is the  `let`  vs  `val`  keyword for defining read-only variables.

## **Optionals / Nullability**

For optional or nullable types, both languages use the same  `?`  char, the only difference is  `nil`  vs  `null`  for the valueless state.


```swift                     
var foundItem: String? = nil
```

```kotlin
var foundItem: String? = null
```



There are several ways to handle the  `nil`  state:

-   by using an  `if`  expression to check for a value (we cover this later in the article),
-   by using optional binding (we cover this later in the article),
-   by providing a fallback/default value,
-   by force unwrapping.

Here is an example of the last two approaches. We can see that Kotlin offers an almost identical approach with a slight difference in syntax (`?? vs ?:`  and  `! vs !!`).


```swift
//  - fallback/default value
let actualFoundItem = foundItem ?? "empty"
//  - force unwrapping
let actualFoundItem = foundItem!
```

```kotlin
//  - fallback/default value
val actualFoundItem = foundItem ?: "empty"
//  - force unwrapping
val actualFoundItem2 = foundItem!!
```



## Control flow

The if statement is almost identical to Kotlin, with a small difference that Swift supports omitting parenthesis.


```swift
if foundItem != nil {       
  // do something            
}
```

```kotlin
if (foundItem != null) {
  // do something
}
```



Both languages use the same syntax for `else if/else` branches and in both it can be used as an expression.

```swift
let description = if delta <= 10 {
  "low"
} else if delta >= 50 {
  "high"
} else {
  "medium"
}
```

```kotlin
val description = if (delta <= 10) {
    "low"
} else if (delta >= 50) {
    "high"
} else {
    "medium"
}
```


## Functions

Functions in Swift are declared using the  `func`  keyword, followed by the function name, input parameters, and the return type.

```swift
func addTwoNumbers(a: Int, b: Int) -> Int {
    return a + b
}
```


In Kotlin we use the `fun` keyword and a `:` to define the return type instead of `->`.

```kotlin
fun addTwoNumbers(a: Int, b: Int): Int {
    return a + b
}
```


# 2. Structures and Classes

Swift supports structures and classes which are similar when it comes to modeling data as both support defining properties and functions. A key difference is that classes are pass-by-reference while structures are pass-by-value.

The recommendation is to use structures by default. Use classes when you need additional functionalities like inheritance, objective-C compatibility, and  [others](https://developer.apple.com/documentation/swift/choosing-between-structures-and-classes).


```swift
struct VehicleStructure {
    var maxSpeed = 0
    
    func printInfo() {
        print("Max speed \(maxSpeed)")
    }
}

class VehicleClass {
    var maxSpeed = 0
    
    func printInfo() {
        print("Max speed \(maxSpeed)")
    }
}
```


To create an instance we reference the structure or class name followed by empty parentheses.

Structures are immutable by default, so if we want to change the value of any of its properties we have to declare it as a `var` instead of `let`.  They have auto-generated initializers which we can use to set values of the member properties.

```swift
// creating mutable struct instance
var car = VehicleStructure()
car.maxSpeed = 250
car.printInfo()

// creating immutable struct instance
let carSimple = VehicleStructure(maxSpeed: 200)
carSimple.printInfo()

// creating a class instance
let bike = VehicleClass()
bike.maxSpeed = 50
bike.printInfo()
```


## Kotlin

The main building block in Kotlin is a class. Its declaration and usage are almost identical to Swift.

```kotlin
class Vehicle {
    var maxSpeed = 0

    fun printInto(){
        println("Max speed is $maxSpeed")
    }
}

// creating a class instance
val vehicle = Vehicle()
vehicle.maxSpeed = 250
vehicle.printInfo()
```



Kotlin also supports class constructors, meaning we must provide values for all the class properties when creating an instance.


```kotlin
class Vehicle(var maxSpeed: Int) {
    
    fun printInfo(){
        println("Max speed is $maxSpeed")
    }
}

val vehicle = Vehicle(250)
```


Kotlin supports other related structures such as abstract classes, data classes, interfaces, and sealed classes and interfaces. Read more at  [https://kotlinlang.org/docs/classes.html](https://kotlinlang.org/docs/classes.html)

# 3. Optional Binding

## If let

One pattern that you might have come  across frequently  in a Swift codebase is the following:


```swift
let fetchedUserId: String? = "Optional id of the fetched user"
if let userId = fetchedUserId {
    // userId can be used as non-optional constant
    print(userId)
} else {
    // fetchedUserId is nil/null
    throw Error("Missing user id")
}

// fetchedUserId and userId can be used outside the if statement
// but both are stil optional, require unwrapping
```


This is called  [optional binding](https://drewag.me/posts/2014/07/05/what-is-an-optional-in-swift#optional-binding)  and it does the following:

-   checks if the optional  `fetchedUserId`  variable has a value present that is not  `nil`,
-   if true it assigns the value to a new non-optional constant named  `userId`,
-   the new constant  `userId`  can be referenced inside the code block,
-   if  `fetchedUserId`  is  `nil`  then it will execute the  `else`  block.

It’s possible to simplify this even further by using the existing variable name:


```swift
let fetchedUserId: String? = "Optional id of the fetched user"
if let fetchedUserId {
    // fetchedUserId can be used as non-optional constant
    print(fetchedUserId)
}
```



In both cases the  `fetchedUserId`  and the  `userId`  constants can be used outside of the  `if`  statement, but require additional unwrapping as both are still considered optional.

**Kotlin**  <br>
Kotlin doesn’t have an equivalent special pattern for this. One option is to use an  `if/else`  statement. However, that only works for locally scoped variables and not global ones. We must first assign the value to a new local variable/constant to support global variables.


```kotlin
// global class property
var fetchedUserId: String? = "Optional id of the fetched user"

val userId = fetchedUserId
if (userId != null) {
  // userId can be used as non-optional
} else {
  throw Exception("Missing user id")
}

// userId can be used as non-optional anywhere
```



In the above case, we can reference the  `userId`  constant as non-optional even after the  `if`  statement, which is unsupported in Swift using the optional binding pattern.

An alternative solution would be to use one of the scope functions like  `.let {}`. The code inside the function will only be executed if  `fetchedUserId`  is not  `null`. Any references to  `fetchedUserId`  after this code block still require null-safety as the variable is considered optional.


```kotlin
fetchedUserId?.let { userId ->
  // userId can be used as non-optional
} ?: throw Exception("Missing user id")
```


## **Guard**

Another common pattern is a  `guard`  statement, which is similar to the  `if let`  pattern. It’s commonly used for early exits from a function. Another difference is that the  `else`  block is required.

```swift
func checkUsernameValid(username: String?): Bool {
    guard let username else {
        // username is nil, can't evaluate
        return false
    }
    // username can be used non-optional
    return username.count > 3
}
```



In the code above, the function receives an optional variable  `username`. It then checks if  `username`  has a value using the  `guard`statement. If it doesn’t, it will return from the function. If it has, we can use  `username`  as if it were non-optional in the rest of the function.

**Kotlin**<br>  
In Kotlin we could write this in a few different ways, here are two suggestions:


```kotlin
fun checkUsernameValid(username: String?): Boolean {
    if (username.isNullOrEmpty()){
        return false
    }
    return username.length > 3
}

// or

fun checkUsernameValid(username: String?): Boolean {
    val actualUsername = username ?: return false
    return actualUsername.length > 3
}
```



# 4. Enums

We use the  `enum`  keyword in Swift to define enumerations. Values are defined using the  `case`  keyword followed by the name of the enumeration case, which is recommended to be lowercase and in singular form. It’s required to use  `case`  when each case is placed in a new line. When placing multiple cases in a single line, we can separate them with a comma.

```swift
enum Direction {
    case left
    case up
    case right
    case down
}

// or

enum Direction {
    case left, up, right, down
}
```


To use the enum case we reference the type (`Direction`) and the case we want to use. Later we can skip the type and use the case directly using the shorter dot syntax.

```swift
var selectedDirection = Direction.up
selectedDirection = .right
```


To check for the enumeration value we can use the `switch` statement. Xcode will automatically write all the branches, as it’s required for the `switch` statement over enums to be exhaustive.

```swift
switch(selectedDirection){
case .left:
    goLeft()
case .up:
    goForward()
case .right:
    goRight()
case .down:
    goBackward()
}
```

Swift enums also support  [associated values](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/enumerations#Associated-Values)  meaning each enum case can have a different number of value types. This acts as a powerful domain modeling tool, similar to a  `sealed class`  in Kotlin.

Read more:  [https://docs.swift.org/swift-book/documentation/the-swift-programming-language/enumerations](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/enumerations)

## **Kotlin**

Enums in Kotlin are defined using the  `enum class`  keyword. We define the values separated by a comma. The convention is for enum value names to be uppercase, but this can differ based on the style of the project.

```kotlin
enum class Direction {
    LEFT, UP, RIGHT, DOWN
}
```

To use the enum we reference the class name and the value we want to use.

```kotlin
var selectedDirection = Direction.UP
```


We can use a `when` statement to check for the value. It has to be exhaustive to cover all the possible enum values.

```kotlin
when(selectedDirection){
    Direction.LEFT -> goLeft()
    Direction.UP -> goForward()
    Direction.RIGHT -> goRight()
    Direction.DOWN -> goBackward()
}
```

Kotlin enum class also supports defining additional properties for which each enum value has to provide a value. However, unlike Swift’s associated values, the properties are at class level, not value level, meaning they must be the same type for each value.

Read more:  [https://kotlinlang.org/docs/enum-classes.html](https://kotlinlang.org/docs/enum-classes.html)

# 5. Dictionary / Map

The syntax between dictionaries in Swift and maps in Kotlin is quite different but uses a similar underlying concept.

## **Swift**

A  `dictionary`  is a data structure in Swift that stores associations between keys and values of the same type in an unordered manner. Each key represents a unique value based on which we can access the associated value.

To declare a  `dictionary`  we define key-value pairs inside the square brackets as  `[Key: Value]`, separated by a comma. We can omit the type declaration when we define at least one key-value pair so that the compiler can determine the types.


```swift
var httpErrorCodes: [Int: String] = [404: "Not found", 401: "Unauthorized"]
```


To read the value from a dictionary using a key we can use the subscript syntax (`dictionary[key]`). If the key does not exist in the dictionary, it will return `nil`. We can provide a default value by using the `??` operator.

```swift
func getHttpErrorCodeMessage(code: Int) -> String {
    let errorCodeMessage = httpErrorCodes[code] ?? "Unknown"
    return "Http error code \(errorCodeMessage)"
}
```


To write a new value to a dictionary, we assign a value to a key. If the key doesn’t exist, it will add a new key-value pair to the collection. If the key already exists, it will update its value.

```swift
// add new key:value pair
httpErrorCodes[500] = "Internal Server Error"

// update value for an existing key
httpErrorCodes[401] = "Requires authentication"
```

Whether we use a mutable or immutable (read-only) dictionary depends on the assignment. By using  `let`  we define a dictionary from which we can only read after it’s declared. To support writing, we have to declare it as a  `var`.

Read more:  [https://docs.swift.org/swift-book/documentation/the-swift-programming-language/collectiontypes#Dictionaries](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/collectiontypes#Dictionaries)

## **Kotlin**

A  `map`  is a collection that holds pairs of unique keys and values and supports efficiently retrieving the value corresponding to each key.

To declare an immutable (read-only) map in Kotlin we use the  `Map<KeyType, ValueType>`  type with a  `mapOf(varargs pairs: Pair<KeyType, ValueType>)`  standard library function to initialize it. We can omit the explicit variable type declaration when we provide at least one key-value pair as the compiler can determine the types.

Note that when declaring the values we can use either the  `Pair(key, value)`  class directly or the  `to`  infix function which creates the object for us.


```kotlin
val httpErrorCodes: Map<Int, String> = mapOf(
    404 to "Not found",
    Pair(401, "Unauthorized"),
)
```

To read the value from a map using a key we can use the bracket notation (`map[key]`). If the key does not exist in the map, it will return  `null`. We can provide a default value by using the  `?:`  operator.


```kotlin
fun getHttpErrorCodeMessage(code: Int): String {
    val errorCodeMessage = httpErrorCodes[code] ?: "Unknown"
    return "Http error code $errorCodeMessage"
}
```


To write a new value to a map, we have to make sure we declare a mutable map using the `MutableMap<KeyType, ValueType>` type and `mutableMapOf()` factory function. Then we assign a value to a key. If the key doesn’t exist, it will add a new key-value pair to the collection. If the key already exists, it will update its value.


```kotlin
// add new key:value pair
httpErrorCodes[500] = "Internal Server Error"

// update value for an existing key
httpErrorCodes[401] = "Requires authentication"
```

Read more:  [https://www.baeldung.com/kotlin/maps](https://www.baeldung.com/kotlin/maps)

# 6. Extensions

Extensions are a way to add new functionality to an existing class or structure, including the ones to which we don’t have access to the code.

In  [Swift](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/extensions)  we can define them using the  `extension`  keyword, followed by the name of the class or structure we want to extend. Extensions must be declared at the top level, outside other classes or structures.


```swift
extension String {
    func doubled() -> String {
        return self + self
    }
}
```


In the example above we defined a new extension function `doubled()` on the `String` type. We can now call this function on any instance of a string like it was part of the original definition.


```swift
let originalStr = "Swift"
let doubledStr = originalStr.doubled()
print(doubledStr) // prints "SwiftSwift"
```

## **Kotlin**

Kotlin uses extension functions that behave the same way and allow us to add new functionality to existing classes. We define them as a top-level function with the class name we want to extend, followed by a dot and function name.

```kotlin
fun String.doubled(): String {
    return this + this
}
```

We can now call this function on any instance of a string like it was part of the original definition.

```kotlin
val originalStr = "Kotlin"
val doubledStr = originalStr.doubled()
println(doubledStr) // prints "KotlinKotlin"
```


# 7. Protocols

A protocol in Swift is a set of properties, methods, and other requirements that a class, structure, or enumeration can adopt by providing actual implementation of those requirements.

We can define a protocol using the  `protocol`  keyword followed by the protocol name, similar to a structure or class declaration. Inside the protocol we can define the properties that can be gettable (`{ get }`) or gettable and settable (`{ get set }`).

```swift
protocol RequestError {
    var errorCode: Int { get }
    var isRecoverable: Bool { get set}
}

protocol PrintableError {
    func buildErrorMessage() -> String
}
```


In the above example, we have defined a protocol  `RequestError`  with a gettable property named  `errorCode`  and another settable property named  `isRecoverable`. Additionally, we defined a protocol  `PrintableError`  that includes a function  `buildErrorMessage()`  that adopters have to implement.

To adopt a protocol, we need to define a class or struct and add  `: ProtocolName`  after its name. We use a comma to declare multiple protocols. The body of the class or struct then needs to define the requirements from the protocol.

```swift
class ServerHttpError: RequestError, PrintableError {
    var errorCode: Int = 500
    var isRecoverable: Bool = false
    
    func buildErrorMessage() -> String {
        return "Server side http error with error code \(errorCode)"
    }
}

struct ConnectionError: RequestError, PrintableError {
    var errorCode: Int
    var isRecoverable: Bool
    
    func buildErrorMessage() -> String {
        return "Local connection error"
    }
}
```


We defined a class  `ServerHttpError`  that adopts the  `RequestError`  and  `PrintableError`  protocol, and defines default values for the two properties and an implementation of the function. Additionally, we have a struct  `ConnectionError`  that declares the two properties and provides an implementation for the function.

We can now create instances of  `ServerHttpError`  and  `ConnectionError`  and pass them as if they were of type  `RequestError`  or  `PrintableError`  as they adopt this protocol. In the  `onRequestError()`  function that accepts a  `RequestError`  type, we check if the error conforms to the  `PrintableError`  protocol to build the error message.


```swift
func onRequestError(error: RequestError) {
    if let printableError = error as? PrintableError {
        print(printableError.buildErrorMessage())
    }
    print("Is recoverable: \(error.isRecoverable)")
}

let firstError = ServerHttpError()
firstError.errorCode = 503
firstError.isRecoverable = false

let secondError = ConnectionError(errorCode: 404, isRecoverable: true)

// "Server side http error with error code 503. Is recoverable: false"
onRequestError(error: firstError)
// "Local connection error. Is recoverable: true"
onRequestError(error: secondError)
```


This is a simple example of how to use protocols. Protocols in Swift support more advanced use cases like inheritance, composition, associated types, generics, and others. Read more at  [https://docs.swift.org/swift-book/documentation/the-swift-programming-language/protocols](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/protocols)

## Kotlin

The above example could be written in Kotlin in several ways, including interfaces, abstract classes, and sealed classes. However, the closest representation of Swift protocol in Kotlin is probably an interface. It supports defining properties and functions, inheritance, composition, and generics.

We use the  `interface`  keyword to define an interface, followed by the name. In the body, we define the properties and functions. Gettable properties are defined using the  `val`  keyword and gettable and settable using the  `var`  keyword.

To let a class implement this interface, we use  `: InterfaceName`  after its name. We use a comma to implement multiple interfaces. We then need to define all the properties and functions with the  `override`  keyword.

```kotlin
interface RequestError {
    val errorCode: Int
    var isRecoverable: Boolean
}

interface PrintableError {
    fun buildErrorMessage(): String
}

class ServerHttpError(
    override val errorCode: Int,
    override var isRecoverable: Boolean
) : RequestError, PrintableError {
    override fun buildErrorMessage(): String {
        return "Server side http error with error code $errorCode"
    }
}

class ConnectionError : RequestError, PrintableError {
    override val errorCode: Int
        get() = 404
    override var isRecoverable: Boolean = true

    override fun buildErrorMessage(): String {
        return "Local connection error"
    }
}
```


We can now create instances of `ServerHttpError` and `ConnectionError` and pass them to functions as they were of `RequestError` type.

```kotlin
fun onRequestError(error: RequestError) {
    if (error is PrintableError) {
        println(error.buildErrorMessage())
    }
    println("$errorMessage. Is recoverable: ${error.isRecoverable}")
}

val firstError = ServerHttpError(errorCode = 503, isRecoverable = false)
val secondError = ConnectionError()
// "Server side http error with error code 503. Is recoverable: false"
onRequestError(firstError)
// "Local connection error. Is recoverable: true"
onRequestError(secondError)
```

Read more at  [https://kotlinlang.org/docs/interfaces.html](https://kotlinlang.org/docs/interfaces.html)

# Conclusion

Knowing common Swift patterns and how they translate to Kotlin can help us understand better what the code does. Whether to see how some feature is implemented on the neighbor platform, perform code reviews, review or write tech specifications/proposals, or work with Kotlin Multiplatform.

We looked at some of the basics of the Swift language and how it compares to Kotlin. Additionally, we covered common patterns that you might find in a typical iOS project like optional bindings, dictionaries, extensions, structures, and protocols.

Let me know in the comments if you found this useful and I encourage you to share your experiences with having to read/review/write Swift code and how you approach it.

References:

-   [https://kotlinlang.org/docs/home.html](https://kotlinlang.org/docs/home.html)
-   [https://docs.swift.org/swift-book/documentation/the-swift-programming-language/thebasics](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/thebasics)

Special thanks to [Andrej Rolih](https://medium.com/u/b31cf96a68c9) for his review and feedback.