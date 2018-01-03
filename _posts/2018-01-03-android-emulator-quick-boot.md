---
layout: post
title: Android Emulator - Quick Boot
---

# Android Emulator Quick Boot

Beginning with [Android Emulator version 27.0.2](https://developer.android.com/studio/releases/emulator.html#27-0-2), the Android Emulator now includes a feature called `Quick Boot` that launches the emulator in a few seconds.

For `Quick Boot` to work, your AVD must perform a cold boot on it’s first time booting up. All subsequent starts will use the `Quick Boot` feature which restores the system to the state it was last closed in.

This feature is turned on by default.

## Getting Started

To get started, open up the [Xamarin Android SDK Manager](https://developer.xamarin.com/guides/android/getting_started/installation/android-sdk/) inside your IDE by going to `Tools > Android > SDK Manager`.

![](https://i.imgur.com/cXi20Ca.png)

Under the `Tools` tab, update the following items:

- Android SDK Tools 26.1.1
- Android Emulator 27.0.2

![](https://i.imgur.com/yVBpCD8.png)

Once you’ve installed the requirements for this feature, it’s as easy as booting up your favorite AVD for the first time to start the initial cold boot.

![](https://i.imgur.com/qU0UBEm.png)

When you’re done using the AVD, you can exit out of the emulator and `Quick Boot` will save the state of your emulator.

![](https://i.imgur.com/WpGHpIi.png)

The next time you open your AVD, it will load the existing state and proceed to `Quick Boot`.

![](https://i.imgur.com/fnQ5tgX.png)

## Summary

Your emulator should now start up in under 6 seconds. Now there is no excuse to wait for the emulator to boot up and you can continue coding!

![](https://media.giphy.com/media/xUNda2xaw192P7BtM4/giphy.gif)

## Xamarin.Android Book

If you enjoyed this post, please consider subscribing to my upcoming book's email list:

You can signup at the following link: [Programming Xamarin.Android Email List](https://eepurl.com/cz_fj1)