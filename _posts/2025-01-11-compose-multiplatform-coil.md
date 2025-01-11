---
title: Loading Images with Coil in Compose Multiplatform
description: We take a look at how to add the popular Coil image loading library to a Compose Multiplatform project.
date: 2025-01-11 11:30:00 +0100
categories: [Compose Multiplatform, Kotlin Multiplatform, Compose, Android, iOS, Coil, Ktor]
tags: [compose multiplatform, compose multiplatform, kmp, compose android, jetpack compose, coil, ktor]
image: /assets/img/posts/compose/multiplatform/coil/coil_combined_result_small.png
---


We’ll take a look at how to add the popular  [Coil](https://coil-kt.github.io/coil/)  image loading library to a Kotlin Multiplatform project that uses the Compose Multiplatform framework to share UI across Android, iOS, Desktop, and web.

Recently, I was exploring Kotlin Multiplatform and Compose Multiplatform for a series of articles focused on that topic (released soon). One of the things I needed to do was to load remote images from the network and display them in the app. I use Coil regularly on Android but found integrating it into a Compose Multiplatform app challenging due to a lack of documentation and resources.

This article provides a simple guide on how to add and use Coil in KMP Compose apps.


![Result of loading remote images in Android, iOS, Web and Desktop apps.](/assets/img/posts/compose/multiplatform/coil/coil_combined_result_small.png)
_Result of loading remote images in Android, iOS, Web and Desktop apps._


# Declaring the dependencies

> This article is focused on using the gradle version catalog to manage dependencies. If that is not the case, you can still follow along and declare dependencies directly in the  `build.gradle.kts`  files.

## Coil Compose

Let’s start by declaring the Coil Compose dependency in the  `libs.versions.toml`  file. For the latest version, please check the official  [Coil](https://coil-kt.github.io/coil/)  website.



```toml
[versions]
coil = "3.0.4"

[libraries]
coil-compose = { module = "io.coil-kt.coil3:coil-compose", version.ref = "coil" }
```



## Coil Network

The above core library only allows local images to be loaded. To load remote images from the network, we also have to declare a separate  `coil-network`  dependency. Coil offers different  [network images](https://coil-kt.github.io/coil/network/)  based on OkHttp or Ktor. Since we have a KMP project, we’ll use the Ktor one.

```toml
[versions]
coil = "3.0.4"

[libraries]
coil-compose = { module = "io.coil-kt.coil3:coil-compose", version.ref = "coil" }
coil-network-ktor = { module = "io.coil-kt.coil3:coil-network-ktor3", version.ref = "coil" }
```



## Ktor Engine

Next, we must declare the  [Ktor engine](https://ktor.io/docs/client-engines.html)  dependency for each of our supported platforms: Android, iOS, and JVM.



```toml
[versions]
ktor = "3.0.1"

[libraries]
ktor-client-android = { module = "io.ktor:ktor-client-android", version.ref = "ktor" }
ktor-client-darwin = { module = "io.ktor:ktor-client-darwin", version.ref = "ktor" }
ktor-client-java = { group = "io.ktor", name = "ktor-client-java", version.ref = "ktor" }
```


## Kotlinx Coroutines Swing

Finally, since we are using Compose on JVM/Desktop, we must also add a dependency for Coroutines Swing. Coil relies on  `Dispatchers.Main.immediate`  to resolve images from the memory cache synchronously and  `kotlinx-coroutines-swing`  provides support for that on JVM (non-Android) platforms ([source](https://coil-kt.github.io/coil/compose/)).

Most likely, you already have this dependency in your project. If you do not, here is how to declare it.


```toml
[versions]
kotlinx-coroutines = "1.9.0"

[libraries]
kotlinx-coroutines-swing = { group = "org.jetbrains.kotlinx", name = "kotlinx-coroutines-swing", version.ref = "kotlinx-coroutines" }
```



# Implementing the dependencies

We’ve declared the dependencies in the version catalog, now let’s add them to the apps. Open the  `composeApp/build.gradle.kts`  and add the coil compose and coil network dependencies to the  `commonMain`  source set.



```kotlin
kotlin {
    sourceSets {
        commonMain.dependencies {
            implementation(libs.coil.compose)
            implementation(libs.coil.network.ktor)
        }
}
```



Next, we have to add the specific Ktor engine dependency for each platform. Note that  `desktopMain`  might also be  `jvmMain`  in your project, depending on how you name the JVM source set.



```kotlin
kotlin {
    sourceSets {
        androidMain.dependencies {
            // Ktor client dependency required for Coil
            implementation(libs.ktor.client.android)
        }

        appleMain.dependencies {
            // Ktor client dependency required for iOS
            implementation(libs.ktor.client.darwin)
        }

        // alternatively jvmMain
        desktopMain.dependencies {
            // Ktor client dependency required for JVM/Desktop
            implementation(libs.ktor.client.java)
            implementation(libs.kotlinx.coroutines.swing)
        }
    }
}
```

# Adding the Internet permission on Android

Before we can load remote images on Android, we have to declare the internet permission in the  `AndroidManifest`. Images will fail to load without it.


```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

<uses-permission android:name="android.permission.INTERNET" />
    <application>
       ...
    </application>
</manifest>
```



# Loading Remote Images in Compose

To load remote images in Compose, we can use the  `AsyncImage`  composable that accepts a URL to the image. It executes an asynchronous request in the background to fetch the image and display it.

> To see what else the  `AsyncImage`  supports, like placeholders and error states, check the  [official documentation](https://coil-kt.github.io/coil/compose/#asyncimage).


```kotlin
AsyncImage(
    model = "https://spacenews.com/spacex-launches-uaes-thuraya-4-mobile-connectivity-satellite/",
    contentDescription = null,
    contentScale = ContentScale.Crop,
    modifier = Modifier
        .fillMaxWidth()
)
```


If we run the app on Android, iOS, Desktop, and Web, we see that images are loaded successfully.


![Result of loading remote images in Android, iOS, Web and Desktop apps.](/assets/img/posts/compose/multiplatform/coil/coil_combined_result_small.png)
_Result of loading remote images in Android, iOS, Web and Desktop apps._



# Loading local images

Coil also supports loading local drawables. We’ve added a  `.jpg`  file to the  `commonMain/composeResources/drawable`  folder and want to load it in the app.


![Location of a local image in the project structure.](/assets/img/posts/compose/multiplatform/coil/local_image_res.png)
_Location of a local image in the project structure._




To do that, we can pass  `Res.getUri(path_to_drawable)`  to the  `model`  argument of the  `AsyncImage`  composable. We also need to add the  `@OptIn(ExperimentalResourceApi::class`  annotation since the support for accessing resources via Uri is still experimental.

> `Res.drawable.image`  and other compile-safe references are not supported by Coil until Compose Multiplatform exposes APIs to support it ([ticket](https://youtrack.jetbrains.com/issue/CMP-4360)).



```kotlin
@OptIn(ExperimentalResourceApi::class)
AsyncImage(
    model = Res.getUri("drawable/pexels-pixabay-2166.jpg"),
    contentDescription = stringResource(Res.string.locally_loaded_image_title),
)
```


Here is the result: the local image is successfully loaded on all platforms.


![Result of loading local images in Android, iOS, Web, and Desktop apps.](/assets/img/posts/compose/multiplatform/coil/coil_local_combined_small.png)
_Result of loading local images in Android, iOS, Web, and Desktop apps._



# Conclusion

We’ve looked at how to add the Coil image loading library to a Kotlin Multiplatform project that uses Compose to share UIs across Android, iOS, Desktop, and Web.

As we saw, adding just Coil is not enough. We also must add dependencies for Ktor to load remote images on all platforms.


**You can find the source code and the sample project on [GitHub](https://github.com/landomen/ComposeMultiplatformCoilSample).**


I hope you found this article useful. Let me know your thoughts and experiences with Coil and KMP in the comments.

## Resources:

-   [https://coil-kt.github.io/coil/getting_started/](https://coil-kt.github.io/coil/getting_started/)