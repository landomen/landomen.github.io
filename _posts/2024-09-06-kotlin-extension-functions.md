---
title: Managing the Scope of Extension Functions in Kotlin
description: Extension functions in Kotlin enhance classes without inheritance, but overuse can clutter suggestions, especially in multi-module projects.
date: 2024-09-06 21:00:00 +0100
categories: [Kotlin, Android, Kotlin Multiplatform, Java]
tags: [kotlin, android, kotlin multiplatform, java]
image: /assets/img/posts/kotlin/cover.jpg
---


[Extension functions](https://kotlinlang.org/docs/extensions.html) in Kotlin enable us to extend the functionality of a class that we do not own, without having to inherit from it or use design patterns. They are usually declared as a top-level function, directly under packages, existing outside of a class.


Here is an example of two extension functions that check whether a `String` is a valid username and password. This common and perfectly valid usage of extension functions makes sense in the context of login and register screens.


<script src="https://gist.github.com/landomen/5e0c640529c576208221e07cd6e6396a.js"></script>


## What’s the issue?

Since the above functions have a  `public`  visibility and are defined as top-level functions, they will be accessible from anywhere inside the module where they are defined and in modules that depend on this one.

If we have a single module app and we write these extension functions, anytime we want to call a function on a  `String`, this unrelated  `isValidUsername()`  will show up in the suggestions list.

In a multi-module setup, these functions will appear in any class that belongs to a module depending on the one where the extension functions are defined. Depending on the dependency graph, this could even occur in other modules.

Once we have a large number of extension functions, the developer experience will become degraded, as the suggestions will become more irrelevant.

## How to solve this?

As I shared in the article  [5 Kotlin Tips for a Cleaner Codebase](https://medium.com/@domen.lanisnik/5-kotlin-tips-for-a-cleaner-codebase-3582f2e4e2af), it’s important to limit the scope of extension functions to the file, class, or module where they make contextual sense.

We’ll explore several solutions for managing scope to keep the codebase clean and maintainable.

## Use internal or private visibility

Depending on the module structure of the app and usages, we can simply change the visibility modifier of the extension functions to limit their scope.

For example, if the above extension functions are defined and used inside a single module, we can change their scope to  `internal`  and they will not be accessible to any other modules. However, if we have a single module app, then this approach won’t make a difference.

If they are declared at the top of a specific file, like  `LoginViewModel`, and only used there, then we can change their visibility to  `private`. However, in that case, we cannot reuse them in other classes, like  `RegisterViewModel`.

## Declare them inside a class

If the extension functions only make sense inside the context of a single class, then we can simply move them inside that class and declare them as  `private`.

Sticking with our example, if we only have a login screen, because for example the registration can not be done through the app, we could move them to a  `LoginViewModel`.


<script src="https://gist.github.com/landomen/577413131da115fa78e0487736a89f6a.js"></script>


The downside of this approach is that we cannot use them outside of this class.

## Declare them inside an interface

Improving on the previous solution, we can create a new interface  `UserInputValidation`  and declare our extension functions inside of it. A class has to implement the new interface to be able to access them. This way the scope of the extension functions is kept well contained and the functions are not accessible from unrelated places.


<script src="https://gist.github.com/landomen/1c88a39b9a451861b8ec7dcbf13d2a2a.js"></script>



It also makes it easy to reuse them across the codebase and modules. For example,  `LoginViewModel`  and  `RegisterViewModel`  can both implement this interface and access the functions. If we later introduce a new password change screen, the  `ChangePasswordViewModel`  can just implement the interface and start using the functions.

## Conclusion

This was a short article on how to manage the scope of extension functions to keep our codebase clean. The approaches we covered are using a lower-scope visibility modifier and declaring them inside a class or interface instead of at a top level to make them more contextual.

<hr style="width:50%; margin-left:25% !important; margin-right:25% !important; height:2px;">


Please share your advice in the comments on how you manage extension functions in your projects.