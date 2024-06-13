---
title: Getting Started with Feature Flags on Mobile
description: We’ll take a closer look at what feature flags are, how they can help us, and how to start using them in your project.
date: 2023-07-11 11:00:00 +0100
categories: [Android, iOS, Mobile, Feature Flags, Experimentation, Firebase, Remote Config]
tags: [android, ios, mobile, feature flags, experimentation, firebase, remote config, firebase remote config]
image: /assets/img/posts/feature-flags/firebase_newParameter.png
---



We’ll take a closer look at what feature flags are, how they can help us, and how to start using them in your project by leveraging the Firebase Remote Config cloud service.

# What is a feature flag?

A feature flag is a simple boolean variable with which we can remotely enable or disable some functionality in the app without having to rebuild and re-release the app. This allows us to develop features faster and to have extra protection when something in the new feature goes wrong.

## Example

Let’s say we want to introduce a share feature in our app. In the code, we would check if the feature is enabled and then show a share button in the  `TopAppBar`.


<script src="https://gist.github.com/landomen/ddaf6712300928115f79f065605ef870.js"></script>



The feature flag is named  `shareFunctionEnabled`  and  `FeatureChecker`  is a class with logic to check if a feature is enabled. We will implement this class later using Firebase Remote Config.

If the feature flag is disabled, we will not display the share button and users will not be able to enter the sharing flow.


![State of the TopAppBar with the feature flag disabled.](/assets/img/posts/feature-flags/appbar_initial.png)
_State of the TopAppBar with the feature flag disabled._


But if the feature flag is enabled, the share button will be displayed and users will be able to enter the sharing flow.


![State of the TopAppBar with the feature flag enabled.](/assets/img/posts/feature-flags/appbar_enabled.png)
_State of the TopAppBar with the feature flag enabled._



This is a very simple pattern, but as we’ll learn it can be very useful, especially when things go wrong.

# Benefits and drawbacks of feature flags

Having a feature behind a feature flag means that if a critical bug or crash is found after releasing the app to the users, we can easily disable the feature as soon as we notice the issue. We can then fix the bug or crash at our own pace, as users are no longer affected.

Without feature flags, we would have to fix the bug, prepare a new version, go through the review process, and finally release it to the users, all as soon as possible to fix the broken experience. This can be stressful for everyone involved in the release process: developers, quality assurance, product owners, release managers, and others. Not to mention the business risk in terms of losing customers, revenue, and prestige if a critical issue is released to customers.

Additionally, releasing a new version doesn’t guarantee that users will immediately get the fix. Some users might have automatic updates disabled and will still use the broken version for a while.

## Benefits

Here are some of the positives of using feature flags in projects:

1.  As we already mentioned, having a feature behind a feature flag means that when something goes wrong, we can easily disable the feature with minimal effort.
2.  Allows us to only roll out the feature to a small subset of users, instead of every user. That way we can gather feedback and observe issues before releasing the feature to other users.
3.  Some teams use feature branches for developing new features that will take a long time to complete. That brings the extra cost of keeping that branch up-to-date and dealing with merge conflicts. By hiding the feature behind a feature flag, teams can avoid feature branches and merge parts of the feature into the main branch, as the feature is not accessible to users. This brings the extra benefit of discovering and addressing issues early on, instead of waiting for the whole feature to be ready.
4.  Allows release of one-time features that should only be available to users for a specific time, like some marketing campaign. The feature can be built and released ahead of time, and when we want to market the feature to users, we can simply enable the feature flag, and disable it when the campaign is over, without the need to release another version. Since it takes a while for users to update their apps, this allows us to get the adoption we want, before enabling the campaign.
5.  Feature flags allow us to run A/B experiments to validate new features. We can divide the user base into two halves, where one half has the new feature enabled, and the other half has it disabled. We then gather analytics about the conversion rate, engagement, growth, and other metrics for a specific amount of time. Based on the results we can select whether to ship the feature or not.

