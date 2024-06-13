---
title: Guide to Foreground Services on Android 14
description: Learn how to work with Foreground Services on Android 14.
date: 2024-02-10 11:00:00 +0100
categories: [Android, Foreground Services, Background Work]
tags: [android, foreground services, background work]
image: /assets/img/posts/foreground-services/cover.png
---


Android 14 includes breaking changes related to foreground services that need to be incorporated if you want to target SDK version 34. We’ll take a look at what these changes are and what needs to be done.

> **Reminder:** `_targetSdk_`  [will need to be 34+](https://developer.android.com/google/play/requirements/target-sdk)  for new apps and app updates by August 31, 2024.

We are also going to cover some common exceptions that might turn up and how to fix them.

At the end of this article, you will also find a sample project demonstrating how to properly implement foreground services.



## What is a foreground service?

Foreground service is a service that performs work or operation that is visible to the user and can continue executing even when the user is not directly interacting with the app. Such service must display a system notification to make the user aware that it is active and using system resources.

Examples of apps that use foreground services include:

-   a music player app (like Spotify) that plays music even when the user leaves the app,
-   a fitness app (like Google Fit) tracking the number of steps even when the phone is locked,
-   a navigation app (like Google Maps) providing driving directions.

## Foreground Service Types

Android 10 introduced the  `android:foregroundServiceType`  attribute within the  `[<service>](https://developer.android.com/guide/topics/manifest/service-element)`  element. The idea of it is to explicitly specify what kind of work the service does.  Until now, it was necessary to specify the type only if the service used location, camera, or microphone permission.

Android 14 makes specifying the foreground service type mandatory. This is to ensure the correct usage of foreground services and consistency across device manufacturers.

The currently supported types are:

- [camera](https://developer.android.com/about/versions/14/changes/fgs-types-required#camera)  (required in Android 11) — when accessing the camera from the background, such as video calling apps
-   [connectedDevice](https://developer.android.com/about/versions/14/changes/fgs-types-required#connected-device)  — when interacting with external device, such as Bluetooth fitness device
-   [dataSync](https://developer.android.com/about/versions/14/changes/fgs-types-required#data-sync)  — when uploading or downloading data, will be deprecated and alternatives like  `DownloadManager`,  `BackupManager`, or  `WorkManager`  should be used instead
-   [health](https://developer.android.com/about/versions/14/changes/fgs-types-required#health)  (new in Android 14) — for fitness apps such as exercise trackers
-   [location](https://developer.android.com/about/versions/14/changes/fgs-types-required#location)  (required in Android 10) — when location is required, such as navigation
-   [mediaPlayback](https://developer.android.com/about/versions/14/changes/fgs-types-required#media) — when continuing audio or video playback from the background, like Spotify or Netflix
-   [mediaProjection](https://developer.android.com/about/versions/14/changes/fgs-types-required#media-projection)  — when projecting content to external devices or screens
-   [microphone](https://developer.android.com/about/versions/14/changes/fgs-types-required#microphone)  (required in Android 11) — when accessing the microphone from the background, such as calling apps
-   [phoneCall](https://developer.android.com/about/versions/14/changes/fgs-types-required#phone-call)  — when continuing an ongoing call
-   [remoteMessaging](https://developer.android.com/about/versions/14/changes/fgs-types-required#remote-messaging)  (new in Android 14) — when transferring text messages from one device to another
-   [shortService](https://developer.android.com/about/versions/14/changes/fgs-types-required#short-service)  — when it’s required to quickly finish critical work that can’t be interrupted, can only run for about 3 minutes
-   [specialUse](https://developer.android.com/about/versions/14/changes/fgs-types-required#special-use)  — when other types don’t cover your use case
-   [systemExempted](https://developer.android.com/about/versions/14/changes/fgs-types-required#system-exempted)  — reserved for system apps

**Declare foreground service type**<br>
The first step in supporting Android 14 is to update your service declaration in the  `AndroidManifest`  file and specify the correct foreground service type.

If your service requires multiple types, you can combine them using the  `|`  operator like this:


```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android" ...>
    <service
        android:name=".MyForegroundService"
        android:foregroundServiceType="camera|location|microphone"
        android:exported="false">
    </service>
</manifest>
```



If you try to start a foreground service without declaring its type in the manifest, the system will throw a  `[MissingForegroundServiceTypeException](https://developer.android.com/reference/android/app/MissingForegroundServiceTypeException)`  upon calling  `startForeground()`.

## Request specific foreground service permission

From Android 9 (API 28) onwards apps had to request  `FOREGROUND_SERVICE`  permission in the app manifest, which was granted automatically by the system.


```xml
<uses-permission android:name="android.permission.FOREGROUND_SERVICE"/>
```


Starting from Android 14 (API 34) apps need to additionally request specific permission depending on the type of the foreground service. So, if your service connects to an external Bluetooth device you would need to specify `FOREGROUND_SERVICE_CONNECTED_DEVICE`. The permission is automatically granted by the system.


```xml
<uses-permission android:name="android.permission.FOREGROUND_SERVICE_CONNECTED_DEVICE"/>
```

If your service requires multiple types, you have to declare this permission for every type.

If you forget to declare either of the two permissions, you will receive a  `SecurityException`  with the exact reason why.


```java
java.lang.SecurityException: 
     Permission Denial: startForeground from pid=8589, uid=10623 
     requires android.permission.FOREGROUND_SERVICE

or

java.lang.SecurityException: 
     Starting FGS with type mediaPlayback targetSDK=34 
     requires permissions: 
        all of the permissions allOf=true 
        [android.permission.FOREGROUND_SERVICE_MEDIA_PLAYBACK]
```



## Specify service type in startForeground

Additionally to declaring the foreground service type in the manifest, it is also necessary to specify it when calling the  `startForeground()`  function.

To make the service run in the foreground, we have to call  `[ServiceCompat.startForeground()](https://developer.android.com/reference/androidx/core/app/ServiceCompat#startForeground(android.app.Service,int,android.app.Notification,int))`  inside the service, usually in  `onStartCommand()`. This function takes the service, the id of the notification, the notification object, and the foreground service types identifying the work done by the service.

In previous versions of Android, we could simply pass  `0`  for the  `foregroundServiceType`  argument. However, it’s now required to pass the correct type or a subset of types declared in the manifest. It’s possible to call  `startForeground()`  multiple times with additional types, depending on the use case.


```kotlin
ServiceCompat.startForeground(
    this,
    id,
    notification,
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R) {
        ServiceInfo.FOREGROUND_SERVICE_TYPE_MEDIA_PLAYBACK
    } else {
        0
    }
)
```



If you try to pass  `0`  for  `foregroundServiceType`  to  `startForeground()`, you will receive an exception:



```java
android.app.InvalidForegroundServiceTypeException: 
  Starting FGS with type none 
  targetSDK=34 has been prohibited
```

And if you pass a type that you have not declared in the manifest, you will receive an exception similar to this one:



```java
java.lang.IllegalArgumentException: 
  foregroundServiceType 0x00000002 is not a subset of 
  foregroundServiceType attribute 0x00000000 in service 
  element of manifest file
```


> Reminder: You have to call  `startForeground()`  within 10 seconds of receiving  `onStartCommand()`  after starting your service using  `startForegroundService()`, otherwise, the app will crash with the following exception:


```java
android.app.ForegroundServiceDidNotStartInTimeException: 
  Context.startForegroundService() did not then call Service.startForeground()
```

## Request runtime permissions

Each foreground service type has a list of required permissions. You must request and be granted the needed runtime permissions before you start the foreground service. If the permissions are not granted, and you try to start it, the service will throw an exception.

For example: if your service needs to use the camera even while the app is in the background, you will need to request  `android.permission.CAMERA`.

And if your service connects to a Bluetooth device then at least one of the following conditions must be true:

-   Declare at least one of the following permissions in your manifest:  
    -  `CHANGE_NETWORK_STATE`  
    -  `CHANGE_WIFI_STATE`  
    -  `CHANGE_WIFI_MULTICAST_STATE`  
    -  `NFC`  
    -  `TRANSMIT_IR`
-   Request and be granted at least one of the following runtime permissions:  
    -  `BLUETOOTH_CONNECT`  
    -  `BLUETOOTH_ADVERTICE`  
    -  `BLUETOOTH_SCAN`  
    -  `UWB_RANGING`
-   Call  `UsbManager.requestPermission()`

> You can find the exact requirements for all the supported types in the  [official documentation](https://developer.android.com/about/versions/14/changes/fgs-types-required#use-cases).

In case you don’t fulfill the conditions before starting the service, you will receive an exception which will also contain information on which condition is not met. In the case below, the app has not been granted the needed permissions.




```java
Starting FGS with type connectedDevice targetSDK=34 requires permissions: 
- all of the permissions allOf=true 
  - [android.permission.FOREGROUND_SERVICE_CONNECTED_DEVICE] 
- any of the permissions allOf=false 
  - [android.permission.BLUETOOTH_ADVERTISE, 
     android.permission.BLUETOOTH_CONNECT, 
     android.permission.BLUETOOTH_SCAN, 
     android.permission.CHANGE_NETWORK_STATE, 
     android.permission.CHANGE_WIFI_STATE, 
     android.permission.CHANGE_WIFI_MULTICAST_STATE, 
     android.permission.NFC, 
     android.permission.TRANSMIT_IR, 
     android.permission.UWB_RANGING, 
     USB Device, 
     USB Accessory]
```



## Make sure to have notifications set up correctly

When starting a foreground service it is required to provide a notification that will be visible to the user for the duration of the service execution. Android 13 introduced a runtime permission for posting notifications, requiring that apps ask for this permission and users have to explicitly grant it, otherwise notifications will not be visible.

-   If you request the  `POST_NOTIFICATIONS`  permission and the user grants it, the notification will be displayed normally.




![Foreground service notification.](/assets/img/posts/foreground-services/foreground_service_notification.webp)
_Foreground service notification._



-   If you request the permission and the user denies it, then the notification will not be displayed, but the service will still work as intended. Users will see that the app is performing background work in the task manager.
-   If you  don’t ask for the notification permission and try to post it regardless, the app will behave the same as in the previous point.


![Task manager showing active foreground services.](/assets/img/posts/foreground-services/background_activity.jpg)
_Task manager showing active foreground services._


## Provide details of your use case on Google Play Console

After you upload the new version of the app that targets Android 14 and uses a foreground service type to the Google Play Console, you will see a prompt in your console to provide additional details about your usage.

As Google wants to make sure that apps are using foreground services appropriately, you will need to submit a new declaration on the  [App content](https://play.google.com/console/app/app-content/summary)  page (Policy -> App content).

For each foreground service type* you declare, you’ll need to do the following:

1.  Describe the app functionality that is using that foreground service type.
2.  Describe the user impact if the task is deferred or interrupted by the system.
3.  Include a link to a video demonstrating each foreground service feature. The video should demonstrate the steps the user needs to take in your app to trigger the feature.
4.  Choose your specific use case for each foreground service type. You can choose from a pre-set list of use cases listed  [here](https://support.google.com/googleplay/android-developer/answer/13392821)  or enter it manually.

* types  `shortService`  and  `systemExempted`  do not require this declaration.

> You can find up-to-date guidelines and more information on the  [official support page](https://support.google.com/googleplay/android-developer/answer/13392821).

## Improvements for Samsung devices

Samsung has collaborated with Google on a unified policy to make foreground services run as intended on Galaxy devices running Android 14 and newer. This is an important step towards a unified Android platform, as Samsung has a 34%¹ market share, and previously the foreground services would behave differently in some cases compared to devices like Pixels.

Here is the full quote²:

> “To strengthen the Android platform, our collaboration with Google has resulted in a unified policy that we expect will create a more consistent and reliable user experience for Galaxy users. Since One UI 6.0, foreground services of apps targeting Android 14 will be guaranteed to work as intended so long as they are developed according to Android’s new foreground service API policy.” — Samsung

# Sample app

I’ve prepared a simple sample app demonstrating how to create and start a foreground service on Android 14. Included are the following functionalities:

-   Starting a foreground service with a declared foreground service type of location
-   Requesting location permissions before service is started
-   Binding to the foreground service from Activity to display the service status and receive location updates
-   Stopping the service from the activity
-   Requesting notification permission to show foreground service notification


You can find the source code on [GitHub](https://github.com/landomen/ForegroundService14Sample).


# Conclusion

Android 14 brought several changes related to foreground services, which means additional work for developers to make their apps target API 34. The biggest one is that foreground service types are now mandatory, which requires fulfilling all the requirements before starting the service.

The new changes mean a more standardized approach to foreground services and hopefully better support from different manufacturers.

I hope you found this guide useful and I encourage you to review the sample app and to check out the additional resources linked below to learn more.

Resources:

-   [https://developer.android.com/develop/background-work/services/foreground-services](https://developer.android.com/develop/background-work/services/foreground-services)  — official documentation for foreground services
-   [https://developer.android.com/about/versions/14/changes/fgs-types-required](https://developer.android.com/about/versions/14/changes/fgs-types-required)  — describes in detail the requirements for all the foreground service types
-   [https://www.droidcon.com/2023/11/15/a-guide-to-using-foreground-services-and-background-work-in-android-14/](https://www.droidcon.com/2023/11/15/a-guide-to-using-foreground-services-and-background-work-in-android-14/)  — great talk from Droidcon London 23 by Alice Yuan that covers best practices and official guidelines from Google for background work on Android

References:

-   [1]  [https://www.demandsage.com/android-statistics/](https://www.demandsage.com/android-statistics/)
-   [2]  [https://android-developers.googleblog.com/2023/05/improving-consistency-of-background-work-on-android.html](https://android-developers.googleblog.com/2023/05/improving-consistency-of-background-work-on-android.html)