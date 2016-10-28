---
layout: post
title: Porting Android Libraries to Xamarin.Android
---

When it comes to Android, there is a vast open source community with thousands of libraries to use inside our Android application.

[https://github.com/wasabeef/awesome-android-libraries](https://github.com/wasabeef/awesome-android-libraries)

[https://github.com/codepath/android_guides/wiki/Must-Have-libraries](https://github.com/codepath/android_guides/wiki/Must-Have-libraries)

However not all of these awesome libraries have an equivalent library in the `Xamarin` community. This of course makes it awkward for us as a developer for multiple reasons:

- Not enough time to write port the respective Java -> C# code
- Not familiar enough with Java/C# to know the language equivalents
- Just want to use the library and not care about the internals

Because of this, this gives us only a few options to use one of these libraries:

1. Port over the Java code to C#
2. Create a [Xamarin.Android Binding Project](https://developer.xamarin.com/guides/android/advanced_topics/binding-a-java-library/)
3. Wait for somebody else to do it (Xamarin Community)

However there are pros/cons of each of these items:

### Porting over the Java code to C#:

This is personally my favorite way to create a Xamarin.Android library. You will get to learn how the library was made simply by porting it over.

**Pros**

- Everything will be in C#/Xamarin.Android and easier to maintain in the future
- Easier to debug
- It is fairly easy to port Java to C# ([Guide](https://developer.xamarin.com/guides/android/advanced_topics/xamarin-for-java/))

**Cons**

- It takes awhile to write in terms of keyboard strokes
- Certain language features can be hard to port over (Reflection, Anonymous Inner Classes, Enums, etc)

### Create a Xamarin.Android Binding Project:

This scenario basically allows the Xamarin.Android Binding Generator do all the work for you. In most simple cases it works quite well, but in more complicated cases, you might end up pulling out your hair.

**Pros**

- Easy to generate a quality binding without writing much code
- Ability to transform the binding how you see fit to match C# conventions
- Does not take much time

**Cons**

- Hard to debug generated code
- Have to learn how to [troubleshoot bindings](https://developer.xamarin.com/guides/android/advanced_topics/binding-a-java-library/troubleshooting-bindings/) / [bindings guide](https://gist.github.com/JonDouglas/dda6d8ace7d071b0e8cb)

### Wait for somebody else to do it (Xamarin Community)

Waiting on other people is typically a last resort when it comes to deadlines.

**Pros**

- All the work has been done for you and tested
- Plug and Play
- Officially maintained by somebody

**Cons**

- Have to wait and rely on other people
- Libraries can be abandoned by maintainers

Overall with these three scenarios laid out, it's really up to you to make the decision on what might be best for your project's needs.

Here's a quick guide to help you find if a library already exists:

1. Search [NuGet](https://www.nuget.org/) and [Xamarin Components](https://components.xamarin.com/) for the library you're after
2. Search Google for the `<Library Name>` + `Xamarin` (**Note:** Some libraries might be hosted on other providers like CodePlex, GitLab, etc)
3. Search Github for snippets of the `Java` library to see if there's any `Xamarin` counterparts