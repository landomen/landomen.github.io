---
title: "Building a Space Flight News App with Compose Multiplatform for Android, iOS, and Desktop: Part 5 - Native Interactions"
description: "The fifth part of building a Compose Multiplatform app: native interactions."
date: 2025-07-22 18:35:00 +0200
categories: [Compose Multiplatform, Kotlin Multiplatform, Compose, Android, iOS, Desktop]
tags: [compose multiplatform, compose multiplatform, kmp, compose android, jetpack compose, ios, desktop]
image: /assets/img/posts/compose/multiplatform/space-flight/part-2/mid_combined.png
---




This is the fifth and final part of a series of articles focusing on Compose Multiplatform. We are building an app for Android, iOS, and Desktop that displays the latest Space Flight news.

This part will focus on the following:

-   adding a system share functionality to share links to articles with different apps.
-   opening the article URL in an external web browser.


![Showcase of the final app.](/assets/img/posts/compose/multiplatform/space-flight/part-1/final_app_gif.gif)
_Showcase of the final app._

## Recap of the first four parts

This article continues where the fourth part left off, so make sure to start there if you haven’t followed the series:
[Building a Space Flight News App with Compose Multiplatform for Android, iOS, and Desktop: Part 4](https://landomen.github.io/posts/compose-multiplatform-space-part-4/)


So far, we’ve learned the following:

-   how Kotlin Multiplatform works
-   sharing UI using Compose Multiplatform on iOS, Android, and Desktop
-   loading remote images with Coil
-   adding network layer for fetching remote data with Ktor
-   adding dependency injection with Koin
-   handling loading and error states
-   adding a local database and offline support with SQLDelight
-   adding an article details screen
-   and navigating to it using Compose Navigation.

You can find the code after part 4, and what will be our starting point for this article, on the `part-4` branch of the project repository:
[https://github.com/landomen/KMPSpaceFlightNews/tree/part-4](https://github.com/landomen/KMPSpaceFlightNews/tree/part-4)

# 1. Open the Article in a Web browser


We have a “Read more” button at the bottom of the article details screen. Let’s implement the logic behind it to open the article web page in the default external web browser. This will allow our users to read the article in its entirety.

Open  `ArticleDetailsScreen`  and get an instance of the  `UriHandler`  in the root composable function. This class handles the incoming URI/URL so that it’s correctly opened on all platforms.

Next, let’s implement the  `onReadMoreClick`  callback where we currently have a TODO. We pass the article URL to the  `openUri(String)`  function of the  `UriHandler`.

```kotlin
@Composable  
internal fun ArticleDetailsScreen(  
    articleId: Long,  
    onBackClick: () -> Unit,  
) {  
    val viewModel = koinViewModel<ArticleDetailsViewModel>()  
    val state by viewModel.state.collectAsStateWithLifecycle()  
    // NEW: instance of the UriHandler  
    val uriHandler = LocalUriHandler.current  
  
    LaunchedEffect(Unit) {  
        viewModel.onFetch(articleId)  
    }  
  
    ArticleDetailsScreenContent(  
        state = state,  
        onRetryClick = {  
            viewModel.onFetch(articleId)  
        },  
        onBackClick = onBackClick,  
        onShareClick = { article ->  
            // TODO Open share sheet  
        },  
        onReadMoreClick = { articleUrl ->  
            // NEW: handle URL  
            uriHandler.openUri(articleUrl)  
        }  
    )  
}
```

That’s it! Run the app and check the behavior on Android, iOS, and Desktop. On all three platforms, the article web page is opened in the default external browser, which could be Chrome/Safari or another browser.



![Opening the article web page in an external browser on Android.](/assets/img/posts/compose/multiplatform/space-flight/part-5/android_url.gif)
_Opening the article web page in an external browser on Android._


![Opening the article web page in an external browser on iOS.](/assets/img/posts/compose/multiplatform/space-flight/part-5/ios_url.gif)
_Opening the article web page in an external browser on iOS._

![Opening the article web page in an external browser on Android.](/assets/img/posts/compose/multiplatform/space-flight/part-5/desktop_url.gif)
_Opening the article web page in an external browser on desktop._



# 2. Create a share service

The final feature that we will implement is the ability to share a link to the article through an external app. On Android and iOS, this means opening the system share sheet, where the user can select which app to share it on. On Desktop, this means copying the link to the clipboard, giving the user the option to paste it to their desired app or website.

## Trigger share action in the ViewModel

Let’s start by updating our  `ArticleDetailsScreen`  to handle the click action on the share button and call the  `ArticleDetailsViewModel`.

Open  `ArticleDetailsScreen`  and go to the  `ArticleDetailsSuccessContent`  function. It currently accepts a  `onShareClick: () -> Unit`  argument that we need to extend to get the  `Article`  object. We then need to update the  `IconButton`  to call  `onShareClick(article)`  and pass in the  `article`  object.

```kotlin
@Composable  
private fun ArticleDetailsSuccessContent(  
    article: Article,  
    onBackClick: () -> Unit,  
    // NEW: Added Article argument  
    onShareClick: (Article) -> Unit,  
    onReadMoreClick: (String) -> Unit,  
) {  
    Column(  
        ...  
    ) {  
        Box(...) {  
            AsyncImage(...)  
  
            IconButton(...)  
  
            // NEW: Updated the onClick function  
            IconButton(  
                onClick = { onShareClick(article) },  
                modifier = Modifier.align(Alignment.TopEnd)  
            ) {  
                Icon(  
                    Icons.Default.Share,  
                    contentDescription = stringResource(Res.string.share_content_description),  
                    tint = Color.White  
                )  
            }  
        }  
  
        Column(  
            ...  
        ) {  
            ...  
        }  
    }  
}
```

Next, move to the  `ArticleDetailsScreenContent`  function and update the existing  `onShareClick: () -> Unit`  argument to accept an  `Article`  object.

```kotlin
@Composable  
private fun ArticleDetailsScreenContent(  
    state: ArticleDetailsViewModel.ArticleDetailsViewState,  
    onRetryClick: () -> Unit,  
    onBackClick: () -> Unit,  
    // NEW: Updated to accept an Article  
    onShareClick: (Article) -> Unit,  
    onReadMoreClick: (String) -> Unit,  
) {  
    ...  
}
```

Then, go to the top-level  `ArticleDetailsScreen`  function and update the implementation of the  `onShareClick`  function to call the new function on the view model.

```kotlin
@Composable  
internal fun ArticleDetailsScreen(  
    articleId: Long,  
    onBackClick: () -> Unit,  
) {  
    val viewModel = koinViewModel<ArticleDetailsViewModel>()  
    val state by viewModel.state.collectAsStateWithLifecycle()  
    val uriHandler = LocalUriHandler.current  
  
    LaunchedEffect(Unit) {  
        viewModel.onFetch(articleId)  
    }  
  
    ArticleDetailsScreenContent(  
        state = state,  
        onRetryClick = {  
            viewModel.onFetch(articleId)  
        },  
        onBackClick = onBackClick,  
        // NEW: Updated to call the view model  
        onShareClick = { article ->  
            viewModel.onShareClick(article)  
        },  
        onReadMoreClick = { articleUrl ->  
            uriHandler.openUri(articleUrl)  
        }  
    )  
}
```

Finally, let’s create a new function in the  `ArticleDetailsViewModel`  that will be responsible for triggering the share functionality.

```kotlin
fun onShareClick(article: Article) {  
    // TODO Trigger the share functionality  
}
```

We will return to this function later, once we have implemented the sharing services.

## Define the ShareService interface

We’ll start by defining the interface that we will implement on every device. Create a new interface  `ShareService`  inside the  `composeApp/commonMain/your.package.spaceflightnews/share`  directory. It should have a single function  `share()`  that accepts the title of the article to share and its URL.

```kotlin
interface ShareService {  
    fun share(title: String, url: String)  
}
```

## Implement sharing on Android

Create a new class  `AndroidShareService`  inside the  `composeApp/androidMain/your.package.spaceflightnews/share`  directory. It should extend  `ShareService`  and implement the  `share(title, url)`  function.

It needs to have a constructor that accepts a  `Context`  to be able to create a chooser intent that will trigger the system share sheet.

The rest of the code is standard Android. We’re creating an intent and setting the payload, like the title and the text to be shared. Finally, we’re launching the intent, which opens the sheet.

```kotlin
import android.content.Context  
import android.content.Intent  
import com.landomen.spaceflightnews.R  
  
class AndroidShareService(private val context: Context) : ShareService {  
  
    override fun share(title: String, url: String) {  
        val shareIntent = Intent(Intent.ACTION_SEND).apply {  
            type = "text/plain"  
            putExtra(Intent.EXTRA_SUBJECT, title)  
            putExtra(Intent.EXTRA_TEXT, constructMessage(title = title, url = url))  
        }  
  
        context.startActivity(  
            Intent.createChooser(  
                shareIntent,  
                context.getString(R.string.share_title)  
            ).apply {  
                addFlags(Intent.FLAG_ACTIVITY_NEW_TASK)  
            })  
    }  
  
    private fun constructMessage(title: String, url: String): String {  
        return context.getString(R.string.share_message, title, url)  
    }  
}
```

You’ll notice we’re referencing two new strings in the code above. Open  `composeApp/src/androidMain/res/values/strings.xml`  and add the following two strings:

```xml
<string name="share_title">Share article</string>  
<string name="share_message">%1$s\n\nRead more: %2$s</string>
```

Now that we have our Android share service, we need to provide it through Koin. Open  `AppModule.android.kt`  and add the  `AndroidShareService`  to the graph. We’ll inject it later into the  `ArticleDetailsViewModel.`

```kotlin
actual val platformModule: Module  
    get() = module {  
        single<DatabaseDriverFactory> { AndroidDatabaseDriverFactory(context = get()) }  
        single<ShareService> { AndroidShareService(context = get()) }  
    }
```

## Implement sharing on iOS

Create a new class  `IOSShareService`  inside the  `composeApp/iosMain/your.package.spaceflightnews/share`  directory. It should extend  `ShareService`  and implement the  `share(title, url)`  function.

We then need to use a  `UIActivityViewController`  which is a standard way to offer content from your app to other apps. Presenting this  `ViewController`  displays the share sheet.

```kotlin
import kotlinx.cinterop.BetaInteropApi  
import platform.Foundation.NSString  
import platform.Foundation.create  
import platform.UIKit.UIActivityViewController  
import platform.UIKit.UIApplication  
  
class IOSShareService: ShareService {  
    @OptIn(BetaInteropApi::class)  
    override fun share(title: String, url: String) {  
        val activityItems = listOf(  
            NSString.create(string = "$title\n\nRead more: $url")  
        )  
        val activityViewController =  
            UIActivityViewController(activityItems = activityItems, applicationActivities = null)  
  
        // Get the top-most view controller to present the activity view controller  
        val rootViewController = UIApplication.sharedApplication.keyWindow?.rootViewController  
        rootViewController?.presentViewController(activityViewController, animated = true, completion = null)  
    }  
}
```

To add the iOS share service to the dependency injection, open  `AppModule.ios.kt`  and provide it. We’ll inject it later into the  `ArticleDetailsViewModel.`

```kotlin
actual val platformModule: Module  
    get() = module {  
        single<DatabaseDriverFactory> { IOSDatabaseDriverFactory() }  
        single<ShareService> { IOSShareService() }  
    }
```

## Implement sharing on Desktop

Create a new class  `DesktopShareService`  inside the  `composeApp/desktopMain/your.package.spaceflightnews/share`  directory. It should extend  `ShareService`  and implement the  `share(title, url)`  function.

Implementing share functionality on the Desktop is harder because there is no built-in mechanism like on mobile. Instead, we’ll copy the text to be shared to the clipboard and let users share it manually to their desired app or website.

Then, to notify the users that the content was copied to their clipboard, we’re building and showing a simple toast message.

```kotlin
import java.awt.Color  
import java.awt.Dimension  
import java.awt.Font  
import java.awt.Toolkit  
import java.awt.datatransfer.StringSelection  
import javax.swing.JLabel  
import javax.swing.JWindow  
import javax.swing.SwingConstants  
import javax.swing.Timer  
  
class DesktopShareService : ShareService {  
  
    override fun share(title: String, url: String) {  
        val clipboard = Toolkit.getDefaultToolkit().systemClipboard  
        val selection = StringSelection("$title\n\nRead more: $url")  
        clipboard.setContents(selection, selection)  
  
        showToastMessage("Copied to clipboard")  
    }  
  
    fun showToastMessage(message: String, durationMillis: Int = 5000) {  
        val window = JWindow()  
  
        val label = JLabel(message, SwingConstants.CENTER).apply {  
            foreground = Color.WHITE  
            background = Color(0, 0, 0, 200)  
            isOpaque = true  
            font = Font("SansSerif", Font.PLAIN, 14)  
            preferredSize = Dimension(300, 40)  
        }  
  
        window.contentPane.add(label)  
        window.pack()  
  
        // Position at bottom center of screen  
        val screenSize = Toolkit.getDefaultToolkit().screenSize  
        val x = (screenSize.width - window.width) / 2  
        val y = screenSize.height - window.height - 100  
        window.setLocation(x, y)  
  
        window.isAlwaysOnTop = true  
        window.isVisible = true  
  
        // Auto-hide after timeout  
        Timer(durationMillis) {  
            window.isVisible = false  
            window.dispose()  
        }.start()  
    }  
}
```

Finally, we need to add the desktop share service to Koin in the  `AppModule.desktop.kt`  to later inject it into the  `ArticleDetailsViewModel.`

```kotlin
actual val platformModule: Module  
    get() = module {  
        single<DatabaseDriverFactory> { DesktopDatabaseDriverFactory() }  
        single<ShareService> { DesktopShareService() }  
    }
```

#### Using the ShareService

Now that we have all three implementations of the  `ShareService`, we can inject it into the  `ArticleDetailsViewModel`  and trigger the share functionality.

```kotlin
internal class ArticleDetailsViewModel(  
    private val repository: ArticlesRepository,  
    // NEW: Added ShareService  
    private val shareService: ShareService,  
) : ViewModel() {  
  
    fun onShareClick(article: Article) {  
        shareService.share(  
            title = article.title,  
            url = article.url  
        )  
    }  
}
```

As the final step, open  `AppModule.kt`  and pass the  `ShareService`  to the  `ArticleDetailsViewModel`.

```kotlin
val appModule = module {  
    ...  
    viewModel { ArticleDetailsViewModel(repository = get(), shareService = get()) }  
}
```

That’s it! We’re now ready to test our sharing functionality.

## Testing sharing

Opening an article and then clicking the “Share” button in the top right corner of the image will trigger the share functionality. Here is how it looks on all three platforms.

**Android**<br>
The native share sheet is opened, where users can either copy the text or share it to any app that supports it. Since we’re sharing a simple text, the majority of apps should be able to handle it.

![Native share sheet on Android.](/assets/img/posts/compose/multiplatform/space-flight/part-5/android_share_small.png)
_Native share sheet on Android._

**iOS**<br>
The native share sheet is opened, where users can either copy the text or share it to any app that supports it. Since we’re sharing a simple text, the majority of apps should be able to handle it.


![Native share sheet on iOS.](/assets/img/posts/compose/multiplatform/space-flight/part-5/ios_share_small.png)
_Native share sheet on iOS._


**Desktop**<br>
On desktop we simply copy the text to the clipboard and show a toast to the user.

![Share toast on desktop.](/assets/img/posts/compose/multiplatform/space-flight/part-5/desktop_share_small.png)
_Share toast on desktop._


# 3. Conclusion and thank you!
If you followed to the end, great job! This is the end of the fifth and final part of the series on Kotlin Multiplatform and Compose Multiplatform.

It’s been a long journey, and you should be proud of yourself for building a fully functioning multiplatform app that runs on Android, iOS, and Desktop.

Throughout the series, we’ve learned how to:

-   create a Kotlin Multiplatform and Compose Multiplatform project,
-   build UI with Compose and Material3,
-   share view models across platforms,
-   add dependency injection using Koin,
-   integrate a network layer using Ktor,
-   integrate a database layer using SQLDelight to support offline mode,
-   display remote images using Coil,
-   navigate between screens using Navigation Compose,
-   open a URL in an external web browser,
-   and open the system share sheet.

Here is the final product on all three platforms:


![Completed app running on Android, showing all the features.](/assets/img/posts/compose/multiplatform/space-flight/part-5/android_final.gif)
_Completed app running on Android, showing all the features._


![Completed app running on iOS, showing all the features.](/assets/img/posts/compose/multiplatform/space-flight/part-5/ios_final.gif)
_Completed app running on iOS, showing all the features._


![Completed app running on Desktop, showing all the features.](/assets/img/posts/compose/multiplatform/space-flight/part-5/desktop_final.gif)
_Completed app running on Desktop, showing all the features._


<hr style="width:50%; margin-left:25% !important; margin-right:25% !important; height:4px;">

You can find the source code for this part here:
[https://github.com/landomen/KMPSpaceFlightNews/tree/part-5](https://github.com/landomen/KMPSpaceFlightNews/tree/part-5)

<hr style="width:50%; margin-left:25% !important; margin-right:25% !important; height:4px;">


## Resources

-   [https://kmp.jetbrains.com/](https://kmp.jetbrains.com/)
-   [https://www.jetbrains.com/compose-multiplatform/](https://www.jetbrains.com/compose-multiplatform/)
-   [https://spaceflightnewsapi.net/](https://spaceflightnewsapi.net/)