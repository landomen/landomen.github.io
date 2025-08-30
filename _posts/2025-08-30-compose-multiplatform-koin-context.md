---
title: "Injecting Android Context in Compose Multiplatform with Koin"
description: "How to inject and use Android Context in a Compose Multiplatform app with Koin."
date: 2025-08-30 20:00:00 +0200
categories: [Compose Multiplatform, Kotlin Multiplatform, Compose, Android, iOS, Desktop, Koin]
tags: [compose multiplatform, compose multiplatform, kmp, compose android, koin, jetpack compose, ios, desktop]
image: /assets/img/posts/compose/multiplatform/koin/context/cover.png
---


While developing the  [Space Flight News app](https://medium.com/proandroiddev/building-a-space-flight-news-app-with-compose-multiplatform-for-android-ios-and-desktop-part-3-1efe6545f885), I needed to inject the Android Context into Koin to set up SQLDelight and to trigger a native share intent.

There were quite a few examples of initializing Koin for every platform independently, using the  `initKoin`  function. However, I was using the  `KoinApplication`  composable function wrapper to initiate Koin from within the shared  `App`  composable function and found a lack of examples on how to do this.

This is a short article sharing how to set up your Compose Multiplatform app so that you can access the Android Context in your Koin modules when using the  `KoinApplication`  composable function wrapper.


> **Note:** this article assumes the reader is familiar with Koin, Kotlin Multiplatform, and Compose Multiplatform. If you want to learn more, consider first reading  [Part 2 of Building a Space News App with Compose Multiplatform for Android, iOS, and Desktop.](https://proandroiddev.com/building-a-space-flight-news-app-with-compose-multiplatform-for-android-ios-and-desktop-part-2-d8541e15aaa6)

## ❌ Wrong approach

One solution I saw recommended frequently was to create an empty  `Application`  class in the  `androidMain`  source set and store the context inside a static variable from  `onCreate()`  and access it within your Koin modules.

```kotlin
// composeApp/androidMain/MyApp.kt  
class MyApp: Application() {  
  
    override fun onCreate() {  
        super.onCreate()  
        androidContext = this  
    }  
}  
  
@SuppressLint("StaticFieldLeak")  
lateinit var androidContext: Context  
  
// composeApp/androidMain/AppModule.android.kt  
actual val platformModule: Module  
    get() = module {  
        // provide Context to Koin  
        single<Context> { androidContext }  
        // retrieve Context using get()  
        single<ShareService> { AndroidShareService(context = get()) }  
    }
```

While this might work, it’s not a scalable solution and can lead to bugs, crashes, and memory leaks.

## ✅ Option 1: Providing Context through optional KoinAppDeclaration

The option that I ended up using in my  [Space Flight News app](https://github.com/landomen/KMPSpaceFlightNews/blob/main/composeApp/src/commonMain/kotlin/com/landomen/spaceflightnews/App.kt#L23)  is to add an optional  `KoinAppDeclaration`  argument to the main  `App`  composable function. This enables passing custom config to the Koin initialization that’s platform-specific. It’s only used by Android, while iOS and Desktop are not using it.

```kotlin
// composeApp/commonMain/App.kt  
@Composable  
fun App(koinAppDeclaration: KoinAppDeclaration? = null) {  
    KoinApplication(application = {  
        koinAppDeclaration?.invoke(this)  
        modules(appModule, platformModule)  
    }) {  
        AppTheme {  
            MainNavigationHost()  
        }  
    }  
}
```

Then in the  `MainActivity`  in  `androidMain`, we’re building the  `KoinAppDeclaration`  instance and using the  `androidContext`  function from Koin to declare the specific instance of  `Context`  that Koin should use whenever we want to inject it.

```kotlin
// composeApp/androidMain/MainActivity.kt  
class MainActivity : ComponentActivity() {  
    override fun onCreate(savedInstanceState: Bundle?) {  
        super.onCreate(savedInstanceState)  
  
        setContent {  
            // provide KoinAppDeclaration  
            App({  
                androidContext(this@MainActivity.applicationContext)  
            })  
        }  
    }  
}
```

As  `Context`  is an Android-only concept; we can only access it within the  `androidMain`  source set, meaning that we must have platform-specific modules.

```kotlin
// composeApp/commonMain/AppModule.kt  
val appModule = module {  
    single<ApiService> { ApiService() }  
    single<ArticlesRepository> {  
        ArticlesRepository(  
            databaseDriverFactory = get(),  
            api = get()  
        )  
    }  
  
    viewModel { ArticleListViewModel(repository = get()) }  
}  
  
expect val platformModule: Module

```

Within the actual implementation of the  `platformModule`  in  `androidMain`  source set we can retrieve the  `Context`  using the standard  `get()`  function.

```kotlin
// composeApp/androidMain/AppModule.android.kt  
actual val platformModule: Module  
    get() = module {  
        single<DatabaseDriverFactory> { AndroidDatabaseDriverFactory(context = get()) }  
        single<ShareService> { AndroidShareService(context = get()) }  
    }
```

## ✅ Option 2: Using experimental KoinMultiplatformApplication in Koin 4.1.0

Release 4.1.0 of Koin added a new  `KoinMultiplatformApplication`  composable function that is the same as  `KoinApplication`, but automatically injects the Android  `Context`.

> Note: the  [API is experimental](https://insert-koin.io/docs/reference/koin-compose/compose/#starting-koin-with-a-compose-app---koinapplication)  and might change in future releases.

```kotlin
@OptIn(KoinExperimentalAPI::class)  
@Composable  
fun App() {  
    KoinMultiplatformApplication(config = koinConfiguration {  
        modules(appModule, platformModule)  
    }) {  
        AppTheme {  
            MainNavigationHost()  
        }  
    }  
}
```

## Conclusion

We looked at a couple of options on how to inject and use Android  `Context`  within a Compose Multiplatform app when using Koin and  `KoinApplication`  composable function.

Hopefully, you found this helpful. Share your approach in the comments.

<hr style="width:50%; margin-left:25% !important; margin-right:25% !important; height:4px;">

**Resources:**

-   [https://insert-koin.io/docs/reference/koin-compose/compose/](https://insert-koin.io/docs/reference/koin-compose/compose/)  — official documentation
-   [https://carrion.dev/en/posts/koin-cmp/](https://carrion.dev/en/posts/koin-cmp/)  — helpful article from Ignacio Carrión
-   [https://stackoverflow.com/questions/78257096/injecting-context-from-android-into-sqldelight-driver-using-koin-within-kmp](https://stackoverflow.com/questions/78257096/injecting-context-from-android-into-sqldelight-driver-using-koin-within-kmp)  — some of the alternative recommendations