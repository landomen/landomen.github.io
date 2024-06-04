---
title: Data Objects in Kotlin
description: Learn more about a new Kotlin language feature introduced in version 1.7.20.
date: 2023-01-20 11:00:00 +0100
categories: [Kotlin, Android]
tags: [kotlin, android]
---

Data objects are a new Kotlin language feature introduced in version `1.7.20` and are currently planned to be released in version `1.9`. We’ll take a closer look at what they are and what issue they are trying to solve.

## What issue are data objects solving?

Below we have a typical example of a sealed class hierarchy where we are using a  `sealed interface`  (we could also use a  `sealed class`) to define possible states for a Profile screen. We are using a  `data class`  for the success state and an  `object`  for error and loading states since we don’t need any additional information.


<script src="https://gist.github.com/landomen/0ff3b3885b338fecf15ff7783250cb05.js"></script>


What if we want to log/print the current screen state for either debugging reasons or for sending it to an analytics service? The string representation of the `ProfileScreenState.Success` data class contains the name of the class and all its properties, which is exactly what we want.  

But if we print a plain `object` declaration in Kotlin, we’ll get a string containing the full package name, object name, and the address of where this object is stored in memory. And since objects are singletons in Kotlin, the address part will remain the same each time we print that object and is not relevant to us.

```kotlin
com.dataobjects.example.ProfileScreenState$Loading@6d03e736  
Success(username=exampleUser1)  
com.dataobjects.example.ProfileScreenState$Error@5fd0d5ae
```

One solution would be to override the `toString(): String` function on each object and provide our implementation, but that seems like a lot of boilerplate code for such a trivial issue.


<script src="https://gist.github.com/landomen/807c33487638880bac742b62447fe96a.js"></script>


## Data  objects

Kotlin is planning to solve this using data objects. A `data object` is identical to a regular `object` but provides a default implementation of the `toString()` function that will print its name, removing the need to override it manually, and aligning behavior with a `data class` definition. They are especially useful for sealed class hierarchies to match behavior with data classes.

Data objects are currently an experimental feature, so to try them out, we have to tell the compiler to use version Kotlin  `1.9`:

```kotlin
kotlinOptions.languageVersion = "1.9"
```

After syncing the project we receive a suggestion on our existing objects that are a part of a sealed hierarchy to convert them to a `data object`.