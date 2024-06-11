---
title: Functional Interfaces in Kotlin
description: Learn what functional interfaces are and how to use them in Kotlin.
date: 2024-01-24 11:00:00 +0100
categories: [Kotlin, Android, Kotlin Multiplatform, Java]
tags: [kotlin, android, kotlin multiplatform, java]
image: /assets/img/posts/kotlin-functional-interfaces/cover.jpg
---


Interfaces with a single function in Kotlin can be converted to functional interfaces. Kotlin will use SAM (Single Abstract Method) conversion to make it possible to use a lambda instead of creating a class or extending an object, removing boilerplate code and making it more readable.

## Creating functional interfaces

Let’s take the following  `PageObserver`  interface as an example. It’s a standard interface with a single function and an instance of it needs to be passed to the  `registerPageObserver`  function.

```kotlin
interface PageObserver {
  fun onPageVisible(pageIndex: Int)
}

registerPageObserver(object : PageObserver {
  override fun onPageVisible(pageIndex: Int) {
    // do something
  }
})
```


We can simplify the code by turning the  `PageObserver`  interface into a functional interface. Functional interfaces can be defined by prepending a  `fun`  modifier before the  `interface`  keyword:

```kotlin
fun interface PageObserver {
  fun onPageVisible(pageIndex: Int)
}
```

We can now replace the creation of an instance of this interface with a simple lambda function:


```kotlin
registerPageObserver { pageIndex ->
  // do something
}
```


If trying to add the  `fun`  modifier to an interface with more than one function, the compiler will throw an error “_Fun interfaces must have exactly one abstract method”._

## Java and Kotlin interoperability

Kotlin also supports SAM conversions for Java interfaces. This is why for example on Android you can use the lambda version of existing system Java interfaces like  `View.OnClickListener`.


```kotlin
public interface OnClickListener {
  void onClick(View v);
}

val view = findViewById<Button>(R.id.btn_start)
view.setOnClickListener { view ->
  // SAM converted lamba
}
```



## Conclusion

Functional interfaces in Kotlin are a simple construct that can improve the readability and maintainability of your codebase by removing boilerplate code.

It’s a good construct to be aware of, but you should only use it when it’s needed as there is runtime overhead involved due to conversion.

Alternatively, you can consider using a functional type as a parameter or a  [type alias](https://kotlinlang.org/docs/type-aliases.html).

References and further reading:

-   [https://kotlinlang.org/docs/fun-interfaces.html](https://kotlinlang.org/docs/fun-interfaces.html)
-   [https://kt.academy/article/ek-function-types](https://kt.academy/article/ek-function-types)