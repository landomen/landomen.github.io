---
title: 'Kotlin’s Builder Functions: A Better Way to Create Lists, Maps, Strings & Sets'
description: Kotlin offers several convenience functions to create lists, maps, strings, and more without the usual boilerplate code.
date: 2025-05-09 21:30:00 +0200
categories: [Kotlin, Android, Kotlin Multiplatform]
tags: [kotlin, android, kotlin multiplatform]
image: /assets/img/posts/kotlin/cover.jpg
---

Kotlin offers several convenience functions to create lists, maps, strings, and more without the usual boilerplate code.

In this short post, we’ll examine a few common functions in the Kotlin standard library that make constructing those objects easier.


## Building a List

The usual approaches to creating and populating a dynamic list are, for example:


```kotlin
val newList = mutableListOf<Int>()
newList.add(1)
if (conditionFullfilled) {
    newList.add(2)
}

// OR

val newList = mutableListOf<Int>().apply { 
    add(1)

    if (conditionFullfilled) {
        add(2)
    }
}
```



We can make that easier by using the `buildList {}` function that creates a mutable list under the hood, allows us to call functions on it, and then returns an immutable list.


```kotlin
val newList = buildList { 
    add(1)

    if (conditionFullfilled) {
        add(2)
    }
}
```


Looking at the implementation of the `buildList()` function, we can see that it works similarly to our initial code. It calls the `buildListInternal()` function that constructs a new mutable list and applies the actions to it. And since it’s an `inline` function, it also means that the overhead of the additional lambda is removed, since it will copy the code to the call site.


```kotlin
@SinceKotlin("1.6")
@WasExperimental(ExperimentalStdlibApi::class)
@kotlin.internal.InlineOnly
@Suppress("LEAKED_IN_PLACE_LAMBDA", "WRONG_INVOCATION_KIND")
public inline fun <E> buildList(@BuilderInference builderAction: MutableList<E>.() -> Unit): List<E> {
    contract { callsInPlace(builderAction, InvocationKind.EXACTLY_ONCE) }
    return buildListInternal(builderAction)
}
```

`buildList()`  can be used for any type, including primitives like  `Int`,  `Float`,  `Double`, and  `Long`. However, if we want to avoid auto-boxing when storing and retrieving the elements, we can use dedicated functions:

-   `buildIntList {}`: constructs a  `MutableIntList`  and returns a  `IntList`,
-   `buildLongList {}`: constructs a  `MutableLongList`  and returns a  `LongList`,
-   `buildFloatList {}`: constructs a  `MutableFloatList`  and returns a  `FloatList`,
-   `buildDoubleList {}`: constructs a  `MutableDoubleList`  and returns a  `DoubleList`.

## Building a String

A typical way of constructing a string that requires concatenation based on some condition is to create a  `StringBuilder`  object and then call the  `append`  functions on it and convert it to a string.


```kotlin
val sb = StringBuilder()
sb.append("Some")
if (conditionFullfilled) {
    sb.append(" string")
}
val newString = sb.toString()

// OR

val newString = StringBuilder().apply { 
    append("Some")

    if (conditionFullfilled) {
        append(" string")
    }
}.toString()
```


We can make that easier by using the `buildString {}` function that creates the `StringBuilder` object for us and returns the output string. We can call the usual `append` functions on it inside the code block.



```kotlin
val newString = buildString {
    append("Some")

    if (conditionFullfilled) {
        append(" string")
    }
}
```


Looking at the implementation of the `buildString()` function, we can see that it works the same as our initial code. And since it’s an `inline` function, it also means that the overhead of the additional lambda is removed, since it will copy the code to the call site.



```kotlin
@kotlin.internal.InlineOnly
public inline fun buildString(builderAction: StringBuilder.() -> Unit): String {
    contract { callsInPlace(builderAction, InvocationKind.EXACTLY_ONCE) }
    return StringBuilder().apply(builderAction).toString()
}
```


## Building a Set

Similar to building a list and string, there is a helper function for building a set. It constructs a mutable set, allows us to call functions on it, and then returns an immutable version of the set that preserves the insertion order.


```kotlin
val newSet = buildSet {
    add(1)

    if (conditionFullfilled) {
        addAll(someOtherSet)
    }
}
```

The  `buildSet()`  function can be used for any type. However, there are also other versions of the function for building type-specific sets:

-   `buildIntSet {}`: constructs a  `MutableIntSet`  and returns a  `IntSet`,
-   `buildFloatSet {}`: constructs a  `MutableFloatSet`  and returns a  `FloatSet`,
-   `buildLongSet {}`: constructs a  `MutableLongSet`  and returns a  `LongSet`.

The three type-bound sets use a flat hash table underneath and don’t preserve insertion order.

## Building a Map

We can build a new map using the  `buildMap {}`  function. It constructs a new  `MutableMap`  underneath, allows us to call functions on it, and then returns an immutable  `Map`.


```kotlin
val newMap = buildMap { 
    put("key", "value")
    
    if (conditionFullfilled) {
        putAll(someOtherMap)
    }
}
```



If our keys or values are primitives like  `Int`,  `Float`,  `Double`, or  `Long`, we can use optimized variants of the function:

-   `buildIntIntMap {}`: constructs a  `MutableIntIntMap`  and returns a  `IntIntMap`, where the keys and values are  `Int`  primitives,
-   `buildLongLongMap {}`: constructs a  `MutableLongLongMap`  and returns a  `LongLongMap`, where the keys and values are  `Long`  primitives,
-   `buildFloatFloatMap {}`: constructs a  `MutableFloatFloatMap`  and returns a  `FloatFloatMap`, where the keys and values are  `Float`  primitives,
-   `buildObjectObjectMap {}`: constructs a  `MutableObjectObjectMap`  and returns a  `ObjectObjectMap`, where the keys and values are reference types.

There are even more variants of this function for different combinations of key and value types:  `buildIntFloatMap {}`,  `buildIntLongMap {}`,  `buildObjectFloatMap {}`  and so on.

## Other builders

The above functions are all part of the Kotlin standard library and should be available in most projects. There are other similar builder functions declared in other dependencies, for example:  `buildSpannedString {}`  and  `buildAnnotatedString {}`  that are part of Compose, and  `buildJsonObject {}`  that is part of the Kotlinx Serialization library.

# Conclusion

The Kotlin Standard Library offers several convenience functions for building common types like lists, sets, maps, and strings. They provide a standardized way of constructing those types and avoid common boilerplate.


<hr style="width:50%; margin-left:25% !important; margin-right:25% !important; height:4px;">


I hope you found this useful and learned something new. Let me know what other builder functions you’re using.