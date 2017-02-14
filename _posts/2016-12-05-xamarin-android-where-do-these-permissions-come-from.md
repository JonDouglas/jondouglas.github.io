---
layout: post
title: Xamarin.Android - Where Do These Permissions Come From?
---

## Preface

I get this question quite often:

> Why does my application have extra permissions added to my `AndroidManifest.xml`?

Many developers think it's a problem with `Xamarin.Android`, but they aren't aware of what really happens with third party libraries and their applications.

To clear up that muck, let's first talk about the general process.

Typically when you have an external library, those libraries have their own manifests. Those manifests can have various `<uses-permission>` elements inside, or they can even specify them inside the `AssemblyInfo.cs` like the following:

    [assembly: UsesPermission(Android.Manifest.Permission.AccessCoarseLocation)]

Either way, these permissions will get inserted into an `AndroidManifest.xml` at the end of the day for that library and then eventually into your application's `AndroidManifest.xml`.

## Manifest Merging

What do we know about Android development thus far? Well we know that each `.apk` file can only have **one** `AndroidManifest.xml` defined. Thus that means at the end of the day, everything needs to **merge** into a master `AndroidManifest.xml`.

> Every application must have an AndroidManifest.xml file (with precisely that name) in its root directory. The manifest file provides essential information about your app to the Android system, which the system must have before it can run any of the app's code.

[https://developer.android.com/guide/topics/manifest/manifest-intro.html](https://developer.android.com/guide/topics/manifest/manifest-intro.html)

Great! Now we are one step ahead of the game and can understand a little bit about the Android build process. However that brings us to a little rut...**Does `Xamarin.Android` follow the same build process as say native `Android`?**

The answer is no. They both have their own unique build processes and thus not everything we learn about native `Android` can be converted directly to `Xamarin.Android`. In this specific case, if we learned about how [native `Android` merges multiple manifest files](https://developer.android.com/studio/build/manifest-merge.html) we would find out that this is done via `Gradle` and not `MSBuild` which `Xamarin.Android` uses under the hood.

## Example

Let's take a `File -> New Blank Android Project`. Let's rebuild it from the start and see a final `AndroidManifest.xml`. We would find this file in the `obj\Debug\android\AndroidManifest.xml`:

    <?xml version="1.0" encoding="utf-8"?>
    <manifest xmlns:android="http://schemas.android.com/apk/res/android" package="IDontWantYourPermission.IDontWantYourPermission" android:versionCode="1" android:versionName="1.0">
      <!--suppress UsesMinSdkAttributes-->
      <uses-sdk android:minSdkVersion="16" />
      <uses-permission android:name="android.permission.INTERNET" />
      <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
      <application android:label="IDontWantYourPermission" android:name="android.app.Application" android:allowBackup="true" android:icon="@drawable/icon" android:debuggable="true">
    <activity android:icon="@drawable/icon" android:label="IDontWantYourPermission" android:name="md5bdb5a298d3d71fd07b60a60955b14ddd.MainActivity">
      <intent-filter>
    <action android:name="android.intent.action.MAIN" />
    <category android:name="android.intent.category.LAUNCHER" />
      </intent-filter>
    </activity>
    <provider android:name="mono.MonoRuntimeProvider" android:exported="false" android:initOrder="2147483647" android:authorities="IDontWantYourPermission.IDontWantYourPermission.mono.MonoRuntimeProvider.__mono_init__" />
    <!--suppress ExportedReceiver-->
    <receiver android:name="mono.android.Seppuku">
      <intent-filter>
    <action android:name="mono.android.intent.action.SEPPUKU" />
    <category android:name="mono.android.intent.category.SEPPUKU.IDontWantYourPermission.IDontWantYourPermission" />
      </intent-filter>
    </receiver>
      </application>
    </manifest>

We only see two permissions being added by default in a `Debug` configuration:

      <uses-permission android:name="android.permission.INTERNET" />
      <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />

However, now let's add a NuGet package that we believe might be adding permissions we don't want in our project. For this example, I'm going to use `James Montemagno's` [http://www.nuget.org/packages/Xam.Plugin.Geolocator](http://www.nuget.org/packages/Xam.Plugin.Geolocator)

After adding it, rebuilding the solution, and checking the new `obj\Debug\android\AndroidManifest.xml`:

      <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
      <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />

So surely enough we would never have known these items were added to our project unless we checked the final generated `AndroidManifest.xml`.

## Okay...But that doesn't help me understand where a permission is coming from if my project is huge!

Fair enough. Let's figure out what we can provide to make this easier for us.

Here's a few ways we can figure out this information:

1) Search through all third-party dependencies via a decompiler like dotPeek and view the Assembly's information to see if a permission is being added.

EX: (Using dotPeek)

![](http://content.screencast.com/users/JDouglas2529/folders/Snagit/media/cc36c438-038a-4efb-938c-7fbd14cd30f5/12.05.2016-13.49.png)

2) Use a `grep` tool to search for a `uses-permission` and `UsesPermission` string.

EX: (Using grepWin - [https://sourceforge.net/projects/grepwin/files/](https://sourceforge.net/projects/grepwin/files/))

![](http://content.screencast.com/users/JDouglas2529/folders/Snagit/media/a2d541f4-85e1-451f-838b-f9999014190c/12.05.2016-13.46.png)





