---
title: Pull to Refresh with Compose Material 3
description: Learn how to add pull-to-refresh to your app with the latest version of the Compose Material 3 library.
date: 2024-10-05 19:00:00 +0100
categories: [Jetpack Compose, Android, Compose, Material 3]
tags: [compose, android, jetpack compose, material 3]
image: /assets/img/posts/compose/material3/pull-to-refresh/pull_to_refresh_cut.gif
---


There are many articles online on how to implement pull-to-refresh with the Compose Material 3 library. However, the pull-to-refresh APIs have completely changed since version 1.2.0, introducing breaking changes and making most blog posts outdated.

This article will show you how to use the new APIs to add pull-to-refresh functionality to your app and how to upgrade your existing app to use the newest version of the Compose Material 3 library.

> Google  [recommends](https://developer.android.com/develop/ui/compose/designsystems/material)  using Material 3 with Compose, even if most of the APIs are experimental and can behave differently from Material 2.


# What is pull-to-refresh?

Pull-to-refresh or swipe-to-refresh is a common gesture-based feature in mobile apps that allows users to manually refresh the content of a page by swiping or pulling down on the screen.

It’s typically used on screens that display a list of data loaded from remote services that can change frequently. You can see examples of this in Facebook, Instagram, Twitter, Reddit, and other social and news apps.


![Example of using pull-to-refresh in an app.](/assets/img/posts/compose/material3/pull-to-refresh/pull_to_refresh_cut.gif)
_Example of using pull-to-refresh in an app._



# Starter code

We’ll be adding pull-to-refresh functionality to a simple  `HomeScreen`  that shows a scrollable list of random dog facts. The screen state is provided through a  `ViewModel`  which is responsible for loading the data from a repository and exposing it through a  `StateFlow`. A standard MVVM architecture in most of today’s apps.


<script src="https://gist.github.com/landomen/b10937b58a15f5d02456c4b30e9c755e.js"></script>


<script src="https://gist.github.com/landomen/b390942342d7db2f68df18afe482c6ba.js"></script>



We collect the state from the view model in the view (Compose) and then render a scrollable list of facts.

<script src="https://gist.github.com/landomen/5320e6e3bf11a26c8b37fc256971849f.js"></script>


All of the above results in this screen showing a list of facts. To see new facts, users have to close and re-open the app, resulting in a sub-optimal UX. Therefore, we will add a pull-to-refresh gesture to this screen to let users refresh the data.


![Initial screen showing a list of facts.](/assets/img/posts/compose/material3/pull-to-refresh/pull_to_refresh_initial.png)
_Initial screen showing a list of facts._


# Implementing pull-to-refresh with Compose Material 3

Compose Material 3 library offers an out-of-the-box solution to add pull-to-refresh to your app. It contains the  `PullToRefreshBox`  container and  `.pullToRefresh`  modifier.

## Adding the latest dependency

To get started, make sure you have added the latest version of the Compose Material 3 dependency to your app-level build file:


<script src="https://gist.github.com/landomen/04c58b691558dd411d50c2e7b1a4360d.js"></script>


or if you’re using version catalog dependency management:

<script src="https://gist.github.com/landomen/cd006ef5bf9ff495ea6ba76b3d6b998b.js"></script>


<script src="https://gist.github.com/landomen/51c2473ebe88794deae8c4cde9100986.js"></script>


## Using the PullToRefreshBox

The easiest way to add pull-to-refresh is to use the  `PullToRefreshBox`  container. It requires a scrollable layout as the content and enables gesture support, allowing the user to manually refresh by swiping downward from the top of the content. It also provides a default implementation of the refreshing indicator.

<script src="https://gist.github.com/landomen/8a4625b81d89d045c1c9de04cc125dc3.js"></script>


We have to provide the following:

-   `isRefreshing`  — true/false whether a refresh is happening, needed for the circle animation to happen
-   `onRefresh`  — callback that is triggered when the user’s gesture exceeds the threshold, initiating a refresh request
-   `content`  — vertically scrollable content/layout, such as a  `LazyColumn`, on top of which the refresh indicator will be displayed when triggered


## Adding pull-to-refresh to our example

First, we have to update our screen state object to hold a new property  `isRefreshing`  that we can pass on to the  `PullToRefreshBox`.


<script src="https://gist.github.com/landomen/7a94a0d7051180ce814fbfd3da4e7383.js"></script>


Next, we have to update our  `ViewModel`  to update the  `isRefreshing`  state and to trigger data refresh. We’ve added a new  `_isRefreshing: MutableStateFlow<Boolean>`  to control the refreshing state. Anytime this flow emits, it updates the screen state.

We’ve also added a  `onPullToRefreshTrigger()`  function that’s called from Compose when pull-to-refresh is triggered. It controls the refreshing state and re-fetches the data.


<script src="https://gist.github.com/landomen/1ecbdf4957970a318c77688684f0057b.js"></script>



Finally, we have to update our screen composable to add pull-to-refresh. To do that, we wrap our existing `LazyColumn` with `PullToRefreshBox` and provide it with the `isRefreshing` state and a `onRefresh` callback, which calls the function in the `ViewModel`.

<script src="https://gist.github.com/landomen/61c56a4d358f1280f43af0e7eb2a0786.js"></script>

When we pull down on the screen, the indicator will appear and we receive a  `onRefresh()`  callback. We now have to trigger the refresh of the data through the  `ViewModel`  and set the  `isRefreshing`  to true. If we do not do that, the refresh indicator will be stuck and won’t animate.

It’s also important to set  `isRefreshing`  back to false once the data is refreshed to hide the refreshing indicator.

And that’s it! We’ve added a pull-to-refresh gesture to the app.


![Working pull-to-refresh on the home screen](/assets/img/posts/compose/material3/pull-to-refresh/pull_to_refresh_working.gif)
_Working pull-to-refresh on the home screen_


## Customizing the indicator animation

We can further customize the behavior of the pull-to-refresh component by extending the  `PullToRefreshState`. This enables us to change the animation of the refresh indicator. Here is an example of how to add a spring/bounce animation to the indicator after it’s released.

<script src="https://gist.github.com/landomen/cce1d94470cb1f05216c1f09c4abb064.js"></script>

And this is the final result:

![Pull-to-refresh behavior with a custom spring animation for the refresh indicator.](/assets/img/posts/compose/material3/pull-to-refresh/pull_to_refresh_spring.gif)
_Pull-to-refresh behavior with a custom spring animation for the refresh indicator._


## Further customizations

`PullToRefreshBox`  offers an easy-to-use API but doesn’t provide many customization options. For example, it’s missing a  `enabled`  property with which we could enable or disable the pull-to-refresh gesture.

If we need further control, we can use the  `.pullToRefresh`  modifier directly, which is used by  `PullToRefreshBox`  under the hood. It exposes the  `enabled: Boolean`  property and also allows us to control the threshold of when the refresh is triggered via the  `treshold: Dp`property.

<script src="https://gist.github.com/landomen/eed4943717e25976d34eca150f81e1a2.js"></script>


We can apply the modifier on any layout that wraps a vertically scrollable layout. Or we can create a wrapper composable and use it in the same way as `PullToRefreshBox`.

<script src="https://gist.github.com/landomen/902bca1e7a5cf87504e63c5282b9172e.js"></script>

# Migrating from previous APIs

If you already use an older version of Compose Material 3 in your app and have implemented the pull-to-refresh behavior with  `PullToRefreshContainer`, you will need to migrate to  `PullToRefreshBox.`  The previous APIs have been deprecated in version 1.3.0 and are no longer included in the library, making the upgrade a breaking change.

We’ve shown how to use the new APIs in the first part of the article. Deciding whether to migrate to  `PullToRefreshBox`  or  `pullToRefresh`  modifier will depend on your needs. If you require the option to enable/disable the pull-to-refresh gesture, then you will have to use the  `pullToRefresh`  modifier. If not, you can simply use  `PullToRefreshBox`.

Key changes:

-   `rememberPullToRefreshState()`: no longer accepts an  `enabled`  argument, it can be controlled through the  `pullToRefresh`  modifier
-   `PullToRefreshContainer`: replaced by  `PullToRefreshBox`  or  `pullToRefresh`  modifier
-   `isRefreshing`  state is controlled by the user instead of  `PullToRefreshState`

<script src="https://gist.github.com/landomen/d43f9f6ae0e2c181ae2d3b9603f9aa88.js"></script>


# Conclusion

We’ve taken a look at how to implement a simple pull-to-refresh mechanism in your app using the latest Compose Material 3 library.

The APIs are experimental and can be used in two ways: by applying a wrapper layout or by applying a modifier. Which one you select depends on your requirements, such as custom animations or control over enabling and disabling the gesture.

You can find the full implementation example of pull-to-refresh in my sample project on  [GitHub](https://github.com/landomen/compose-material3-pull-to-refresh-sample/tree/main).


## Resources:

-   [https://developer.android.com/develop/ui/compose/designsystems/material](https://developer.android.com/develop/ui/compose/designsystems/material)
-   [https://developer.android.com/jetpack/androidx/releases/compose-material3#compose_material3_version_14_2](https://developer.android.com/jetpack/androidx/releases/compose-material3#compose_material3_version_14_2)
-   [https://composables.com/material3/pulltorefreshbox](https://composables.com/material3/pulltorefreshbox)