## Drawbacks

The usage of feature flags also brings some negatives, not limited to:

1.  Having too many feature flags can add complexity to the code, as there are more code paths to read and maintain. It’s especially hard when feature flags are dependent on other feature flags, as it’s hard to keep track of the possible combinations. It’s therefore important to be pragmatic with the usage of feature flags.
2.  Having more code paths also means that testing becomes more difficult, as we have to include the feature flag component in our tests.
3.  Once we have confirmed that the new feature is stable, we should remove the feature flag and if applicable, also any old code. Doing so allows us to avoid tech debt and keep the codebase clean.

## Best practices

Here are some best practices regarding feature flags that teams should use:

1.  Put new features under feature flags and ship early.
2.  It’s important to have a good naming convention and to detail how and why this feature flag is needed and who owns it.
3.  Review feature flags regularly to figure out which ones are stale and no longer needed and perform code cleanup.
4.  Avoid having feature flags that are dependent on other feature flags as that increases the code complexity and it’s hard to keep track of.

# Getting started with Firebase Remote Config

There are multiple ways to implement feature flags that can be controlled remotely:

1.  Implement your own feature management service that runs on your servers, that will provide clients with feature flag values. The big drawback here is the engineering effort required to implement this from scratch and maintain it. Additionally, it might not be clear what is needed and required when starting to use feature flags for the first time.
2.  Use open-source libraries that offer feature management and deploy them on your own servers. The benefit of this approach over the first one is that there is no development effort required, but there is still the issue of maintenance.
3.  Using a third-party feature management service that offers everything out-of-the-box. These services usually offer advanced features for more complex use cases, but it’s also the easiest way to get started.

