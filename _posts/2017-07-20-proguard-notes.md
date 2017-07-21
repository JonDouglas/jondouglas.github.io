---
layout: post
title: Xamarin.Android Proguard Notes
---

# Preface

So you've heard of this tool called "Proguard" which can Shrink and Optimize unused code from our apps and the libraries they reference. Little did you know that you actually have to do a bit of work so Proguard can work for you.

## Holy Notes Batman!

Have you ever enabled Proguard in your project and now your Build Output is plagued by logs of `Note:`?

Here's a sample of a Build Output of a project that has the `Xamarin Android Support Library - Design`(<https://www.nuget.org/packages/Xamarin.Android.Support.Design/>) installed. This of course has a large list of dependencies on other support libraries.

<https://gist.github.com/JonDouglas/fe19e8e7be658779e8b68b90c8fc7aad>

You may notice hundreds of notes saying 

```
Note: the configuration doesn't specify which class members to keep for class `package.ClassName`
```

## Proguard Summary

However let's first get some numbers about just how much is affecting us. At the bottom of our `Proguard` Task, we will find a summary that let's us know exactly how much.

```
1>  Note: there were 109 references to unknown classes. (TaskId:247)
1>        You should check your configuration for typos. (TaskId:247)
1>        (http://proguard.sourceforge.net/manual/troubleshooting.html#unknownclass) (TaskId:247)
1>  Note: there were 1 classes trying to access generic signatures using reflection. (TaskId:247)
1>        You should consider keeping the signature attributes (TaskId:247)
1>        (using '-keepattributes Signature'). (TaskId:247)
1>        (http://proguard.sourceforge.net/manual/troubleshooting.html#attributes) (TaskId:247)
1>  Note: there were 10 unkept descriptor classes in kept class members. (TaskId:247)
1>        You should consider explicitly keeping the mentioned classes (TaskId:247)
1>        (using '-keep'). (TaskId:247)
1>        (http://proguard.sourceforge.net/manual/troubleshooting.html#descriptorclass) (TaskId:247)
1>  Note: there were 5 unresolved dynamic references to classes or interfaces. (TaskId:247)
1>        You should check if you need to specify additional program jars. (TaskId:247)
1>        (http://proguard.sourceforge.net/manual/troubleshooting.html#dynamicalclass) (TaskId:247)
1>  Note: there were 6 accesses to class members by means of introspection. (TaskId:247)
1>        You should consider explicitly keeping the mentioned class members (TaskId:247)
1>        (using '-keep' or '-keepclassmembers'). (TaskId:247)
1>        (http://proguard.sourceforge.net/manual/troubleshooting.html#dynamicalclassmember) (TaskId:247)
```

## Proguard Reminder

Before jumping into these items one by one, let's first understand what's going on when we are calling Proguard.

Simply put, we are calling the `Proguard.jar` and providing various `Configuration` files that depict how `Proguard` should process the shrinking.

The three main files to care about are the following:

### proguard_xamarin.cfg

These are rules unique to Xamarin. Such as ensuring to keep the mono, android, and java classes that Xamarin.Android relies on.

```
ProguardCommonXamarinConfiguration=obj\Release\proguard\proguard_xamarin.cfg
```

### proguard_project_references.cfg

These are rules that are generated based on the ACW(Android Callable Wrappers) of your references. In this case the `Xamarin.Android.Support.Design` library and it's dependencies.

```
ProguardGeneratedReferenceConfiguration=obj\Release\proguard\proguard_project_references.cfg
```

### proguard_project_primary.cfg

These are rules that are generated based on your Application.

```
ProguardGeneratedApplicationConfiguration=obj\Release\proguard\proguard_project_primary.cfg
```

### Custom proguard.cfg

Finally it will include your custom `Proguard.cfg` file which is defined with the `Build Action` of `ProguardConfiguration`.

Let's break these notes down one by one:

## references to unknown classes
```
1>  Note: there were 109 references to unknown classes. (TaskId:247)
1>        You should check your configuration for typos. (TaskId:247)
1>        (http://proguard.sourceforge.net/manual/troubleshooting.html#unknownclass) (TaskId:247)
```

> Your configuration refers to the name of a class that is not present in the program jars or library jars. You should check whether the name is correct. Notably, you should make sure that you always specify fully-qualified names, not forgetting the package names.

We can now search for the keywords `unknown class` within our Build Output and see what type of classes it's throwing this `Note:` for.

EX: `Note: the configuration refers to the unknown class 'com.google.vending.licensing.ILicensingService'`

So it is saying that our configuration refers to this unknown class, but if we did a bit of searching, we don't see this class anywhere within our `.cfg` files above. Let's add some diagnostics by adding the `-printconfiguration config.txt` rule into our custom `Proguard.cfg` file.

If we now look through this `config.txt` file, we will see the rule caught red handed:

```
-keep public class com.google.vending.licensing.ILicensingService
```

Unfortunately this is a limitation of generation of Proguard rules. Some of these will be completely out of your control.

