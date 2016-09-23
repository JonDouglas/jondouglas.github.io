---
layout: post
title: Multidex in depth and overriding the main dex list in Xamarin.Android
---

This post is a continuation of [Multidex in Xamarin.Android]({% link _posts/2016-09-05-xamarin-android-multidex.md %})

Today we are going to talk about the basic mechanics of the `classes.dex` file and how multidex handles more than 65k methods.

The `classes.dex` file is used as a main dex list in your application. It houses all of the classes needed for your application. An APK requires at least one `classes.dex`(DEX) file which has all the executable code of our application stored inside. The size of a `DEX` file's method index is 16-bit which means (2^16) or 65,536 total references a single dex list can have.

However we run into scenarios where we surpass the 65,536 reference count. This is when we need to enable multidex which gives us overfill `classes.dex` files in the form of `classes{n}.dex` where n >= 1.

Now we can have as many classes/references as we desire since our application can grow over time. Let's talk about some of the disadvantages to this however:

1) Multidex tries it's best to know what dependencies are needed at startup. Since it only loads the main dex list `classes.dex` at first, you will need to ensure any startup classes/references are inside the *MainDexList*(`classes.dex`) or you will crash at startup.

2) Secondary DEX files will be added to the `classloader` after Multidex is initialized via `install(Context)` or the other two ways I described in the previous blog article - [Multidex in Xamarin.Android]({% link _posts/2016-09-05-xamarin-android-multidex.md %}).

3) Multidex doesn't efficiently store classes/references to the max count in each list it creates. It'll try it's best however.

Okay cool, we have a rundown of what's going on, but let's dig a little deeper to the tooling that generates a *DEX* list. Let's introduce our friend `dx` which is a command line tool to generate respective `.dex` files. There's a couple parameters we want to keep in mind.

1) `--multi-dex`: This will enable multidex and create 1 or more `classes{n}.dex` files.

2) `--main-dex-list=<file>`: This will parse through a list of class file names in which classes defined will be put in the `classes.dex` file.

3) `--minimal-main-dex`: Certain classes selected by `--main-dex-list` above will be put in the main DEX list(`classes.dex`)

## Typical Issues

1) Custom Application class is not found on dexPathList:

This is very straight forward now that we know what's going on with multidex. Simply put, our custom application class is not being put on the main `classes.dex` list. Therefore it cannot even open the entrypoint of the application nor initialize the secondary *DEX* lists.

2) Other classes needed at startup are not found on dexPathList:

This is also quite straight forward. You might have a framework dependency such as `MVVMCROSS` or other items that register at startup which need to be on the main `classes.dex` list.

## How to investigate Multidex issues

There is really one tool that is needed now-a-days to investigate the behavior. That tool is classyshark:

<https://github.com/google/android-classyshark>

This tool can read either `.dex` or `.apk` files. Since we are primarily dealing with `.dex` files, we can directly drag and drop them into classyshark to see all of the classes listed, and also the total method count.

You could technically go further into reading about `dx` tooling, but it's not really worth going that far into unless there's a critical bug. Google recommends: *As a general rule, you should rely on the build tools to call them as needed.*

## Overriding the main dex list(`classes.dex`)

Xamarin.Android now offers a simple way to override this list. You can do the following:

1) Create a new Text file in your main application root. (Name it `multidex.keep`)

2) Set the `Build Action` to `MultiDexMainDexList`

3) Include any classes you want on the main dex list inside

*Note:* It's always a good idea to see a previous `multidex.keep` file in your `obj\Debug` folder for a reference.

I hope this helps!
