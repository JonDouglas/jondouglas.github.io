---
layout: post
title: Xamarin.Android - UI Performance
---

# Android UI Performance

**NOTE: This blog was featured on blog.xamarin.com in which you can view a condensed version here: [https://blog.xamarin.com/tips-for-creating-a-smooth-and-fluid-android-ui/](https://blog.xamarin.com/tips-for-creating-a-smooth-and-fluid-android-ui/)**

As developers, we want our users to have buttery smooth experiences when using our application. When it comes to UI performance, a buttery smooth experience can be defined as a consistent **60 frames per second(fps)**. That means that we have to render a frame every **16 ms** to achieve this experience.

**Note:**

> You can exceed the 16 ms time to render a frame occasionally, as there are often one or two buffered frames ready to go.

## Why `60 fps`?

Watch this wonderful video by Google on this topic for a brief introduction:

[![Why 60 FPS](http://img.youtube.com/vi/CaMTIgxCSqU/0.jpg)](http://www.youtube.com/watch?v=CaMTIgxCSqU "Why 60 fps")

This gives our whole frame process a little less than **16 ms** to fully draw a frame on the screen. **If a frame is dropped or delayed, it is known as Jank.**

## Identifying the problem

Have you ever had an experience like this with your application? Your scrolls are very *choppy* and your UI doesn’t seem to be that responsive.

![](https://content.screencast.com/users/JDouglas2529/folders/Snagit/media/5b6740d6-25b8-4676-a43b-a9a0c13c1464/12.08.2016-12.57.GIF)

**Credit: Doug Sillars** [https://github.com/dougsillars](https://github.com/dougsillars) for a great example.

During the time that the application is unresponsive and choppy, if the user attempts to interact with the application and it takes longer than **5000 ms (5 seconds)** for the application to respond, the Android OS will give us a lovely message:

![](https://content.screencast.com/users/JDouglas2529/folders/Snagit/media/5eb41bf1-2b2f-43d5-bffb-2f83202e9d02/12.08.2016-13.05.png)

*This isn't the best experience we can offer our users.*

## Digging Deeper

Have you ever seen the following in your `adb logcat`?


    I/Choreographer(1200): Skipped 60 frames!  The application may be doing too much work on its main thread.


This is the Android Choreographer warning you in advance that your application is not performing to the buttery smooth experience we are aiming for. In fact it’s telling you that you are skipping frames and not providing your users a **60 FPS** experience. We will need a few tips & tricks to resolve these issues.

## Overall Tips & Tricks for UI Performance

- Measure overall UI performance with **Systrace** to get a baseline.
- Use **Hierarchy Viewer** to identify and flatten view hierarchies.
- Reduce overdraw by flattening layouts and drawing less pixels on screen.
- Enable **StrictMode** to identify and make fewer potentially blocking calls on the main UI thread.

## Getting Started

### Counting Janky Frames

First you’ll want to get an overview of the total **janky frames**. After your application has been running and interacted with to reproduce the jank, run the following command:

`adb shell dumpsys gfxinfo <PACKAGE_NAME>`

**Sample Output:**

```
Stats since: 524615985046231ns
Total frames rendered: 8325
Janky frames: 729 (8.76%)
90th percentile: 13ms
95th percentile: 20ms
99th percentile: 73ms
Number Missed Vsync: 294
Number High input latency: 47
Number Slow UI thread: 502
Number Slow bitmap uploads: 44
Number Slow issue draw commands: 135
```

Numbers never lie, so it’s apparent we have a bit of **Janky frames(~9%)**. Let’s dig in further with `Systrace`. 

## Systrace

Systrace gives you an overview of the whole android system and tells you what’s going on at specific intervals of time.

### Getting Started

Let's start a `systrace` on our device. First open up `Android Device Monitor` to get started.

Once inside, we see the option to start a `Systrace`:

![](https://content.screencast.com/users/JDouglas2529/folders/Snagit/media/c2737e71-e6b8-4935-b77a-fd831005305b/12.08.2016-13.30.png)

We then are prompted with what we would like to trace:

![](https://content.screencast.com/users/JDouglas2529/folders/Snagit/media/a6c51b73-4c5b-41fa-957a-3e2e97b2e8a9/12.08.2016-13.34.png)

This will generate a `trace.html` file that will give us information about our system for the trace duration.

**EX: `Systrace` over 30 seconds:**

![](https://content.screencast.com/users/JDouglas2529/folders/Snagit/media/2d438179-25b9-477e-afd6-06e772956e63/12.08.2016-14.16.png)

Okay great! But what does this all mean? Let's take it a step at a time.

### Alerts & Frames

![](https://content.screencast.com/users/JDouglas2529/folders/Snagit/media/19a2ca0b-4ead-4c08-be98-12292cdd5071/12.08.2016-14.31.png)

**Alerts** will give you a description of what the current situation is with a respective frame(s). It might let you know that there was a long `View.OnDraw()` call and it might give you suggestions on how you can fix the relevant frame(s).

![](https://content.screencast.com/users/JDouglas2529/folders/Snagit/media/6b7cea56-40b4-42ab-95a9-91f408665cce/12.08.2016-14.52.png)

You can then dig straight into the **frame**.

![](https://content.screencast.com/users/JDouglas2529/folders/Snagit/media/1485c0a4-67c9-406e-9f16-92df13d6311d/12.08.2016-14.53.png)

And see how much **time spent** during each step

![](https://content.screencast.com/users/JDouglas2529/folders/Snagit/media/a37f4e1e-b8dd-4569-9d27-f4859bcb32e0/12.08.2016-14.58.png)

Finally you can **mark** that frame using the `m` hotkey and see what work is being done on various threads such as various `CPU Threads`, the `UI Thread`, and the `RenderThread`.

![](https://content.screencast.com/users/JDouglas2529/folders/Snagit/media/479ec2ef-0728-4d56-8748-394f349d928e/12.08.2016-15.05.png)

This example is showing a **Yellow** Frame, but we can get a general idea of what an idea performant **Frame** might look like.

From our **Alert**, we can see that we might want to avoid significant work in `View.OnDraw()` or `Drawable.Draw()`, especially allocations or drawing to `Bitmaps`. In other words, Google gives us a tip to watch this video:

[![Avoiding Allocations in onDraw()](http://img.youtube.com/vi/HAK5acHQ53E/0.jpg)](http://www.youtube.com/watch?v=HAK5acHQ53E "Avoiding Allocations in onDraw()")

For our **Frame**, we can see that our `Adapter.GetView()` should recycle the incoming `View` instead of creating a new one.

### Systrace Terms

#### Frame Color

Green - Great performance

Yellow - Less than ideal performance

Red - Bad performance

#### Scheduling Delay

Scheduling delays happen when the thread that is processing a specific slice was not scheduled on the CPU for a long amount of time. Thus it takes longer for this thread to fire up and complete.

#### Wall Duration

The amount of time that passed from the moment a slice is started until it's finished.

#### CPU Duration

The amount of time the CPU spent processing that slice.

## Overdraw

Debugging overdraw is fairly easy. You can enable this in your Android device’s settings:
 
`Settings -> Developer Options -> Debug GPU overdraw -> Show overdraw areas`.

Once enabled, you will see many different colors on your layouts.

- White - No overdraw
- Blue - Pixels that are 1x overdrawn
- Green - Pixels that are 2x overdrawn
- Pink - Pixels that are 3x overdrawn
- Red - Pixels that are 4x overdrawn

**Bad Layout Performance:**

![](https://content.screencast.com/users/JDouglas2529/folders/Snagit/media/c01274a3-ae72-4efb-afaf-33111730f4eb/12.08.2016-15.22.png)

**Good Layout Performance:**

![](https://content.screencast.com/users/JDouglas2529/folders/Snagit/media/7ea37179-77a4-4a26-8f7d-02b7fcd4dc56/12.08.2016-15.24.png)

You can then identify why this layout might be overdrawing so much via a tool like [Hierarchy Viewer](https://developer.android.com/studio/profile/optimize-ui.html#HierarchyViewer).

## Hierarchy Viewer

The first thing we want to know regarding our View Hierarchy is how deep or nested our layouts are.

Let's take the previous example of bad layout performance:

![](https://content.screencast.com/users/JDouglas2529/folders/Snagit/media/66bdd866-9860-4630-966a-3ebd8bf72ecc/12.14.2016-15.45.png)

We can see that we are **4 layers deep** which is less than ideal for our `ListView`. We really need to flatten this out.

![](https://content.screencast.com/users/JDouglas2529/folders/Snagit/media/cd600c91-a42d-42f1-8bc1-19de675de49b/12.14.2016-15.49.png)

Okay that's a little better! We are only **3 layers deep** now. However it's still not great. Let's try to remove one more layer.

![](https://content.screencast.com/users/JDouglas2529/folders/Snagit/media/afe71740-21a7-47b1-9f24-33dbac8c332c/12.14.2016-15.51.png)

Much better! We just optimized our whole view hierarchy and we will reap the performance benefits. Let's take a look at the overall `Layout` and `Draw` timings for proof.

- Bad Layout - 31 views / **Layout: 0.754 ms** / **Draw: 7.273 ms**
- Better Layout - 26 views / **Layout: 0.474 ms** / **Draw: 6.191 ms**
- Good Layout - 17 views / **Layout: 0.474 ms** / **Draw: 1.888 ms**

## StrictMode

StrictMode is a very useful tool for battle testing your application. Enabling StrictMode as a means to ensure you are not putting extra work in certain places of your application like disk reads, disk writes, and network calls is ideal for a great user experience. In a nutshell StrictMode does the following:

1. Logs a message to `LogCat` under the `StrictMode` tag
2. Display a dialog (If `PenaltyLog()` is enabled)
3. Crash your application (If `PenaltyDeath()` is enabled)

This can help you determine what type of policies you'd like to battle test your application with.

**Enabling `StrictMode` in your Android Application**

```
protected override void OnCreate(Bundle bundle)
{
    StrictMode.SetThreadPolicy(new StrictMode.ThreadPolicy.Builder().DetectAll().PenaltyLog().Build());

    StrictMode.SetVmPolicy(new StrictMode.VmPolicy.Builder().DetectLeakedSqlLiteObjects().DetectLeakedClosableObjects().PenaltyLog().PenaltyDeath().Build());

    base.OnCreate(bundle);
}
```

## Summary

There are a plethora of tools available to use on your Xamarin.Android application. Use them to track the important performance-related items about your application such as rendering performance to achieve a buttery smooth 60 fps experience for your customers. Use tools like **Systrace**, **GPU overdraw**, **Hierarchy Viewer**, and **StrictMode** to pinpoint performance related issues in your application and fix them.