## access generic signatures using reflection
```
1>  Note: there were 1 classes trying to access generic signatures using reflection. (TaskId:247)
1>        You should consider keeping the signature attributes (TaskId:247)
1>        (using '-keepattributes Signature'). (TaskId:247)
1>        (http://proguard.sourceforge.net/manual/troubleshooting.html#attributes) (TaskId:247)
```

> Your code uses reflection to access metadata from the code, with an invocation like "class.getAnnotations()". You then generally need to preserve optional class file attributes, which ProGuard removes by default. The attributes contain information about annotations, enclosing classes, enclosing methods, etc. In a summary in the log, ProGuard provides a suggested configuration, like -keepattributes *Annotation*. If you're sure the attributes are not necessary, you can switch off these notes by specifying the -dontnote option.

This one however is straight forward by just adding the `-keepattributes Signature` to our Custom `Proguard.cfg` file.

## unkept descriptor classes in kept class members.
```
1>  Note: there were 10 unkept descriptor classes in kept class members. (TaskId:247)
1>        You should consider explicitly keeping the mentioned classes (TaskId:247)
1>        (using '-keep'). (TaskId:247)
1>        (http://proguard.sourceforge.net/manual/troubleshooting.html#descriptorclass) (TaskId:247)
```

> Your configuration contains a -keep option to preserve the given method (or field), but no -keep option for the given class that is an argument type or return type in the method's descriptor. You may then want to keep the class too. Otherwise, ProGuard will obfuscate its name, thus changing the method's signature. The method might then become unfindable as an entry point, e.g. if it is part of a public API. You can automatically keep such descriptor classes with the -keep option modifier includedescriptorclasses (-keep,includedescriptorclasses ...). You can switch off these notes by specifying the -dontnote option.

You know the process now, let's search for "descriptor class" in our Build Output.

EX: `Note: the configuration keeps the entry point 'android.support.design.widget.BaseTransientBottomBar$SnackbarBaseLayout { void setOnLayoutChangeListener(android.support.design.widget.BaseTransientBottomBar$OnLayoutChangeListener); }', but not the descriptor class 'android.support.design.widget.BaseTransientBottomBar$OnLayoutChangeListener'`

Again, search the `config.txt` for the first class to ensure it's there:

i.e. `-keep class android.support.design.widget.BaseTransientBottomBar$SnackbarBaseLayout`

We do see it inside. So it's saying we need a `-keep` rule for `android.support.design.widget.BaseTransientBottomBar$OnLayoutChangeListener`. Let's now add that to our Custom `Proguard.cfg` file:

```
-keep class android.support.design.widget.BaseTransientBottomBar$OnLayoutChangeListener
```

## unresolved dynamic references to classes or interfaces.
```
1>  Note: there were 5 unresolved dynamic references to classes or interfaces. (TaskId:247)
1>        You should check if you need to specify additional program jars. (TaskId:247)
1>        (http://proguard.sourceforge.net/manual/troubleshooting.html#dynamicalclass) (TaskId:247)
```

> ProGuard can't find a class or interface that your code is accessing by means of introspection. You should consider adding the jar that contains this class.

Let's search for `dynamically referenced class` in our Build Output.

EX: `Note: android.support.v4.media.ParceledListSliceAdapterApi21: can't find dynamically referenced class android.content.pm.ParceledListSlice`

Unfortunately this one isn't that straight forward as something is referencing classes or interfaces that are not present in our project. To solve this, we would have to find the dependency and add it via NuGet.

## accesses to class members by means of introspection.
```
1>  Note: there were 6 accesses to class members by means of introspection. (TaskId:247)
1>        You should consider explicitly keeping the mentioned class members (TaskId:247)
1>        (using '-keep' or '-keepclassmembers'). (TaskId:247)
1>        (http://proguard.sourceforge.net/manual/troubleshooting.html#dynamicalclassmember) (TaskId:247)
```

> Your code uses reflection to find a fields or a method, with a construct like ".getField("myField")". Depending on your application, you may need to figure out where the mentioned class members are defined and keep them with an option like "-keep class MyClass { MyFieldType myField; }". Otherwise, ProGuard might remove or obfuscate the class members, since it can't know which ones they are exactly. It does list possible candidates, for your information. You can switch off these notes by specifying the -dontnote option.

EX: `Note: android.support.v4.app.NotificationCompatJellybean accesses a declared field 'icon' dynamically`

We may just want to let Proguard know "Hey please don't note me on this". We can use the `-dontnote` rule for this specific class. Let's add this to our Custom `Proguard.cfg` file:

```
-dontnote android.support.v4.app.NotificationCompatJellybean
```

## Summary

Now you should know the basics of how Proguard shrinks our code and how we can adjust to the notes that proguard gives us. For the most part, Proguard does a great job of documenting what is going on and how you can fix certain scenarios. Take the time to read through your Proguard logs to optimize the ruleset so Proguard can work for you!

As a bonus, you can always start off with some of these community created Proguard rules to help give you a jump start with these libraries and proguard.

<https://github.com/yongjhih/android-proguards>

<https://github.com/krschultz/android-proguard-snippets>

If you enjoyed this post, please consider subscribing to my upcoming book's email list:

You can signup at the following link: [Programming Xamarin.Android Email List](http://eepurl.com/cz_fj1)