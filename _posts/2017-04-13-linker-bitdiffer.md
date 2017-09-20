---
layout: post
title: Xamarin.Android Linker Tricks Part 1 - Bitdiffer
---

# Preface

The linker used in Xamarin.Android applications has historically been a painpoint for developers.

This is mainly because as developers, we only use a small portion of the assemblies we consume. For the rest of those assemblies, we consider using a `linker` mechanism so we reap the benefits of having a smaller assembly at the end of the day.

To give a real world example of what the `linker` does, I've created three `.apk` files with the different `linker` options:

- None
- Sdk Assemblies Only
- Sdk and User Assemblies

```
04/13/2017  11:15 AM        52,160,463 LinkerSample.None.apk
04/13/2017  11:12 AM        21,995,031 LinkerSample.SDK.apk
04/13/2017  11:16 AM        10,130,214 LinkerSample.UserAndSDK.apk
```

**Note:** This project includes a reference to the `Xamarin.Android.Support.Design` NuGet to demonstrate something more than "Hello World"

As you can see, we see the size of `52mb`(None), `21mb`(SDK), and `10mb`(User and SDK). What this means is that in a perfect world, we would want to use `Sdk and User Assemblies` so we can keep our `.apk` size down. However using that setting is considered the most aggressive linker setting and it can strip away things that you need to use in your application. Finding out exactly what it strips out is a bit difficult and we should fallback to other tooling.

## Using bitdiffer

`bitdiffer` is a tool that helps compare assembly files. It is extremely useful in the sense of a practical GUI/CLI that lets us see the difference between assemblies. You can download it here:

<https://github.com/bitdiffer/bitdiffer>

One of the most useful things bitdiffer does is allows you to add not only 2 assemblies, but a full list for comparison. This is extremely useful for us as developers to see what exactly each `linker` option does to our assembly.

Consider the following example with the three `.apk` we created earlier comparing against the `Mono.Android.dll` assembly. They are ordered `None`, `Sdk Assemblies Only`, `Sdk and User Assemblies` in the tabs:

![](https://content.screencast.com/users/JDouglas2529/folders/Snagit/media/ce8db931-fb44-4338-8218-2a7b432403ec/04.13.2017-11.34.png)

You can notice off a quick glance that we had quite a difference in `Members Changed` or `Removed` between the linker setting `None` -> `Sdk Assemblies Only` -> `Sdk and User Assemblies`.

Now that's not the only cool feature of this tool, you can also dive down into a specific class to see the `diff` between your assemblies:

![](https://content.screencast.com/users/JDouglas2529/folders/Snagit/media/445fb42b-e0f5-455d-923d-07d626ebc409/04.13.2017-11.40.png)

And finally we can dig into what the difference is between members in our class:

![](https://content.screencast.com/users/JDouglas2529/folders/Snagit/media/c297819c-f32b-4eba-81f5-4f9b394b51d0/04.13.2017-11.44.png)

Although this is only part 1 of various linker tricks I use to diagnose a problem, this is one of the more powerful ways to approach any linker issues in your project to understand the problem.

If you enjoyed this post, please consider subscribing to my upcoming book's email list:

You can signup at the following link: [Programming Xamarin.Android Email List](https://eepurl.com/cz_fj1)