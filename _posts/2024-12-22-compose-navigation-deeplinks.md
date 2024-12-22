---
title: Type Safe Deep Links in Compose
description: We cover what deep links are and how to implement them using the new type-safe APIs of the Navigation Compose library.
date: 2024-12-22 12:00:00 +0100
categories: [Jetpack Compose, Android, Compose, Navigation, Compose Navigation]
tags: [compose, android, jetpack compose, navigation]
image: /assets/img/posts/compose/navigation/deeplinks/sample_app_preview.jpg
---



Version 2.8.0 of Navigation Compose brought much-needed type safety to the official Compose navigation library. Instead of string routes with path arguments, we can use classes with specific property types.

The  [official documentation](https://developer.android.com/develop/ui/compose/navigation#deeplinks)  provides a good overview of the new type-safe approach for defining destinations and navigation between them. However, when it comes to deep links, it only covers basic usage and doesn’t provide additional information on how deep link generation and matching work.

While performing migration on one of the pet projects, I encountered an issue with migrating the deep links. This article aims to clarify the usage of deep links in the latest versions of the navigation library and help others in the migration process.

> Some level of familiarity with the  [Navigation Compose](https://developer.android.com/develop/ui/compose/navigation)  library is required.

# What is a deep link?

In Android, a deep link is a link that takes you directly to a specific destination within an app. It’s typically in the form of a URL with either a custom scheme (`sample://product/123`) or with a regular web scheme (`https://sample.com/product/123?available=true`).

A deep link consists of several parts that help define how it will be handled:

-   **scheme**: defines the protocol or type of link being used. For example,  `http`,  `https`,  `mailto`, or a custom scheme like  `sample`.
-   **host**: the domain or main part of the URL after the scheme. In web URLs, this is the domain name (e.g.,  `sample.com`). In custom schemes, you can define any string as the host.
-   **path:**  specifies the location of a specific resource or page in the app. It comes after the host and typically begins with a forward slash (`/products/1234`). The path can have multiple levels, like  `/products/1234/details`.
-   **query parameters:** provide additional information in the form of key-value pairs. These parameters come after a question mark (`?`) and are used to pass extra data in the URL. Multiple query parameters can be chained using an ampersand (`&`), such as  `?available=true&source=fb`.

When the user clicks on this link (from an email, web page, etc.), Android checks if any app is registered to handle this URL scheme and, if so, opens the associated app and screen.

# Sample app

We have a simple app with two screens:

-   Input screen: has inputs for first name, last name, and age. Pressing the Submit button displays the inputs on the second screen
-   Result screen: shows the user’s inputs from the input screen.

We want to allow navigating to the result screen directly by leveraging deep links. Clicking on a deep link  `https://deeplink.sample.com/result/smith/john?age=28`  should open the app and jump straight to the result screen showing the correct data from the deep link.


![Sample app with the input screen on the left and the result screen on the right.](/assets/img/posts/compose/navigation/deeplinks/sample_app_preview.jpg)
_Sample app with the input screen on the left and the result screen on the right._

You can find the full implementation on [GitHub](https://github.com/landomen/ComposeNavigationDeeplinksSample).



# Declaring deep links in the app

To enable deep linking into an app, we first have to expose the deep links to the Android system. This will allow it to associate the deep link with the app and open it when pressed.

We need to add a new intent filter to the  `AndroidManifest.xml`  file and declare the format of the deep link. This can be a classic web URL or a custom schema. For this sample app, we want to support  `[https://deeplink.sample.com](https://deeplink.sample.com./)`  web URL format, so we have to declare the  `schema="https"`  and  `host="deeplink.sample.com”`. If we had a custom deep link such as  `sample://deeplink/`  , then we would need to declare  `scheme="sample"`  and  `host="deeplink"`.

It’s not needed at this stage to declare any additional path parameters or arguments. That matching is done later at the Compose level.


<script src="https://gist.github.com/landomen/c9df29c0e3de3a60e1345a625af4c791.js"></script>


## Testing deep links

We have two main options to test deep links:

-   sending the deep link via a message app or storing it in the text editor and clicking on it,
-   by triggering it through an adb command.

To trigger a deep link through adb, we have to use the following command that simulates a click on the deep link:


```bash
adb shell am start -W -a android.intent.action.VIEW -d "deeplink_to_test" app_package
```

`"deeplink_to_test"`  is the deep link we want to trigger, for example  `"https://deeplink.sample.com”`.  `app_package`  is the app’s package name, for example  `com.landomen.composenavigationdeeplinks`.

```bash
adb shell am start -W -a android.intent.action.VIEW -d "https://deeplink.sample.com" com.landomen.composenavigationdeeplinks
```


If everything is configured correctly, the app should open when the command is executed. If it doesn’t, then most likely the  `AndroidManifest`  configuration is wrong and the Android system can’t resolve the deep link or associate it with our app. In that case, we have to make sure we correctly defined the schema and host in the intent filter.

# Deep link implementation before Type Safety update in 2.8.0

> Feel free to skip this part if you are not performing a migration from an older version of navigation library.

In older versions of the navigation library, we defined a composable deep link by using the  `navDeepLink { }`  builder and specifying the  `uriPattern`  string. It describes the entire deep link, with placeholders for path and query arguments, similar to how a composable route is defined. The main difference is that we have to provide the full URL, including scheme and host.

The names of the path and query arguments should ideally match the ones specified in the route, so we can easily retrieve them from the back stack entry arguments.


<script src="https://gist.github.com/landomen/3e7a422feb9ed014c3a58af16e527f8b.js"></script>


Triggering the following command in the terminal should open the app and navigate directly to the result screen displaying the correct data:


```bash
adb shell am start -W -a android.intent.action.VIEW -d "https://deeplink.sample.com/result/smith/john?age=20" com.landomen.composenavigationdeeplinks
```


# Type Safe Implementation

## Pre-requirements

A pre-requisite for implementing type-safe deep links is to add the latest version of  [Navigation Compose](https://developer.android.com/develop/ui/compose/navigation)  dependency to the app. Since it works with serializing objects, we also need to add the Kotlin serialization dependency and Gradle plugin. Here is how the two dependencies can be declared in the version catalog file (`libs.versions.toml`):


<script src="https://gist.github.com/landomen/49bf7e425e523238c7ee8602b043a546.js"></script>



And how to apply them to the app module’s `build.gradle.kts` file:

<script src="https://gist.github.com/landomen/c5500850cc662cea6966bf60d1424821.js"></script>


## Creating destinations

While we could previously use strings to define destination routes for screens, we now have to declare classes and objects with the  `@Serializable`  annotation. This allows the navigation library to generate an underlying string route based on the class definition, thus providing type safety.

Input arguments are defined using regular class properties. We can use any primitive data type and even custom types, but for that, we have to write a custom serializer.

<script src="https://gist.github.com/landomen/61d800e9be9a3b6e97790f539400d5f9.js"></script>




## Define a deep link

We can create a deep link by using the  `navDeepLink`  function that accepts a destination type and a base path string. The destination type has to be of the same type as the composable for which the deep link is defined.

The base path should be the deep link URL up to (but not including) any path parameters. The navigation library will append the path parameters and query arguments to it automatically based on the destination class structure (more in the next section).

Parameters and arguments can be retrieved from the back stack entry using the  `.toRoute<DestinationType>()`  extension function. It converts the back stack entry to the destination object, allowing us to read the properties from it.


<script src="https://gist.github.com/landomen/42f02826e3ef23c07d2f70ea0f0b1c29.js"></script>



## Understanding deep link generation

As mentioned, the navigation library uses serialization to convert the destination class/object into a string representation. In our case, the  `Result`  class has three properties:

-   `lastName`  of type  `String`,
-   `firstName`  of type  `String`,
-   and  `age`  of type  `Int`  with a default value of 0.


<script src="https://gist.github.com/landomen/a805dac1a006ea272f8601aeb9900dd9.js"></script>



The navigation library does the following to generate the final deep link URL from the above class:

-   it takes all non-optional properties and adds them to the URL’s path, based on the order they’re declared,
-   adds all optional properties and properties with default values as query parameters of the URL, based on the order they’re declared.

That means that with the base path of  `[https://deeplink.sample.com/result](https://deeplink.sample.com/result,)`  and the class definition above, the generated appended part is  `/{lastName}/{firstName}?age={age}`.

We can change the name of the properties with the  `@SerialName`  annotation on the property. This is useful if we have an existing deep link that is also used on the web app and we can’t change the names.

## Testing

Triggering the following command in the terminal should open the app and navigate directly to the result screen displaying the correct data:



```bash
adb shell am start -W -a android.intent.action.VIEW -d "https://deeplink.sample.com/result/smith/john?age=20" com.landomen.composenavigationdeeplinks
```


# Conclusion

Deep links are a common navigation pattern in Android apps that allow us to directly jump to a specific screen in the app. Their implementation is quite simple when using the Navigation Compose library. However, with the recent type-safe APIs, their declaration and usage have slightly changed.

In this article, we’ve explored what deep links are, how to declare them, and how to implement them using the new type-safe API. We’ve also looked at migrating the old implementation.

Hope you found this helpful and let me know your experience with deep links in the comments.


You can find the full implementation on [GitHub](https://github.com/landomen/ComposeNavigationDeeplinksSample).


## Resources

-   [https://developer.android.com/develop/ui/compose/navigation#deeplink](https://developer.android.com/develop/ui/compose/navigation#deeplinks)
-   [https://developer.android.com/guide/navigation/design/deep-link](https://developer.android.com/guide/navigation/design/deep-link)
-   [https://tkhs0604.medium.com/implementation-of-deeplinks-with-type-safe-navigation-compose-apis-601b3c9e381c](https://tkhs0604.medium.com/implementation-of-deeplinks-with-type-safe-navigation-compose-apis-601b3c9e381c)
-   [https://medium.com/androiddevelopers/type-safe-navigation-for-compose-105325a97657](https://medium.com/androiddevelopers/type-safe-navigation-for-compose-105325a97657)