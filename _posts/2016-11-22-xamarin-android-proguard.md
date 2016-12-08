---
layout: post
title: Proguard
---

## Why ProGuard?

`ProGuard` detects and removes unused classes, fields, methods, and attributes from your packaged application. It can even do the same for referenced libraries which can help you avoid the 64k reference limit.

The `ProGuard` tool from the Android SDK will also optimize the bytecode, remove unused code instructions, and obfuscates the remaining classes, fields, and methods with short names.

![](http://content.screencast.com/users/JDouglas18/folders/Snagit/media/e1c2a095-d29e-4905-bea0-db9770b647e4/11.21.2016-12.10.png)

There are four *optional* steps that ProGuard uses:

- In the **shrinking step**, ProGuard starts from these seeds and recursively determines which classes and class members are used. All other classes and class members are discarded.

- In the **optimization step**, ProGuard further optimizes the code. Among other optimizations, classes and methods that are not entry points can be made private, static, or final, unused parameters can be removed, and some methods may be inlined.

- In the **obfuscation step**, ProGuard renames classes and class members that are not entry points. In this entire process, keeping the entry points ensures that they can still be accessed by their original names.

- The **preverification step** is the only step that doesn't have to know the entry points.

ProGuard reads **input jars** in which it will shrink, optimize, obfuscate, and preverify them. ProGuard will then write these results to one or more **output jars**.

[Reference Documentation](https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/index.html#manual/introduction.html)

**Please note only the following steps are run in Xamarin.Android:**

> The Xamarin.Android `ProGuard` configuration does not  obfuscate the `.apk` and it is not possible to enable obfuscation through ProGuard even through the use of custom configuration files. Thus Xamarin.Android's `ProGuard` will only run the **shrinking step**. 

## How does ProGuard work with Xamarin.Android

One important item to know ahead of time before using `ProGuard` is how it works within the `Xamarin.Android` build process. This can be thought of in two separate steps:

1. Xamarin Android Linker
2. Proguard

### Xamarin.Android Linker

First, the linker employs static analysis of your application to determine which assemblies are actually used, which types are actually used, and which members are actually used. It will always run before the `ProGuard` step. Because of this, the linker can strip an assembly/type/member that you might expect `ProGuard` to run on.

You can find more information on this topic here: [https://developer.xamarin.com/guides/android/advanced_topics/linking/](https://developer.xamarin.com/guides/android/advanced_topics/linking/) 

### Proguard

Secondly, ProGuard will then run and remove unused Java bytecode to optimize the `.apk`. (shrinking step)

## Enabling ProGuard

ProGuard can be enabled by checking the `Enable Proguard` option inside of your [Packaging Properties](https://developer.xamarin.com/guides/android/deployment,_testing,_and_metrics/publishing_an_application/part_1_-_preparing_an_application_for_release/#Set_Packaging_Properties). You must also ensure your project is set to the `Release` configuration as the `Linker` must run in order for `ProGuard` to run.

![](http://content.screencast.com/users/JDouglas18/folders/Snagit/media/716eb5a7-77c3-46f6-b13b-0e94c1cdc574/11.21.2016-13.21.png)

![](http://content.screencast.com/users/JDouglas18/folders/Snagit/media/ba6364fb-4d84-4ecb-a7bb-4246f6f810fe/11.21.2016-13.20.png)

*Optional:* You can add a custom ProGuard Configuration file for more control with the ProGuard tooling. To do this, you can create a new `.cfg` file and apply the build action of `ProguardConfiguration`.

![](http://content.screencast.com/users/JDouglas18/folders/Snagit/media/49625f16-cb76-4387-b1be-017f5e69e1b8/11.21.2016-13.19.png)

## Customize which code to keep

For majority of situations, the default ProGuard configuration file that `Xamarin.Android` provides will be sufficient to remove all-and only-the unused code.

You can find the default `ProGuard Configuration File` at `obj\Release\proguard\proguard_xamarin.cfg` if you wanted to see what by default is added to the configuration.

EX:

    # This is Xamarin-specific (and enhanced) configuration.
    
    -dontobfuscate
    
    -keep class mono.MonoRuntimeProvider { *; <init>(...); }
    -keep class mono.MonoPackageManager { *; <init>(...); }
    -keep class mono.MonoPackageManager_Resources { *; <init>(...); }
    -keep class mono.android.** { *; <init>(...); }
    -keep class mono.java.** { *; <init>(...); }
    -keep class mono.javax.** { *; <init>(...); }
    -keep class opentk.platform.android.AndroidGameView { *; <init>(...); }
    -keep class opentk.GameViewBase { *; <init>(...); }
    -keep class opentk_1_0.platform.android.AndroidGameView { *; <init>(...); }
    -keep class opentk_1_0.GameViewBase { *; <init>(...); }
    
    -keep class android.runtime.** { <init>(***); }
    -keep class assembly_mono_android.android.runtime.** { <init>(***); }
    # hash for android.runtime and assembly_mono_android.android.runtime.
    -keep class md52ce486a14f4bcd95899665e9d932190b.** { *; <init>(...); }
    -keepclassmembers class md52ce486a14f4bcd95899665e9d932190b.** { *; <init>(...); }
    
    # Android's template misses fluent setters...
    -keepclassmembers class * extends android.view.View {
       *** set*(***);
    }
    
    # also misses those inflated custom layout stuff from xml...
    -keepclassmembers class * extends android.view.View {
       <init>(android.content.Context,android.util.AttributeSet);
       <init>(android.content.Context,android.util.AttributeSet,int);
    }
    

However there might be cases where ProGuard might not be able to properly analyze and can potentially remove code your application actually needs.

If this happens, you can add a `-keep` line to your custom ProGuard configuration file.

EX:

    -keep public class MyClass

## What command is ProGuard running?

ProGuard is simply a `.jar` provided with the Android SDK. Thus it invokes a command like the following:

    java -jar proguard.jar options ...

## The ProGuard Task

The `Proguard` task is found inside the `Xamarin.Android.Build.Tasks.dll` assembly. It is apart of the `_CompileToDalvikWithDx` `Target`  which is apart of the `_CompileDex` `Target`.

**EX: Default Parameters in a File->New Project**

    ProguardJarPath = C:\Android\android-sdk\tools\proguard\lib\proguard.jar
    AndroidSdkDirectory = C:\Android\android-sdk\
    JavaToolPath = C:\Program Files (x86)\Java\jdk1.8.0_92\\bin
    ProguardToolPath = C:\Android\android-sdk\tools\proguard\
    JavaPlatformJarPath = C:\Android\android-sdk\platforms\android-25\android.jar
    ClassesOutputDirectory = obj\Release\android\bin\classes
    AcwMapFile = obj\Release\acw-map.txt
    ProguardCommonXamarinConfiguration = obj\Release\proguard\proguard_xamarin.cfg
    ProguardGeneratedReferenceConfiguration = obj\Release\proguard\proguard_project_references.cfg
    ProguardGeneratedApplicationConfiguration = obj\Release\proguard\proguard_project_primary.cfg
    ProguardConfigurationFiles
    
    		{sdk.dir}tools\proguard\proguard-android.txt;
    		{intermediate.common.xamarin};
    		{intermediate.references};
    		{intermediate.application};
    		;
    	
    JavaLibrariesToEmbed = C:\Program Files (x86)\Reference Assemblies\Microsoft\Framework\MonoAndroid\v7.0\mono.android.jar
    ProguardJarInput = obj\Release\proguard\__proguard_input__.jar
    ProguardJarOutput = obj\Release\proguard\__proguard_output__.jar
    DumpOutput = obj\Release\proguard\dump.txt
    PrintSeedsOutput = obj\Release\proguard\seeds.txt
    PrintUsageOutput = obj\Release\proguard\usage.txt
    PrintMappingOutput = obj\Release\proguard\mapping.txt

**EX: Command generated from File -> New Project**

    C:\Program Files (x86)\Java\jdk1.8.0_92\\bin\java.exe -jar C:\Android\android-sdk\tools\proguard\lib\proguard.jar -include obj\Release\proguard\proguard_xamarin.cfg -include obj\Release\proguard\proguard_project_references.cfg -include obj\Release\proguard\proguard_project_primary.cfg "-injars 'obj\Release\proguard\__proguard_input__.jar';'C:\Program Files (x86)\Reference Assemblies\Microsoft\Framework\MonoAndroid\v7.0\mono.android.jar'" "-libraryjars 'C:\Android\android-sdk\platforms\android-25\android.jar'" -outjars "obj\Release\proguard\__proguard_output__.jar" -optimizations !code/allocation/variable 

**Optional Parameter:** 

Custom `proguard.cfg` file with `Build Action` of `ProguardConfiguration`. This will add the `proguard.cfg` file to the `ProguardConfigurationFiles` parameter.

## ProGuard Options

- [Input/Output Options](https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/manual/usage.html#iooptions)
- [Keep Options](https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/manual/usage.html#keepoptions)
- [Shrinking Options](https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/manual/usage.html#shrinkingoptions)
- ~~[Optimization Options](https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/manual/usage.html#optimizationoptions)~~ (This step is skipped in Xamarin.Android)
- ~~[Obfuscation Options](https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/manual/usage.html#obfuscationoptions)~~ (This step is skipped in Xamarin.Android)
- ~~[Preverification Options](https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/manual/usage.html#preverificationoptions)~~ (This step is skipped in Xamarin.Android)
- [General Options](https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/manual/usage.html#generaloptions)
- [Class Paths](https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/manual/usage.html#classpath)
- [File Names](https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/manual/usage.html#filename)
- [File Filters](https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/manual/usage.html#filefilters)
- [Filters](https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/manual/usage.html#filters)
- [Overview of `Keep` Options](https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/manual/usage.html#keepoverview)
- [Keep Option Modifiers](https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/manual/usage.html#keepoptionmodifiers)
- [Class Specifications](https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/manual/usage.html#classspecification)

## Examples of ProGuard Configurations

### A simple Android activity

    -injars  bin/classes
    -outjars bin/classes-processed.jar
    -libraryjars /usr/local/java/android-sdk/platforms/android-9/android.jar
    
    -dontpreverify
    -repackageclasses ''
    -allowaccessmodification
    -optimizations !code/simplification/arithmetic
    
    -keep public class mypackage.MyActivity

### A complete Android application

    -injars  bin/classes
    -injars  libs
    -outjars bin/classes-processed.jar
    -libraryjars /usr/local/java/android-sdk/platforms/android-9/android.jar
    
    -dontpreverify
    -repackageclasses ''
    -allowaccessmodification
    -optimizations !code/simplification/arithmetic
    -keepattributes *Annotation*
    
    -keep public class * extends android.app.Activity
    -keep public class * extends android.app.Application
    -keep public class * extends android.app.Service
    -keep public class * extends android.content.BroadcastReceiver
    -keep public class * extends android.content.ContentProvider
    
    -keep public class * extends android.view.View {
    public <init>(android.content.Context);
    public <init>(android.content.Context, android.util.AttributeSet);
    public <init>(android.content.Context, android.util.AttributeSet, int);
    public void set*(...);
    }
    
    -keepclasseswithmembers class * {
    public <init>(android.content.Context, android.util.AttributeSet);
    }
    
    -keepclasseswithmembers class * {
    public <init>(android.content.Context, android.util.AttributeSet, int);
    }
    
    -keepclassmembers class * implements android.os.Parcelable {
    static android.os.Parcelable$Creator CREATOR;
    }
    
    -keepclassmembers class **.R$* {
    public static <fields>;
    }

Please note that in these cases above, the `Xamarin.Android` build process will supply the **input, output, and library jars**. Thus you can focus on other options like `-keep`.

## Troubleshooting

### File Issues

    Unknown option '-keep' in line 1 of file 'proguard.cfg'

This issue happens mainly on Windows because the `.cfg` file has the wrong encoding. You need to ensure it has encoding set to `UTF-8` in your favorite text editor.

### Proguard Processing Issues

Majority of the issues that can happen when ProGuard is processing can be found [here](https://stuff.mit.edu/afs/sipb/project/android/sdk/android-sdk-linux/tools/proguard/docs/index.html#manual/troubleshooting.html).

## ProGuard Versions

You can find all of the ProGuard versions at the [SourceForge page](https://sourceforge.net/projects/proguard/files/).

Note: If you are trying to use `ProGuard` against Android 7.0, you will need to download a newer version of `ProGuard` as the Android SDK does not ship a new version that is compatible with JDK 1.8

[http://stackoverflow.com/questions/39514518/xamarin-android-proguard-unsupported-class-version-number-52-0/39514706#39514706](http://stackoverflow.com/questions/39514518/xamarin-android-proguard-unsupported-class-version-number-52-0/39514706#39514706)

You will be able to provide a custom `ProGuard` path in the future via the following Pull Request:

https://github.com/xamarin/xamarin-android/pull/267

Otherwise you can use this [NuGet package](https://www.nuget.org/packages/name.atsushieno.proguard.facebook/5.3.0) to support a new version of `proguard.jar`.