We are going to use the  [Firebase Remote Config](https://firebase.google.com/docs/remote-config)  cloud service, which is easy to set up and works on all major clients (Android, Flutter, iOS, Web, and Backend).

**The guide will focus on Android, but the logic should be similar on the rest of the platforms.**

> This guide assumes that you have already set up a Firebase and have it working on your mobile project. If not, please see follow the  [Firebase setup instructions](https://firebase.google.com/docs/android/setup)  first.

## 1. Add dependencies

Add Firebase Remote Config dependency to your app module  `build.gradle.kts`  file:


<script src="https://gist.github.com/landomen/1baab790663e53eeedf4889d73cc3383.js"></script>




## 2. Create a new feature flag in the Firebase console

To use the new feature flag in our code, we first have to create the corresponding parameter in the Firebase console. Select the “Create configuration” or “Add parameter” button and a new popup will appear. We have to enter the name of the parameter/feature flag (`shareFeatureEnabled`), select the data type (Boolean), write a meaningful description, and set the default value.


![Creating a new parameter/feature flag.](/assets/img/posts/feature-flags/firebase_newParameter.png)
_Creating a new parameter/feature flag._



Click “Save” and then “Publish changes” to make them available. The next step is to read the value of the feature flag in our app.

## 3. Initialize remote config

To read the value of the feature flag we set up, we have to first initialize and fetch the remote config. We’ll create a helper class called  `FeatureChecker`  that will encapsulate the logic.

We need to get a singleton instance of  `FirebaseRemoteConfig`  and set a lower fetch interval for testing purposes. For a release build, one would either use the default values or set a reasonable custom interval to avoid issues with  [throttling](https://firebase.google.com/docs/remote-config/get-started?platform=android#throttling)  due to excessive requests.


<script src="https://gist.github.com/landomen/559d01a720b1277eea891a2b20724fbf.js"></script>



## 4. Fetch the latest remote config

Next, we need to actually fetch and activate the latest remote values to be able to use them in our app.

We’ll define a new function inside the  `FeatureChecker`  that simply calls the  `FirebaseRemoteConfig.fetchAndActivate()`  function as a suspend function. This will fetch the latest values from the Firebase Remote Config cloud service and apply them to our app. We have to wait until this function completes to make sure we are displaying the correct state.


<script src="https://gist.github.com/landomen/27b174ca7ddc2cfe8614762eb0fac8fb.js"></script>




It’s best to call this function as soon as the app starts so that the latest values are available immediately when needed to provide the best user experience and avoid app UI updates as a result.

## 5. Read the feature flag value

To get the value of a feature flag, we have to define a new function inside  `FeatureChecker`  that reads the value from the previously fetched config.


<script src="https://gist.github.com/landomen/a6ca13baf9eb0baa5f01a02eac8292f1.js"></script>


Now we can call this function after we have successfully fetched the latest remote config and use the result to show/hide some UI or enable/disable some feature in our app.


<script src="https://gist.github.com/landomen/ddaf6712300928115f79f065605ef870.js"></script>



To see that this works as expected, you can change the value on the Firebase Remote Config console and then restart the app to see the new value applied.

## 6. Using it in Compose with a ViewModel

Here is a simple example of how to use feature flags in a codebase with Compose and View Models. We are fetching the latest features when  `ViewModel`  is initialized and then checking if the share feature flag is enabled and posting the result to the  `StateFlow`  that the  `Composable`  function is observing.


<script src="https://gist.github.com/landomen/41b9f5a5527ce4e0362ef05fba11c020.js"></script>


<script src="https://gist.github.com/landomen/fe1d9cfb4bd3496a93698723c6cf95d4.js"></script>



## 7. Setting a default value

Sometimes we want to have a default value for a feature flag available locally before we can fetch the latest remote values so that the app behaves properly. We can do that by providing a  `Map`  object of key/value pairs to the  `FirebaseRemoteConfig`  when initializing the  `FeatureChecker`.


<script src="https://gist.github.com/landomen/72f728d11f7bf46d748e205745e8d68a.js"></script>



## 8. Observe updates in real-time

Our solution so far allows us to fetch the latest values when the app is launched. However, any changes made to the remote config on the Firebase console will only be reflected once the app is launched again (and the fetch interval is exceeded in the case of the production app). But sometimes we might need to react to changes in the config in real time. Firebase Remote Config allows us to do that by attaching an update listener.

Once a change is made on the Firebase console we will receive a callback with the names of all parameters/feature flags that were changed. We then have to apply those changes by calling  `activate()`  and after they are applied, we can use them to update the UI.



<script src="https://gist.github.com/landomen/2694009a3bd01b4e9260277ec8422175.js"></script>




> _Note: we are using_ `_callbackFlow_` _builder to convert the callback-based API to a Kotlin Flow._

And then in our  `ViewModel`  we can use this function to observe the  `Flow`  and update the UI.



<script src="https://gist.github.com/landomen/2150214e0ca758ed1b1864a3f9e0c3f2.js"></script>



Now we will see the UI reflect changes as soon as they are published on the Firebase console.

# Conclusion

Feature flags are a powerful tool that makes building and releasing mobile apps both safer and faster. Developers and teams can merge code faster and release more often, without having to worry about breaking something. Additionally, it allows the product owners and the team to target specific users and to experiment via A/B testing.

We’ve also taken a look at how we can start using feature flags by leveraging the Firebase Remote Config service. It offers all the necessary infrastructure out of the box and makes it easy to get up and running, but also supports complex use cases.

All in all, the benefits of feature flags far outweigh the drawbacks and are a must-have when working on any quickly evolving mobile application.

## References:

-   [A guide to feature flags in mobile development](https://bitrise.io/blog/post/a-guide-to-feature-flags-in-mobile-development)
-   [Get started with Firebase Remote Config](https://firebase.google.com/docs/remote-config/get-started?platform=android)
-   [Adding Firebase Remote Config to a Jetpack Compose app](https://firebase.blog/posts/2022/11/adding-remote-config-to-jetpack-compose-app/)