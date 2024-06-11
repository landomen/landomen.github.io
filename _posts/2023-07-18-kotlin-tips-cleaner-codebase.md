---
title: 5 Kotlin Tips for a Cleaner Codebase
description: Let’s take a look at a 5 Kotlin tips for writing code that is easy to read, understand and maintain.
date: 2023-07-18 11:00:00 +0100
categories: [Kotlin, Android]
tags: [kotlin, android]
image: /assets/img/posts/kotlin/cover.jpg
---


Kotlin offers a lot of useful  concepts  and structures that make it easy to write  concise  code. But when working  in  a team, the primary goal should be to write code that is easy to read, understand and maintain. We’re going to take a look at a few good practices that should maintain a healthier codebase.

> Note: these are just suggestions and do not imply this is the correct way to do things. The style of the code you write is down to your and your team’s preference.

## 1. Pay attention to the visibility of classes

Pay close attention to the visibility modifier that you apply to new classes and functions. Classes are public by default, which means that the class will be accessible from any other module that depends on this one.

**Supported visibilities for classes:**

-   `public`: default modifier, visible to all the classes inside the module plus any module that depends on this module
-   `internal`: visible to all the classes inside the module, but not outside the module
-   `private`: only visible inside the file or class

Additionally, for class members (functions and properties), there is a  `protected`  modifier, that makes them visible to any class that extends this one.

Try to use the `internal` modifier whenever possible for classes to limit their visibility to only inside the current module. This way you reduce the external API of the module.

If you’re building a library or an SDK, consider enabling the  [Explicit API mode in Kotlin](https://github.com/Kotlin/KEEP/blob/master/proposals/explicit-api-mode.md). You can set it up so that every new class or function needs to have an explicit visibility modifier defined, otherwise, the compiler will show a warning or even throw an exception during build time.

## 2. Keep the number of top-level declarations to a minimum

Top-level functions (functions that exist outside of a class) can be very useful for defining helper/utility functions without the need to declare a class. Especially useful can be [extension functions](https://kotlinlang.org/docs/extensions.html), that enable us to extend the functionality of a class that we do not own, without having to inherit from it or use design patterns.

But it can be very easy to overuse top-level functions. Let’s take the below extension function as an example. It’s defined at the top level, with  `public`  visibility, and it checks if a  `String`  is a valid username. This is a perfectly valid function and it makes sense in the context of a login screen.


<script src="https://gist.github.com/landomen/f4227cbbb73795cb1fd0bc437030b547.js"></script>




However, since the function is  `public`  and defined as a top-level function, it means that it will be accessible from anywhere inside the module where it’s defined and in modules that depend on this one. If you have a single module app and you write this extension function, anytime you want to call a function on a  `String`, this unrelated  `isValidUsername()`  will show up in the suggestions list. Once you have a large number of these types of functions, the developer experience will become degraded, as the suggestions will become more irrelevant.

Try to limit the scope/visibility of extension functions to the file, class, or module where they make contextual sense. We could change the visibility modifier of the above function to either `internal` or even `private`.

Additionally, top-level functions are usually harder to discover, meaning that new developers who don’t know about them, probably won’t use them. It’s better to group related functions inside a dedicated class.

## 3. Prefer readability over saving a few lines of code

Kotlin offers a powerful syntax that makes it easy to do multiple things in a single line. However, sometimes being clever makes the code harder to read for other developers. Prefer clear, simple syntax over complicated chained operators, even if that requires a few additional lines of code.

Here is a bit of a far-fetched example, but it should hopefully illustrate the point. We have a function that should return the square of the input number if the input number is larger than 0. Otherwise, it should return 0. We try to be clever and write it in a single line using multiple operators.


<script src="https://gist.github.com/landomen/90d7ceb79ba5adee45b8c04de6443fcf.js"></script>



And here is the same function but written in a more boring, almost Java-like, way.



<script src="https://gist.github.com/landomen/78745b8ba0db72bf1d1687dd08f1b7c6.js"></script>



While not as clever as the one-line solution, the “boring” solution is arguably easier to read and understand to whoever comes across it. As software engineers, we spend more time reading code than writing it, so make it easier for your teammates and your future self.

## 4.  Prefer creating a dedicated data class to using a Pair or Triple

The built-in  `Pair`  and  `Triple`  classes can be helpful when you need to return two or three values from a function. However, there is no context attached to the properties of those two classes. That makes it harder for your teammates to understand what the result of a function that returns  `Pair<String, String>`  means. It might require them to read the function body to figure out what the first  `String`  represents and what the second one is.

Let’s say we have a function to authenticate a user with the backend that returns access and refresh tokens as a  `Pair<String, String>`.


<script src="https://gist.github.com/landomen/9781e5031eb7f6ec58ea4db545938aa2.js"></script>


We can improve it by creating a new dedicated data class named AuthenticationTokens that contains explicitly named properties accessToken: String and refreshToken: String. It makes it clearer what is being returned and what each value represents.



<script src="https://gist.github.com/landomen/14b1c0e34f4cb31c7d6daecd49a387b2.js"></script>



## 5.  Prefer exhaustive when statements

When using a `when` statement to check for a value of a limited class hierarchy, like `enum class`, `sealed class`, or `sealed interface`, prefer defining all possible values, instead of using `else` branch as a catch-all.

Using the  `else`  branch can cause potential bugs when a new value is added, as it’s up to the developer who is adding a new value to be aware of all the usages.


<script src="https://gist.github.com/landomen/82e550a2c320ff4462fa2f43d25384e9.js"></script>




Let’s say we have possible analytics events defined as an  `enum`. These events are tracked in multiple functions across the codebase, and we use an  `else`  branch for tracking the  `PROFILE_DELETED`  event. A new developer joins the team and is tasked with adding a new event called  `PROFILE_CANCELLED`. Because they are not familiar with all the places where these events are checked, they miss this one check and it results in  `trackProfileDeleted()`  being called for both  `PROFILE_DELETED`  and  `PROFILE_CANCELLED`  due to the  `else`  branch condition. Maybe the bug is caught in code review, but maybe it gets released to production and it affects the analytics metrics.

This can easily be avoided by declaring all the possible values explicitly and making the  `when`  statement  exhaustive. When a new value is added, the compiler will complain that the  `when`  statement is not exhaustive, making sure that we do not miss a usage.



<script src="https://gist.github.com/landomen/a53f5a389ecf0738473fcc521116d711.js"></script>



# Conclusion

We’ve taken a look at five tips on how to improve the readability and maintainability of your code. Here’s a short recap:

1.  Try to use the  `internal`  modifier whenever possible for classes to limit their visibility to only inside the current module.
2.  Keep the number of top-level declarations to a minimum and be aware of their visibility.
3.  Prefer readability even if that means additional few lines of code.
4.  Prefer creating a dedicated `data class` to using a `Pair` or `Triple`.
5.  Prefer exhaustive when statements as using the `else` branch can result in bugs.

I hope these tips will help you improve the readability and maintainability of your code. Let me know your thoughts in the comments and share tips from your experiences.