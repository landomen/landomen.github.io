---
title: Modeling Domain Values and Restricted Class Hierarchies in Kotlin
description: Learn different ways how to model domain values and restricted class hierarchies in Kotlin.
date: 2024-01-06 11:00:00 +0100
categories: [Kotlin, Android, Kotlin Multiplatform]
tags: [kotlin, android, kotlin multiplatform]
image: /assets/img/posts/kotlin-modeling-domain/cover.jpg
---



There are many ways how to model domain values and restricted class hierarchies in Kotlin: enum classes, type aliases, inline value classes, sealed classes, and sealed interfaces. We’ll take a look at each of them and describe what they offer and when to use them.

## Enum classes

[**Enums**](https://kotlinlang.org/docs/enum-classes.html)  are the most simple way to define a limited set of values. An enum class can have none, one, or multiple properties that each of the entries has to provide a value for. These values are usually static/hardcoded and don’t change at runtime.

The benefit of using enums is that we get type safety. The compiler will warn us when we reference them in a  `when`  statement and do not cover all cases. This can prevent bugs when adding new values, as it helps us to not forget to update existing usages.


```kotlin
enum class VehicleStyle(val id: String, val sortOrder: Int) {
    CONVERTIBLE("convertible", 3),
    COUPE("coupe", 1),
    HATCHBACK("hatchback", 2)
}

fun onStyleSelected(style: VehicleStyle) {
    // reading properties
    Analytics.trackStyleSelected(style.id)
    
    // checking for specific enum entry
    when(style){
        VehicleStyle.CONVERTIBLE -> TODO()
        VehicleStyle.COUPE -> TODO()
        VehicleStyle.HATCHBACK -> TODO()
    }
}

// cannot pass anything to this function but VehicleStyle
onStyleSelected(VehicleStyle.COUPE)
```



Enums can be used whenever we have many possible states that are mutually exclusive. Prefer using enums over constants to model values as they offer type safety and provide more context over what the input represents.

## Type aliases

[**Type aliases**](https://kotlinlang.org/docs/type-aliases.html)  allow us to provide new names for existing types. They are helpful for modeling as we can give more meaningful names for generic types like lambdas or primitives, making our code more readable and easier to understand by providing more context.


```kotlin
typealias LoadingCallback = (Boolean) -> Unit

class DataLoader(loadingCallback: LoadingCallback){
    // some class
}

DataLoader(loadingCallback = { success ->
    // same usage as the underlying type
})
```




Type aliases do not introduce a new type, but are equal to the underlying type they reference. Therefore it doesn’t guarantee true type safety, as it’s possible to pass any instance of the underlying type as a value.

Below is an example of this where we define a new type alias for an integer to represent a distance unit. The  `convertMileToKilometer(miles: Mile)`  function accepts a  `Mile`  type as an input argument and returns a  `Kilometer`. But because both  `Mile`  and  `Kilometer`  represent an  `Int`  underneath, it’s possible to pass a  `Kilometer`  as input. As far as the compiler is concerned, we are passing integers around, so no error is raised.


```kotlin
typealias Kilometer = Int
typealias Mile = Int

fun convertMileToKilometer(miles: Mile): Kilometer {
    return (miles * 1.6f).toInt()
}

// allowed to pass Kilometer (or any Int) as Mile
val miles: Mile = 1
val kilometers: Kilometer = 5
convertMileToKilometer(miles = kilometers) // OK
convertMileToKilometer(miles = 10) // OK
```



For a more strongly typed solution, we can use inline value classes.

## Inline value classes

[**Inline value classes**](https://kotlinlang.org/docs/inline-classes.html)  are a way to wrap existing types into more domain-specific types. As the name implies, inline value classes are inlined into their usages similar to  [inline functions](https://kotlinlang.org/docs/inline-functions.html). This means that the code for the value class is copied over to the place where it’s called, without creating an actual object. This provides less overhead compared to introducing a new regular class wrapper.

The main difference compared to type aliases is that inline value classes introduce a truly new type underneath as opposed to just an alternate name. This solves the issue we mentioned with type aliases, where it was possible to pass a  `Kilometer`  or any  `Int`  as a  `Mile`.


```kotlin
@JvmInline
value class Kilometer(val value: Int)

@JvmInline
value class Mile(val value: Int)

fun convertMileToKilometer(miles: Mile): Kilometer {
    return Kilometer((miles.value * 1.6f).toInt())
}

// not allowed to pass Kilometer as Mile
val miles = Mile(1)
val kilometers = Kilometer(5)
convertMileToKilometer(miles = kilometers) // error
convertMileToKilometer(miles = 10) // error
convertMileToKilometer(miles = miles) // OK
```


Value classes can also contain other functions and simple properties, just like regular classes.


```kotlin
@JvmInline
value class Kilometer(val value: Int) {

    val meters: Int
        get() = value * 1000

    fun printWithUnit() {
        println("$value km")
    }
}

// usage
val kilometers = Kilometer(5)
println(kilometers.meters) // prints "5000"
kilometers.printWithUnit() // prints "5 km"
```



Use value classes to provide more meaning to otherwise generic input types like integers, strings, and so on.

## Sealed classes and interfaces

Regular interfaces and abstract classes are the typical approaches to defining limited hierarchies. However, when using them in Kotlin, they have a few drawbacks. The biggest one is that not all possible subclasses are known ahead of time and new subclasses can be defined outside of the module, making it easy to not cover all the cases.


```kotlin
interface ScreenState
data object Loading: ScreenState
data class Success(val data: ScreenData): ScreenState
data class Error(val exception: Exception): ScreenState

// IDE doesn't offer to prefill branches
// compiler doesn't complain that we didn't check Loading and Error states
when(screenState){
    is Success -> TODO()
}
```



[**Sealed classes and interfaces**](https://kotlinlang.org/docs/sealed-classes.html)  allow us to define a restricted class hierarchy where all the direct subclasses are known at compile time. We can think of a sealed class as a sort of enum where all the possible values are known ahead of time, but each value (subclass) can have different properties and there can be multiple instances of the same subclass. All subclasses need to be defined inside the same module. This allows us to check for all the possible cases when using a  `when`  statement, making the code more robust.

Sealed interfaces are similar to sealed classes but don’t require any properties. A class can also implement multiple sealed interfaces, making it a part of multiple hierarchies.


```kotlin
sealed interface ScreenState
data object Loading : ScreenState
data class Success(val data: ScreenData) : ScreenState
data class Error(val exception: LoadingError) : ScreenState

sealed class LoadingError(val errorCode: Int) {
    data object Unknown : LoadingError(0)
    data class ServerError(val message: String) : LoadingError(500)
}

// IDE offers to prefill all the possible values
// compiler warns us if we don't check all values or add else branch
when (screenState) {
    is Success -> TODO()
    is Error -> {
        println(screenState.exception.errorCode)
        
        when(screenState.exception){
            is LoadingError.ServerError -> TODO()
            LoadingError.Unknown -> TODO()
        }
    }
    Loading -> TODO()
}
```


Sealed classes and interfaces are helpful when defining all sorts of models with limited values. This ranges from operation results like network calls, and errors to app states, and so on.

# Conclusion

We took a look at a few different approaches how to model domain values to provide the most context and make your Kotlin code more readable and safer.

Using the correct constructs when modeling the domain values is an important part of defining your business logic. It affects your code in multiple ways:

-   **readability**: using the correct construct can make your code easier to read, understand, and search through. This can be achieved by providing additional context over generic types using enums, type aliases, and value classes.
-   **extensibility**: using the correct construct can make your code easier to extend with new functionality in the future. Adding a new value to an enum or a sealed class is straightforward and the compiler will help us make sure we don’t miss any usages.
-   **safety**: using the correct construct, such as enum class, value class, and sealed class, can make your code safer and less error-prone, as the compiler will help us make sure we cover all cases and don’t pass wrong values.

**Reference and further reading:**

-   [https://kotlinlang.org/docs/enum-classes.html](https://kotlinlang.org/docs/enum-classes.html)
-   [https://kotlinlang.org/docs/type-aliases.html](https://kotlinlang.org/docs/type-aliases.html)
-   [https://kotlinlang.org/docs/inline-classes.html](https://kotlinlang.org/docs/inline-classes.html)
-   [https://kotlinlang.org/docs/sealed-classes.html](https://kotlinlang.org/docs/sealed-classes.html)